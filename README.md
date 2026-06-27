# askpass

A small askpass helper that shows what is asking for a password before it
returns the secret to the caller.

## Features

- Modern GTK 4 / libadwaita dialog that leads with what matters: the command
  being invoked, an optional reason, and the password field.
- The calling process chain is tucked into a collapsible "Process details"
  section so the default view stays uncluttered while the audit trail remains
  one click away.
- Lets agents explain why they are invoking the command with `ASKPASS_REASON`
  or the `askpass-run --why` wrapper.
- Falls back gracefully: GTK first, then `zenity`, then `kdialog`, and finally
  a terminal prompt for headless/manual testing.
- Keeps the implementation compact and dependency-light (Python standard
  library, plus PyGObject when a GUI is available).

## Usage

Use the helper directly with tools that understand askpass:

```sh
export SSH_ASKPASS="$PWD/askpass"
export GIT_ASKPASS="$PWD/askpass"
export SUDO_ASKPASS="$PWD/askpass"
export ASKPASS_REASON="Fetching a private dependency for the build"
export ASKPASS_COMMAND="git fetch origin main"
```

Then run the command that may need credentials.

For agent-driven commands, prefer the wrapper so the dialog can show a reason
and the exact command:

```sh
./askpass-run --why "Fetch private dependency requested by the build step" -- \
  git fetch origin main
```

For `sudo`, pass `-A`:

```sh
./askpass-run --why "Install local package dependencies" -- sudo -A apt install ./pkg.deb
```

For SSH in environments without a terminal prompt, set `DISPLAY` or
`WAYLAND_DISPLAY` as appropriate. `SSH_ASKPASS_REQUIRE=prefer` is set by the
wrapper unless already provided.

## Environment

- `ASKPASS_COMMAND`: command text shown in the dialog. If omitted, the helper
  shows the immediate parent process command from `/proc`.
- `ASKPASS_REASON`: human-readable reason shown in the dialog.
- `ASKPASS_TERMINAL=1`: force the terminal fallback.

## Auditing notes

The secret only travels from the GUI program to stdout, which is the askpass
contract. Context is shown in the dialog and written only to stderr by the
terminal fallback. The helper reads process metadata from `/proc`. It relies on
the Python standard library, plus PyGObject (GTK 4 / libadwaita) for the GUI
dialog when a display is available.
