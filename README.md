# harmony-sase-fix

Recover the **Harmony SASE** (Check Point, formerly Perimeter 81) macOS VPN when
it won't connect because its privileged helper daemon is wedged — a vendor bug
where the daemon migration leaves the helper empty, locked (`schg`), disabled in
`launchd`, or missing entirely, and the app's own retry (running unprivileged)
can't repair it.

`harmony-fix` detects the wedge, restores the daemon from the app's own payload,
and re-registers it with `launchd` as root. It also triages the common
*authorization* failures (session expired / device not recognized) so you don't
tamper with the daemon when the real problem is just a login.

> ⚠️ **What it does to your system.** With `sudo` it clears the `schg`
> (immutable) flag, may `rm -rf` and re-copy the daemon bundle under `/Library`,
> and quits/relaunches the app. Nothing persists beyond a working daemon. Read
> `harmony-fix --help` first, or preview with `--dry-run`.
>
> Tested only on **Harmony SASE 12.7.4 (build 9577) / macOS 26.4.1** — it's tied
> to this vendor build. Re-verify before running against other versions.

## Requirements

- macOS, with the Harmony SASE app at `/Applications/Harmony SASE.app`.
- An admin account (for the single `sudo` prompt).

## Install (macOS)

```sh
git clone https://github.com/akirayamamoto/harmony-sase-fix.git
cd harmony-sase-fix

# symlink onto your PATH — pick one:
ln -sf "$PWD/harmony-fix" ~/.local/bin/harmony-fix     # if ~/.local/bin is on PATH
sudo ln -sf "$PWD/harmony-fix" /usr/local/bin/harmony-fix   # system-wide
```

If macOS quarantines the script, clear it with `xattr -d com.apple.quarantine harmony-fix`.

## Usage

```sh
harmony-fix --check     # diagnose only — exit 0 healthy · 2 wedged · 3 unknown · 4 access expired · 5 re-login
harmony-fix --dry-run   # show the repair plan, change nothing
harmony-fix             # detect + repair (one sudo prompt)
harmony-fix --help      # full runbook
```

## Optional configuration

Org-specific hints (access-request guide, escalation contact, where to reinstall
the app) come from environment variables or an env file at
`~/.config/harmony-sase-fix.env`; generic defaults apply otherwise:

```sh
HARMONY_FIX_ACCESS_URL="https://your-wiki/vpn-access"
HARMONY_FIX_ESCALATE="Contact your VPN / IT admin."
HARMONY_FIX_REINSTALL_SRC="your MDM / self-service portal"
```

## License

[MIT](LICENSE)
