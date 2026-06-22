# xcode-mcp-tools-skill

A skill for AI agents that work with Xcode. Covers all 47 `mcp__xcode__*` tools exposed by Xcode's built-in MCP server: when to call each one, what parameters it takes, which version of Xcode it requires, and how to chain tools across common workflows.

## What It Covers

All tools across both Xcode 26.3 and Xcode 27+, organized by category:

| Category | Tools |
|----------|-------|
| **Build & Run** | `BuildProject`, `GetBuildLog`, `RunProject`, `StopProject` |
| **Testing** | `GetTestList`, `RunAllTests`, `RunSomeTests` |
| **Code Execution** | `RenderPreview`, `ExecuteSnippet` / `RunCodeSnippet` |
| **Documentation** | `DocumentationSearch` |
| **Debugging** | `InvokeDebuggerCommand`, `GetConsoleOutput` |
| **Analytics & Crash** | `GetTopCrashIssues`, `GetCrashIssueLogs`, `GetTopFieldPerformanceIssues`, `GetFieldPerformanceIssueLogs` |
| **Project Config** | `GetTargetBuildSettings`, `UpdateTargetBuildSetting`, `AddEntitlement`, `AddInfoPlist`, `GetFileCompilerFlags`, `UpdateFileCompilerFlags` |
| **Localization** | `LocalizationPlanner`, `StringCatalogRead`, `StringCatalogContext`, `StringCatalogEdit` |
| **File Operations** | `XcodeRead/Write/Update/LS/Glob/Grep/MV/MakeDir/RM`, `XcodeGetCurrentFile` |
| **Navigation & Schemes** | `XcodeListWindows`, `XcodeListNavigatorIssues`, `XcodeRefreshCodeIssuesInFile`, `XcodeListSchemes`, `XcodeSwitchScheme`, `XcodeListRunDestinations`, `XcodeSwitchRunDestination` |
| **Device Interaction** | `DeviceInteractionStartSession`, `DeviceInteractionEndSession`, `DeviceInteractionInstallAndRun`, `DeviceInteractionSynthesize` |

For each tool the skill provides: when to use it, key parameters, version requirement (26.3+ or 27+), gotchas, and preferred alternatives where a built-in tool is faster.

## Usage

Once installed, the skill triggers automatically. When your AI agent is asked to build, test, debug, rename files, investigate crashes, or interact with a simulator while the Xcode MCP server is active, it will consult the skill to pick the right tool, pass the correct parameters, and chain tools in the right order.

No manual invocation needed.

## Installation

### Any agent (universal)

Works with Claude Code, Codex, Cursor, Gemini CLI, Kimi Code, OpenCode, Pi, and [60+ other agents](https://github.com/vercel-labs/skills):

```bash
npx skills add vincenzpascarella/xcode-mcp-tools-skill
```

### Claude Code

#### Personal usage

```
/plugin marketplace add vincenzpascarella/xcode-mcp-tools-skill
/plugin install xcode-mcp-tools@xcode-mcp-tools-skill
/reload-plugins
```

#### From a local clone

```
/plugin marketplace add /path/to/xcode-mcp-tools-skill
/plugin install xcode-mcp-tools@xcode-mcp-tools-skill
/reload-plugins
```

### OpenCode

Add to your `opencode.json`:

```json
{
  "plugin": ["xcode-mcp-tools@git+https://github.com/vincenzpascarella/xcode-mcp-tools-skill.git"]
}
```

### Other agents

Follow your tool's official documentation on installing Agent Skills. The skill content lives in `xcode-mcp-tools/SKILL.md`.

## Requirements

- Xcode 26.3 or later with the MCP server enabled
- Xcode 27 beta or later for the `[27+]` tools

## Plugin Structure

```
xcode-mcp-tools-skill/                      Repo root
├── .claude-plugin/
│   ├── marketplace.json                    Marketplace entry
│   └── plugin.json                         Plugin manifest
├── .codex-plugin/
│   └── plugin.json                         Codex plugin manifest
├── .cursor-plugin/
│   └── plugin.json                         Cursor plugin manifest
├── .kimi-plugin/
│   └── plugin.json                         Kimi plugin manifest
├── .opencode/
│   └── plugins/
│       └── xcode-mcp-tools.js              OpenCode plugin entry point
├── xcode-mcp-tools/
│   └── SKILL.md                            All 47 tools, distilled reference
├── package.json
├── README.md
└── LICENSE
```

## License

MIT. See [LICENSE](LICENSE).
