# Coding Style

## Immutability (CRITICAL)

ALWAYS create new objects, NEVER mutate:

```rust
#[derive(Clone)]
struct User {
    name: String,
}

fn update_user(user: &User, name: String) -> User {
    User { name, ..user.clone() }
}
```

## File Organization

MANY SMALL FILES > FEW LARGE FILES:
- High cohesion, low coupling
- 200-400 lines typical, 800 max
- Extract utilities from large components
- Organize by feature/domain, not by type

## Error Handling

ALWAYS handle errors comprehensively:

```rust
fn load_user(id: UserId) -> Result<User, ApplicationError> {
    repository
        .find_by_id(id)?
        .ok_or(ApplicationError::NotFound)
}
```

## Input Validation

ALWAYS validate user input:

```rust
impl TryFrom<CreateUserRequest> for NewUser {
    type Error = ValidationError;

    fn try_from(request: CreateUserRequest) -> Result<Self, Self::Error> {
        let email = EmailAddress::try_from(request.email)?;
        let age = Age::try_from(request.age)?;
        Ok(Self { email, age })
    }
}
```

## Code Quality Checklist

Before marking work complete:
- [ ] Code is readable and well-named
- [ ] Functions are small (<50 lines)
- [ ] Files are focused (<800 lines)
- [ ] No deep nesting (>4 levels)
- [ ] Proper error handling
- [ ] No console.log statements
- [ ] No hardcoded values
- [ ] No mutation (immutable patterns used)
