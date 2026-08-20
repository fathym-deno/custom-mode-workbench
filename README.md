# custom-mode-workbench

Track 6 Phase 11 sample workbench demonstrating a **custom mode type** authored with `DefineModeHolon`.

Sibling to [`api-workbench`](https://github.com/fathym-deno/api-workbench) (Phase 9 API-mode sample), [`ui-workbench`](https://github.com/fathym-deno/ui-workbench) (Phase 10 WebMode + Consumes sample), and [`hello-workbench`](https://github.com/fathym-deno/hello-workbench) (v1 MCP-mode reference).

## What it demonstrates

Phase 11 opens the door for community-defined mode types beyond OpenX's built-in MCP / API / Web. This sample walks the full path:

1. **`DefineModeHolon` primitive** (Phase 11 D.11.3) — [`workbenches/cron/mode-holon.ts`](./workbenches/cron/mode-holon.ts) defines a `Cron` mode. One `DefineModeHolon` call yields two outputs: a `ModeBuilder` factory (runtime behavior) and a `ModeHolonBinding` (workspace registration data).

2. **`eac.ModeHolons` registration** (Phase 11 D.11.1 + D.11.2) — a workspace admin writes `CronModeHolon.Binding` to `eac.ModeHolons['Cron']` once. That entry unlocks the mode Kind for every workbench in that workspace.

3. **SOP validation with clear failure** (Phase 11 D.11.4) — attempting to deploy this workbench BEFORE the registration step fails with `HostingStatus: Failed` and a message pointing to `eac.ModeHolons['Cron']`. Built-in MCP/API/Web modes bypass this check.

## The Cron mode itself

Kept **intentionally minimal** — this is a primitive demonstration, not a production Cron mode. The mode's `.Execute()`:

- Reads an `EveryMs` interval from mode-config.
- Starts a `setInterval` that logs a heartbeat every N ms. Visible in the Container App's stdout.
- Serves `GET /health` on `HealthPort` so the SOP readiness probe passes.
- Blocks on the HTTP server until the container is stopped.

A real Cron mode would parse cron expressions (or use `Deno.cron`), support multiple jobs, propagate handler errors, etc. Out of scope for a primitive demonstration.

## Requires `@fathym/fai` post-Phase-11 release

**⚠️ This sample doesn't run today.** `DefineModeHolon` ships on the `feature/track-6-phases-9-10-11` branch of [`fathym-deno/power-ai`](https://github.com/fathym-deno/power-ai); the `EaCModeHolon` type ships on the same-named branch of [`open-industrial-reference-architecture`](https://github.com/o-industrial/open-industrial-reference-architecture). JSR publish is pending as part of the Phase 9+10+11 cascade release. The `deno.jsonc` pins `@fathym/fai@0.0.406` as a syntactically-valid placeholder — the import in [`workbenches/cron/mode-holon.ts`](./workbenches/cron/mode-holon.ts) will fail until the pin is bumped to the post-cascade version.

## Deploy via OpenX (post cascade release)

**One-time workspace admin step: register the mode holon.**

```typescript
import { CronModeHolon } from 'https://raw.githubusercontent.com/fathym-deno/custom-mode-workbench/main/workbenches/cron/mode-holon.ts';

eac.ModeHolons ??= {};
eac.ModeHolons['Cron'] = CronModeHolon.Binding;
// commit the EaC update
```

**Then deploy the workbench:**

1. Drop a **SurfaceWorkbench** onto a surface. In the inspector:
   - **Source** tab: Repo `https://github.com/fathym-deno/custom-mode-workbench`, Ref `main`, Entry `workbenches/cron/local.ts`
   - **Hosting** tab: APISlug `cron-sample`
   - **Modes** tab (after first deploy): enable `Cron`
2. Deploy. Once `HostingStatus` is `Running`, tail the Container App logs:
   ```
   [cron] starting, everyMs=60000, healthPort=4970
   [cron] tick @ 2026-08-21T00:15:00.000Z
   [cron] tick @ 2026-08-21T00:16:00.000Z
   ...
   ```

**Deploying WITHOUT the registration step** — the SOP short-circuits with:

```
Mode 'Cron' is not registered in this workspace. Built-in modes are MCP, API, Web.
Register a custom mode holon under eac.ModeHolons['Cron'] (Phase 11 D.11.1 primitive;
DefineModeHolon helper in '@fathym/fai/workbenches' emits the binding data) to enable
a workbench to declare it.
```

That's Phase 11 D.11.4 in action.

## Local run (post pin bump)

```
deno task cron
```

Runs the Cron mode against a local `Deno.serve` on port 4970. `Ctrl+C` to stop.

## Related

- **Track 6 v2 execution tracker**: [`o-industrial/oi-core-pack#61`](https://github.com/o-industrial/oi-core-pack/issues/61)
- **Phase 11 spec** (on `fathym-dev-space`): [`.workbench/.workstreams/2026-04-06-NewNodeCapabilities/track-6-workbench-node/phase-11-custom-mode-types.md`](https://github.com/fathym-deno/fathym-dev-space/blob/feature/track-6-phases-9-10-11/.workbench/.workstreams/2026-04-06-NewNodeCapabilities/track-6-workbench-node/phase-11-custom-mode-types.md)
- **Phase 9 API-mode sample**: [`fathym-deno/api-workbench`](https://github.com/fathym-deno/api-workbench)
- **Phase 10 WebMode sample**: [`fathym-deno/ui-workbench`](https://github.com/fathym-deno/ui-workbench)
- **v1 MCP-mode reference sample**: [`fathym-deno/hello-workbench`](https://github.com/fathym-deno/hello-workbench)

## License

MIT
