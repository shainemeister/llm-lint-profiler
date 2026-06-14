# Model Profiler v2.0 — Core Library + CLI

**Project Intent**  
Extend the solid v1.0 library with a professional command-line interface (CLI) while preserving the clean, maintainable architecture. The CLI makes the tool usable directly from the terminal and enables easy integration into CI/CD pipelines, scripts, and developer workflows.

v2.0 **does not** change the core library classes or behavior. It adds a thin CLI layer on top.

---

## Version Scope

### v1.0 (Foundation)
- Pure Python library
- 5 focused classes (`ModelNamer`, `ErrorAnalyzer`, `ProfileWriter`, `DataLogger`, `ModelProfiler`)
- Clean separation of `profiles/` and `data/`
- Programmatic error processing only

### v2.0 Additions (This Document)
- Command-line interface using **Typer** (recommended) or Click
- Console script entry point (`model-profiler` command after installation)
- Basic configuration support (optional `config.toml` or `pyproject.toml` section)
- Helpful terminal output (using `rich` for tables, colors, progress)
- Commands for generating profiles and viewing existing ones
- Clear error messages and verbose mode

**Important:** The core library API remains stable. CLI is an optional add-on.

---

## Design Principles (v2.0)

- **Library First, CLI Second**: The CLI is a thin wrapper around the existing `ModelProfiler` class.
- **No Convolution of Core**: All new CLI code lives in its own module (`src/cli.py`).
- **User Experience**: Simple, discoverable commands with helpful help text.
- **Extensibility**: Easy to add subcommands later (e.g. `compare`, `trends`, `export`).
- **Professional Packaging**: Proper `pyproject.toml` entry point so `pip install` gives users the `model-profiler` command.

---

## Updated Folder Structure (v2.0)

```
model-profiler/
├── profiles/                    # Clean AI profiles (unchanged)
├── data/                        # Analytical data (unchanged)
├── src/
│   ├── __init__.py
│   ├── namer.py
│   ├── analyzer.py
│   ├── writer.py
│   ├── logger.py
│   ├── profiler.py              # Core orchestrator (unchanged public API)
│   └── cli.py                   # NEW: Typer-based CLI application
├── config.py                    # Optional configuration handling
├── pyproject.toml               # Console script entry point defined here
└── README.md
```

---

## Core Components (v1.0 classes remain unchanged)

The five classes from v1.0 stay exactly the same.  
Only `src/cli.py` and `pyproject.toml` are new.

---

## New Component: CLI (`src/cli.py`)

**Purpose**  
Provides a user-friendly command-line interface that calls into the existing `ModelProfiler` class.

**Recommended Library:** Typer (modern, type-hinted, excellent help text and autocompletion)

**Class / Application Structure:**

```python
import typer
from rich import print
from pathlib import Path
from .profiler import ModelProfiler

app = typer.Typer(
    name="model-profiler",
    help="Generate and manage model-specific error profiles from Ruff + Pylint output.",
    add_completion=True
)

@app.command()
def generate(
    model: str = typer.Argument(..., help="Ollama model name (e.g. deepseek-coder:6.7b)"),
    ruff: Path = typer.Option(..., "--ruff", "-r", help="Path to Ruff JSON output"),
    pylint: Path = typer.Option(..., "--pylint", "-p", help="Path to Pylint JSON output"),
    iteration: int = typer.Option(0, "--iteration", "-i", help="Iteration number for tracking"),
    profile_dir: str = typer.Option("profiles", "--profile-dir"),
    data_dir: str = typer.Option("data", "--data-dir"),
):
    """
    Run Ruff + Pylint analysis and generate both clean profile and analytical data.
    """
    profiler = ModelProfiler(profile_dir=profile_dir, data_dir=data_dir)
    success = profiler.generate(
        model_name=model,
        ruff_json=str(ruff),
        pylint_json=str(pylint),
        iteration=iteration
    )
    if success:
        print(f"[green]✓ Profile generated for[/green] {model}")
    else:
        print("[red]✗ Failed to generate profile[/red]")
        raise typer.Exit(code=1)


@app.command()
def show(
    model: str = typer.Argument(..., help="Model name to display profile for"),
    profile_dir: str = typer.Option("profiles", "--profile-dir"),
):
    """Display the clean profile for a given model."""
    profiler = ModelProfiler(profile_dir=profile_dir)
    path = profiler.get_profile_path(model)
    if path.exists():
        print(path.read_text())
    else:
        print(f"[red]No profile found for[/red] {model}")
        raise typer.Exit(code=1)


@app.command()
def list(
    profile_dir: str = typer.Option("profiles", "--profile-dir"),
):
    """List all generated model profiles."""
    # Implementation lists JSON files in profile_dir
    ...
```

**Key Design Notes for CLI:**
- Each command is a thin wrapper — all heavy logic stays in the library classes.
- Uses `typer.Option` and `typer.Argument` for clean, self-documenting interfaces.
- Rich output for better readability (tables, colors, progress bars in future commands).
- Graceful error handling with meaningful messages.

---

## Recommended `pyproject.toml` Additions (v2.0)

```toml
[project.scripts]
model-profiler = "src.cli:app"

[project.optional-dependencies]
cli = ["typer[all]>=0.9.0", "rich>=13.0.0"]
```

Users can then install with:

```bash
pip install "model-profiler[cli]"
```

And run:

```bash
model-profiler generate deepseek-coder:6.7b --ruff ruff.json --pylint pylint.json
model-profiler show deepseek-coder:6.7b
model-profiler list
```

---

## Recommended Commands for v2.0

| Command                  | Description                                      | Status     |
|--------------------------|--------------------------------------------------|------------|
| `model-profiler generate` | Analyze Ruff + Pylint and create profile + data | Core       |
| `model-profiler show`     | Display clean profile for a model                | Core       |
| `model-profiler list`     | List all available profiles                      | Core       |
| `model-profiler trends`   | Show error trend analysis (future)               | v2.x / v3  |
| `model-profiler compare`  | Compare two models side-by-side (future)         | v2.x / v3  |

---

## Migration Path from v1.0 to v2.0

- v1.0 users continue to import and use `ModelProfiler` directly — **no breaking changes**.
- v2.0 users get the additional convenience of the `model-profiler` CLI command.
- All new functionality is additive.

---

## Summary

**v2.0** adds a professional, discoverable CLI layer on top of the rock-solid v1.0 library without increasing complexity in the core classes.

This phased approach keeps the initial development focused and clean while providing a clear, low-risk path to a full-featured developer tool.

The architecture remains highly maintainable because:
- Core logic never mixes with presentation (CLI)
- Each class continues to have a single responsibility
- Future features (trends, comparison, web UI, etc.) can be added as new modules or subcommands without touching existing code
