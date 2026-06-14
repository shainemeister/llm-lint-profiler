# Model Profiler v1.0 — Core Library

**Project Intent**  
A lightweight, maintainable Python library that processes Ruff and Pylint JSON error outputs generated from local LLM code generation runs. It programmatically identifies the most frequent errors per model and produces two distinct outputs:

1. **Clean, minimal profiles** (`profiles/`) — designed to be injected into future prompts to help the model avoid repeating its common mistakes.
2. **Detailed analytical data** (`data/`) — used for tracking error trends, frequency weighting, and long-term model behavior analysis.

All error processing is done **programmatically** (no AI summarization) to ensure accuracy, speed, and zero hallucination risk.

---

## Design Principles (v1.0)

- **Strict Separation of Concerns**: Clean AI-facing profiles are kept completely separate from raw analytical data.
- **Single Responsibility Principle**: Each class has one clear job.
- **Programmatic Determinism**: All counting, sorting, and file generation is done in pure Python.
- **Ollama Compatibility**: Model names are sanitized safely for any operating system.
- **Maintainability First**: Small, focused modules that are easy to test, extend, and refactor.
- **Minimal Public API**: Users primarily interact with one orchestrator class.

---

## Folder Structure (v1.0)

```
model-profiler/
├── profiles/                    # Clean, minimal JSON profiles for AI prompting
├── data/                        # Full analytical data for analysis & trends
├── src/
│   ├── __init__.py
│   ├── namer.py                 # Model name sanitization & path handling
│   ├── analyzer.py              # Ruff + Pylint JSON parsing & counting
│   ├── writer.py                # Clean profile JSON writer
│   ├── logger.py                # Detailed analytical data logger
│   └── profiler.py              # Thin orchestrator (public API)
├── pyproject.toml
└── README.md
```

---

## Core Components & Detailed Definitions

### 1. ModelNamer (`src/namer.py`)

**Purpose**  
Handles all model name transformation and filesystem-safe path generation. This class ensures consistent, collision-free filenames across different Ollama model naming conventions.

**Class:** `ModelNamer`

**Methods:**

```python
def sanitize(self, model_name: str) -> str:
    """
    Convert any Ollama model name into a safe, lowercase filename.
    
    Replaces forbidden or problematic characters:
    - ':' → '-'
    - '.' → '-'
    - spaces → '-'
    - other unsafe chars → '-'
    
    Strips leading/trailing hyphens.
    
    Examples:
        "deepseek-coder:6.7b"           → "deepseek-coder-6-7b"
        "qwen2.5-coder:32b-instruct"    → "qwen2-5-coder-32b-instruct"
        "llama3.2:3b"                   → "llama3-2-3b"
    """
```

```python
def get_profile_path(self, model_name: str) -> Path:
    """Return full Path object for the clean AI profile JSON."""
```

```python
def get_data_path(self, model_name: str) -> Path:
    """Return full Path object for the detailed analytical data JSON."""
```

---

### 2. ErrorAnalyzer (`src/analyzer.py`)

**Purpose**  
The single source of truth for parsing Ruff and Pylint JSON outputs. It combines results from both tools and produces a clean frequency count of every error code/symbol.

**Class:** `ErrorAnalyzer`

**Methods:**

```python
def analyze(self, ruff_json_path: str, pylint_json_path: str) -> dict[str, int]:
    """
    Parse both Ruff and Pylint JSON files and return error frequency counts.
    
    - Loads Ruff JSON (array of violations)
    - Loads Pylint JSON (usually under 'messages' or top-level array depending on format)
    - Counts occurrences of each error code/symbol (e.g. 'E501', 'C0114', 'W0612')
    - Returns a dictionary sorted by frequency (most common first)
    
    This method is the core of the entire system and must remain fast and deterministic.
    """
```

---

### 3. ProfileWriter (`src/writer.py`)

**Purpose**  
Writes the **clean, minimal profile** intended for consumption by LLMs. This file should stay small, focused, and free of noise so it can be safely injected into prompts without bloating context.

**Class:** `ProfileWriter`

**Methods:**

```python
def write(self, model_name: str, error_counts: dict[str, int], output_path: Path) -> bool:
    """
    Write a clean, minimal JSON profile for AI prompting.
    
    The output should contain only:
    - model name
    - sanitized name
    - top N most frequent errors (with counts)
    - total error count
    - timestamp (optional but recommended)
    
    This file is what gets fed back into future generation prompts.
    """
```

---

### 4. DataLogger (`src/logger.py`)

**Purpose**  
Persists the **full analytical dataset** for every run. This enables future features like trend analysis, error weighting, improvement tracking over iterations, and model comparison.

**Class:** `DataLogger`

**Methods:**

```python
def log(self, model_name: str, ruff_json_path: str, pylint_json_path: str, 
        iteration: int, output_path: Path) -> bool:
    """
    Save complete analytical data for a single iteration/run.
    
    Stores:
    - Full raw error lists (or references)
    - Per-iteration error counts
    - Timestamp
    - Model name + sanitized name
    - Iteration number
    
    This data lives in `data/` and is never directly fed to AI models.
    """
```

---

### 5. ModelProfiler (`src/profiler.py`)

**Purpose**  
The **main public interface** and thin orchestrator. It coordinates the other four classes so end users only need to call one or two methods.

**Class:** `ModelProfiler`

**Methods:**

```python
def __init__(self, profile_dir: str = "profiles", data_dir: str = "data"):
    """Initialize with configurable output directories."""
```

```python
def generate(self, model_name: str, ruff_json: str, pylint_json: str, 
             iteration: int = 0) -> bool:
    """
    Main entry point for v1.0.
    
    Workflow:
    1. Sanitize model name
    2. Analyze Ruff + Pylint JSON files
    3. Write clean profile to `profiles/`
    4. Log full analytical data to `data/`
    
    Returns True on success, False on any failure.
    This is the primary method users will call.
    """
```

```python
def get_profile_path(self, model_name: str) -> Path:
    """Convenience method to retrieve the clean profile path."""
```

```python
def get_data_path(self, model_name: str) -> Path:
    """Convenience method to retrieve the analytical data path."""
```

---

## Recommended Usage (v1.0)

```python
from src.profiler import ModelProfiler

profiler = ModelProfiler(profile_dir="profiles", data_dir="data")

success = profiler.generate(
    model_name="deepseek-coder:6.7b",
    ruff_json="ruff_output.json",
    pylint_json="pylint_output.json",
    iteration=1
)
```

---

## Summary

v1.0 delivers a clean, modular, and highly maintainable library focused purely on the core value: turning raw linter JSON into actionable, model-specific error profiles — all without AI summarization or unnecessary complexity.

This foundation is designed to be extended cleanly in v2.0 (CLI, configuration, richer analytics, etc.) without requiring changes to the existing class structure.
