# Android Device Control

A local ADB command-line tool and MCP server for inspecting authorized Android
devices. Find connected devices, see the foreground app, and read the screen's
accessibility tree. The MCP tools are read-only by default, with taps available
when explicitly enabled for one device.

[![Android Device Control on mcprush](https://mcprush.com/badge/android-device-control-mcp/compact.svg)](https://mcprush.com/to/android-device-control-mcp?ref=badge) [![Android Device Control on Glama](https://glama.ai/mcp/servers/lisn0/android-device-control/badges/score.svg)](https://glama.ai/mcp/servers/lisn0/android-device-control)

[PyPI](https://pypi.org/project/android-device-control/) ·
[npm](https://www.npmjs.com/package/android-device-control) ·
[MCP Registry](https://registry.modelcontextprotocol.io/v0.1/servers/io.github.lisn0%2Fandroid-device-control/versions/latest)

[Install the CLI](#install) · [Run the MCP server](#run-the-mcp-server)

## Install

Python 3.10 or newer is required. Install `adb` from
[Android SDK Platform-Tools](https://developer.android.com/tools/releases/platform-tools)
and put it on your PATH. In your Python environment, install the CLI.

```sh
python -m pip install android-device-control
```

The CLI uses the Python standard library. If you are working from a clone, you
can run the scripts directly with `python3 -B phone.py`, `device_info.py`,
`messages.py`, and `receipts.py`.

## Get started

1. Connect your test device over USB, enable USB debugging, and approve your
   computer on the device. See Google's [ADB setup guide](https://developer.android.com/tools/adb)
   for USB troubleshooting and wireless pairing.
2. List devices and run a read-only check. Replace
   `DEVICE_SERIAL` with the serial shown for your authorized device.

   ```sh
   android-control devices
   android-control-device-info --device DEVICE_SERIAL
   android-control foreground --device DEVICE_SERIAL
   ```

Open an ordinary test screen on the device to read its accessibility tree.

```sh
android-control snapshot --device DEVICE_SERIAL
android-control --help
```

Use `android-control-sms --help` for SMS diagnostics and
`android-control-receipts --help` for the local setup receipt ledger.

## Run the MCP server

With Node.js, `uvx`, and `adb` on your MCP client's PATH, launch the local
stdio server from npm:

```sh
npx --yes android-device-control
```

The launcher uses the published PyPI package and forwards options such as
`--device DEVICE_SERIAL`. Call `list_devices` to check the connection.

### Install from PyPI or run with uvx

```sh
python -m pip install 'android-device-control[mcp]'
```

For a client that uses `mcpServers` JSON, use the installed command on its PATH
or set `command` to the full path to the executable.

```json
{
  "mcpServers": {
    "android-control": {
      "command": "android-control-mcp"
    }
  }
}
```

With `uvx` on the client's PATH, use `"command": "uvx"` and
`"args": ["--from", "android-device-control[mcp]", "android-control-mcp"]`.

### Run from a checkout

Create a Python environment with the MCP SDK.

```sh
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements-mcp.txt
```

Replace the example paths with the absolute paths to the environment's Python
executable and `android_mcp.py`.

```json
{
  "mcpServers": {
    "android-control": {
      "command": "/absolute/path/to/android-control/.venv/bin/python",
      "args": ["/absolute/path/to/android-control/android_mcp.py"]
    }
  }
}
```

### Scope tools to a device

The default tools are `list_devices` (authorized serials and ADB states),
`foreground` (focused package and activity), and `snapshot` (guarded UI summary).
All three are read-only. Supply the target `serial` to `foreground` and `snapshot`. To scope every tool
to one phone, append `"--device", "DEVICE_SERIAL"` to `args`, using the serial
shown by `android-control devices` or `python3 -B phone.py devices`. The scoped
read tools can then omit `serial`.

To enable `tap_label`, append `"--allow-input"` after the device arguments. The
tool acts only on that configured phone and requires a single enabled clickable
match for the exact accessibility label. Inspect the screen again after a tap
to verify its effect.

Use a trusted local client and a test screen. Snapshots can contain private app
content. Handle sensitive prompts manually and keep login, messaging, and
payment screens out of the MCP workflow.

## Client plugins

The Claude Code, Antigravity, Cursor, and Gemini CLI packages launch the same local
MCP server. Before installing, put Node.js, `uvx`, and Android SDK
Platform-Tools on the client's PATH and authorize a test device over ADB.

Open the plugin from this repository. In Claude Code, run `claude --plugin-dir ./plugins/claude`
from a checkout. In Antigravity, use `agy plugin install .`. In Cursor, use
**Import from Disk** in **Customize**. For Gemini CLI, install this repository with `gemini extensions install https://github.com/lisn0/android-device-control`.

Ask the client to list authorized Android devices, then inspect the foreground
app on a test device. The plugin enables read-only inspection by default.

## Workflows

- [App navigation and UI interaction](https://github.com/lisn0/android-device-control/blob/main/docs/workflows.md)
- [Device diagnostics and SMS inspection](https://github.com/lisn0/android-device-control/blob/main/docs/device-information.md)
- [Manual social-account setup and receipts](https://github.com/lisn0/android-device-control/blob/main/docs/social-account-setup.md)
- [Design references](https://github.com/lisn0/android-device-control/blob/main/docs/research.md)

The authentication guard uses English accessibility labels and password-field
metadata. It can miss unlabeled or non-English screens. Handle passwords,
verification codes, recovery, identity, and payment prompts directly on the device.
Keep secrets out of command arguments and receipts.

## Tests and bug reports

```sh
python3 -B -m unittest discover -s tests -v
```

The tests use mock ADB responses. For a bug report, provide the command, Android
and ADB versions, and a sanitized error or synthetic reproduction. Device lists,
snapshots, screenshots, and SMS output can contain serials, account identifiers,
private messages, or codes. Remove those details before sharing.

## License

[MIT](https://github.com/lisn0/android-device-control/blob/main/LICENSE)

## Maintainer releases

Set the same version in `pyproject.toml` and `package.json`. Build and test the
Python distributions and the npm archive from that source. Attach these three
files to the stable GitHub release before publishing it.

- `android_device_control-X.Y.Z-py3-none-any.whl`
- `android_device_control-X.Y.Z.tar.gz`
- `android-device-control-X.Y.Z.tgz`

Publishing the GitHub release with tag `vX.Y.Z` uploads the Python distributions
to PyPI first and the npm launcher second. Both registries require a trusted
publisher for `publish-pypi.yml`. The PyPI job uses the `pypi` environment.

To update the MCP Registry listing, match the versions in `server.json` to the
published npm package. Push the manifest, then use `mcp-publisher publish server.json`
from a machine authenticated to the official registry.
