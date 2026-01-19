# Common Patterns - Rust

## API Response Structs with Serde

```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
#[serde(rename_all = "camelCase")]
pub struct ApiResponse<T> {
    pub success: bool,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub data: Option<T>,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub error: Option<String>,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub meta: Option<PaginationMeta>,
}

#[derive(Debug, Serialize, Deserialize)]
#[serde(rename_all = "camelCase")]
pub struct PaginationMeta {
    pub total: u64,
    pub page: u32,
    pub limit: u32,
}

impl<T> ApiResponse<T> {
    pub fn success(data: T) -> Self {
        Self {
            success: true,
            data: Some(data),
            error: None,
            meta: None,
        }
    }

    pub fn success_with_meta(data: T, meta: PaginationMeta) -> Self {
        Self {
            success: true,
            data: Some(data),
            error: None,
            meta: Some(meta),
        }
    }

    pub fn error(message: impl Into<String>) -> Self {
        Self {
            success: false,
            data: None,
            error: Some(message.into()),
            meta: None,
        }
    }
}
```

Usage:

```rust
async fn fetch_users(pool: &PgPool) -> ApiResponse<Vec<User>> {
    match sqlx::query_as!(User, "SELECT * FROM users")
        .fetch_all(pool)
        .await
    {
        Ok(users) => ApiResponse::success_with_meta(
            users.clone(),
            PaginationMeta {
                total: users.len() as u64,
                page: 1,
                limit: 50,
            },
        ),
        Err(e) => {
            tracing::error!("Failed to fetch users: {}", e);
            ApiResponse::error("Failed to fetch users")
        }
    }
}
```

## Repository Pattern with Traits

Define the trait with `async_trait`:

```rust
use async_trait::async_trait;

#[async_trait]
pub trait Repository<T, ID> {
    type Error;

    async fn find_all(&self) -> Result<Vec<T>, Self::Error>;
    async fn find_by_id(&self, id: ID) -> Result<Option<T>, Self::Error>;
    async fn create(&self, entity: &T) -> Result<T, Self::Error>;
    async fn update(&self, id: ID, entity: &T) -> Result<T, Self::Error>;
    async fn delete(&self, id: ID) -> Result<(), Self::Error>;
}

// User-specific repository with additional methods
#[async_trait]
pub trait UserRepository: Repository<User, u64> {
    async fn find_by_email(&self, email: &str) -> Result<Option<User>, Self::Error>;
}
```

Implementation:

```rust
pub struct PgUserRepository {
    pool: PgPool,
}

impl PgUserRepository {
    pub fn new(pool: PgPool) -> Self {
        Self { pool }
    }
}

#[async_trait]
impl Repository<User, u64> for PgUserRepository {
    type Error = sqlx::Error;

    async fn find_all(&self) -> Result<Vec<User>, Self::Error> {
        sqlx::query_as!(User, "SELECT id, name, email FROM users")
            .fetch_all(&self.pool)
            .await
    }

    async fn find_by_id(&self, id: u64) -> Result<Option<User>, Self::Error> {
        sqlx::query_as!(
            User,
            "SELECT id, name, email FROM users WHERE id = $1",
            id as i64
        )
        .fetch_optional(&self.pool)
        .await
    }

    async fn create(&self, user: &User) -> Result<User, Self::Error> {
        sqlx::query_as!(
            User,
            "INSERT INTO users (name, email) VALUES ($1, $2) RETURNING id, name, email",
            user.name,
            user.email
        )
        .fetch_one(&self.pool)
        .await
    }

    async fn update(&self, id: u64, user: &User) -> Result<User, Self::Error> {
        sqlx::query_as!(
            User,
            "UPDATE users SET name = $1, email = $2 WHERE id = $3 RETURNING id, name, email",
            user.name,
            user.email,
            id as i64
        )
        .fetch_one(&self.pool)
        .await
    }

    async fn delete(&self, id: u64) -> Result<(), Self::Error> {
        sqlx::query!("DELETE FROM users WHERE id = $1", id as i64)
            .execute(&self.pool)
            .await?;
        Ok(())
    }
}

#[async_trait]
impl UserRepository for PgUserRepository {
    async fn find_by_email(&self, email: &str) -> Result<Option<User>, Self::Error> {
        sqlx::query_as!(
            User,
            "SELECT id, name, email FROM users WHERE email = $1",
            email
        )
        .fetch_optional(&self.pool)
        .await
    }
}
```

## Debounce with Tokio

```rust
use std::sync::Arc;
use tokio::sync::Mutex;
use tokio::time::{sleep, Duration, Instant};

pub struct Debouncer<T> {
    delay: Duration,
    last_call: Arc<Mutex<Option<Instant>>>,
    pending_value: Arc<Mutex<Option<T>>>,
}

impl<T: Clone + Send + 'static> Debouncer<T> {
    pub fn new(delay_ms: u64) -> Self {
        Self {
            delay: Duration::from_millis(delay_ms),
            last_call: Arc::new(Mutex::new(None)),
            pending_value: Arc::new(Mutex::new(None)),
        }
    }

    pub async fn call<F, Fut>(&self, value: T, action: F)
    where
        F: FnOnce(T) -> Fut + Send + 'static,
        Fut: std::future::Future<Output = ()> + Send,
    {
        let now = Instant::now();

        {
            let mut last = self.last_call.lock().await;
            let mut pending = self.pending_value.lock().await;
            *last = Some(now);
            *pending = Some(value.clone());
        }

        let delay = self.delay;
        let last_call = Arc::clone(&self.last_call);
        let pending_value = Arc::clone(&self.pending_value);

        tokio::spawn(async move {
            sleep(delay).await;

            let should_execute = {
                let last = last_call.lock().await;
                last.map(|t| now == t).unwrap_or(false)
            };

            if should_execute {
                let value = {
                    let mut pending = pending_value.lock().await;
                    pending.take()
                };

                if let Some(v) = value {
                    action(v).await;
                }
            }
        });
    }
}
```

Usage:

```rust
let debouncer = Debouncer::new(300);

// Each call resets the timer
debouncer.call("search query 1".to_string(), |query| async move {
    perform_search(&query).await;
}).await;

debouncer.call("search query 2".to_string(), |query| async move {
    perform_search(&query).await;  // Only this one executes after 300ms
}).await;
```

## Builder Pattern

```rust
#[derive(Debug, Clone)]
pub struct HttpClient {
    base_url: String,
    timeout: Duration,
    headers: HashMap<String, String>,
    retry_count: u32,
}

#[derive(Default)]
pub struct HttpClientBuilder {
    base_url: Option<String>,
    timeout: Option<Duration>,
    headers: HashMap<String, String>,
    retry_count: Option<u32>,
}

impl HttpClientBuilder {
    pub fn new() -> Self {
        Self::default()
    }

    pub fn base_url(mut self, url: impl Into<String>) -> Self {
        self.base_url = Some(url.into());
        self
    }

    pub fn timeout(mut self, timeout: Duration) -> Self {
        self.timeout = Some(timeout);
        self
    }

    pub fn header(mut self, key: impl Into<String>, value: impl Into<String>) -> Self {
        self.headers.insert(key.into(), value.into());
        self
    }

    pub fn retry_count(mut self, count: u32) -> Self {
        self.retry_count = Some(count);
        self
    }

    pub fn build(self) -> Result<HttpClient, BuilderError> {
        let base_url = self.base_url.ok_or(BuilderError::MissingField("base_url"))?;

        Ok(HttpClient {
            base_url,
            timeout: self.timeout.unwrap_or(Duration::from_secs(30)),
            headers: self.headers,
            retry_count: self.retry_count.unwrap_or(3),
        })
    }
}

impl HttpClient {
    pub fn builder() -> HttpClientBuilder {
        HttpClientBuilder::new()
    }
}
```

Usage:

```rust
let client = HttpClient::builder()
    .base_url("https://api.example.com")
    .timeout(Duration::from_secs(10))
    .header("Authorization", "Bearer token123")
    .header("Content-Type", "application/json")
    .retry_count(5)
    .build()?;
```

## Type State Pattern

Compile-time state machines that prevent invalid state transitions:

```rust
use std::marker::PhantomData;

// State markers (zero-sized types)
pub struct Draft;
pub struct Published;
pub struct Archived;

pub struct Article<State> {
    id: u64,
    title: String,
    content: String,
    _state: PhantomData<State>,
}

// Methods available in all states
impl<State> Article<State> {
    pub fn title(&self) -> &str {
        &self.title
    }

    pub fn content(&self) -> &str {
        &self.content
    }
}

// Methods only available for Draft articles
impl Article<Draft> {
    pub fn new(id: u64, title: String, content: String) -> Self {
        Self {
            id,
            title,
            content,
            _state: PhantomData,
        }
    }

    pub fn edit_content(&mut self, content: String) {
        self.content = content;
    }

    pub fn publish(self) -> Article<Published> {
        Article {
            id: self.id,
            title: self.title,
            content: self.content,
            _state: PhantomData,
        }
    }
}

// Methods only available for Published articles
impl Article<Published> {
    pub fn archive(self) -> Article<Archived> {
        Article {
            id: self.id,
            title: self.title,
            content: self.content,
            _state: PhantomData,
        }
    }

    pub fn unpublish(self) -> Article<Draft> {
        Article {
            id: self.id,
            title: self.title,
            content: self.content,
            _state: PhantomData,
        }
    }
}

// Methods only available for Archived articles
impl Article<Archived> {
    pub fn restore(self) -> Article<Draft> {
        Article {
            id: self.id,
            title: self.title,
            content: self.content,
            _state: PhantomData,
        }
    }
}
```

Usage:

```rust
// Create draft
let draft = Article::<Draft>::new(1, "Title".into(), "Content".into());

// Can edit draft
let mut draft = draft;
draft.edit_content("Updated content".into());

// Publish
let published = draft.publish();

// Cannot edit published article - compile error!
// published.edit_content("..."); // ERROR: method not found

// Can archive
let archived = published.archive();

// Can restore to draft
let draft_again = archived.restore();
```

This pattern ensures invalid state transitions are caught at compile time rather than runtime.
