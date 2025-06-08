# Contributing to Azure Notification Hubs Go SDK

Thank you for your interest in contributing to the Azure Notification Hubs Go SDK! This document provides guidelines for contributing to this project.

## Development Setup

### Prerequisites

- Go 1.21 or later
- Git
- Make (optional, but recommended)

### Getting Started

1. Fork the repository
2. Clone your fork:
   ```bash
   git clone https://github.com/your-username/azure-notificationhubs-go.git
   cd azure-notificationhubs-go
   ```

3. Install development tools:
   ```bash
   make install-tools
   ```

4. Run tests to ensure everything works:
   ```bash
   make test
   ```

## Development Workflow

### Before Making Changes

1. Create a new branch for your feature/fix:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. Make sure all tests pass:
   ```bash
   make test
   ```

### Making Changes

1. Write your code following Go best practices
2. Add tests for new functionality
3. Update documentation if needed
4. Run pre-commit checks:
   ```bash
   make pre-commit
   ```

### Code Quality Standards

- **Formatting**: Use `go fmt` (run `make fmt`)
- **Linting**: Code must pass `golint` and `staticcheck`
- **Testing**: Maintain or improve test coverage
- **Documentation**: Add godoc comments for public APIs

### Testing Guidelines

- Write unit tests for all new functionality
- Use table-driven tests where appropriate
- Mock external dependencies
- Ensure tests are deterministic and fast

### Commit Guidelines

- Write clear, descriptive commit messages
- Use conventional commit format when possible:
  - `feat:` for new features
  - `fix:` for bug fixes
  - `docs:` for documentation changes
  - `test:` for test improvements
  - `refactor:` for code refactoring

### Submitting Changes

1. Push your branch to your fork
2. Create a Pull Request against the main branch
3. Fill out the PR template with:
   - Description of changes
   - Related issues
   - Testing performed
   - Breaking changes (if any)

## Project Structure

```
├── README.md              # Project documentation
├── go.mod                 # Go module definition
├── Makefile              # Build and development commands
├── *.go                  # Main source files
├── *_test.go             # Test files
├── fixtures/             # Test fixtures
├── utils/                # Utility packages
├── .github/              # GitHub Actions workflows
└── docs/                 # Additional documentation
```

## Code Review Process

All submissions require review. We use GitHub pull requests for this purpose.

### Review Criteria

- Code follows Go best practices
- Tests are included and passing
- Documentation is updated
- No breaking changes without discussion
- Performance impact is considered

## Release Process

Releases are handled by maintainers and follow semantic versioning.

## Getting Help

- Open an issue for bug reports or feature requests
- Join discussions in existing issues
- Reach out to maintainers for guidance

## Code of Conduct

Be respectful and inclusive in all interactions. We want this to be a welcoming community for everyone. 