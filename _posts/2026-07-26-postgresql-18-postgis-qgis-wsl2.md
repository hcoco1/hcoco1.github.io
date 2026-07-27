---
title: "PostgreSQL 18 + PostGIS on WSL2, Connected to QGIS: A Zero-to-Working Runbook"
author: hcoco1
date: 2026-07-26 10:00:00 +0200
categories: [Tutorial, PostgreSQL]
tags: [postgresql, postgis, qgis, wsl2, ubuntu]
render_with_liquid: false
---

A complete, reproducible runbook: install PostgreSQL 18 and PostGIS inside WSL2 Ubuntu, configure it for network access, and connect QGIS running on Windows. Each step includes the reasoning behind it so the goal is understanding, not pasting.

## The Mental Model

Four concepts explain everything below.

- **Debian cluster system.** Ubuntu doesn't manage PostgreSQL as a single server. It uses `postgresql-common`, which wraps one or more *clusters* — a cluster is one data directory plus one running server on one port. Tools: `pg_lsclusters` (list), `pg_ctlcluster` (control one), `systemctl` (control all). Config lives in `/etc/postgresql/<ver>/<name>/`{: .filepath}, data in `/var/lib/postgresql/<ver>/<name>/`{: .filepath} — deliberately separated.
- **Two config files run the show.** `postgresql.conf`{: .filepath} controls the server (which addresses and port it listens on). `pg_hba.conf`{: .filepath} controls who may connect, from where, and how they authenticate. Nearly every "can't connect" problem is one of these two.
- **Authentication differs by path.** Connecting over the local Unix socket uses `peer` (matches your OS username — no password, which is why `sudo -u postgres psql` just works). Connecting over TCP (`-h localhost`, what QGIS does) uses `scram-sha-256` (password). Same server, different rules per path.
- **Roles vs database vs schema vs extension.** A *role* is a login. A *database* is a container. A *schema* namespaces tables inside a database (default `public`). *PostGIS* is an *extension* — geometry types and functions enabled per-database.

The full connection path this runbook builds:

![PostgreSQL and QGIS connection path](/assets/img/posts/postgresql-qgis-connection-path.svg)
_QGIS on Windows connects over `localhost:5432`, forwarded by WSL2 (NAT) to PostgreSQL 18 listening on all interfaces; `pg_hba.conf` authenticates via `scram-sha-256` into the PostGIS-enabled `gis` database._

## Phase 1 — Install

Ubuntu's default repository only carries PostgreSQL 16, so add the official PostgreSQL (PGDG) apt source first.

```bash
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
```

The first line installs the tooling from Ubuntu's repo; the script it provides adds the PGDG source. Press Enter when it prompts. Then install the server, client, and the matching PostGIS package:

```bash
sudo apt update
sudo apt install -y postgresql-18 postgresql-client-18 postgresql-18-postgis-3
```

`postgresql-18-postgis-3` is the server-side extension built for PG 18. Optionally add `postgis` for the CLI loaders (`shp2pgsql`, `raster2pgsql`). Confirm the cluster was created:

```bash
pg_lsclusters
```

Installing the server auto-creates and starts `18/main` on port 5432. Expect one row: version `18`, cluster `main`, port `5432`, status `online`.

## Phase 2 — Survive WSL Restarts

```bash
sudo systemctl enable postgresql
```

> `wsl --shutdown` stops the PostgreSQL service. Enabling auto-start makes it come back with WSL (requires systemd, which recent WSL provides). Verify next session with `sudo ss -tlnp | grep 5432`.
{: .prompt-tip }

Cluster control commands worth knowing: `sudo pg_ctlcluster 18 main start|stop|restart` (one cluster) and `sudo systemctl restart postgresql` (all clusters).

## Phase 3 — Roles, Database, PostGIS

Enter as the superuser over the socket — peer authentication, no password:

```bash
sudo -u postgres psql
```

Inspect the empty state to learn the introspection commands:

```sql
\conninfo      -- who and where you are connected as
\du            -- list roles
\l             -- list databases
```

Create a login role with a password:

```sql
CREATE ROLE gisuser WITH LOGIN CREATEDB;
\password gisuser
```

`LOGIN` makes it a usable account; `CREATEDB` lets it own and create databases.

> Set the password with `\password`, not plaintext `ALTER USER ... PASSWORD 'x'`. `\password` prompts twice and hashes client-side, so the plaintext never lands in your shell history or the server log.
{: .prompt-tip }

Create a database owned by that role, then switch into it:

```sql
CREATE DATABASE gis OWNER gisuser;
\c gis
```

> Extensions are enabled *per-database*. You must be connected to `gis` (via `\c`) before enabling PostGIS — enabling it while connected to `postgres` would install it in the wrong database.
{: .prompt-info }

Enable PostGIS and verify:

```sql
CREATE EXTENSION postgis;
SELECT postgis_full_version();
\dx
```

`\dx` lists installed extensions; `postgis_full_version()` confirms the geometry engine is live. Exit with `\q`.

## Phase 4 — Open It to Windows / QGIS

Ask the server for its exact config paths rather than guessing:

```bash
sudo -u postgres psql -c "SHOW config_file; SHOW hba_file;"
```

This returns `/etc/postgresql/18/main/postgresql.conf`{: .filepath} and `/etc/postgresql/18/main/pg_hba.conf`{: .filepath}. Make the server listen beyond the local socket:

```bash
sudo sed -i "s/^#\?listen_addresses.*/listen_addresses = '*'/" /etc/postgresql/18/main/postgresql.conf
grep "^listen_addresses" /etc/postgresql/18/main/postgresql.conf
```

The default is `localhost` (socket and 127.0.0.1 only); `'*'` listens on all interfaces so the TCP connection from Windows is accepted. Confirm it printed `listen_addresses = '*'`. Add an authentication rule for that TCP connection:

```bash
echo "host    all    all    127.0.0.1/32    scram-sha-256" | sudo tee -a /etc/postgresql/18/main/pg_hba.conf
```

Read the rule as: connection type `host` (TCP), any database, any user, from `127.0.0.1`, authenticate with a password. Under NAT networking, a Windows to WSL connection arrives as 127.0.0.1, so QGIS matches this rule. Apply the changes:

```bash
sudo systemctl reload postgresql          # pg_hba.conf changes need only a reload
sudo systemctl restart postgresql         # listen_addresses change needs a full restart
sudo ss -tlnp | grep 5432
```

> Rule of thumb: `pg_hba.conf`{: .filepath} changes need a **reload**; `postgresql.conf`{: .filepath} listen/port changes need a **restart**. A restart covers both. Expect a line showing `0.0.0.0:5432`.
{: .prompt-info }

## Phase 5 — Connect and Verify

Test the TCP path from inside WSL first — this isolates server configuration from Windows networking:

```bash
psql -h localhost -U gisuser -d gis -c "SELECT postgis_full_version();"
```

`-h localhost` forces the TCP-plus-password path (exercising Phase 4), unlike `sudo -u postgres` which used the socket. It prompts for the password from Phase 3. Success here means the server is fully configured. Confirm the networking mode:

```bash
wslinfo --networking-mode
```

> This **must** report `nat`. Mirrored mode has a documented loopback bug where host-to-WSL connections over 127.0.0.1 are refused even when the server is listening correctly. NAT mode forwards `localhost` reliably. If it reports `mirrored`, set `networkingMode=nat` in `%USERPROFILE%\.wslconfig`{: .filepath }, run `wsl --shutdown`, and reopen.
{: .prompt-warning }

In QGIS (Windows), open Data Source Manager (`Ctrl+L`) → PostgreSQL → New:

- Host `localhost` · Port `5432` · Database `gis` · SSL `disable`
- Authentication → Basic → Username `gisuser`, password from Phase 3
- Click **Test Connection**

> Use `localhost` as the host, not the raw WSL IP from `hostname -I`. Under NAT with localhost forwarding, `localhost` is the working address; the `172.x` NAT IP will not be reachable from Windows.
{: .prompt-warning }

## Phase 6 — Prove It End to End

Create a spatial table with one feature so QGIS has something to render:

```bash
psql -h localhost -U gisuser -d gis <<'SQL'
CREATE TABLE test_points (id serial PRIMARY KEY, name text, geom geometry(Point, 4326));
INSERT INTO test_points (name, geom)
VALUES ('Den Haag', ST_SetSRID(ST_MakePoint(4.3007, 52.0705), 4326));
SQL
```

`geometry(Point, 4326)` declares a point column in WGS84 (SRID 4326, standard lat/lon).

> `ST_MakePoint` takes **longitude first**, then latitude — `ST_MakePoint(lon, lat)`. Reversing them is the single most common PostGIS mistake and silently places features in the wrong hemisphere.
{: .prompt-warning }

In QGIS, expand the `gis` connection → `public` → `test_points` and add the layer. A point appears over The Hague. That single feature confirms server, PostGIS, authentication, networking, and QGIS are all correct simultaneously.

## Reference Commands

Keep these for day-to-day work and troubleshooting:

| Command | Purpose |
| --- | --- |
| `pg_lsclusters` | Cluster version, port, and status |
| `\du` / `\l` / `\dn` / `\dt` | List roles / databases / schemas / tables |
| `\conninfo` | Show the current session's user, database, and host |
| `\dx` | List installed extensions |
| `SHOW config_file;` / `SHOW hba_file;` | Locate the active config files |
| `sudo ss -tlnp \| grep 5432` | Confirm the server is listening on 5432 |
| `sudo systemctl reload postgresql` | Apply `pg_hba.conf` changes |
| `sudo systemctl restart postgresql` | Apply `postgresql.conf` listen/port changes |