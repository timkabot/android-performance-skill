# Android Performance Skill

A portable agent skill for investigating Android and Jetpack Compose performance. Works with **Claude Code, Codex, and Cursor** using the same `SKILL.md` and reference files.

It helps an agent diagnose slow startup, jank, ANRs, memory growth, slow data loading, and background drain. It connects a symptom to evidence, a focused fix, and a before/after check while separating hypotheses from measured improvements.

## Quick start

From your Android project's root, install for Claude Code, Codex, and Cursor:

```sh
npx skills add timkabot/android-performance-skill --agent claude-code codex cursor
```

Requires Node.js/npm and Git. The [Skills CLI](https://github.com/vercel-labs/skills) downloads the skill and handles the agent directories. Keep only the agent names you use. Add `--global` to install for your user across projects, or `--copy` if you prefer copies instead of symlinks.

Prefer to choose your agents interactively?

```sh
npx skills add timkabot/android-performance-skill
```

Start a new agent session, then ask: **"Use the android-performance skill to investigate scroll jank in FeedScreen."**

Project installation for all three agents was tested with Skills CLI 1.7.2 on 2026-10-12. The installed skill and all reference files were checked against the source. This checks installation, not runtime behavior inside each agent.

## What it covers

- Main-thread work, coroutines, Flow, cancellation, and scheduling.
- Startup readiness, SDK initialization, first-use latency, and ANR deadlines.
- Compose stability, strong skipping, recomposition, and effects.
- Lazy lists/grids, keys, nesting, pagination, and scroll restoration.
- Coherent state publication, image loading, and bounded prefetch.
- Memory/resource leaks, native/graphics footprint, allocation churn, and process loss.
- SQL/query plans, transactions, HTTP caching, duplicate requests, and retries.
- Background jobs, wake locks, network radio use, and thermal behavior.
- R8, Baseline/Startup Profiles, release artifacts, and SDK footprint.
- Views/Compose interoperability, GPU work, WebView, and animation cost.
- Perfetto, measurement guardrails, field cohorts, and benchmark validation.

The entry point routes to **14 focused reference guides**. Each new diagnostic guide connects concrete patterns to confirmation steps, constrained fixes, and validation. The agent reads only the relevant topics.

## Evidence and evaluation

The [2026-10-12 research review](.docs_generated/research/2026-10-12-android-performance-landscape.md) connects the guidance to Google documentation, production cases from Google and Meta, Google I/O / Android Dev Summit resources, and empirical research. It records source limitations, including videos whose full recordings were not reviewed. Case-study improvements are not promises for your app.

There are [18 behavioral evaluation scenarios](.docs_generated/evals/performance-scenarios.md) for checking diagnosis, fix semantics, uncertainty, and validation. They are a manual review/evaluation specification; no independent before/after model evaluation has been completed.

No company-specific architecture, custom ViewModel, MCP server, API key, or runtime dependency is required. This is an instruction-only skill; it uses the tools and build setup already available in your Android project.

## Manual installation

<details>
<summary>Install with Git or copy files without Node.js</summary>

Requires Git and an agent version supporting local Agent Skills. Commands below work in macOS/Linux shells and Git Bash. Run only the installation for the tool you use; **Codex and Cursor share the same installation** below.

### Codex and Cursor

Install for your user:

```sh
mkdir -p "$HOME/.agents/skills"
git clone https://github.com/timkabot/android-performance-skill.git "$HOME/.agents/skills/android-performance"
```

### Claude Code

Install for your user:

```sh
mkdir -p "$HOME/.claude/skills"
git clone https://github.com/timkabot/android-performance-skill.git "$HOME/.claude/skills/android-performance"
```

If you already installed the shared Codex/Cursor copy, you can link it instead on macOS/Linux:

```sh
mkdir -p "$HOME/.claude/skills"
test ! -e "$HOME/.claude/skills/android-performance" && test ! -L "$HOME/.claude/skills/android-performance" &&
  ln -s "$HOME/.agents/skills/android-performance" "$HOME/.claude/skills/android-performance"
```

Choose either clone or link for Claude Code. These commands intentionally do not overwrite an existing installation.

### Project-only installation

Copy `SKILL.md`, `references/`, and `agents/` from this repository into one of these folders in the target Android project:

| Tool | Folder |
| --- | --- |
| Codex and Cursor | `.agents/skills/android-performance/` |
| Claude Code | `.claude/skills/android-performance/` |

Copy the files, not the repository's `.git` directory. Commit the copied skill if teammates or cloud agents should use it. For both locations, a relative symlink can keep one source on systems that support symlinks. Follow any skill-directory conventions already enforced by the target project.

Local user installations do not automatically reach remote/cloud environments. Install or commit the skill where the agent actually runs.

</details>

## Use

Start a new agent session after installation. Explicit examples:

**Codex**

```text
Use $android-performance to investigate scroll jank in FeedScreen.
Inspect the full state-to-render path and propose a focused fix.
```

**Claude Code / Cursor**

```text
/android-performance Investigate scroll jank in FeedScreen.
Inspect the full state-to-render path and propose a focused fix.
```

You can also ask naturally: "Use the android-performance skill to review this Compose screen for expensive recompositions." Automatic selection depends on the agent and its settings.

Useful requests:

- "Review this diff for performance risks; do not edit files."
- "Our refresh is fast on the network but slow to render. Find the critical path."
- "Investigate duplicate lazy keys without losing item state."
- "Interpret this Perfetto trace and suggest a before/after benchmark."
- "This receiver ANRs despite goAsync. Find the deadline and blocked work."
- "Memory grows after repeated navigation. Separate leaks from cache growth."
- "Review our release configuration and Baseline Profile coverage."
- "Find why background sync drains battery without sacrificing freshness."

The skill can inspect code without a device. Runtime improvement claims still need runtime measurements; the agent should state when those are unavailable.

## Update or remove

For a project installed with the Skills CLI:

```sh
npx skills update android-performance --project
npx skills remove android-performance
```

For a global installation, use `npx skills update android-performance --global` or `npx skills remove android-performance --global`. These are separate actions: run update to refresh the skill, or remove to uninstall it.

For a Git-cloned installation:

```sh
git -C "$HOME/.agents/skills/android-performance" pull --ff-only
```

Use the `.claude/skills` path if you cloned there instead. A linked Claude installation updates with the shared copy. Project copies must be updated explicitly. To remove the skill, remove only its installation folder or symlink, preserving any local changes you want to keep.

## Structure

```text
SKILL.md          Shared entry point and reference routing
references/       Focused performance investigation guides
agents/openai.yaml  Optional Codex display metadata
.docs_generated/  Research rationale and evaluation scenarios
```

The skill is framework-neutral within Android/Kotlin. Its code examples are illustrative and should be adapted to the target app. References link to public Android, Kotlin, and Perfetto documentation.

Installation follows the documented local skill directories: [Codex](https://learn.chatgpt.com/docs/build-skills), [Claude Code](https://code.claude.com/docs/en/skills), [Cursor](https://cursor.com/docs/skills). Installation paths checked on 2026-10-10. Client discovery has not been end-to-end tested in all three applications.

## Contributing

Prefer small, evidence-backed improvements. Keep the entry point concise, add specialized guidance to a relevant reference, and avoid assuming a particular app architecture. Include the scenario and evidence that justify a new rule. Do not add private code, internal endpoints, production traces, or credentials.

## License

[MIT](LICENSE).
