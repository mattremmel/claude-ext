# Testing - Rust

## Unit Testing Basics

Use `#[test]` attribute and `#[cfg(test)]` module:

```rust
// src/lib.rs or src/main.rs
pub fn calculate_total(items: &[Item]) -> u32 {
    items.iter().map(|item| item.price).sum()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_calculate_total_sums_correctly() {
        let items = vec![
            Item { price: 10 },
            Item { price: 20 },
        ];
        assert_eq!(calculate_total(&items), 30);
    }

    #[test]
    fn test_calculate_total_empty_array() {
        assert_eq!(calculate_total(&[]), 0);
    }

    #[test]
    #[should_panic(expected = "divide by zero")]
    fn test_division_by_zero_panics() {
        divide(10, 0);
    }
}
```

### Common Assertion Macros

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_assertions() {
        // Equality
        assert_eq!(actual, expected);
        assert_ne!(actual, unexpected);

        // Boolean
        assert!(condition);
        assert!(!should_be_false);

        // With custom message
        assert_eq!(result, 42, "Expected 42, got {}", result);

        // Approximate equality for floats
        assert!((actual - expected).abs() < 0.001);
    }
}
```

## Test Organization

### Unit Tests (in same file)

```rust
// src/user.rs
pub struct User {
    pub id: u64,
    pub name: String,
}

impl User {
    pub fn new(id: u64, name: String) -> Self {
        Self { id, name }
    }

    pub fn is_valid(&self) -> bool {
        !self.name.is_empty()
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_user_creation() {
        let user = User::new(1, "Alice".to_string());
        assert_eq!(user.id, 1);
        assert_eq!(user.name, "Alice");
    }

    #[test]
    fn test_user_validation() {
        let valid = User::new(1, "Alice".to_string());
        let invalid = User::new(2, String::new());

        assert!(valid.is_valid());
        assert!(!invalid.is_valid());
    }
}
```

### Integration Tests (in tests/ directory)

```
project/
├── src/
│   ├── lib.rs
│   └── user.rs
├── tests/
│   ├── common/
│   │   └── mod.rs      # Shared test utilities
│   ├── user_tests.rs
│   └── api_tests.rs
└── Cargo.toml
```

```rust
// tests/common/mod.rs
pub fn setup_test_db() -> TestDb {
    // Setup code shared across integration tests
    TestDb::new()
}

// tests/user_tests.rs
mod common;

use my_crate::User;

#[test]
fn test_user_workflow() {
    let db = common::setup_test_db();
    let user = User::new(1, "Alice".to_string());
    db.save(&user);

    let loaded = db.find(1).unwrap();
    assert_eq!(loaded.name, "Alice");
}
```

## Mocking with mockall

Define traits and mock them:

```rust
use mockall::{automock, predicate::*};

#[automock]
pub trait UserRepository {
    fn find_by_id(&self, id: u64) -> Option<User>;
    fn save(&self, user: &User) -> Result<(), RepositoryError>;
}

pub struct UserService<R: UserRepository> {
    repo: R,
}

impl<R: UserRepository> UserService<R> {
    pub fn new(repo: R) -> Self {
        Self { repo }
    }

    pub fn get_user(&self, id: u64) -> Option<User> {
        self.repo.find_by_id(id)
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_get_user_found() {
        let mut mock_repo = MockUserRepository::new();

        mock_repo
            .expect_find_by_id()
            .with(eq(1))
            .times(1)
            .returning(|_| Some(User::new(1, "Alice".to_string())));

        let service = UserService::new(mock_repo);
        let user = service.get_user(1);

        assert!(user.is_some());
        assert_eq!(user.unwrap().name, "Alice");
    }

    #[test]
    fn test_get_user_not_found() {
        let mut mock_repo = MockUserRepository::new();

        mock_repo
            .expect_find_by_id()
            .with(eq(999))
            .times(1)
            .returning(|_| None);

        let service = UserService::new(mock_repo);
        assert!(service.get_user(999).is_none());
    }
}
```

### Cargo.toml for mockall

```toml
[dev-dependencies]
mockall = "0.12"
```

## Async Testing

Use `#[tokio::test]` for async tests:

```rust
use tokio;

async fn fetch_user(id: u64) -> Result<User, Error> {
    // Async database call
    db.query_one("SELECT * FROM users WHERE id = $1", &[&id]).await
}

#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_fetch_user() {
        let user = fetch_user(1).await.unwrap();
        assert_eq!(user.name, "Alice");
    }

    #[tokio::test]
    async fn test_concurrent_fetches() {
        let (user1, user2) = tokio::join!(
            fetch_user(1),
            fetch_user(2)
        );

        assert!(user1.is_ok());
        assert!(user2.is_ok());
    }
}
```

### Cargo.toml for async tests

```toml
[dev-dependencies]
tokio = { version = "1", features = ["macros", "rt-multi-thread"] }
```

## Coverage with cargo-tarpaulin

Install and run:

```bash
# Install
cargo install cargo-tarpaulin

# Run with coverage
cargo tarpaulin

# With HTML report
cargo tarpaulin --out Html

# With minimum threshold
cargo tarpaulin --fail-under 80

# Exclude test files from coverage
cargo tarpaulin --exclude-files "tests/*"
```

### CI Integration

```yaml
# .github/workflows/test.yml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable

      - name: Run tests
        run: cargo test --all-features

      - name: Install tarpaulin
        run: cargo install cargo-tarpaulin

      - name: Generate coverage
        run: cargo tarpaulin --out Xml --fail-under 80

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: cobertura.xml
```

### Cargo.toml Configuration

```toml
# Cargo.toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2024"

[dependencies]
# ...

[dev-dependencies]
mockall = "0.12"
tokio = { version = "1", features = ["macros", "rt-multi-thread"] }

# Optional: Configure test features
[features]
test-utils = []
```

## Test Utilities Pattern

```rust
// src/test_utils.rs (only compiled in test)
#[cfg(test)]
pub mod fixtures {
    use crate::User;

    pub fn test_user() -> User {
        User::new(1, "Test User".to_string())
    }

    pub fn test_users(count: usize) -> Vec<User> {
        (0..count)
            .map(|i| User::new(i as u64, format!("User {}", i)))
            .collect()
    }
}

// Usage in tests
#[cfg(test)]
mod tests {
    use crate::test_utils::fixtures;

    #[test]
    fn test_with_fixture() {
        let user = fixtures::test_user();
        assert!(user.is_valid());
    }
}
```

## Running Tests

```bash
# Run all tests
cargo test

# Run specific test
cargo test test_user_creation

# Run tests in specific module
cargo test user::tests

# Run tests with output
cargo test -- --nocapture

# Run ignored tests
cargo test -- --ignored

# Run tests in parallel (default) or single-threaded
cargo test -- --test-threads=1
```
