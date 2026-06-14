# llm-lint-profiler

**Python library that analyzes Ruff and Pylint JSON output from local LLM code generation and creates clean, model-specific error profiles to improve future generations.**

Local models often repeat the same mistakes. This tool profiles those mistakes **programmatically** (no AI summarization) and produces concise, injectable profiles you can feed back into prompts.

---

## Why This Exists

When using local models (Ollama, Continue.dev, etc.) for code generation, the same errors (style violations, missing docstrings, incorrect argument counts, etc.) tend to appear repeatedly. Manually fixing them every time is tedious.

`llm-lint-profiler` solves this by:

1. Running Ruff (fast, broad) + Pylint (deeper analysis) on generated code
2. Counting error frequencies across runs
3. Producing two outputs:
   - **Clean profiles** (`profiles/`) — minimal JSON designed to be injected into future prompts
   - **Analytical data** (`data/`) — full details for tracking model behavior over time

All processing is deterministic Python. No hallucinations, no context bloat.

---

## Key Features (v1.0)

- **Programmatic error analysis** — Pure Python `Counter` + JSON parsing
- **Strict separation of concerns** — AI-facing profiles vs raw analytical data
- **Ollama-friendly naming** — Safely converts any model name (e.g. `deepseek-coder:6.7b`) into valid filenames
- **Maintainable architecture** — 5 focused classes following Single Responsibility Principle
- **Minimal public API** — One main `generate()` method to rule them all

---

## Installation

```bash
pip install llm-lint-profiler    # Not yet available
```

> **Note:** v1.0 is the core library only. A CLI (Typer + Rich) is planned for v2.0.

---

## Quick Start

```python
from llm_lint_profiler import ModelProfiler

profiler = ModelProfiler(profile_dir="profiles", data_dir="data")

success = profiler.generate(
    model_name="deepseek-coder:6.7b",
    ruff_json="ruff_output.json",
    pylint_json="pylint_output.json",
    iteration=1
)

if success:
    print("Profile generated successfully!")
    print(profiler.get_profile_path("deepseek-coder:6.7b"))
```

The resulting `profiles/model-profile-deepseek-coder-6-7b.json` contains the top errors the model makes most frequently — ready to paste into your system prompt or few-shot examples.

---

## How It Works (High-Level)

```
Generated Code
      │
      ▼
Ruff JSON + Pylint JSON
      │
      ▼
ErrorAnalyzer (counts frequencies)
      │
      ├──────────────► ProfileWriter → profiles/model-profile-*.json   (for AI)
      │
      └──────────────► DataLogger   → data/model-data-*.json         (for analysis)
```

**Core Classes:**

| Class            | File            | Responsibility |
|------------------|-----------------|----------------|
| `ModelNamer`     | `namer.py`      | Sanitize model names & generate safe paths |
| `ErrorAnalyzer`  | `analyzer.py`   | Parse Ruff + Pylint JSON and count errors |
| `ProfileWriter`  | `writer.py`     | Write minimal, AI-ready profile JSON |
| `DataLogger`     | `logger.py`     | Persist full analytical data per iteration |
| `ModelProfiler`  | `profiler.py`   | Thin orchestrator (public API) |

---

## Project Structure

```
llm-lint-profiler/
├── profiles/                 # Clean JSON profiles for prompting
├── data/                     # Detailed analytical data
├── src/
│   ├── namer.py
│   ├── analyzer.py
│   ├── writer.py
│   ├── logger.py
│   └── profiler.py
├── pyproject.toml
└── README.md
```

---

## Roadmap

| Version | Focus                              | Status      |
|---------|------------------------------------|-------------|
| v1.0    | Core library + clean architecture  | In progress |
| v2.0    | Typer CLI + `model-profiler` command | Planned    |
| v2.x    | Trend analysis, model comparison   | Future     |

---

## Design Philosophy

- **Library first** — CLI and other interfaces come later
- **No AI in the hot path** — All counting and file generation is pure Python
- **Small, focused modules** — Easy to test, easy to extend
- **Future-proof naming** — Works with any Ollama model name today and tomorrow

---

## Contributing

Contributions are welcome! Please open an issue first to discuss major changes.

When contributing:
- Keep classes small and focused (Single Responsibility)
- Maintain the clean separation between `profiles/` and `data/`
- Add type hints and docstrings
- Update this README when behavior changes

---

## License

MIT License — see `LICENSE` file for details.

---

**Built to make local LLM code generation less repetitive and more reliable.**

If this project helps you, consider starring the repo or sharing your use case!
