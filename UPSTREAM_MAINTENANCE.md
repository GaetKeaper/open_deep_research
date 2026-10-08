# Upstream archive and final dependency reconciliation

Reviewed 2026-10-08 (Australia/Melbourne; 2026-10-07 UTC).

## Maintenance ownership

`langchain-ai/open_deep_research` was archived on **2026-08-21** and is
read-only. This was confirmed from the GitHub repository banner and the
repository API (`archived: true`). Do not assume further upstream merges or
security releases. Dependency monitoring, compatibility checks, and security
maintenance now belong to this fork. Reassess this only if upstream explicitly
resumes maintenance; unrelated open upstream PRs are not reviewed releases.

Source: https://github.com/langchain-ai/open_deep_research

## Exact source and scope

Fork starting point: `9fe9713feeb39029e0955d410bcebcb8b0285944` (220 commits).
Archived upstream tip: `1b7d2e80db9faa586165c60e09096dbbfd483a64` (224 commits).
The fork main branch had zero unique commits and was four commits behind.
The reconciliation retains these original commits, in order, on a maintenance
branch. No application source, project dependency declarations, or unrelated PRs
were changed or imported.

| Upstream commit | Change |
| --- | --- |
| `d337ae32ed4ff8f4c6fbe192ba3bf1b2d6610799` | pyasn1 0.6.3 → 0.6.4 |
| `7882410280a77085862e51d1db4b4a3a3818a738` | aiohttp 3.14.1 → 3.14.3 |
| `20aaa0d422bd290c83f93574810ef1244e8d5955` | cryptography 48.0.1 → 50.0.0 |
| `1b7d2e80db9faa586165c60e09096dbbfd483a64` | h2 4.3.0 → 4.4.1; required hpack 4.1.0 → 4.2.0 |

The lockfile is byte-for-byte identical to the archived upstream tip. Its exact
diff is 283 insertions and 286 deletions. Most lines are distribution URLs,
hashes, sizes, and upload timestamps. The aiohttp commit also rewrites dependency
markers in existing package records, including Python-specific package variants;
these upstream changes were retained, not silently described as version-only
changes. The top-level Python range and resolution partitions are unchanged.
Exactly five package names change versions; hpack is required by h2, not a
discretionary upgrade. No packages are added or removed.

## Compatibility review

- Published Python requirements: pyasn1 >=3.8; aiohttp >=3.10; cryptography
  >=3.9 excluding 3.9.0/3.9.1; h2 >=3.10. These permit the project's >=3.10 floor.
- h2 4.4.1 requires `hpack>=4.2,<5` and `hyperframe>=6.1,<7`; retaining hpack
  4.1.0 would be incompatible. Source:
  https://github.com/python-hyper/h2/blob/v4.4.1/pyproject.toml
- Published cryptography constraints from locked consumers allow 50.0.0:
  msal 1.37.0 requires >=2.5,<51; azure-identity 1.23.0 requires >=2.5;
  google-auth 2.48.0 requires >=38.0.3; langgraph-api 0.10.0 requires >=42.0.0.
  Version-specific metadata was read from `https://pypi.org/pypi/NAME/VERSION/json`.
- cryptography 49.0.0 removed Intel macOS and 32-bit Windows support and several
  deprecated key-type aliases; it changed ChaCha20 counter behavior and rejects
  invalid certificate signature parameters. Version 50.0.0 further tightens
  DER/certificate parsing and changes PKCS7 decryption failure behavior. No direct
  use of the removed APIs was found in this project's Python source. This is not
  a guarantee for every third-party integration or externally supplied certificate.
  Source: https://github.com/pyca/cryptography/blob/50.0.0/CHANGELOG.rst
- The retained upstream marker rewrites were validated by the locked resolver and
  installed dependency checker on Linux x86_64, CPython 3.12.14, uv 0.12.19.
  Other Python/platform combinations were not executed. Intel macOS and 32-bit
  Windows deployments require a separate platform decision before rollout.

## Validation performed

| Check | Result |
| --- | --- |
| `uv sync --locked --extra dev` | PASS; resolved 243 packages, installed the locked environment |
| `uv lock --check --offline` | PASS; no lockfile rewrite |
| `uv pip check` | PASS; checked 228 installed packages; no conflicts |
| `python -m compileall -q src tests` | PASS |
| Main `deep_researcher_builder.compile()` | PASS; CompiledStateGraph |
| Imports of azure.identity, google.auth, msal, langgraph_api | PASS |
| aiohttp local HTTP server/client JSON round trip | PASS |
| cryptography RSA sign/verify and private-key PEM reload | PASS |
| pyasn1 DER integer encode/decode round trip | PASS |
| h2/hpack in-memory client/server handshake and request headers | PASS |
| `.venv/bin/python -m pytest -q -m 'not langsmith'` | Exit 5: 1 live-service test deselected; no offline tests selected; 24 warnings |
| `git diff --check` | PASS |

The pytest result is **not** a passing test suite. Existing tests consist of a
LangSmith-marked live report-quality test and evaluation scripts requiring model,
search, and LangSmith services. Those live evaluations were not run. Existing
LangGraph/Pydantic deprecation warnings remain outside this dependency-only scope.
The current GitHub workflows provide Claude automation, not an install/test gate.

## Review and future updates

Reproduce the exact dependency diff with:

```sh
git diff 9fe9713feeb39029e0955d410bcebcb8b0285944 1b7d2e80db9faa586165c60e09096dbbfd483a64 -- uv.lock
```

Future upgrades should be separately scoped and validated against this fork's
deployment platforms and live integrations. Use locked installs for this change;
do not run a broad `uv lock --upgrade`. This reconciliation is not a claim that
the entire archived dependency set is current or vulnerability-free.
