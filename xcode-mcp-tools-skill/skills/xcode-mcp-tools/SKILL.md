---
name: xcode-mcp-tools
description: Guides which mcp__xcode__* tool to call, what parameters it takes, and how to chain tools. Use whenever mcp__xcode__* tools are available in the session: building, testing, debugging, renaming files, investigating crashes, adding entitlements, translating strings, or driving a simulator. Also use when unsure whether to prefer an Xcode MCP tool over a built-in tool or shell command.
when_to_use: Invoke when the user asks to build, run, test, debug, rename or move files, investigate a production crash, update build settings, add a privacy key, translate strings, or interact with a simulator; and mcp__xcode__* tools are available. Also invoke when deciding between an Xcode MCP tool and xcodebuild, mv, rm, mkdir, or a built-in Read/Edit/Grep tool.
---

# Xcode MCP Tools

Call `XcodeListWindows` first in every session; it returns the `tabIdentifier` required by every other tool. Prefer `mcp__xcode__*` tools over shell equivalents when the MCP server is active.

## Version Labels

| Label | Meaning |
|-------|---------|
| *(none)* | Xcode 26.3+ |
| `[27+]` | Xcode 27+ only |
| `[cond]` | Both versions; requires prerequisite to register |

## Build & Run

| Tool | When to use | Gotcha |
|------|-------------|--------|
| `BuildProject` | After every code change | No CLI equivalent matches Xcode's exact build behavior |
| `GetBuildLog` | After a failed `BuildProject` | Most recent build only; truncates at 100 entries |
| `RunProject` `[27+]` | Launch app with optional debugger | Does not stream output; follow with `GetConsoleOutput` |
| `StopProject` `[27+]` | End a `RunProject` session | Always call; open sessions degrade Xcode performance |

## Testing

| Tool | When to use | Gotcha |
|------|-------------|--------|
| `GetTestList` | Before `RunSomeTests` | Returns exact `targetName` + `testIdentifier` required as input |
| `RunAllTests` | Pre-commit | Honors test plan enabled/disabled state; `xcodebuild test` does not |
| `RunSomeTests` | TDD loop | Requires identifiers from `GetTestList`; no guessing |

## Code Execution & Preview

- **`RenderPreview`**: Snapshots a `#Preview` or `PreviewProvider`. Only programmatic way to verify SwiftUI output without a simulator. 120s timeout.
- **`ExecuteSnippet`** (26.3 only) / **`RunCodeSnippet`** `[27+]`: Scoped REPL in a file's context. `RunCodeSnippet` requires a `purpose` param that must NOT contain "test". Prefer over throwaway unit tests when verifying API behavior.

## Documentation

- **`DocumentationSearch`** `[cond]`: Semantic local search over Apple docs; no network needed. Activate via `Window > Developer Documentation` (index download required). Use before any unfamiliar Apple API; do not assume overloads exist.

## Debugging `[27+]`

| Tool | When to use | Gotcha |
|------|-------------|--------|
| `InvokeDebuggerCommand` | Send any lldb command | Requires `RunProject` with `attachDebugger: true` first; increase timeout when issuing `continue` |
| `GetConsoleOutput` | Read stdout/stderr/OSLog | Tail-limited to 500 lines; use `outputType: "oslog"` for structured Apple logs |

## Analytics & Crash Diagnostics `[27+]`

| Tool | When to use | Gotcha |
|------|-------------|--------|
| `GetTopCrashIssues` | Investigate production crashes | 14-day window only; `bundle_id` is case-sensitive |
| `GetCrashIssueLogs` | Drill into a crash signature | Requires exact `signature_name` from `GetTopCrashIssues` |
| `GetTopFieldPerformanceIssues` | Query field performance data | `diagnostic_type` always required; omitting `app_version` returns a version list, not data |
| `GetFieldPerformanceIssueLogs` | Detailed performance diagnostics | Requires both `app_version` and `signature_name`; platform support varies by diagnostic type |

## Project Configuration `[27+]`

Never edit `project.pbxproj`, `.entitlements`, or `Info.plist` directly. Use these instead:

| Tool | Purpose | Gotcha |
|------|---------|--------|
| `GetTargetBuildSettings` | Read resolved build settings | Call before `UpdateTargetBuildSetting` to see current state |
| `UpdateTargetBuildSetting` | Write a build setting | Omitting `buildSettingValue` deletes the setting entirely |
| `AddEntitlement` | Add a code-signing entitlement | Code-signing capabilities only; use `AddInfoPlist` for everything else |
| `AddInfoPlist` | Add/update any `Info.plist` key | Works even when `GENERATE_INFOPLIST_FILE` is enabled |
| `GetFileCompilerFlags` / `UpdateFileCompilerFlags` | Per-file compiler flags | Swift often ignores per-file flags; prefer `UpdateTargetBuildSetting` for most cases |

## Localization `[27+]`

Call in strict order:

```
LocalizationPlanner -> StringCatalogRead -> StringCatalogContext -> StringCatalogEdit
```

- **`LocalizationPlanner`**: Prerequisite; ensures project is translatable for target locale. Call once per locale task.
- **`StringCatalogRead`**: Lists keys by state (`new`, `translated`, `needs_review`). Supports pagination.
- **`StringCatalogContext`**: Returns source text for a key. Mandatory before `StringCatalogEdit`.
- **`StringCatalogEdit`**: The only correct way to write to `.xcstrings`. Direct file edits corrupt the catalog.

## File Operations

MCP file tools update both the filesystem and the project graph atomically. Always prefer them over shell equivalents for project files:

| MCP Tool | Instead of | Why |
|----------|-----------|-----|
| `XcodeWrite` | `Write` (new files) | Registers file in project navigator and target membership |
| `XcodeMV` | `mv` / `git mv` | Updates all Xcode references and target memberships |
| `XcodeRM` | `rm` | Removes both the project reference and the file; avoids broken red references |
| `XcodeMakeDir` | `mkdir` | Creates directory and corresponding navigator group |
| `XcodeGetCurrentFile` `[27+]` | n/a | Returns path + content of the file active in the Xcode editor right now |

Use built-in tools for these; the MCP wrappers add unnecessary XPC overhead:

`XcodeRead` → `Read` | `XcodeUpdate` → `Edit` | `XcodeLS` → `ls`/`Glob` | `XcodeGlob` → `Glob` | `XcodeGrep` → `Grep`

## Navigation & Schemes

- **`XcodeListWindows`**: **Call first in every session.** Returns `tabIdentifier` values required by all other tools.
- **`XcodeListNavigatorIssues`**: Broader than `GetBuildLog`; includes package resolution failures, config warnings, signing issues.
- **`XcodeRefreshCodeIssuesInFile`**: SourceKit diagnostics for one file in seconds. First pass after edits; follow with `BuildProject` for full verification.
- **`XcodeListSchemes`** `[27+]`: Lists all schemes with sharing status.
- **`XcodeSwitchScheme`** `[27+]`: Changes active scheme; destination may silently change, verify via `activeDestinationDisplayTitle`.
- **`XcodeListRunDestinations`** `[27+]`: Lists run destinations; pass `includeIncompatible: true` to surface hidden ones.
- **`XcodeSwitchRunDestination`** `[27+]`: Accepts `displayTitle` from `XcodeListRunDestinations` directly; values chain cleanly.

## Device Interaction `[27+]`

Strict session lifecycle:

```
DeviceInteractionStartSession -> DeviceInteractionInstallAndRun -> DeviceInteractionSynthesize -> DeviceInteractionEndSession
```

- **`DeviceInteractionStartSession`**: Boots simulator or prepares device. Start early to parallelize boot time with other work.
- **`DeviceInteractionEndSession`**: Always call; open sessions waste resources and degrade Xcode UI performance.
- **`DeviceInteractionInstallAndRun`**: Re-call after any project change, destination change, or debug disconnect.
- **`DeviceInteractionSynthesize`**: Tap/swipe/type/rotate. Use coordinates from the UI hierarchy dump, never inferred from a screenshot.

## Common Workflows

```
Fix build:        BuildProject -> GetBuildLog (severity: error) -> fix -> BuildProject
TDD:              GetTestList -> RunSomeTests -> fix -> RunSomeTests
Rename file:      XcodeMV -> XcodeGrep (find references) -> XcodeUpdate (fix imports) -> BuildProject
Debug session:    RunProject (attachDebugger: true) -> InvokeDebuggerCommand -> GetConsoleOutput -> StopProject
Production crash: GetTopCrashIssues -> GetCrashIssueLogs -> fix -> BuildProject
Translate:        LocalizationPlanner -> StringCatalogRead (state: new) -> StringCatalogContext -> StringCatalogEdit
UI smoke test:    DeviceInteractionStartSession -> DeviceInteractionInstallAndRun -> DeviceInteractionSynthesize -> DeviceInteractionEndSession
```

## Fallback

If any MCP tool fails, fall back to the equivalent shell command and report the MCP failure.
