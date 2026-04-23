# Python argparse — CLI Argument Parsing (Tier 3)

> **Tier 3** | Source: Python Argparse HOWTO, docs.python.org/3/howto/argparse.html | Enforces/Derives From: wiki/tier3-working/python/overview.md, wiki/tier1-sources/python-peps/pep-020-zen.md

## Summary

Python's `argparse` module provides structured command-line argument parsing with automatic help generation, type conversion, and error messaging. It is the standard library solution for CLI tools — more capable than `sys.argv` manual parsing and more Pythonic than `optparse` (deprecated). A coding agent writing any CLI tool must use `argparse` and follow the patterns below for testability and consistency.

## Key Concepts

### Basic Structure

```python
import argparse

def build_parser() -> argparse.ArgumentParser:
    parser = argparse.ArgumentParser(
        description="Process data files",
        formatter_class=argparse.ArgumentDefaultsHelpFormatter,  # shows defaults in help
    )
    return parser

def main() -> None:
    parser = build_parser()
    args = parser.parse_args()
    # ... use args

if __name__ == "__main__":
    main()
```

Separate `build_parser()` from `main()` to enable testing without calling `sys.exit`.

### Positional vs Optional Arguments

```python
# Positional — required, order matters
parser.add_argument("input_file", help="Path to input CSV file")

# Optional — prefixed with --
parser.add_argument("--output", "-o", default="output.csv", help="Output file path")

# Flag (boolean switch)
parser.add_argument("--verbose", "-v", action="store_true", help="Enable verbose output")
parser.add_argument("--quiet",   "-q", action="store_true", help="Suppress output")
```

### Types and Validation

```python
# Built-in types
parser.add_argument("--count", type=int, default=10, help="Number of records")
parser.add_argument("--ratio", type=float, default=0.5)
parser.add_argument("--path",  type=argparse.FileType('r'), help="Input file")

# Restricted choices
parser.add_argument(
    "--format",
    choices=["json", "csv", "parquet"],
    default="json",
    help="Output format"
)

# Custom type converter (callable that takes str, returns value or raises ValueError)
def positive_int(value: str) -> int:
    ivalue = int(value)
    if ivalue <= 0:
        raise argparse.ArgumentTypeError(f"{value} is not a positive integer")
    return ivalue

parser.add_argument("--workers", type=positive_int, default=4)
```

### nargs — Variable Number of Values

```python
# One or more values
parser.add_argument("files", nargs="+", help="One or more input files")

# Zero or more
parser.add_argument("--tags", nargs="*", default=[], help="Optional tags")

# Exactly N
parser.add_argument("--range", nargs=2, type=int, metavar=("START", "END"))

# Optional single value
parser.add_argument("--config", nargs="?", const="config.yaml", default=None)
```

### Mutually Exclusive Groups

```python
group = parser.add_mutually_exclusive_group()
group.add_argument("--verbose", action="store_true")
group.add_argument("--quiet",   action="store_true")

# Required mutual exclusion
group = parser.add_mutually_exclusive_group(required=True)
group.add_argument("--create", action="store_true")
group.add_argument("--delete", action="store_true")
```

### Subcommands

```python
def build_parser() -> argparse.ArgumentParser:
    parser = argparse.ArgumentParser(description="Data pipeline")
    subparsers = parser.add_subparsers(dest="command", required=True)

    # 'ingest' subcommand
    ingest = subparsers.add_parser("ingest", help="Ingest data from source")
    ingest.add_argument("source", help="Source URL")
    ingest.add_argument("--limit", type=int, default=1000)

    # 'export' subcommand
    export = subparsers.add_parser("export", help="Export processed data")
    export.add_argument("--format", choices=["json", "csv"], default="json")

    return parser

def main() -> None:
    args = build_parser().parse_args()
    if args.command == "ingest":
        run_ingest(args.source, args.limit)
    elif args.command == "export":
        run_export(args.format)
```

### Logging Level from CLI

```python
import logging

parser.add_argument(
    "--log-level",
    default="INFO",
    choices=["DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"],
    help="Set the logging level"
)

args = parser.parse_args()
logging.basicConfig(level=getattr(logging, args.log_level))
```

### Testing argparse

```python
def test_defaults() -> None:
    parser = build_parser()
    args = parser.parse_args([])
    assert args.output == "output.csv"
    assert args.verbose is False

def test_verbose_flag() -> None:
    parser = build_parser()
    args = parser.parse_args(["--verbose"])
    assert args.verbose is True

def test_invalid_format_exits(capsys) -> None:
    import pytest
    parser = build_parser()
    with pytest.raises(SystemExit):
        parser.parse_args(["--format", "xml"])
```

## Agent Guidance

### Do

- Separate `build_parser()` from `main()` to enable unit testing of argument parsing.
- Use `type=` with a custom converter function for domain validation — it integrates with argparse error handling and help text.
- Use `ArgumentDefaultsHelpFormatter` to automatically include defaults in generated help output.
- Use `add_mutually_exclusive_group()` rather than manual checks for arguments that cannot coexist.
- Use `dest="command"` with `required=True` on `add_subparsers()` for tools with multiple subcommands.
- Derive the logging level from a `--log-level` argument using `choices=` to restrict valid values.

### Do Not

- Do not call `parser.parse_args()` inside functions that should be testable — pass `args` in instead.
- Do not use `sys.argv` directly when `argparse` is available.
- Do not perform argument validation in `main()` — use `type=` or `choices=` to push validation into the parser so error messages are consistent.
- Do not use positional arguments for optional configuration — positional arguments imply required inputs.
- Do not suppress `--help` (`add_help=False`) without a documented reason.

## Checklist

- [ ] `ArgumentParser` built in a dedicated `build_parser()` function (separate from `main`)
- [ ] Type conversion uses `type=` parameter, not post-parse casting
- [ ] `choices=` used to restrict discrete-value arguments
- [ ] Mutually exclusive arguments use `add_mutually_exclusive_group()`
- [ ] Log level driven by `--log-level` argument with `choices=` constraint
- [ ] Tests pass argument lists directly to `parser.parse_args([...])` without `sys.argv`

## See Also

- wiki/tier3-working/python/overview.md
- wiki/tier3-working/python/logging.md
- wiki/tier1-sources/python-peps/pep-020-zen.md
- wiki/tier3-working/python/type-system.md

## Source

Python Argparse HOWTO, docs.python.org/3/howto/argparse.html. Python `argparse` module documentation.
