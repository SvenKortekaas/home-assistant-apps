# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in this app, please report it privately by
creating a [security advisory](https://github.com/cyberjunky/home-assistant-apps/security/advisories)
on GitHub.

**Please do NOT open a public issue for security vulnerabilities.** This allows us to
address the issue before it becomes public knowledge.

Vulnerabilities in pgAdmin 4 itself should be reported to the
[pgAdmin project](https://www.pgadmin.org/support/). If you are unsure where an issue
belongs, report it here and we will coordinate.

## Security Considerations

### Access to pgAdmin

- pgAdmin is served through **Home Assistant ingress**. Only authenticated Home Assistant
  users can open the sidebar panel; pgAdmin's own login is bypassed for them.
- Ports 80 and 443 are **not published by default**. If you map one, pgAdmin becomes
  reachable without Home Assistant on that network. Only do this on a trusted LAN, never
  expose it to the internet. With `ssl: true` the direct port uses the certificate and
  key from the `ssl` folder (`certfile` and `keyfile`).
- The `leave_front_door_open` option disables Home Assistant authentication entirely.
  Leave it off unless you know why you need it.
- pgAdmin can run arbitrary SQL on every server registered in it. Anyone who can reach
  pgAdmin can do whatever the stored database credentials allow.

### Credential and data storage

- The `servers` option registers database connections **without passwords**. Passwords you
  enter and save in pgAdmin are stored, encrypted by pgAdmin, in its configuration
  database under `/data`.
- The app's data folder is included in Home Assistant backups, so backups of this app
  contain the pgAdmin configuration database and any saved database passwords. Treat
  them as sensitive.
- The TimescaleDB app from this repository is registered automatically over the internal
  Supervisor network; its password is not stored by this app.

### Start-up commands

- `system_packages` and `init_commands` run **as root inside the container** on every
  start. Only add packages and commands you trust.

### Container hardening

- pgAdmin is built from a **pinned release tag** of the upstream sources on a pinned base
  image, and the PostgreSQL client tools come from Alpine packages.
- The app uses the Supervisor's default AppArmor profile and runs the pgAdmin server
  process as root inside its container.

### Best Practices

1. **Keep Home Assistant updated** - Security patches are released regularly
2. **Install from the official repository** - `https://github.com/cyberjunky/home-assistant-apps`
3. **Leave ports 80 and 443 unmapped** unless you need direct LAN access, and keep
   `leave_front_door_open` off
4. **Use dedicated database roles** - Register servers with the least privileged role that
   does the job, rather than `postgres`
5. **Review the code** - As an open-source project, you can audit both this app and
   pgAdmin before use
6. **Secure your network** - Restrict access to your Home Assistant instance and your
   database servers
7. **Use strong authentication** - Enable multi-factor authentication for Home Assistant
   users

## Disclosure Timeline

When a vulnerability is confirmed:

1. We will assess the severity and impact
2. A fix will be prepared for the latest version
3. A new release will be published
4. A security advisory will be published on GitHub (with credit to the reporter if
   desired)

Thank you for helping keep this project secure!
