# AIOS — repo instructions for Claude

This repo is a Claude Code plugin marketplace that ships Hebrew AI Operating Systems to clients. `plugins/aios-core` is the standard layer installed for every client; per-client vertical plugins install on top. Full build conventions live in `BUILD-SPEC.md`. Read it before adding or changing a skill.

## OS-agnostic is non-negotiable

Every skill must work identically on **macOS**, **Windows** (inside Git Bash), and **Linux**. This is the single most important constraint in this repo, for one reason: the clients are non-technical business owners, and we do not control their machine. A skill that quietly assumes macOS will fail in the client's hands with no one technical nearby to fix it. A broken skill on a client laptop is a broken product.

The clients run a real mix of Windows and Mac. Windows support is deliberate (see `SETUP-WINDOWS.md` and the `git log` history: "Windows client support (Git Bash, gws, Desktop Routines)", "connect self-validates tools (OS-agnostic preflight)"). `/connect` already detects the OS and gives per-OS instructions. Hold every new skill to that same bar.

### Rules

1. **Detect, never assume.** Any skill that branches on the platform starts with the standard block (copy it from `/connect`):
   ```bash
   case "$(uname -s)" in
     Darwin) OS=mac ;;
     Linux) OS=linux ;;
     MINGW*|MSYS*|CYGWIN*) OS=windows ;;
     *) OS=unknown ;;
   esac
   ```
   On Windows the commands run in Git Bash, where `$HOME` is the user profile and paths use forward slashes. Match install instructions and download links to the detected OS. Never hardcode a Mac-only path, `brew`, or a `.app`.

2. **Depend only on tools guaranteed on a fresh client.** The required set is small: `git`, `node`, `gws`, and the `claude` CLI (`wacli` optional). Do NOT assume `unzip`, `python3`, `jq`, `tar` with zip support, GNU-only flags, or any Homebrew tool. If a tool is not in the required set, either install it per-OS (as `/connect` installs `wacli`) or avoid it.

3. **Prefer the model over the shell for parsing.** These skills are model-executed. To read a JSON manifest or config, use the Read tool and take the field, do not pipe through `python3 -c` or `jq`. This removes a whole class of cross-platform breakage.

4. **Use portable file operations.** `mkdir -p`, `cp -R`, `rm -rf`, `find`, `ls`, `grep`, `sed` are fine (present in Git Bash). For zip extraction use a tool cascade that tries what exists (`unzip` -> `ditto` on macOS -> `powershell Expand-Archive` on Windows -> `tar`), never a bare `unzip`. The `/install` skill's `extract_zip` is the reference implementation.

5. **PATH after install.** When a skill installs a CLI, tell the client to open a NEW terminal (on Windows: a new Git Bash window) before the re-check, because the current shell will not see the new tool. `/connect` Troubleshooting is the model for this.

6. **No em dashes, Hebrew client-facing output.** Same as `BUILD-SPEC.md`. These are global, not OS-specific, but they ship with every skill.

### Before you call a skill done

- Re-read the bash blocks and confirm each command runs in Git Bash on Windows, not just macOS.
- Confirm no `unzip` / `python3` / `jq` / `brew` / Mac-only path crept in.
- If the skill branches on OS, confirm all three branches (mac / windows / linux) are present and correct.
- Exemplars to copy: `plugins/aios-core/skills/connect/SKILL.md` (per-OS install + preflight) and `plugins/aios-core/skills/install/SKILL.md` (portable extraction + model-side JSON read).
