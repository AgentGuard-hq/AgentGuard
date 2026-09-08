# Security policy

AgentGuard is a Linux eBPF LSM supervisor (Apache-2.0, v0.1.2). It is a **research prototype**, not a Seatbelt/bubblewrap replacement and not a hosted sandbox.

Hooks (`agentguard init --claude` / `--codex`) are **not** the TCB. Missing or untrusted hooks is not a vulnerability if the kernel still returns `EPERM`.

## How to report

**Contact:** [pramathshukla275@gmail.com](mailto:pramathshukla275@gmail.com) ([@shuklapramath](https://github.com/shuklapramath))

**Do not** open a public issue for a working bypass of enforcement.

Prefer [GitHub Private Vulnerability Reporting](https://github.com/AgentGuard-hq/AgentGuard/security/advisories/new) on this repository.

If that form is unavailable, use the contact above and say you have a **private** security report. Do not paste a full exploit in email or a public issue.

Please include:

- AgentGuard version (`agentguard version`) and how you installed it
- Kernel (`uname -r`) and `cat /sys/kernel/security/lsm` (must contain `bpf`)
- Policy file (redact secrets)
- Exact command: `sudo agentguard -- …` vs attach-by-PID vs `agentguard up`
- What you expected to be denied, what happened, and a **minimal** repro

You will get an ack when the report is seen. There is no bug bounty.

## Supported versions

| Version | Supported |
| :--- | :--- |
| Latest `v0.1.x` tag / `master` | Yes |
| Older tags, unofficial builds, `go build` trees not matching a release | No |

## What is in scope

A report is in scope if **BPF LSM is loaded** (`bpf` in the LSM list, program attached) and a **tracked** agent can still:

- Read or write a path the loaded YAML is meant to deny (e.g. credential `path_patterns`)
- `connect` / proxy to a host that is not on the allow-list while `egress: proxy_only`
- `execve` a command the destructive policy is meant to block
- Escape tracking so children are unenforced after a normal `sudo agentguard -- <agent>` launch
- Tamper with AgentGuard so policy is not what the operator loaded (without already being root)

Root on the box, or a kernel without `lsm=bpf`, is not a bypass. The supervisor loads as root on purpose.

## What is not a vulnerability

These are documented limits or non-goals. Do not file them as advisories:

- Native **Darwin** `claude` / `codex` (use `agentguard up`; the Mac host is not supervised)
- Hooks missing, untrusted, or silent chat — kernel deny without `feedback:` is expected
- `install.sh` not setting `lsm=bpf` — boot the **Linux/guest** cmdline; remounting securityfs does not enable LSM
- Policy you wrote that is too wide (e.g. `/proc` as an allow prefix is a known hole)
- `getattr` / `ls` of a denied path (name lookup is not the same as `open`)
- Hardlinks, `/proc/<pid>/root`, or other path-equivalence tricks called out in `policies/default.yaml`
- IPv6 connect fail-closed (logged as an unusable target) or proxy 403s on telemetry hosts
- Codex requiring `/hooks` trust — that is Codex, not AgentGuard
- “The agent could still think about secrets” — we do not inspect model weights
- DoS against the box, or breaking your own project files inside the workspace

## Operator notes

- `agentguard up` is privileged by design (`--privileged`, BPF caps). Treat the image and `~/.agentguard` like root.
- `agentguard login` writes `~/.agentguard/anthropic_key` mode `0600`. Do not pass keys on `argv`.
- A deny is only as tight as `policies/default.yaml` in `$PWD` (or `--policy` / `AGENTGUARD_POLICY`).
- After a Go or BPF change, the binary that is running is the one you `install`ed, not necessarily `./agentguard` in the git tree.

## Disclosure

Fixes land on this repo. Please give a reasonable window before public write-ups. We will credit reporters who want to be named.
