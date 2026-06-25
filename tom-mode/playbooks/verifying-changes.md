# Verifying changes

For proving a change works end to end, beyond what unit tests show. This is the most important step in the loop and the one most often skipped, because it is still manual prompting. Use it whenever a change has runtime behaviour that tests in isolation do not exercise: a new data path, a pipeline stage, an API a UI consumes, anything that crosses a service or storage boundary.

The bar is `principle-prove-it-works`. A change is verified when real data has moved through a running deployment and you have read what every layer actually did, not when the code compiles and the unit suite is green. Green tests with monochrome seed data are a proxy, not proof. For any runtime question, read the live signals before reading the source.

The moves are generic; the named tools below are one example toolchain. Substitute your repo's equivalents.

1. **Name what "exercises the change" means, then seed it.** Write down the dimension the feature exists to handle (multiple currencies, enough rows to paginate, the case where two code paths must diverge) and seed that, not the happy-path single row. If the seed data cannot tell the new path from the old, the deploy proves nothing.

2. **Deploy to a real running stack.** Deploy to a real preview or ephemeral environment rather than a local stub (e.g. a PR label that spins one up, or a hot-reload sync of local changes into it), then confirm it is actually healthy before driving it. A preview env that is one action away beats verifying against a mock.

3. **Drive real data through the surface.** Trigger the actual flow, do not just poke the endpoint. For example, submit the real request or upload the real input so the pipeline runs the stages the change touches. The point is to make the system do the work a user would cause, on data shaped like step 1.

4. **Inspect the data store.** Confirm the rows landed with the right shape, not just that the call returned 200. Query the database directly (a DB client or an MCP) and read the rows back. A write that "succeeded" but stored the wrong value, a null where a value belongs, or a duplicate is a defect the API response hides.

5. **Inspect the runtime signals.** Read the story from observability, not from the code. Query your observability platform for logs, traces, and metrics, via its CLI or skills. Confirm the logs narrate what happened and carry the identifying context, the trace crosses the boundary you changed, and any metric you expected to move actually moved. Discover the exact service name first; do not guess it.

6. **Inspect the UI when the change is user-facing, and drive its edge states.** Confirm what the user sees, not just what the API returns, and exercise the states that fail silently rather than throw: an empty dataset, a row with a null id, a clock in a non-UTC timezone, and every clickable path (does the link open the right thing, or a 404?). drive the rendered page with a browser-automation tool (open, snapshot, assert on what is shown). Reuse the repo's own frontend-testing skill, since frontends differ across repos. These frontend edges (empty-state, null, timezone, dead click) are the usual silent-failure sources and rarely show up in a happy-path screenshot.

7. **Confirm or iterate, and own the integration test.** If the result is wrong, fix and re-drive from step 3. A parity or reconciliation gate proves sameness, not correctness; if the baseline is itself unverified, prove the new path is right, not just equal to the old. Do not hand the user a change that only compiles and leave them to find the runtime break.

**Done means:** real, representative data driven through a running deployment, with the outcome confirmed across the data store, the runtime signals, and the UI where the change is user-facing. Not "tests pass", not "it deployed".
