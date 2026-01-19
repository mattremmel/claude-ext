# Security Guidelines - Rust

## Secret Management with Environment Variables

```rust
// NEVER: Hardcoded secrets
const API_KEY: &str = "sk-proj-xxxxx";

// ALWAYS: Environment variables
fn get_api_key() -> Result<String, std::env::VarError> {
    std::env::var("OPENAI_API_KEY")
}

// With validation
fn get_required_env(key: &str) -> Result<String, AppError> {
    std::env::var(key)
        .map_err(|_| AppError::Config(format!("{} not configured", key)))
}
```

### Loading Environment Variables with dotenvy

```rust
use dotenvy::dotenv;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Load .env file (optional, won't fail if missing)
    dotenv().ok();

    let config = Config {
        api_key: std::env::var("API_KEY")?,
        database_url: std::env::var("DATABASE_URL")?,
        port: std::env::var("PORT")
            .unwrap_or_else(|_| "3000".to_string())
            .parse()?,
    };

    Ok(())
}
```

### Validated Config with config Crate

```rust
use config::{Config, ConfigError, Environment, File};
use serde::Deserialize;

#[derive(Debug, Deserialize)]
pub struct Settings {
    pub database_url: String,
    pub api_key: String,
    pub port: u16,
    pub debug: bool,
}

impl Settings {
    pub fn new() -> Result<Self, ConfigError> {
        Config::builder()
            // Start with defaults
            .set_default("port", 3000)?
            .set_default("debug", false)?
            // Load from config file (optional)
            .add_source(File::with_name("config").required(false))
            // Override with environment variables (APP_ prefix)
            .add_source(Environment::with_prefix("APP").separator("__"))
            .build()?
            .try_deserialize()
    }
}
```

## SQL Injection Prevention with sqlx

Use compile-time checked queries with `sqlx`:

```rust
use sqlx::PgPool;

// WRONG: String formatting (SQL injection vulnerable)
async fn get_user_unsafe(pool: &PgPool, id: &str) -> Result<User, sqlx::Error> {
    let query = format!("SELECT * FROM users WHERE id = '{}'", id);
    sqlx::query_as(&query).fetch_one(pool).await  // DANGEROUS!
}

// CORRECT: Parameterized query (compile-time checked)
async fn get_user(pool: &PgPool, id: &str) -> Result<User, sqlx::Error> {
    sqlx::query_as!(
        User,
        "SELECT id, name, email FROM users WHERE id = $1",
        id
    )
    .fetch_one(pool)
    .await
}

// CORRECT: Using query builder
async fn search_users(pool: &PgPool, name_filter: &str) -> Result<Vec<User>, sqlx::Error> {
    sqlx::query_as!(
        User,
        "SELECT id, name, email FROM users WHERE name ILIKE $1",
        format!("%{}%", name_filter)  // Parameter is escaped
    )
    .fetch_all(pool)
    .await
}
```

For dynamic queries, use `sqlx::QueryBuilder`:

```rust
use sqlx::QueryBuilder;

async fn search_users_dynamic(
    pool: &PgPool,
    filters: &UserFilters,
) -> Result<Vec<User>, sqlx::Error> {
    let mut builder = QueryBuilder::new("SELECT id, name, email FROM users WHERE 1=1");

    if let Some(name) = &filters.name {
        builder.push(" AND name ILIKE ");
        builder.push_bind(format!("%{}%", name));
    }

    if let Some(status) = &filters.status {
        builder.push(" AND status = ");
        builder.push_bind(status);
    }

    builder.build_as::<User>().fetch_all(pool).await
}
```

## Template Escaping with askama

Askama auto-escapes by default:

```rust
use askama::Template;

#[derive(Template)]
#[template(path = "user.html")]
struct UserTemplate<'a> {
    username: &'a str,  // Auto-escaped in template
    bio: &'a str,       // Auto-escaped in template
}
```

```html
<!-- templates/user.html -->
<!-- Safe: username and bio are auto-escaped -->
<h1>{{ username }}</h1>
<p>{{ bio }}</p>

<!-- DANGEROUS: Only use |safe for trusted content -->
<div>{{ trusted_html|safe }}</div>
```

For HTML that must be rendered:

```rust
use ammonia::clean;

// Sanitize user HTML before marking as safe
fn sanitize_html(input: &str) -> String {
    ammonia::clean(input)
}

#[derive(Template)]
#[template(path = "post.html")]
struct PostTemplate {
    title: String,
    content: String,  // Pre-sanitized with ammonia
}
```

## Security Scanning with cargo-audit

Install and run `cargo-audit` to check for known vulnerabilities:

```bash
# Install
cargo install cargo-audit

# Check dependencies for known vulnerabilities
cargo audit

# Auto-fix where possible
cargo audit fix

# Generate JSON report for CI
cargo audit --json > audit-report.json
```

### CI Integration

```yaml
# .github/workflows/security.yml
name: Security Audit
on:
  push:
    paths:
      - '**/Cargo.toml'
      - '**/Cargo.lock'
  schedule:
    - cron: '0 0 * * *'  # Daily

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: rustsec/audit-check@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

## Additional Security Practices

### Secure Random Generation

```rust
use rand::{rngs::OsRng, RngCore};

// WRONG: Using thread_rng for security-sensitive operations
let mut rng = rand::thread_rng();

// CORRECT: Use OsRng for cryptographic operations
fn generate_token() -> [u8; 32] {
    let mut token = [0u8; 32];
    OsRng.fill_bytes(&mut token);
    token
}
```

### Password Hashing

```rust
use argon2::{
    password_hash::{rand_core::OsRng, PasswordHash, PasswordHasher, PasswordVerifier, SaltString},
    Argon2,
};

fn hash_password(password: &str) -> Result<String, argon2::password_hash::Error> {
    let salt = SaltString::generate(&mut OsRng);
    let argon2 = Argon2::default();
    Ok(argon2.hash_password(password.as_bytes(), &salt)?.to_string())
}

fn verify_password(password: &str, hash: &str) -> Result<bool, argon2::password_hash::Error> {
    let parsed_hash = PasswordHash::new(hash)?;
    Ok(Argon2::default()
        .verify_password(password.as_bytes(), &parsed_hash)
        .is_ok())
}
```

### Constant-Time Comparison

```rust
use subtle::ConstantTimeEq;

// WRONG: Timing attack vulnerable
fn verify_token(provided: &[u8], expected: &[u8]) -> bool {
    provided == expected  // Early return leaks timing info
}

// CORRECT: Constant-time comparison
fn verify_token_secure(provided: &[u8], expected: &[u8]) -> bool {
    provided.ct_eq(expected).into()
}
```
