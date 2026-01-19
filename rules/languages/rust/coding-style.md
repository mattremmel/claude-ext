# Coding Style - Rust

## Dependencies

When adding a dependency, first check [blessed.rs/crates](https://blessed.rs/crates) for community-vetted recommendations. This curated list covers common needs like async runtimes, serialization, HTTP clients, and more.

## Idiomatic Rust

### Expression-Oriented Style

Rust is expression-based. Avoid explicit `return` for the final expression:

```rust
// WRONG: Unnecessary return
fn calculate_total(items: &[Item]) -> u32 {
    let total = items.iter().map(|i| i.price).sum();
    return total;
}

// CORRECT: Implicit return
fn calculate_total(items: &[Item]) -> u32 {
    items.iter().map(|i| i.price).sum()
}

// CORRECT: Expression blocks
fn categorize(n: i32) -> &'static str {
    if n < 0 {
        "negative"
    } else if n == 0 {
        "zero"
    } else {
        "positive"
    }
}
```

### Iterator Patterns

Prefer iterators over manual loops:

```rust
// WRONG: Manual loop
fn get_names(users: &[User]) -> Vec<String> {
    let mut names = Vec::new();
    for user in users {
        names.push(user.name.clone());
    }
    names
}

// CORRECT: Iterator chain
fn get_names(users: &[User]) -> Vec<String> {
    users.iter().map(|u| u.name.clone()).collect()
}

// CORRECT: Filter and map
fn active_emails(users: &[User]) -> Vec<String> {
    users
        .iter()
        .filter(|u| u.is_active)
        .map(|u| u.email.clone())
        .collect()
}

// CORRECT: fold for aggregation
fn total_age(users: &[User]) -> u32 {
    users.iter().fold(0, |acc, u| acc + u.age)
}
// Or simply:
fn total_age(users: &[User]) -> u32 {
    users.iter().map(|u| u.age).sum()
}
```

### Option and Result Combinators

Use combinators instead of manual matching:

```rust
// WRONG: Verbose match
fn get_username(user: Option<&User>) -> String {
    match user {
        Some(u) => u.name.clone(),
        None => "anonymous".to_string(),
    }
}

// CORRECT: Use combinators
fn get_username(user: Option<&User>) -> String {
    user.map(|u| u.name.clone())
        .unwrap_or_else(|| "anonymous".to_string())
}

// CORRECT: Chaining with and_then
fn get_user_email(id: u64) -> Option<String> {
    find_user(id)
        .and_then(|u| u.email.clone())
}

// CORRECT: Converting Option to Result
fn require_user(id: u64) -> Result<User, AppError> {
    find_user(id).ok_or_else(|| AppError::NotFound(id))
}

// CORRECT: ? operator with Option (in functions returning Option)
fn get_nested_value(data: &Data) -> Option<i32> {
    let inner = data.outer.as_ref()?;
    let value = inner.middle.as_ref()?;
    Some(value.inner)
}
```

### Pattern Matching

Use `if let` and `let else` for single-pattern matches:

```rust
// WRONG: Match for single pattern
fn process_some(opt: Option<Value>) {
    match opt {
        Some(v) => handle(v),
        None => {},
    }
}

// CORRECT: if let
fn process_some(opt: Option<Value>) {
    if let Some(v) = opt {
        handle(v);
    }
}

// CORRECT: let-else for early return
fn process_required(opt: Option<Value>) -> Result<(), Error> {
    let Some(v) = opt else {
        return Err(Error::Missing);
    };
    handle(v);
    Ok(())
}

// CORRECT: Destructuring in parameters
fn print_point(&(x, y): &(i32, i32)) {
    println!("({}, {})", x, y);
}

// CORRECT: Match guards
fn describe(n: i32) -> &'static str {
    match n {
        x if x < 0 => "negative",
        0 => "zero",
        x if x < 10 => "small positive",
        _ => "large positive",
    }
}
```

### Function Parameters

Prefer slices and `&str` over owned types:

```rust
// WRONG: Takes ownership unnecessarily
fn process_items(items: Vec<Item>) { ... }
fn greet(name: String) { ... }

// CORRECT: Borrow slices
fn process_items(items: &[Item]) { ... }
fn greet(name: &str) { ... }

// CORRECT: Use impl Trait for flexibility
fn process<I: IntoIterator<Item = Item>>(items: I) { ... }

// Or with impl Trait syntax:
fn log_all(messages: impl IntoIterator<Item = impl AsRef<str>>) {
    for msg in messages {
        println!("{}", msg.as_ref());
    }
}
```

### From/Into Conversions

Implement `From` for type conversions (you get `Into` for free):

```rust
struct UserId(u64);

impl From<u64> for UserId {
    fn from(id: u64) -> Self {
        UserId(id)
    }
}

// Now both work:
let id1 = UserId::from(42);
let id2: UserId = 42.into();

// Use in function signatures for flexibility:
fn find_user(id: impl Into<UserId>) -> Option<User> {
    let id = id.into();
    // ...
}

// Can call with either:
find_user(42_u64);
find_user(UserId(42));
```

### Default Trait

Use `Default` for structs with sensible defaults:

```rust
#[derive(Default)]
struct Config {
    timeout: u64,
    retries: u32,
    verbose: bool,
}

// Custom Default
impl Default for Config {
    fn default() -> Self {
        Self {
            timeout: 30,
            retries: 3,
            verbose: false,
        }
    }
}

// Usage with struct update syntax
let config = Config {
    verbose: true,
    ..Default::default()
};
```

### Method Chaining

Design APIs for chaining:

```rust
struct QueryBuilder {
    table: String,
    conditions: Vec<String>,
    limit: Option<u32>,
}

impl QueryBuilder {
    fn new(table: impl Into<String>) -> Self {
        Self {
            table: table.into(),
            conditions: Vec::new(),
            limit: None,
        }
    }

    fn where_eq(mut self, field: &str, value: &str) -> Self {
        self.conditions.push(format!("{} = '{}'", field, value));
        self
    }

    fn limit(mut self, n: u32) -> Self {
        self.limit = Some(n);
        self
    }

    fn build(self) -> String {
        // ...
    }
}

// Usage
let query = QueryBuilder::new("users")
    .where_eq("status", "active")
    .where_eq("role", "admin")
    .limit(10)
    .build();
```

## Ownership and Borrowing (Immutability by Default)

Rust is immutable by default. Use `&` for shared references and `&mut` for mutable references:

```rust
// WRONG: Unnecessary mutation
fn update_user(user: &mut User, name: String) {
    user.name = name;  // MUTATION!
}

// CORRECT: Return new instance (functional style)
fn update_user(user: &User, name: String) -> User {
    User {
        name,
        ..user.clone()
    }
}
```

For collections:

```rust
// WRONG: Mutating in place
fn add_item(items: &mut Vec<Item>, item: Item) {
    items.push(item);  // MUTATION!
}

// CORRECT: Return new collection
fn add_item(items: &[Item], item: Item) -> Vec<Item> {
    let mut new_items = items.to_vec();
    new_items.push(item);
    new_items
}

// ALSO CORRECT: Using iterators
fn add_item(items: &[Item], item: Item) -> Vec<Item> {
    items.iter().cloned().chain(std::iter::once(item)).collect()
}
```

## Clone and Copy Traits

Use `Copy` for small, stack-allocated types. Use `Clone` for heap-allocated or larger types:

```rust
// Copy: Small, stack-allocated (automatic bitwise copy)
#[derive(Copy, Clone, Debug)]
struct Point {
    x: i32,
    y: i32,
}

// Clone only: Contains heap data or is large
#[derive(Clone, Debug)]
struct User {
    id: u64,
    name: String,  // String is heap-allocated
    email: String,
}
```

Performance considerations:

```rust
// WRONG: Unnecessary clone
fn process(data: &LargeStruct) {
    let copy = data.clone();  // Expensive clone when not needed
    println!("{}", copy.field);
}

// CORRECT: Use reference
fn process(data: &LargeStruct) {
    println!("{}", data.field);
}
```

## Error Handling

Use `Result<T, E>` with the `?` operator and `thiserror` for custom errors:

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("User not found: {0}")]
    UserNotFound(String),

    #[error("Database error: {0}")]
    Database(#[from] sqlx::Error),

    #[error("Validation failed: {0}")]
    Validation(String),
}

// Use ? for propagation
async fn get_user(id: &str) -> Result<User, AppError> {
    let user = db.query_one("SELECT * FROM users WHERE id = $1", &[&id])
        .await?  // Automatically converts sqlx::Error to AppError
        .ok_or_else(|| AppError::UserNotFound(id.to_string()))?;

    Ok(user)
}
```

For functions that can fail in multiple ways:

```rust
// WRONG: Using unwrap/expect in library code
fn parse_config(path: &str) -> Config {
    let content = std::fs::read_to_string(path).unwrap();  // PANIC!
    toml::from_str(&content).expect("Invalid config")      // PANIC!
}

// CORRECT: Propagate errors
fn parse_config(path: &str) -> Result<Config, AppError> {
    let content = std::fs::read_to_string(path)
        .map_err(|e| AppError::Io(e))?;
    let config = toml::from_str(&content)
        .map_err(|e| AppError::Config(e.to_string()))?;
    Ok(config)
}
```

## Input Validation with validator Crate

Use the `validator` crate with derive macros:

```rust
use validator::{Validate, ValidationError};

#[derive(Debug, Validate)]
struct CreateUserRequest {
    #[validate(email)]
    email: String,

    #[validate(length(min = 3, max = 50))]
    username: String,

    #[validate(range(min = 0, max = 150))]
    age: u8,

    #[validate(custom(function = "validate_password"))]
    password: String,
}

fn validate_password(password: &str) -> Result<(), ValidationError> {
    if password.len() < 8 {
        return Err(ValidationError::new("password_too_short"));
    }
    if !password.chars().any(|c| c.is_uppercase()) {
        return Err(ValidationError::new("password_needs_uppercase"));
    }
    Ok(())
}

// Usage
fn create_user(req: CreateUserRequest) -> Result<User, AppError> {
    req.validate()
        .map_err(|e| AppError::Validation(e.to_string()))?;

    // Proceed with validated data
    Ok(User::new(req.email, req.username))
}
```

## Clippy Lints

Enable useful Clippy lints in `Cargo.toml` or `clippy.toml`:

```toml
# Cargo.toml
[lints.clippy]
# Correctness
unwrap_used = "warn"
expect_used = "warn"
panic = "warn"

# Style
needless_pass_by_value = "warn"
clone_on_ref_ptr = "warn"
redundant_closure_for_method_calls = "warn"

# Performance
unnecessary_to_owned = "warn"
```

Common patterns:

```rust
// WRONG: Clippy warns
fn process(data: String) {  // needless_pass_by_value
    println!("{}", data);
}

// CORRECT: Use reference
fn process(data: &str) {
    println!("{}", data);
}

// WRONG: Clippy warns
let result = items.iter().map(|x| x.to_string()).collect();

// CORRECT: Use method reference
let result: Vec<String> = items.iter().map(ToString::to_string).collect();
```

When a lint must be suppressed, prefer targeted `#[allow]`:

```rust
// WRONG: Global allow
#![allow(clippy::unwrap_used)]

// CORRECT: Targeted allow with justification
#[allow(clippy::unwrap_used)]  // Safe: validated input
fn parse_known_valid(s: &str) -> u32 {
    s.parse().unwrap()
}
```
