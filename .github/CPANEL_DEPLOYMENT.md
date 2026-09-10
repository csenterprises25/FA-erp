# cPanel deployment

The `Deploy to cPanel` workflow deploys the `master` branch with explicit FTPS.
It can also be started manually from the repository's **Actions** tab. Manual
runs provide a `Deploy installer` option for the one-time application setup.

## GitHub environment and secrets

Create a GitHub environment named `production`. Add these environment secrets:

| Name | Value |
| --- | --- |
| `CPANEL_FTP_SERVER` | The cPanel server hostname covered by its TLS certificate, for example `server.example-host.com` (no `ftp://` prefix) |
| `CPANEL_FTP_USERNAME` | The FTP account restricted to this application's directory |
| `CPANEL_FTP_PASSWORD` | A strong, unique FTP password |

Optionally add the environment variable `APP_URL`, such as
`https://erp.example.com/`. The workflow uses it for the environment link and a
post-deployment HTTP check.

The optional environment variable `CPANEL_FTP_PORT` changes the FTPS port. It
defaults to `21` when the variable is not configured.

The workflow assumes explicit FTPS on port 21 with strict certificate
validation. If the hosting provider specifies another port, set the
`CPANEL_FTP_PORT` environment variable. Do not change the protocol to
unencrypted `ftp`.

## One-time cPanel setup

1. Point the domain or subdomain at the deployment directory and enable HTTPS.
2. Create a MariaDB/MySQL database and a database user with access to it.
3. Create an FTP account whose home is the application's deployment directory.
   The workflow deploys directly to that FTP home (`./`).
4. In GitHub, open **Actions → Deploy to cPanel → Run workflow**, enable
   **Deploy installer**, and start the workflow. Then open `/install/` on the
   application domain and run the FrontAccounting installer to create
   `config.php`, `config_db.php`, the database schema, and the first
   administrator.
5. Remove or rename the remote `install` directory after setup, as recommended
   by FrontAccounting.
6. Confirm the `company` and `tmp` runtime directories are writable by PHP.
7. Take an off-server database backup before the first automated deployment.

The workflow deliberately preserves the live configuration, extension
registry, attachments, backups, images, generated reports, PDFs, and caches.
These contain installation-specific or user-generated data and must not be
stored in GitHub.

Normal pushes never upload the `install` directory. Only a manual run with the
installer option enabled uploads it. Remove the remote `install` directory as
soon as setup is complete.

Mail server details are configured on the server only if outgoing email is
needed. Database, mail, domain, and FrontAccounting administrator credentials
are not FTP workflow secrets.
