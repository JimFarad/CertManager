# Certificate Management System – Backend API

A robust certificate management backend built for issuing, signing, tracking, revoking, and auditing internal and ACME-based TLS certificates.  
This project supports OpenID Connect authentication, role-based permissions, and full API-driven integration.

---

## Overview

This backend service powers certificate operations for internal teams and infrastructure systems. It is intended to be deployed in secure environments and extended with a web frontend or CLI tooling.

Key capabilities include:

- Certificate issuance (private/internal or public/ACME)
- CSR and key generation
- OIDC authentication with group-based access control
- Full audit logging and certificate metadata tracking
- Admin dashboards and statistics endpoints
- Email notifications (e.g. for issuance)
- REST API for automation and frontend clients

The frontend is currently under development.

---

## Requirements

- PHP 8.1 or newer
- Composer
- OpenSSL CLI
- SQLite or MySQL
- Web server (PHP built-in server or Nginx/Apache)
- OIDC provider (e.g. Keycloak)

---

## Installation

### 1. Clone and install dependencies

```bash
git clone https://github.com/your-org/certificate-api.git
cd certificate-api
composer install
````

### 2. Directory structure

Ensure the web root is set to the `public/` directory. This is required for serving API requests correctly via `index.php`.

### 3. Configuration

Copy or create the following config files:

* `.env` – environment variables
* `ca.cnf` – OpenSSL configuration for your internal CA

### 4. Database Setup

#### MySQL

Create a database manually, then run the migration script to create tables and schema:

```bash
php scripts/migrate.php
```

#### SQLite (Default)

The system supports SQLite out of the box, requiring no additional database setup. The SQLite file will be created automatically in the configured storage path (default: `storage/`).

Ensure that the configured storage directory is writable by the web server or CLI user.


#### Notes on Database Drivers

* The `DB_DRIVER` environment variable in `.env` controls which driver is used (`mysql` or `sqlite`).
* MySQL requires standard setup and migration.
* SQLite requires only a writable storage directory.

---

## System Scripts

### 1. CRL Generation (for CA)

Used to create a CRL file that can be published with your internal root or intermediate CA.

```bash
php scripts/generate_crl.php
```

Run this on your offline or signing server after revoking certificates.

### 2. Expiration Monitor (cron job)

Run daily to detect and report certificates nearing expiration.

```bash
php scripts/expiration_check.php
```

Suggested cron:

```cron
0 2 * * * php /path/to/certificate-api/scripts/expiration_check.php
```

---

## API Reference

All API routes are served via `public/index.php` using path-based routing.

### Authentication

| Method | Endpoint           | Description                       |
| ------ | ------------------ | --------------------------------- |
| GET    | `/api/v1/auth`     | Generate OIDC login URL           |
| POST   | `/api/v1/token`    | Exchange OIDC code for tokens     |
| GET    | `/api/v1/logout`   | Get logout URL from OIDC provider |
| GET    | `/api/v1/userinfo` | Return current user session info  |

### Certificates

| Method | Endpoint                                    | Description                               |
| ------ | ------------------------------------------- | ----------------------------------------- |
| POST   | `/api/v1/certificates/private/generate`     | Generate a CSR and key (private/internal) |
| POST   | `/api/v1/certificates/public/request`       | Request a certificate via ACME            |
| POST   | `/api/v1/certificates/{id}/sign`            | Sign a CSR with internal CA               |
| POST   | `/api/v1/certificates/{id}/revoke`          | Revoke an issued certificate              |
| GET    | `/api/v1/certificates/search`               | List/filter certificates                  |
| GET    | `/api/v1/certificates/expiring`             | List soon-to-expire certificates          |
| GET    | `/api/v1/certificates/{id}`                 | View full metadata + logs                 |
| GET    | `/api/v1/certificates/{id}/download/{type}` | Download cert, key, csr, or chain         |

### Notifications

| Method | Endpoint                | Description                |
| ------ | ----------------------- | -------------------------- |
| POST   | `/api/v1/notifications` | Trigger email notification |

### System & Monitoring

| Method | Endpoint                     | Description                          |
| ------ | ---------------------------- | ------------------------------------ |
| GET    | `/api/v1/config`             | Return internal config info (secure) |
| GET    | `/api/v1/status`             | System health (uptime, PHP, DB)      |
| GET    | `/api/v1/stats/certificates` | Certificate statistics (admin only)  |
| GET    | `/api/v1/logs`               | Latest activity logs (if enabled)    |

---

## Permission Model

Permissions are granted based on group membership resolved via OIDC. Example mapping:

```yaml
group_permissions:
  view_public:   ["secgroup:certs:private:viewer"]
  view_private:  ["secgroup:certs:public:viewer"]
  issue_public:  ["secgroup:certs:public:issuer"]
  issue_private: ["secgroup:certs:private:issuer"]
  admin:         ["secgroup:certs::admin"]
```

Claims are mapped from OIDC tokens based on `OIDC_GROUPS_CLAIM`.

---

## Architecture

### Core Components

* `Utils.php`: utility and helper functions (auth, response, validation)
* `Database.php`: singleton PDO connection with query helpers
* `MailSender.php`: pluggable email delivery system
* `handlers/`: contains all route logic (modular, by function)

### Storage Directories

* `storage/cert/`: issued certificate files
* `storage/keys/`: private keys
* `storage/csr/`: certificate signing requests
* `storage/chain/`: CA chain files
* `storage/records/`: JSON metadata

---

## Frontend (Under Development)

A modern single-page app is in progress, built to consume this API.

### Planned Capabilities

* Authenticated dashboard
* Certificate request wizard
* Group-based visibility and management
* Tagging and metadata search
* Auto-renewable certificate interface

### Screenshots

*To be added when frontend is available.*

---

## Security Notes

* Ensure `/storage` and non-public files are not served via web.
* All API endpoints should be served over HTTPS.
* Never expose private key or CA signing keys through API.
* Limit access to `generate_crl.php` and similar tooling to trusted servers.

---

## Contribution

1. Fork this repo
2. Create a feature branch
3. Follow PSR-12 and use `composer format`
4. Submit pull request with description


## License

This project is licensed under the **MIT License with Commercial Use Conditions**.
(c) 2025 Dimitris Farantouris and contributors

You are free to use, modify, and distribute this software with **attribution** to the original author(s).

**Commercial use**, such as incorporating this software into products or services that are sold or generate revenue, requires a separate commercial license or agreement, which may include revenue sharing or licensing fees.

For questions or reporting security issues, contact: `d.farantouris@mak-net.net`
