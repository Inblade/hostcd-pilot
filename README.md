# hostcd-pilot

Host-config manifests for a hostcd pilot on Frankfurt-2 Ubuntu VMs.

- `redis/` — installs `redis-server`, keeps the service enabled + running.
- `postgres/` — installs `postgresql`, keeps the service enabled + running.

Pointed at by two hostcd Applications (revision `main`), one host each.
