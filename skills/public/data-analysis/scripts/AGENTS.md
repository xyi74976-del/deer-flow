# Data-analysis cache ownership

`open_analysis_cache` owns every DuckDB connection through success, action
failure, loader failure and interruption. Each miss writes a private generation;
close its writer and reopen read-only before publishing a single atomic manifest
containing generation identity and table map. Never overwrite a published DB.

Validate manifest version, owned generation name, table-map types and referenced
tables before accepting a hit. Invalid/missing caches rebuild into another
generation. Close a reader even if validation is interrupted. Close writers
before removing unfinished private directories. Retain completed generations
once publication might have occurred: another process may have read their ID.
Cache publication is not a power-loss durability or automatic GC guarantee.

Root `tests/skills/test_data_analysis_cache_lifecycle.py` uses real DuckDB,
spawned processes and real CSV loading, bypassing only the unrelated spatial
extension initializer. Keep cache-hit and cold-build action connections equally
read-only; temporary views remain allowed. Cache identity and SQL/source naming
are separate concerns, not changed by this lifecycle repair.

Missing Python dependencies fail with an actionable setup instruction. Do not
launch pip implicitly during import or analysis; setup precedes concurrent calls.
