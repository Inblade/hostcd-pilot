# hostcd-pilot

Host-config manifests for a hostcd pilot on Frankfurt-2 Ubuntu VMs.

## Apps

- **`redis/`** — installs `redis-server`, keeps the service enabled + running.
- **`postgres/`** — installs `postgresql`, keeps the service enabled + running.
- **`webapp/`** — a richer, **templated** nginx web service that renders its config
  and landing page from the host's **labels (the "values") + facts**. Shows multiple
  resource kinds working together: `groups`, `users`, `packages`, `directories`,
  `files` (two of them templated), `services` (reloaded on config change via
  `notify`), `sysctls`, and `crons`.

Each app is pointed at by a hostcd **Application** (revision `main`).

## webapp — required host labels ("values")

`webapp` templates render with `missingkey=error`, so a targeted host **must** carry
all of these labels or the app shows a render error:

| label | example | used for |
|-------|---------|----------|
| `role` | `webapp` | targeting (the Application selects `role=webapp`) |
| `app_name` | `shop` | vhost name + page title |
| `env` | `staging` | vhost name + page |
| `http_port` | `80` | nginx `listen` port |
| `max_body_mb` | `16` | nginx `client_max_body_size` |
| `owner_team` | `platform` | shown on the page |

Change any value label on the host and hostcd re-renders `webapp.conf` + `index.html`
on the next sync (nginx is reloaded automatically because the file `notify`s it).
