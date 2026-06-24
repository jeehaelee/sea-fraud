# SEA Dasher Fraud Report

Personal container repo for the Cursor Cloud Agent automation that runs the SEA dasher
fraud daily report. **This repo intentionally contains no application code.** The work is
performed by a Cloud Agent using external services accessed through MCP servers.

## Cursor Cloud specific instructions

### What "the application" is here

There is no buildable/runnable code in this repo (no `package.json`, `requirements.txt`,
lockfiles, Dockerfile, etc.). The "application" is the daily report workflow itself, which a
Cloud Agent performs by orchestrating external services:

1. **Snowflake** — run the fraud query/queries to pull the SEA dasher data.
2. **Google Sheets** — build/update the report spreadsheet from the query results.
3. **Slack** — schedule/post the report message to the target channel.

Because the logic lives in the agent run (not in committed code), there are **no lint, test,
or build steps** to run, and the update script is effectively a no-op (nothing to install).
Node 22 and Python 3.12 are preinstalled if a future task needs to write helper scripts.

### Required services / access (these gate running the report)

The workflow cannot run unless these MCP servers are authenticated and healthy in the run:

- **Snowflake** MCP — used to run the fraud query. If `serverStatus` is `error`/`needsAuth`,
  it is unusable; the connection must be fixed / re-authed in the Cursor IDE.
- **Slack** MCP — used to post/schedule the report. If `serverStatus` is `needsAuth`, it must
  be authenticated in the Cursor IDE before its tools work.
- **Google Sheets** access — needed to build the report sheet. Provide credentials (e.g. a
  service-account JSON / OAuth token) if no Google MCP/integration is wired up.

Always check live MCP availability with the MCP tool-discovery step at the start of a run; do
not assume a server is usable just because it appears in the catalog. Servers in `error` or
`needsAuth` states have no usable tools.

### Running the report (hello-world)

The end-to-end "hello world" is: run the Snowflake fraud query → write results into the
Google Sheet → post/schedule the Slack message. This requires the three services above to be
authenticated; it cannot be exercised while any of them is in an `error`/`needsAuth` state.
