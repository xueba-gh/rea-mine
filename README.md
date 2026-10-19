<div align="center">

# Build Anything

### A reverse-engineering MCP for binaries, applications, and runtime behavior.

**See a feature you like. Understand how it works, down to the binary level, then build your own.**

[![MCP tool catalog](https://img.shields.io/badge/MCP-tool_catalog-5c4ee5?style=flat-square)](docs/mcp-contracts.md#generated-catalog)
[![MIT license](https://img.shields.io/badge/license-MIT-f4c430?style=flat-square)](LICENSE)

[Quick start](#quick-start) · [How it works](#how-build-anything-works) · [What you can analyze](#what-you-can-analyze) · [FAQ](#faq) · [Documentation](#documentation)

<code>npx build-anything setup</code>

</div>

> [!NOTE]
> **Build Anything is an independent fork of [REA ("Reverse Engineer Anything")](https://github.com/morluto/rea) by morluto, used under the MIT License.**
> It is not the official REA project and is not affiliated with or endorsed by the upstream maintainer.
> See [`NOTICE.md`](NOTICE.md) for attribution and [`LICENSE`](LICENSE) for terms.

---

## Quick start

On Linux, Ghidra, JADX, firmware tool launches and static JavaScript/Electron
analysis require a procps-compatible `ps` on Build Anything's `PATH` for process ownership
checks. Recovery steps are in `docs/installation.md` under
"Linux owned-process prerequisite".

### Set up your agent

With Node.js and npm installed, run:

```bash
npx build-anything setup
```

Choose your agents, review the proposed changes, and approve them. Setup adds
Build Anything's MCP server and matching workflow instructions, with backups of existing
configuration. Restart your agent afterward.

Setup supports Claude Code, Codex, Cursor, Gemini CLI, Grok Build and
[other agents](docs/installation.md#supported-agents). See
[installation and setup](docs/installation.md) for provider configuration and
manual MCP registration.

### Ask your agent

```text
Understand how search works in the Notes app, show me the evidence, and build a
similar feature for my project.
```

Replace Notes with your target app and the feature you want to understand.

### Use the terminal

Inspect an extracted JavaScript/Electron app directory or ASAR:

```bash
npx -y build-anything@latest analyze-javascript-application /absolute/path/to/app --json
```

The result includes modules, imports, Electron boundaries and their evidence.
Replace the path with your target, such as `"D:/apps/example"` on Windows.

To install the `rea` command for regular use:

```bash
npm install --global build-anything
bant --help
```

For native analysis, configure a provider first. See the
[CLI and Evidence guide](docs/cli.md) for native commands, provider selection,
snapshots and scripting.

### Update Build Anything

Build Anything changes quickly, and new releases include frequent bug fixes. Keep your
installation up to date.

For a CLI installed with `npm install --global` or `bun add --global`:

```bash
bant update
```

`bant --update` is an alias for this verified workflow and accepts the same global
output flags. Use `bant update` when combining it with other command controls.

Updater commands retain at most 4 MiB of stdout and stderr combined. Oversized
metadata replies are rejected after the command exits; verbose installs continue
with bounded diagnostics. Human progress output respects terminal backpressure
and waits for submitted writes to finish.

Bun global updates require Bun 1.2.17 or later. With an older Bun release,
upgrade Bun manually before running `bant update`.

To refresh your agent registrations and skill, run the setup command printed
by the update.

If you use `npx`, update your agent setup with:

```bash
npx build-anything@latest setup
```

Review the setup changes and restart your agent. For one-off CLI commands,
use `npx build-anything@latest` followed by the command. With Bun, use
`bunx build-anything@latest` in place of `npx build-anything@latest`.

## How Build Anything works

Your agent calls Build Anything through MCP to inspect the target and trace relevant code.
Build Anything returns findings with their evidence. The agent uses them to ask follow-up
questions, explain the behavior, or write and test an implementation.
CLI commands use the same workflows.

![Build Anything investigation flow: your agent asks about a local target, Build Anything inspects and traces it using analysis tools, and the agent uses the returned code, references and unknowns to explain, implement and test.](website/public/assets/figures/rea-investigation-flow.svg)

[Open the full-size figure](website/public/assets/figures/rea-investigation-flow.svg).

<a id="current-status"></a>

## What you can analyze

Build Anything requires Node.js 22.x (>=22.19), 24.x (>=24.11), or 26+, plus npm.
Additional tools and host support depend on the target:

| Target                 | What Build Anything returns                                                          | Requirements and guide                                                                                                                |
| ---------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------- |
| Native binaries        | Pseudocode, assembly, strings, symbols, calls and references                         | Hopper, Ghidra or IDA; [native analysis](docs/native-investigation.md)                                                                |
| Offline ELF layout     | Sections, segments, original symbols/relocations and static mitigation candidates    | Caller-supplied pwntools on Linux x64; [binary diagnostics](docs/binary-diagnostics.md)                                               |
| Go binaries            | Embedded compiler version, modules, replacements and build settings                  | Built-in static ELF/PE/thin Mach-O inspection; no Go installation required                                                            |
| Rust binaries          | Embedded cargo-auditable crate names, versions, sources and dependency edges         | Built-in static ELF/PE/thin Mach-O inspection; [Rust binaries](docs/rust-binaries.md)                                                 |
| WASM artifact          | Validation, sections, exact import/export WAT forms and decoded WAT                  | Caller-supplied WABT 1.0.42; `inspect_wasm_artifact`                                                                                  |
| EVM bytecode           | Dispatch selectors, byte offsets, inferred arguments and mutability                  | Local raw/hex carrier on Linux x64; [offline EVM guide](docs/evm-bytecode.md)                                                         |
| SQLite databases       | Tables, columns, indexes, views, triggers and selected ordinary-table rows           | Local database/WAL snapshot; Node.js 24.x >=24.15 or 26+ with native SQLite limits                                                    |
| Recorded Linux crashes | Raw notes, every recorded thread's registers/signals and optional mapping candidates | Caller-supplied pwntools on Linux x64; optional GDB/pwndbg; [recorded crashes](docs/recorded-crashes.md)                              |
| JavaScript / Electron  | Modules, imports, source maps, routes, IPC and native add-on relationships           | Node.js and npm; Linux: procps-compatible `ps`; [application analysis](docs/javascript-application-workflows.md)                      |
| Websites               | Page structure, scripts, network observations and requested screenshots              | A Chrome-family browser; [browser analysis](docs/browser-observation.md)                                                              |
| Saved network captures | Requests, responses, exposed payloads and source locations                           | HAR or Fiddler Everywhere SAZ; mitmdump on Linux for native mitmproxy; [capture guide](docs/web-network-captures.md)                  |
| .NET assemblies        | Metadata, CIL instructions, declared native dependencies and build comparisons       | None (static only); [managed-code guide](docs/managed-code-analysis.md)                                                               |
| Android APKs           | Manifest declarations, classes, decompiled methods and references                    | Headless JADX and a full JDK on Linux/macOS/Windows x64; [Android guide](docs/android-analysis.md)                                    |
| Android devices        | Devices, packages, processes, logs, dumps, screen captures, and APK/file transfers   | Caller-supplied adb (emulator or hardware); [ADB guide](docs/adb-device-analysis.md)                                                  |
| JEB projects           | Engine-backed project units, artifact digests and type/method pseudo-code            | A caller-started JEB client serving MCP, selected with REA_JEB_MCP_URL; [JEB guide](docs/jeb-analysis.md)                             |
| Android resources      | Decoded manifest, string tables, locales and version facts                           | Caller-supplied Apktool launcher; [resource guide](docs/apktool-resource-analysis.md)                                                 |
| Flutter apps           | Dart snapshot hash, per-ABI library digests and engine build provenance              | None (static APK parsing); [Flutter guide](docs/flutter-analysis.md)                                                                  |
| Firmware               | Regions, extraction results and native-analysis handoffs                             | Binwalk / Unblob on Linux; [firmware guide](docs/firmware-analysis.md)                                                                |
| Recorded packets       | Selected fields, repeated values, filters and decode-as rules                        | Linux: TShark 4.x + prlimit; [capture guide](docs/packet-captures.md)                                                                 |
| Packet summaries       | Endpoints, DNS, TLS SNI, protocol distribution and HTTP requests                     | Caller-supplied tshark and capinfos (Linux-verified); [pcap guide](docs/pcap-analysis.md)                                             |
| Recorded logic signals | Decoder annotations, sample ranges and diagnostics                                   | Linux: sigrok-cli 0.7.2 + prlimit; [signal recordings](docs/signal-recordings.md)                                                     |
| Packages and resources | File inventories, digests, plists, Apple bundle anatomy and extracted resources      | [Artifact and JavaScript guide](docs/javascript-artifact-reconstruction.md), [Apple applications](docs/apple-application-analysis.md) |
| Process behavior       | Terminal output, interactions, exit and filesystem observations, and run comparisons | Linux/macOS with a native PTY; [process capture](docs/process-capture.md)                                                             |

Static JavaScript and .NET inspection read the supplied files without running
the application. Runtime capture runs or interacts with the selected target
using your user permissions; each runtime guide describes its effects.

For offline native-binary inspection, Build Anything also supports caller-supplied GNU
objdump and Rizin. See the [reverse-engineering tools guide](docs/reverse-engineering-tools.md)
for provider setup, CLI commands, and Evidence limitations. Build Anything does not install
or update either provider.

<a id="choosing-a-deep-analysis-provider"></a>

Native formats and host support vary by provider. See
[Hopper and Ghidra setup](docs/installation.md#hopper), the
[IDA guide](docs/ida-provider.md) (including an optional native MCP bridge), the
[Kuna decompiler guide](docs/kuna-provider.md), and
[experimental Windows Ghidra support](docs/windows-ghidra-p0.md).
Ghidra also supports [16-bit DOS analysis](docs/ghidra-dos.md).
For large binaries, raise the provider's startup deadline with
`REA_GHIDRA_STARTUP_TIMEOUT_MS` or `REA_HOPPER_STARTUP_TIMEOUT_MS`.
For provider selection, see the [CLI guide](docs/cli.md#choose-a-provider).
Check [release availability](docs/installation.md#released-package-and-main)
for features added since the latest npm release.

## FAQ

<details>
<summary><strong>Which agents can use Build Anything?</strong></summary>

Any agent that supports local MCP servers. Setup configures the
[supported agents](docs/installation.md#supported-agents); other clients can use
[manual MCP registration](docs/installation.md#mcp-registry).

</details>

<details>
<summary><strong>Do I need Hopper, Ghidra or IDA?</strong></summary>

Deep native analysis uses one of them. Static JavaScript and .NET inspection
work without a native analysis engine. Setup can install Hopper after approval;
Ghidra and IDA use your existing installations. See [provider setup](docs/installation.md#hopper).

</details>

<details>
<summary><strong>Do I need to start Hopper first?</strong></summary>

Build Anything starts Hopper when an operation needs it. On macOS, a first-run dialog may
ask you to choose demo mode or activate your license. Complete that prompt in the
same user session before your first CLI/MCP operation or unattended job. Open
Hopper manually if needed. See
[Hopper startup and troubleshooting](docs/installation.md#launcher-paths-and-troubleshooting).

</details>

<details>
<summary><strong>What does installing the skill from skills.sh do?</strong></summary>

The skill supplies investigation instructions for your agent. Use `npx build-anything setup`
to register Build Anything's MCP server and install the matching instructions, then restart
your agent. See [skill-only installation](docs/installation.md#skill-only-installation).

</details>

<details>
<summary><strong>What code does Build Anything return?</strong></summary>

Native analysis returns pseudocode and assembly. JavaScript/Electron analysis
recovers modules and their relationships. Your agent uses these findings to
write and test an implementation; the [showcases](#showcases) give worked examples.

</details>

<details>
<summary><strong>Does Build Anything upload my app?</strong></summary>

Build Anything analyzes targets locally. Your agent receives the tool results, and its
model provider has its own data policy.

</details>

<details>
<summary><strong>What should I do if I hit a bug?</strong></summary>

Update first; a recent release may already fix it.

For a CLI installed with `npm install --global` or `bun add --global`:

```bash
bant update
```

For agent setup through `npx`:

```bash
npx build-anything@latest setup
```

If you're using an agent, complete the [setup refresh](#update-rea) and restart
it. Retry the same task. If the problem persists, [open an issue](https://github.com/xueba-gh/build-anything/issues)
with your Build Anything version, target type, steps to reproduce and error output.

</details>

## Documentation

Start with the [documentation](docs/).
For exact options, prerequisites and result contracts:

- [Installation and setup](docs/installation.md): agent registration, provider configuration, updates and uninstall.
- [Readiness and troubleshooting](docs/installation.md#check-readiness-for-your-task): diagnose one agent or analysis engine.
- [CLI and Evidence](docs/cli.md): commands, provider selection, snapshots, import/export and exit statuses.
- [MCP contracts](docs/mcp-contracts.md) and [agent prompts](docs/mcp-prompts.md): tool results, sessions and guided investigations.
- [Tool catalog](docs/mcp-contracts.md#generated-catalog): generated inventory of tools, providers and CLI commands.
- [Roadmap](docs/roadmap.md): planned work and capability trackers.

Report vulnerabilities through [SECURITY.md](SECURITY.md).

## Disclaimer

See [DISCLAIMER.md](DISCLAIMER.md) for responsible-use guidance.

The project's position on illegal or unauthorized use covers all Build Anything versions,
distribution channels, and platforms; each copy retains its applicable licenses.

Build Anything is for lawful research, education, interoperability, software preservation,
and analysis of software you own or are authorized to examine.

## Contributing

We'd love your help with Build Anything! [Open an issue](https://github.com/xueba-gh/build-anything/issues) to
report a bug or suggest a feature, or [send a pull request](https://github.com/xueba-gh/build-anything/pulls)
to improve the code or docs.

See [CONTRIBUTING.md](CONTRIBUTING.md) for development setup and checks,
[testing](docs/testing.md) for verification lanes, and the
[architecture map](docs/architecture.mermaid) for the project structure.

## License

[MIT](LICENSE)

Code in this fork is licensed under MIT; the upstream REA project is (c) morluto under MIT (see [`NOTICE.md`](NOTICE.md)). Third-party material retains its own
licenses and copyright notices; see
[Third-party notices](THIRD_PARTY_NOTICES.md) for the dependency inventory,
preserved license texts, and external-tool terms.

**You must legally obtain and be authorized to use commercial tools**, including
IDA Pro / Hex-Rays decompilers, Hopper, and JEB, under their applicable
licenses. Build Anything grants no tool license, key, or activation entitlement. Trials and
demos remain subject to vendor terms, including user, feature, automation, and
output sharing restrictions. Cracked copies, stolen keys, unauthorized key
sharing, and bypassing license restrictions are not authorized. This guidance
does not amend MIT or third-party licenses.
