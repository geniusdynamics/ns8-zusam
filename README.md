# ns8-zusam

Zusam (/tsuˈzam/) is a free and open-source way to self-host private forums for groups of friends. Composed of a server written in PHP exposing a REST API and a lightweight webapp, Zusam is extensible and easy to install.

Module id: `zusam`. Images are pinned in `build-images.sh` and updated through reviewed changes.

## Install

```bash
add-module ghcr.io/geniusdynamics/zusam:latest 1
```

The command returns the instance id, for example `zusam1`. Use the id NS8 returns.

## Configure

```bash
api-cli run configure-module --agent module/zusam1 --data - <<'EOF'
{
  "host": "zusam.example.org",
  "http2https": true,
  "lets_encrypt": false,
  "init_user": "",
  "init_password": "",
  "init_group": ""
}
EOF
```

| Parameter | Required | Type | Description |
|---|---|---|---|
| `host` | yes | string | Host name for the application, like 'zusam.domain.org' |
| `lets_encrypt` | yes | boolean | Request a valid Let's Encrypt certificate. |
| `http2https` | yes | boolean | Redirect all the HTTP requests to HTTPS |
| `init_user` | yes | string | Initial admin user |
| `init_password` | yes | string | Initial admin password |
| `init_group` | yes | string | Initial admin group |

## Configuration output

`get-configuration` returns: `host`, `lets_encrypt`, `http2https`.

## Images

| Image | Version |
|---|---|
| `docker.io/zusam/zusam` | `0.6.1` |

## Network and access

- Reserves 1 TCP port(s) on the node. The web application is published through Traefik on the configured host name.
- Runs rootless.
- Declares these NS8 authorizations: `traefik@node:routeadm`.

## Backup

The following paths are included in the NS8 backup (`imageroot/etc/state-include.conf`):

- `state/zusam.sql`
- `volumes/zusam-app`

## Services

- `mariadb-app.service`
- `zusam-app.service`
- `zusam.service`

## Get the configuration

```bash
api-cli run get-configuration --agent module/zusam1
```

## Update and uninstall

```bash
api-cli run update-module --data '{"module_url":"ghcr.io/geniusdynamics/zusam:latest","instances":["zusam1"],"force":true}'
remove-module --no-preserve zusam1
```

## Testing

Continuous integration:

- `Publish images` builds the module image on every push.
- `Test module` installs the image on a QEMU NS8 node and runs the Robot Framework suite in `tests/`. It runs after `Publish images` succeeds on `dev` and on `main`.
- `Build API JSON Schema docs` and the security scan run alongside.

The suite installs the module, configures a host name, checks that the web application answers through Traefik, takes UI screenshots and removes the module.

Run it locally with the NS8 standard infrastructure described in the [ns8-github-actions README](https://github.com/NethServer/ns8-github-actions/blob/v1/README.md#running-tests-locally).

## Open items

Found by the NS8 module standards check. These must be resolved before a stable release:

- tests/zusam.robot generated from template: set real configure payload and URL check
- metadata: categories empty (use standard categories)
- mariadb-app.service: uses %S/state, kickstart uses %E/state
- zusam-app.service: uses %S/state, kickstart uses %E/state
- zusam.service: uses %S/state, kickstart uses %E/state
- discover-smarthost still present: remove if the app does not use SMTP
- README still has the template boilerplate
- mariadb-app.service: %S/state changed to %E/state (review the bind mounts that remain)
- zusam-app.service: %S/state changed to %E/state (review the bind mounts that remain)
- zusam.service: %S/state changed to %E/state (review the bind mounts that remain)
- imageroot/actions/configure-module/80start_services: smarthost lines removed, check the unit still starts
- imageroot/systemd/user/zusam-app.service: smarthost lines removed, check the unit still starts
- ZUSAM_IMAGE is used by the module but no image in the label provides it: leftover from the template?

## Upstream

Application documentation: https://github.com/zusam/zusam/tree/master/documentation

## License

GPL-3.0-or-later. See [LICENSE](LICENSE).
