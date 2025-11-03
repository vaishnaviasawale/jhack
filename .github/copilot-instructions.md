# jhack - Copilot Agent Instructions

## Repository Overview

**jhack** is a Python-based CLI tool packed with Juju hacks and utilities for charm development. It provides an opinionated collection of scripts and utilities to make charm development more efficient and debugging easier.

**Key Stats:**
- **Size:** ~66MB (includes dependencies)
- **Language:** Python 3.10+ (supports up to 3.12)
- **Type:** CLI application with multiple subcommands
- **Python Files:** ~1200+ files
- **Package Manager:** uv (preferred), pip (fallback)
- **Build System:** setuptools via pyproject.toml
- **Distribution:** PyPI package and Snap package

**Primary Dependencies:**
- ops[testing] >= 3.0.0 (Juju operator framework)
- typer 0.18.0 (CLI framework)
- rich 13.3.0 (terminal formatting)
- Various utilities: parse, requests, urllib3, asttokens, astunparse, toml

## Environment Setup

### Initial Setup (Required Steps)

**ALWAYS run these commands in order when setting up the environment:**

1. **Install uv package manager** (if not already installed):
   ```bash
   pip install uv
   ```

2. **Create virtual environment and install dependencies:**
   ```bash
   uv venv
   uv pip install --all-extras -r pyproject.toml
   ```
   
   **Note:** This command takes ~20-30 seconds to complete and is required before any other development work.

3. **Alternative: Install in editable mode with pip:**
   ```bash
   pip install -e .
   ```
   This is faster but doesn't include dev dependencies. Use the uv method for development work.

### Environment Activation

**ALWAYS activate the virtual environment before running commands:**
```bash
source .venv/bin/activate
```

## Build, Lint, Test Commands

### Linting and Formatting

**Format code (auto-fix):**
```bash
make fmt
```
- Uses: ruff check --fix-only and ruff format
- Time: ~0.1-1 second
- Run this before committing code

**Lint code (check only):**
```bash
make lint
```
- Uses: ruff check, ruff format --check --diff, pyright
- Time: ~0.1-1 second
- **Note:** The repository currently has 21 pre-existing lint errors that can be ignored. These are NOT introduced by new changes and should not be fixed unless specifically addressing them.
- Pre-existing errors include: F821 (undefined names), F841 (unused variables), and TypedDict warnings

### Testing

**Run unit tests:**
```bash
make unit
```
- Uses: pytest with coverage
- Time: ~2-5 seconds for collection, longer for full run
- **Known Issue:** Test import error in `jhack/tests/utils/test_tail.py` (ImportError: cannot import name 'Level'). This is a pre-existing issue.
- Tests automatically enable devmode via conftest.py fixture

**Test specific module:**
```bash
uv run --isolated --extra dev -m pytest jhack/tests/specific_test.py -v
```

### Building

**Build package distributions:**
```bash
pip install build  # if not already installed
python -m build
```
- Creates source distribution (sdist) and wheel in `dist/` directory
- **Note:** Build may timeout on slow connections when downloading dependencies
- Time: 1-2 minutes (network dependent)

**Build snap package:**
```bash
snapcraft pack
```
- Requires snapcraft installed
- Creates .snap file for distribution

### Cleaning

**Clean build artifacts:**
```bash
make clean
```
- Removes: .coverage, .pytest_cache, .ruff_cache, .venv, *.charm, *.rock, __pycache__, *.egg-info, requirements*.txt

## Project Architecture and Layout

### Root Directory Files

```
.
├── .gitignore              # Git ignore patterns
├── .pre-commit-config.yaml # Pre-commit hooks (black, isort, yaml checks)
├── CONTRIBUTING.md         # Contribution guidelines
├── LICENSE.txt             # Apache 2.0 license
├── MANIFEST.in             # Package manifest (includes .toml, .sh files)
├── Makefile                # Build automation (lint, fmt, unit, clean, lock)
├── README.md               # Comprehensive usage documentation
├── README_SNAP.md          # Snap-specific documentation
├── pyproject.toml          # Python project configuration
├── setup.py                # Minimal setuptools entry point
├── snapcraft.yaml          # Snap build configuration
└── uv.lock                 # Locked dependencies for uv
```

### Source Code Structure

**jhack/** (main package directory):
- **main.py** - CLI entry point, imports all commands
- **config.py** - Configuration management
- **helpers.py** - Shared utilities (juju commands, file operations)
- **logger.py** - Logging configuration
- **version.py** - Version information

**Key Subdirectories:**

1. **jhack/utils/** - Core utility commands
   - `simulate_event.py` - Fire events on units (jhack fire)
   - `show_relation.py` - Display relation databags (jhack show-relation)
   - `show_stored.py` - Display stored state (jhack show-stored)
   - `sync.py` - Sync local files to remote units (jhack sync)
   - `nuke.py` - Destroy resources (jhack nuke)
   - `ffwd.py` - Fast-forward update-status hooks
   - `tail_charms/` - Event log monitoring (jhack tail)
   - `event_recorder/` - Event recording and replay
   - `integrate.py` - Integration matrix management (jhack imatrix)
   - `unbork_juju.sh` - Shell script to fix broken Juju installations

2. **jhack/charm/** - Charm development utilities
   - `functional.py` - Charm repack and refresh (jhack charm repack)
   - `provision.py` - Provision units with scripts
   - `vinfo.py` - Version information display (jhack charm-info)
   - `lobotomy.py` - Disable event processing (jhack charm lobotomy)
   - `update.py` - Update packed charm files
   - `sync.py` - Sync charm directories

3. **jhack/scenario/** - Scenario testing integration
   - `snapshot.py` - Snapshot live charm state
   - `state_apply.py` - Apply state to live charms
   - `integrations/` - Scenario backend integrations

4. **jhack/chaos/** - Chaos testing tools
   - `mancioppi.py` - Scale up/down chaos testing
   - `flicker.py` - Relation flicker testing

5. **jhack/conf/** - Configuration management
   - `jhack_config_defaults.toml` - Default config
   - `jhack_config_destructive.toml` - Destructive mode config
   - `jhack_config_yolo.toml` - YOLO mode config
   - `conf.py` - Config loading and management

6. **jhack/model/** - Juju model utilities

7. **jhack/tests/** - Test suite
   - `conftest.py` - Pytest configuration (auto-enables devmode)
   - `test_devmode.py` - Devmode tests
   - `test_scenario/` - Scenario integration tests
   - `utils/` - Utility tests
   - `charm/` - Charm utility tests
   - `config/` - Config tests

### Configuration Files

**pyproject.toml** - Python project metadata:
- Build system: setuptools
- Dependencies: ops, typer, rich, etc.
- Dev dependencies: pytest, coverage, black, ruff, isort
- Entry point: `jhack = "jhack.main:main"`
- Tool configs: ruff (line-length 99), black, isort (profile black)

**.pre-commit-config.yaml** - Pre-commit hooks:
- trailing-whitespace, end-of-file-fixer
- check-yaml, check-added-large-files
- black, isort

**Makefile targets:**
- `make lock` - Update uv.lock dependencies
- `make requirements` - Generate requirements.txt
- `make lint` - Run linters (ruff, pyright)
- `make fmt` - Format code (ruff, black)
- `make unit` - Run tests with coverage
- `make clean` - Remove artifacts

### Key Architectural Patterns

1. **CLI Structure**: Uses typer for nested command groups (utils, charm, scenario, chaos, etc.)
2. **Juju Integration**: Heavy reliance on subprocess calls to `juju` CLI commands
3. **Remote Execution**: Uses `juju ssh` and `juju scp` for unit operations
4. **Configuration**: TOML-based config at `~/.config/jhack/config.toml`
5. **Devmode**: Environment variable `JHACK_PROFILE=devmode` or config setting for destructive commands

## Validation and CI/CD

**Pre-commit Hooks:**
The repository uses pre-commit hooks for automatic validation:
```bash
pre-commit install  # Install hooks (one-time setup)
pre-commit run --all-files  # Run all hooks manually
```

**No GitHub Workflows:**
There are currently no GitHub Actions workflows or CI/CD pipelines configured. All validation must be done locally using the Makefile targets.

**Manual Validation Steps:**
Before committing changes, ALWAYS run:
1. `make fmt` - Auto-format code
2. `make lint` - Check for errors (ignore 21 pre-existing errors)
3. `make unit` - Run tests (ignore pre-existing import error in test_tail.py)

## Common Pitfalls and Workarounds

### 1. Import Errors
**Problem:** "cannot find jhack modules" error
**Solution:** Ensure you're in the repo root and jhack is installed (`pip install -e .`)

### 2. Test Import Error (Pre-existing)
**Problem:** `ImportError: cannot import name 'Level' from 'jhack.utils.tail_charms.core.juju_model_loglevel'`
**Workaround:** This is a known issue in test_tail.py. It doesn't affect functionality.

### 3. Lint Errors (Pre-existing)
**Problem:** 21 lint errors including undefined names and unused variables
**Workaround:** These are pre-existing. Don't fix them unless specifically addressing them in your changes.

### 4. Build Timeouts
**Problem:** `python -m build` times out downloading dependencies
**Workaround:** Use `pip install -e .` for development or increase timeout

### 5. Juju Commands Require Juju
**Problem:** Many commands require `juju` CLI to be installed and configured
**Note:** Tests mock juju interactions, but real usage requires Juju environment

### 6. Syntax Warnings
**Problem:** Various SyntaxWarning about invalid escape sequences
**Workaround:** These are pre-existing and appear when running commands. They're not errors.

## Development Workflow

### Making Changes

1. **Setup environment:**
   ```bash
   uv venv && uv pip install --all-extras -r pyproject.toml
   source .venv/bin/activate
   ```

2. **Make your changes to Python files in jhack/**

3. **Test your changes:**
   ```bash
   jhack <command> --help  # Test CLI
   make fmt                # Format code
   make lint               # Check lint (ignore pre-existing 21 errors)
   make unit               # Run tests (ignore pre-existing test_tail import error)
   ```

4. **If adding new commands:**
   - Add to appropriate module (utils/, charm/, chaos/, scenario/)
   - Import and register in jhack/main.py
   - Add tests in jhack/tests/

5. **If changing dependencies:**
   - Update pyproject.toml
   - Run `make lock` to update uv.lock
   - Test installation fresh: `uv venv && uv pip install --all-extras -r pyproject.toml`

### Testing Locally

**Quick test of CLI:**
```bash
source .venv/bin/activate
jhack --help
jhack utils --help
jhack charm --help
```

**Test specific functionality:**
```bash
# Enable debug logging
jhack --loglevel=DEBUG <command>

# Log to file
jhack --log-to-file=./jhack.log <command>
```

## Additional Context

### Destructive Commands
Many jhack commands are destructive (nuke, lobotomy, etc.). They:
- Require confirmation unless `JHACK_PROFILE=devmode` is set
- Are controlled by `~/.config/jhack/config.toml`
- Auto-enable devmode in test environment (conftest.py)

### Snap Distribution
The snap package:
- Based on core24
- Bundles juju/3.6/stable snap
- Requires plugs: network, dot-local-share-juju, ssh-read, home-read
- Built with snapcraft.yaml

### Key Design Decisions
- **CLI Tree:** Organized by command category (utils, charm, chaos, scenario)
- **Shortcuts:** Common commands available at top level (fire, sync, tail, etc.)
- **Juju Interaction:** Wraps juju CLI rather than using libjuju for simplicity
- **Rich Output:** Uses rich library for formatted terminal output
- **Typer CLI:** All commands use typer decorators for argument parsing

## Trust These Instructions

These instructions have been validated by:
- Testing environment setup from scratch
- Running all Makefile targets (lint, fmt, unit, clean)
- Verifying build process with both uv and pip
- Checking pre-existing errors are documented
- Testing CLI functionality and imports

**Only search for additional information if:**
- You encounter errors not documented here
- You need details about specific command implementations
- Instructions prove incorrect or outdated
