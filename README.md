# secret-redact-io

`secret-redact-io` is a small Python SDK and CLI for guarded IO. It wraps file
reads, file writes, HTTP fetches, and subprocess execution with redaction and
hash-only audit receipts.

The default posture is conservative:

- user-facing output is redacted before it is returned;
- guarded writes persist redacted content by default;
- receipts store byte counts, hashes, status metadata, and redaction counts;
- receipts do not archive raw input or raw secret values.

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

## Boundary

This package is a public, self-contained guardrail utility. It does not include
credentials, secrets, or any deployment-specific configuration.
