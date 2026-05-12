# Release Notes · Calyntro v0.9.3
**May 12, 2026**

---

## Author Alias Resolution

Contributors who commit under multiple identities — different e-mail addresses, name variants, or machine accounts — are now recognised as a single person across all analyses.

Aliases are configured in `config.yaml` under `users`:

```yaml
users:
  - name: "Alice M."
    aliases:
      - "alice@company.com"
      - "alice.m@personal.io"
      - "alice@ci-bot.internal"
```

Once configured, every view that shows contributor data — Knowledge Silos, Ownership, Contributors, Trends — reflects the resolved identity. Historical data is recomputed on the next import; no schema changes are needed.

Without alias configuration, behaviour is unchanged: raw author names and e-mail addresses are used as before.

---

## Dashboard Performance

Initial load times for the main dashboard have been significantly reduced.

The Knowledge Risk, Complexity, and Warnings panels previously required dozens of sequential database queries per page load. These have been replaced with batch queries that fetch all time buckets in a single round-trip each. Observed improvement on a mid-sized repository: **3,800 ms → 830 ms** for the trends overview alone.

In addition, the three slowest dashboard endpoints are now pre-computed during server startup and held in a short-lived in-memory cache (default: 5 minutes, configurable via `CALYNTRO_CACHE_TTL`). The first user to open the dashboard after a server start gets pre-cached results rather than waiting for a cold query.

Cache behaviour:
- TTL resets on each write — results stay fresh relative to the last computation
- Set `CALYNTRO_CACHE_TTL=0` to disable caching entirely
- Cache is invalidated automatically on server restart

---

## Deterministic Rankings

Several analysis views — Contributors, Knowledge Silos, Team Alignment, Commits — previously produced non-deterministic ordering when multiple entries shared the same primary score. A secondary sort criterion has been added to all affected queries. Results are now stable across repeated requests and re-imports.

---

## Version Display

The current backend version is now shown at the bottom of the sidebar. This makes it straightforward to confirm which version is running without consulting logs or the `/api/info` endpoint.

---

## Under the Hood

- Component assignments in the analysis database are re-resolved at the end of each import run. If you change component prefixes in `config.yaml` and re-run the import, all file–component mappings are updated consistently.
- A `/api/health` endpoint is available for uptime monitoring and container health checks.
- Ranking SQL queries across all modules now include deterministic tie-breaking on secondary columns.
