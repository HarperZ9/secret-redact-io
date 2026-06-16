# io-guardrails

`io-guardrails` is a small Python SDK and CLI for guarded IO. It wraps file
reads, file writes, HTTP fetches, and subprocess execution with redaction and
hash-only audit receipts.

The default posture is conservative:

- user-facing output is redacted before it is returned;
- guarded writes persist redacted content by default;
- receipts store byte counts, hashes, status metadata, and redaction counts;
- receipts do not archive raw input or raw secret values.

## Install

```bash
python -m pip install io-guardrails
```

## CLI

```bash
io-guard read README.md --json
io-guard write out.txt --content "note=hello" --dry-run --json
io-guard fetch https://example.com --json
io-guard exec --json -- python -c "print('hello')"
```

## Python API

```python
from io_guardrails import read_text_guarded, run_guarded

read_result = read_text_guarded("README.md")
print(read_result.text)
print(read_result.receipt.to_json())

exec_result = run_guarded(["python", "-c", "print('hello')"])
print(exec_result.stdout)
```

## Boundary

This package is a public clean-room guardrail utility. It does not include
private policies, private corpora, credentials, operational runbooks, or
environment-specific routing rules.
