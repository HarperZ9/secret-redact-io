<p align="center"><img src="docs/art/secret-redact-io-header.svg" alt="secret-redact-io: guarded io for tools. Read, write, fetch and run with the secrets stripped on the way out." width="100%"></p>

# Secret Redact IO

Brand assets: `.github/assets/zentropy-banner.png` and `docs/brand/secret-redact-io-hero.png`.

> Safe IO for agent tools: read, write, fetch, and exec with redaction and receipts.

## Why it matters

Agents need IO, but raw IO can leak credentials or private payloads into logs and model context. Secret Redact IO gives tools a small guarded boundary: outputs are redacted, receipts are hash-only, and the original secret-shaped values are not archived.

## What to test first

- Read a file that contains a fake token and confirm the returned text is redacted.
- Run the dry-run write path and inspect the receipt before anything is persisted.
- Execute a subprocess that prints a fake secret and confirm stdout is redacted.

## Technical framing

> Stdlib-only guarded file/fetch/subprocess IO: strips API keys, tokens, and PEM keys; emits hash-only receipts.

[![license: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![python](https://img.shields.io/badge/python-3.10%2B-blue.svg)
![version](https://img.shields.io/badge/version-0.1.0-informational.svg)
[![CI](https://github.com/HarperZ9/secret-redact-io/actions/workflows/tests.yml/badge.svg)](https://github.com/HarperZ9/secret-redact-io/actions/workflows/tests.yml)
![deps: none](https://img.shields.io/badge/deps-none-success.svg)
[![part of: AI-accountability toolkit](https://img.shields.io/badge/part_of-AI--accountability_toolkit-7a5cff.svg)](https://harperz9.github.io)

`secret-redact-io` is a small Python SDK and CLI for guarded IO. It wraps file
reads, file writes, HTTP fetches, and subprocess execution with redaction and
hash-only audit receipts.

The default posture is conservative:

- user-facing output is redacted before it is returned;
- guarded writes persist redacted content by default;
- receipts store byte counts, hashes, status metadata, and redaction counts;
- receipts do not archive raw input or raw secret values.

<p align="center"><img src="docs/art/guarded-lane.svg" alt="Eight stages from a guarded call to the caller, ending in redacted, untouched, or still leaking." width="100%"></p>

## Install

```bash
python -m pip install secret-redact-io
```

## CLI

```bash
secret-redact-io read README.md --json
secret-redact-io write out.txt --content "note=hello" --dry-run --json
secret-redact-io fetch https://example.com --json
secret-redact-io exec --json -- python -c "print('hello')"
```

## Python API

```python
from secret_redact_io import read_text_guarded, run_guarded

read_result = read_text_guarded("README.md")
print(read_result.text)
print(read_result.receipt.to_json())

exec_result = run_guarded(["python", "-c", "print('hello')"])
print(exec_result.stdout)
```

## What a receipt records

<p align="center"><img src="docs/art/receipt-fields.svg" alt="The nine keys a guardrail receipt returns, one to a row, each with what fills it. Two are digests, over the input bytes and over the returned text. Three are counts: the length of the input, the length of what came back, and how many times each rule fired. Two are names: which operation ran, and what it acted on, trimmed to a path or a program name. One is a clock stamp. The last is metadata, and its row is accented, because the caller fills it and nothing redacts what goes in." width="100%"></p>

Nine keys, and seven of them are a hash, a count, or a name the tool trimmed itself. `target` keeps a path, a host and path, or a program name, and drops arguments and query strings before it is set. `metadata` is the exception: it holds what the caller handed over, and nothing redacts it.

## Usage

See [USAGE.md](USAGE.md) for an install line, the full CLI and Python API
surface, and worked examples with expected output. A runnable end-to-end
demo lives in [examples/demo.py](examples/demo.py).

## Boundary

This package is a public, self-contained guardrail utility. It does not include
credentials, secrets, or any deployment-specific configuration.

<p align="center"><img src="docs/art/blind-spots-lane.svg" alt="Eight stages across the honesty surface, ending in hash only, unmatched, or open field." width="100%"></p>

The policy is seven patterns and stays seven. Each one matches a shape, so a private key block, a key carrying a known prefix, or a value beside the word `password` is caught. A credential that reads as ordinary prose has no shape to match and survives the pass. That is the honest edge of the tool, and it is why the receipt reports which rules fired rather than claiming the text is clean.

---
**Zain Dana Harper** -- small tools with explicit edges.
[Portfolio](https://harperz9.github.io) · [HarperZ9](https://github.com/HarperZ9)
<sub>Built with Claude Code; reviewed, tested, and owned by me.</sub>

## For developers

Keep the public README, package metadata, and examples aligned with current behavior. Before opening a PR or pushing a release, run the local package verification path.

```bash
python -m pip install -e ".[test]"
python -m pytest
```

See [AGENTS.md](AGENTS.md) for the repo-specific operating boundary and
[CHANGELOG.md](CHANGELOG.md) for current delivery status.

---

**[Zentropy Labs](https://github.com/ZentropyLabs-ai)** · order out of entropy. An independent lab building evidence-first tools that leave a re-checkable artifact behind. Built by Zain Dana Harper in Seattle. The full workbench is at [Project Telos](https://harperz9.github.io).
