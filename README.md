# Taskfiles

Reusable [Task](https://taskfile.dev) configurations for Go and Python projects. These Taskfiles provide common development workflows including building, testing, linting, and cleaning, with cross-platform support for Windows, macOS, and Linux.

## Repository Contents

### [Taskfile.go.yaml](Taskfile.go.yaml)
Shared Taskfile for Go projects. Provides tasks for:
- **init**: Initialize Go module and download dependencies
- **build**: Build all binaries defined in the `COMMANDS` array
- **run**: Run a specific command
- **debug**: Debug a command with Delve
- **test**: Run tests (excluding `/test/` and `/mocks/` directories)
- **test-with-cover**: Run tests with HTML coverage report
- **ci-test**: Run tests with coverage for CI environments
- **fmt**: Format Go code with `go fmt`
- **lint**: Run `golangci-lint`
- **check**: Run formatting and linting checks
- **generate-mocks**: Generate mocks using mockery
- **clean**: Remove build artifacts and generated files

**Required variables**: `USER_NAME`, `ARTIFACT_NAME`, `COMMANDS` (array)

### [Taskfile.python.yaml](Taskfile.python.yaml)
Shared Taskfile for Python projects using [uv](https://github.com/astral-sh/uv). Provides tasks for:
- **init**: Initialize Python project with uv and install dependencies
- **install**: Install dependencies
- **sync**: Sync dependencies from requirements files
- **run**: Run a specific Python module
- **test**: Run pytest tests
- **test-with-cover**: Run tests with HTML coverage report
- **ci-test**: Run tests with XML coverage for CI
- **fmt**: Format code with ruff
- **lint**: Lint code with ruff
- **lint-fix**: Auto-fix linting issues
- **check**: Run formatting and linting checks
- **type-check**: Run type checking with mypy
- **verify**: Run all checks (format, lint, type-check, test)
- **build**: Build Python package distribution
- **publish**: Publish package to PyPI
- **lock**: Generate requirements lock file
- **upgrade**: Upgrade all dependencies
- **clean**: Remove build artifacts, cache, and generated files

**Required variables**: `USER_NAME`, `ARTIFACT_NAME`, `COMMANDS` (array)

### [Taskfile.helpers.yaml](Taskfile.helpers.yaml)
Internal helper tasks providing cross-platform abstractions for common operations. These tasks are automatically included by the language-specific Taskfiles and are marked as internal (not shown in task lists). Includes:
- `delete-paths`: Delete specified paths (files or directories)
- `delete-pattern`: Delete files matching a pattern recursively
- `open-file`: Open a file in the default browser/viewer
- `file-exists`: Check if a file exists
- `dir-exists`: Check if a directory exists

**Note**: This file is not meant to be included directly in your projects; it's automatically included by the language-specific Taskfiles.

## Usage

### Including a Language-Specific Taskfile

You can include these Taskfiles in your local project using [go-getter](https://github.com/hashicorp/go-getter), which is built into Task. Add an `includes` section to your local `Taskfile.yml`:

#### For Go Projects

```yaml
version: '3'

includes:
  go:
    taskfile: https://github.com/paigemae/taskfiles/raw/main/Taskfile.go.yaml

vars:
  USER_NAME: your-github-username
  ARTIFACT_NAME: your-project-name
  COMMANDS:
    - command1
    - command2

tasks:
  # Your local tasks here
  default:
    cmds:
      - task: go:build
```

#### For Python Projects

```yaml
version: '3'

includes:
  python:
    taskfile: https://github.com/paigemae/taskfiles/raw/main/Taskfile.python.yaml

vars:
  USER_NAME: your-github-username
  ARTIFACT_NAME: your-project-name
  COMMANDS:
    - command1
    - command2

tasks:
  # Your local tasks here
  default:
    cmds:
      - task: python:test
```

### How Includes Work

When you include a language-specific Taskfile (e.g., `Taskfile.go.yaml`) using go-getter, Task will:

1. **Download the remote Taskfile** to a local cache directory
2. **Parse the includes** section in that Taskfile
3. **Resolve relative paths** in the includes section relative to the downloaded Taskfile's location
4. **Download any nested includes** (like `Taskfile.helpers.yaml`) to the same cache directory

This means `Taskfile.helpers.yaml` is automatically downloaded alongside the language-specific Taskfile you include, and the relative path `./Taskfile.helpers.yaml` resolves correctly within the cache.

#### Example: Double-Layer Include Resolution

When your local `Taskfile.yml` includes:
```yaml
includes:
  go:
    taskfile: https://github.com/paigemae/taskfiles/raw/main/Taskfile.go.yaml
```

Task will:
1. Download `Taskfile.go.yaml` to `~/.task/cache/<hash>/Taskfile.go.yaml`
2. Parse the includes in `Taskfile.go.yaml`:
   ```yaml
   includes:
     helpers:
       taskfile: ./Taskfile.helpers.yaml
       internal: true
   ```
3. Resolve `./Taskfile.helpers.yaml` relative to the cached `Taskfile.go.yaml`
4. Download `Taskfile.helpers.yaml` to `~/.task/cache/<hash>/Taskfile.helpers.yaml`
5. Make all tasks available in your local project (prefixed with `go:`)

### Running Tasks

Once included, you can run tasks from the remote Taskfile using the namespace prefix:

```bash
# List all available tasks
task --list

# Run a Go task
task go:build
task go:test

# Run a Python task
task python:test
task python:fmt

# Pass variables to tasks
task go:run CMD=my-command
task python:run CMD=my-module
```

## Required Variables

Both language-specific Taskfiles expect the following variables to be defined in your local `Taskfile.yml`:

- **USER_NAME**: Your GitHub username (used in `go mod init`)
- **ARTIFACT_NAME**: Your project name
- **COMMANDS**: Array of command/module names (used for building multiple binaries or entry points)

Example:
```yaml
vars:
  USER_NAME: paigemae
  ARTIFACT_NAME: my-awesome-project
  COMMANDS:
    - api-server
    - worker
    - cli-tool
```

## Local Development

If you want to modify these Taskfiles locally before pushing to GitHub, you can use a local file path instead of a URL:

```yaml
includes:
  go:
    taskfile: ../path/to/taskfiles/Taskfile.go.yaml
```

## Requirements

### For Go Projects
- [Go](https://golang.org/) 1.21+
- [golangci-lint](https://golangci-lint.run/) (for linting)
- [mockery](https://vektra.github.io/mockery/) (for mock generation, optional)
- [Delve](https://github.com/go-delve/delve) (for debugging, optional)

### For Python Projects
- [Python](https://www.python.org/) 3.9+
- [uv](https://github.com/astral-sh/uv) (Python package manager)
- [ruff](https://github.com/astral-sh/ruff) (linter/formatter)
- [pytest](https://pytest.org/) (testing framework)
- [mypy](https://mypy-lang.org/) (type checking, optional)

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
