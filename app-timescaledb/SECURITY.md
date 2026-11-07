# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in this app, please report it privately by
creating a [security advisory](https://github.com/cyberjunky/home-assistant-apps/security/advisories)
on GitHub.

**Please do NOT open a public issue for security vulnerabilities.** This allows us to
address the issue before it becomes public knowledge.

Vulnerabilities in PostgreSQL itself should be reported to the
[PostgreSQL project](https://www.postgresql.org/support/security/), and vulnerabilities in
TimescaleDB or its extensions to [Timescale](https://github.com/timescale/timescaledb/security).
If you are unsure where an issue belongs, report it here and we will coordinate.

## Security Considerations

### Access to the database

- The app has no web interface. Other apps on the same Home Assistant host, such as the
  pgAdmin 4 app from this repository, reach it over the internal Supervisor network.
- Port 5432 is **not published by default**. Mapping it exposes PostgreSQL to your LAN.
  Only do this on a trusted network, never expose it to the internet.
- The `postgres` superuser starts with the password `homeassistant`. **Change it right
  after the first start**, with the `postgres_password` option or `ALTER USER`.
- The default `pg_hba.conf` allows local peer logins and password logins from the
  network. Use the `pg_hba_config` option to restrict addresses and to require
  `scram-sha-256`. A wrong rule set can lock you out, so keep one working rule.
- Database roles can read and write the Home Assistant `share`, `backup` and `media`
  folders through server-side `COPY` and file functions, because those folders are
  mapped read-write into the container for imports and exports.

### Backups and data storage

- The raw PostgreSQL data directory under `/data` is **excluded** from Home Assistant
  backups. Instead, a full `pg_dumpall` dump is written to the data folder before each
  backup and stored with it. That dump contains every database and the password hashes
  of all roles. Treat backups of this app as sensitive.
- While the app runs, temporary files live in RAM (`tmpfs`).

### Telemetry

- TimescaleDB sends **anonymous usage statistics** to Timescale by default
  (`timescaledb.telemetry: basic`). Set it to `off` if nothing may leave your network.
  No other data is sent anywhere.

### Start-up commands

- `system_packages` and `init_commands` run **as root inside the container** on every
  start. Only add packages and commands you trust.

### Container hardening

- PostgreSQL runs as the unprivileged **`postgres`** user; only the service supervisor
  runs as root.
- PostgreSQL, TimescaleDB and every extension (PostGIS, pgvector, pg_cron, pgAgent,
  TimescaleDB Toolkit, system_stats) are built from **pinned versions** declared at the
  top of the Dockerfile, on a pinned base image.
- The app uses the Supervisor's default AppArmor profile.

### Best Practices

1. **Keep Home Assistant updated** - Security patches are released regularly
2. **Install from the official repository** - `https://github.com/cyberjunky/home-assistant-apps`
3. **Change the default password** immediately and use a strong one
4. **Leave port 5432 unmapped** unless you need LAN access, and tighten `pg_hba_config`
   when you do
5. **Use dedicated roles** - Give each application its own role with only the privileges
   it needs, instead of sharing `postgres`
6. **Review the code** - As an open-source project, you can audit this app before use
7. **Secure your network** - Restrict access to your Home Assistant instance
8. **Use strong authentication** - Enable multi-factor authentication for Home Assistant
   users

## Disclosure Timeline

When a vulnerability is confirmed:

1. We will assess the severity and impact
2. A fix will be prepared for the latest version
3. A new release will be published
4. A security advisory will be published on GitHub (with credit to the reporter if
   desired)

Thank you for helping keep this project secure!
