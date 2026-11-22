# hostcd-pilot

Host-config manifests for a hostcd pilot on Frankfurt-2 Ubuntu VMs.

## Commits must be signed

The pilot server runs with `-allowed-signers`: it deploys a commit only if it is
signed by a key in `/etc/hostcd/allowed_signers` on the control plane (today the
`gitops@hostcd-pilot` SSH key, `~/.ssh/hostcd-pilot-gitops`). An unsigned head is
rejected and hosts stay on the last verified revision. Commits made in the GitHub
web editor are not accepted. In your clone, once:

```bash
git config gpg.format ssh
git config user.signingkey ~/.ssh/hostcd-pilot-gitops.pub
git config commit.gpgsign true
```

Check before pushing: `git log --show-signature -1`. To allow another person's key,
add a line `name@example namespaces="git" ssh-ed25519 AAAA…` to the server's
`allowed_signers` and restart `hostcd-server`.

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
