# uf-config

Configures an already-installed Universal Forwarder: points it at a deployment
server and forwards its `_internal` logs to the indexing tier.

Used by `uf-config.yml` and `uf-combo.yml`.

## Variables

| Variable | Default | Purpose |
| --- | --- | --- |
| `uf_deploy_server_address` | `127.0.0.1` | Deployment server address |
| `uf_deploy_server_port` | `8089` | Deployment server management port |
| `uf_forward_internal_logs_address` | `127.0.0.1` | Where `_internal` logs are sent |
| `uf_forward_internal_logs_port` | `9997` | Receiving port on that host |

Only a single forwarding address is supported.

Plus the shared variables from `splunk-common`.

## Notes

- The deployment server is set by templating `deploymentclient.conf` rather
  than running `splunk set deploy-poll`. That command took admin credentials on
  its command line, where they were visible in the process list and in Ansible
  output, and it ran on every play whether or not anything had changed.
- Both files notify the `restart splunk uf` handler, so the forwarder restarts
  only when its configuration actually changed.
