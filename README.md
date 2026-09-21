# Setting up your laptop for the MLDS servers

`mlds-setup` is a small program you run **on your laptop**. It checks that
Git and the GitHub CLI are installed, signs you in to GitHub and sets your `git` username + email, 
makes sure you have an SSH key and that it is on your GitHub account, and then registers you
with the MLDS servers so you can `ssh` in. It takes about two minutes.

The application is idempotent; can run it again any time. If you ever can't 
log in, running it again will usually be the fix.

## Before you start

1. **A GitHub account**, with the username you gave us on the intake survey.
   If you changed your username since, tell the MLDS sysadmin (Mateusz).
2. **Git** and the **GitHub CLI (`gh`)** installed:

   | | Git | GitHub CLI |
   |---|---|---|
   | Mac | `xcode-select --install` in Terminal | `brew install gh` (or https://cli.github.com) |
   | Windows | https://git-scm.com/download/win | `winget install --id GitHub.cli` in PowerShell |
   | Linux / WSL | your package manager (`apt install git`) | https://github.com/cli/cli#installation |

   On Windows, open a **new** terminal after installing.
3. **Be on campus Wi-Fi or the NU VPN** for the last step. Everything else
   works from anywhere.

## Download

Go to https://github.com/NUMLDS/mlds-setup-releases/releases/latest and grab
the file for your machine:

| Machine | File |
|---|---|
| Mac with Apple Silicon (M1/M2/M3/M4) | `mlds-setup-darwin-arm64` |
| Mac with Intel | `mlds-setup-darwin-amd64` |
| Windows | `mlds-setup-windows-amd64.exe` |
| Linux or WSL | `mlds-setup-linux-amd64` |

Not sure which Mac you have? Apple menu → About This Mac. "Chip: Apple M…"
is Apple Silicon.

## Run it

**Mac** (Terminal):

```
cd ~/Downloads
xattr -d com.apple.quarantine mlds-setup-darwin-arm64    # or -amd64
chmod +x mlds-setup-darwin-arm64
./mlds-setup-darwin-arm64
```

If macOS still refuses: System Settings → Privacy & Security → scroll down →
**Open Anyway**, then run it again.

**Windows** (PowerShell):

```
cd ~\Downloads
.\mlds-setup-windows-amd64.exe
```

If SmartScreen appears: **More info → Run anyway**. The window waits for
Enter at the end so you can read the output.

**Linux / WSL**:

```
cd ~/Downloads
chmod +x mlds-setup-linux-amd64
./mlds-setup-linux-amd64
```

## What happens

The tool prints a checklist. `[ok]` is good, `[--]` is just information,
`[!!]` needs your attention. Along the way it may:

- **Open your browser to sign in to GitHub.** Copy the one-time code from the
  terminal, paste it into the browser, approve. This is the normal `gh auth
  login` flow.
- **Ask to set your Git name and email** if they aren't set. Answering `Y` is
  fine; it uses your GitHub profile.
- **Create an SSH key** (`~/.ssh/id_ed25519`) if you don't have one. No
  passphrase, so it just works.
- **Ask before adding a key to GitHub** if you have several. Say `Y` for the
  key you want to use for MLDS. Nothing is ever removed from your account.
- **Ask for a key's passphrase** if the key you chose is passphrase-protected.
  That's the tool proving the key is really yours before uploading it.

At the end it registers you and then tries to log in, with your key, to each
server your account is enabled on, one line per server:

```
== MLDS servers
[ok] linked GitHub octocat to netid abc1234: 1 key(s) on file, 1 added, 0 removed

== Login test
[ok] wolf.mlds.private
[ok] posit.mlds.northwestern.edu
[--] deepdish5.mlds.private: not enabled for your account

All done.
```

A green host is ready for you. A `[--]` host exists but isn't enabled for
your account (that's a roster setting, not a key problem). A red host is
enabled for you but refused the key; tell Mateusz which one.

## Log in

```
ssh <your-netid>@wolf.mlds.private
```

Your username on the servers is your **NetID**, not your GitHub name and not
your laptop username. Your instructor(s) will tell you which hosts to use for
a specific class, if one is required. You are of course free to use whichever
server you like outside of classes too, for its intended teaching purpose. If
you are ever unsure what a server can or can't be used for, please ask Mateusz.

## Your MLDS server password

You have **one MLDS server password, the same on every MLDS server**. It is
what anything with a password prompt asks for: Posit Workbench in the
browser, JupyterHub login, `psql` and any other database client. `ssh` is the
exception: it uses your key and never asks. Set the password once, from your laptop:

```
./mlds-setup-darwin-arm64 password      # Mac; Windows: .\mlds-setup-windows-amd64.exe password
```

It asks for the password twice (nothing is shown as you type), at least 12
characters. **Do not reuse your Northwestern NetID password.** Pick a new one
just for the MLDS servers. It is hashed on your laptop before anything is
sent, and it works on every MLDS server within a few seconds.

| Where | Username | Password |
|---|---|---|
| Posit Workbench, JupyterHub, anything in a browser | your NetID | your MLDS server password |
| `psql -h pg.mlds.northwestern.edu -U <netid> <db>` (and any DB client) | your NetID | your MLDS server password |
| `ssh` | your NetID | none — your key |

Forgot it, or want a new one? Run `mlds-setup password` again; the new
password replaces the old one. There is nothing to reset and nobody to ask.

**If a normal `mlds-setup` run ends with `no MLDS server password set yet`,
this is the step it means.**

## If something goes wrong

| You see | Do this |
|---|---|
| `gh is not installed` / `git is not installed` | install it (table above), open a new terminal, re-run |
| `cannot reach …` (a server name) | connect to campus Wi-Fi or the NU VPN, re-run |
| `GitHub account '…' is not on file for anyone` | tell your instructor or TA your exact GitHub username, then re-run |
| `report timestamp is off by more than 10 minutes` | your computer's clock is wrong; fix it, re-run |
| `…pub does not belong to …` | that key file and its `.pub` don't match. Move both out of `~/.ssh`, re-run, and the tool makes a fresh pair |
| `upload of … failed` | re-run; if it keeps failing, add the key yourself at https://github.com/settings/keys |
| `no private key on this machine is on github.com/<you>.keys` | you said no to every upload, so nothing can be registered; re-run and answer `Y` for the key you want to use |
| `report saved to ~/mlds-setup-report.json` | the MLDS server couldn't be reached; get on campus or the VPN and re-run, or send that file to your instructor |
| a host in the login test says `permission denied` or `timed out during login` | that server isn't set up for your account yet; tell your instructor which one. Your key is fine |
| `host key changed` | don't log in to that host; tell your instructor |
| `Permission denied` when you `ssh` later | run `mlds-setup` again. Still stuck? Send your instructor the output |
| `no private key on this machine is on github.com/<you>.keys — run plain mlds-setup first` (from `mlds-setup password`) | this laptop hasn't been set up yet; run `mlds-setup` with no arguments, then `mlds-setup password` |
| `stdin is not a terminal` (from `mlds-setup password`) | run it in a normal terminal window, not from a script or an IDE's output pane |
| `…not newer than the last password change` / "run mlds-setup password again" | just run it again |
| Posit or psql rejects the password you set | give it a few seconds and retry; still no? tell your instructor which server |
| `TLS: the server's certificate could not be verified` | a hotel/café network or proxy is in the way; use campus Wi-Fi or the NU VPN |

## What it sends, and what it doesn't

The tool sends the MLDS server your GitHub username, basic facts about your
laptop (OS, version, RAM, free disk, which dev tools are installed) and the
fingerprints of your SSH keys, cryptographically signed with your SSH key so we know it's you.
`mlds-setup password` sends a one-way hash of the server password you typed,
signed the same way, over HTTPS; the password itself never leaves your
laptop. It never sends a private key, never sends your GitHub password or
token, and never deletes anything. The server fetches your public keys straight from
GitHub, so what's at https://github.com/settings/keys is exactly what gets
you in.
