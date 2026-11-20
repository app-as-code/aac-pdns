<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/icon-dark.svg">
    <img src="assets/icon.svg" alt="PowerDNS logo" width="128">
  </picture>
</p>

<h1 align="center">aac-pdns</h1>

<p align="center">Self-hosted deployment definition for PowerDNS Authoritative DNS with the PowerDNS-Admin web interface.</p>

<p align="center">
  <a href="https://github.com/PowerDNS-Admin/PowerDNS-Admin"><img alt="Upstream" src="https://img.shields.io/badge/upstream-PowerDNS--Admin-blue?logo=github"></a>
  <img alt="Image" src="https://img.shields.io/badge/image-pschiffe%2Fpdns--admin%3Alatest-2496ED?logo=docker&logoColor=white">
  <a href="https://github.com/PowerDNS-Admin/PowerDNS-Admin/blob/master/LICENSE"><img alt="License" src="https://img.shields.io/badge/license-MIT-green"></a>
</p>

## About

This service combines an authoritative DNS server with a web interface to manage its zones.
PowerDNS Authoritative Server answers DNS queries for the zones it hosts and reads its zone
data from a MySQL-compatible database. PowerDNS-Admin is the web front end that lets you
create and edit zones and records through the PowerDNS HTTP API.

The deployment consists of four containers: the PowerDNS-Admin web interface (main service),
the PowerDNS authoritative server, and a MariaDB database that backs the authoritative server
and the web interface.

Key features (as provided by upstream):

- Web-based management of zones and records
- Authoritative DNS backed by a MySQL database
- DNSSEC signing support (enabled in this deployment for the MySQL backend)
- PowerDNS HTTP API for automation

## Upstream

- Web interface: <https://github.com/PowerDNS-Admin/PowerDNS-Admin>
- DNS server: <https://www.powerdns.com>
- Web interface container image: `pschiffe/pdns-admin:latest`
- DNS server container image: `pschiffe/pdns-mysql:4.9`
- Database container image: `mariadb:10.11` (deployed as `aac-pdns-db`)
- Licenses: PowerDNS-Admin is MIT-licensed (upstream project). PowerDNS Authoritative Server is
  licensed under GPL-2.0 (upstream project).

## Quick start (Docker Compose)

The Compose file is rendered from `service.yml` by the App-as-Code template engine. Do not
edit the rendered output. For a manual run, create the external networks that the rendered
Compose file references, then start the stack:

```bash
docker compose up -d
```

Secrets are not stored in this repository. They are injected by the deployment pipeline
from the controller configuration. Do not commit them.

## Configuration

Secrets are declared in `service.yml` as `CHANGE_ME_IN_CONTROLLER` placeholders and are
provided per deployment (controller configuration or GitLab CI variables).

Main service (web interface) environment:

| Variable | Default | Description |
|---|---|---|
| `PDNS_ADMIN_SQLA_DB_TYPE` | `mysql` | Database type of the web interface |
| `PDNS_ADMIN_SQLA_DB_HOST` | `aac-pdns-db` | Database host |
| `PDNS_ADMIN_SQLA_DB_PORT` | `3306` | Database port |
| `PDNS_ADMIN_SQLA_DB_USER` | `pdns` | Database user (`vars.PDNS_DB_USER`) |
| `PDNS_ADMIN_SQLA_DB_PASSWORD` | `CHANGE_ME` (secret) | Database password (`PDNS_DB_PASSWORD`) |
| `PDNS_ADMIN_SQLA_DB_NAME` | `pdns` | Database name (`vars.PDNS_DB_NAME`) |
| `PDNS_API_URL` | `http://aac-pdns-server:8081/` | URL of the PowerDNS HTTP API |
| `PDNS_API_KEY` | `CHANGE_ME` (secret) | API key for the PowerDNS HTTP API |
| `PDNS_VERSION` | `4.9.3` | PowerDNS version the web interface targets |
| `PDA_BIND_ADDRESS` | `0.0.0.0` | Bind address inside the container |
| `PDA_BIND_PORT` | `8080` | Bind port inside the container |
| `PDA_SECRET_KEY` | `CHANGE_ME` (secret) | Flask secret key |
| `PDA_SALT` | `CHANGE_ME` (secret) | Password salt |
| `PDA_PROXY` | `True` | Run behind a reverse proxy |
| `UPLOAD_FOLDER` | `/opt/powerdns-admin/upload` | Upload path inside the container |
| `TZ` | `Europe/Berlin` | Time zone |

Authoritative DNS server environment (`aac-pdns-server`):

| Variable | Default | Description |
|---|---|---|
| `PDNS_gmysql_host` | `aac-pdns-db` | Database host |
| `PDNS_gmysql_port` | `3306` | Database port |
| `PDNS_gmysql_user` | `pdns` | Database user |
| `PDNS_gmysql_password` | `CHANGE_ME` (secret) | Database password |
| `PDNS_gmysql_dbname` | `pdns` | Database name |
| `PDNS_gmysql-dnssec` | `yes` | Store DNSSEC data in the database |
| `PDNS_api` / `PDNS_api_key` | `yes` / `CHANGE_ME` (secret) | Enable the HTTP API and set its key |
| `PDNS_webserver_password` | `CHANGE_ME` (secret) | Password of the PowerDNS built-in web server |
| `PDNS_webserver` / `PDNS_webserver_address` | `yes` / `0.0.0.0` | Built-in web server listener |
| `PDNS_primary` | `yes` | Run as primary |
| `PDNS_default_ttl` | `600` | Default TTL |
| `PDNS_default-ksk-algorithm` / `PDNS_default-zsk-algorithm` | `ecdsa256` | DNSSEC key algorithms |
| `PDNS_version_string` | `anonymous` | Hide the version in responses |

Database container (`aac-pdns-db`, MariaDB):

| Variable | Default | Description |
|---|---|---|
| `MYSQL_ROOT_PASSWORD` | `CHANGE_ME` (secret) | Root password |
| `MYSQL_DATABASE` | `pdns` | Database created on first start |
| `MYSQL_USER` | `pdns` | Application user |
| `MYSQL_PASSWORD` | `CHANGE_ME` (secret) | Application user password |

Ports:

| Port | Protocol | Purpose |
|---|---|---|
| `8882` (host) -> `8080` | TCP | Web interface (routed through Traefik when enabled) |
| `53` | TCP and UDP | DNS queries, bound to the host LAN address only |
| `8081` | TCP | PowerDNS HTTP API |

Volumes:

| Mount | Description |
|---|---|
| `upload` -> `/opt/powerdns-admin/upload` | PowerDNS-Admin uploads |
| `keys` -> `/var/lib/powerdns` | DNSSEC and PowerDNS state of the authoritative server |
| `db` -> `/var/lib/mysql` | MariaDB data (database dependency) |

## Deploy with App-as-Code

1. Edit `service.yml`. It is the single source of truth for image, ports, environment,
   volumes, dependencies and per-stage overrides. Do not edit the rendered output.
2. Pushing to `main` (or a configured stage branch) runs the service pipeline, which renders
   the manifests with `aac-template-engine` and triggers the deployment.
3. Releases of this definition are published under <https://github.com/app-as-code/aac-pdns>.

## Credits and license

PowerDNS-Admin is developed by the PowerDNS-Admin project and licensed under MIT. PowerDNS
Authoritative Server is developed by PowerDNS and licensed under GPL-2.0. This repository only
contains the deployment definition for these components and is not a fork of either application.
The icon is provided by the selfh.st icon collection; see [`assets/ICON_SOURCE.md`](assets/ICON_SOURCE.md).
