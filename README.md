# nymea-cli

A small command-line client for [nymead](https://nymea.io/)'s JSON-RPC API,
built for the "edit → restart → `grep journalctl` → repeat" phase of
plugin debugging. Lets you list things, read state values, execute
actions, stream state-change notifications, and (if the running `nymead`
supports it) toggle log categories without a restart.

Single-file Python script, one third-party dependency (`websocket-client`).
Runs equally well on the box running `nymead` and from your laptop
pointing at it over the network.

## Install

```sh
# requires Python 3.8+
pip install websocket-client

# copy or symlink anywhere on PATH
install -m 0755 nymea-cli ~/.local/bin/nymea-cli
```

Or just `scp nymea-cli pi@host:/usr/local/bin/` — no package install needed.

## Quick start

```sh
# One-time login (caches a token in ~/.config/nymea-cli/config.json)
nymea-cli login --host 192.168.1.195

# What's configured?
nymea-cli things list                       # all
nymea-cli things list küche                 # substring filter on name

# Read state values
nymea-cli state get "Stüberl 1"             # all states
nymea-cli state get "Stüberl 1" status      # one state by name

# Trigger an action (name resolution via thingClass)
nymea-cli action exec "Stüberl 1" close
nymea-cli action exec "Relay Küche Rollo auf" power power=true

# Stream state changes live
nymea-cli events watch
nymea-cli events watch --filter-thing "Stüberl 1"

# Toggle a log category at runtime (if nymead version supports it;
# falls back to a note about editing nymead.conf if it doesn't)
nymea-cli log set GenericShading debug

# Escape hatch — any JSON-RPC call
nymea-cli raw Integrations.GetThingClasses
nymea-cli raw Integrations.GetStateValue --params '{"thingId":"...","stateTypeId":"..."}'
```

## Resolving things and actions by name

Where the API expects a UUID — thing id, action type id, param type id,
state type id — this CLI accepts either the UUID or the human name and
resolves against `Integrations.GetThings` and
`Integrations.GetThingClasses`. Names are matched case-insensitively,
exact match preferred, substring match used otherwise. If more than one
thing matches, the CLI prints the candidates and exits.

Action parameters are passed as `name=value`. The value is parsed as
JSON when possible (so `power=true`, `duration=5000`, and
`color=[255,0,0]` all work) and falls back to a plain string.

## Config file

`~/.config/nymea-cli/config.json` (or `$XDG_CONFIG_HOME/nymea-cli/`):

```json
{
  "host": "192.168.1.195",
  "port": 4444,
  "username": "admin",
  "token": "…"
}
```

Written on `login`. Any field (except `token`) can also be overridden on
the command line with `--host`, `--port`, or by re-running `login`.

## Connection details

Default endpoint is `wss://<host>:4444`, which matches the stock nymead
`WebSocketServer.default` configuration. `nymead` serves a self-signed
certificate, so TLS verification is **off by default** — pass `--secure`
to enforce it. `--plain` switches to `ws://` for plaintext.

## Scope / non-goals

- Not a general-purpose nymea admin tool. No rules, no scripts, no user
  management, no device discovery / pairing. If you need those, use the
  mobile/web app.
- Doesn't try to abstract over nymead API versions. If a method name has
  moved in your `nymead`, use `raw` and open an issue.

## License

MIT — see [LICENSE](LICENSE).
