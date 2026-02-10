# MySQL schema (site content + leads)

This repo’s MySQL container uses `database/startup.sh` to start MySQL and create the `myapp` database + `appuser`.
There is no migration framework in this project; schema changes are applied manually.

## Connection

Authoritative connection command (from `db_connection.txt`):

```bash
mysql -u appuser -pdbuser123 -h localhost -P 5000 myapp
```

## Tables

### `services`
Stores the service cards/sections shown on the corporate website.

Columns:
- `slug` unique identifier used by the app (e.g., `bsp-development`)
- `title`, `short_desc`, `body`
- `icon` optional token for UI (e.g., `chip`, `network`)
- `sort_order`, `is_active`
- `created_at`, `updated_at`

Indexes:
- `uq_services_slug` unique
- `idx_services_active_sort (is_active, sort_order)`

### `testimonials`
Customer quotes for the testimonials section.

Columns:
- `client_name`, `client_title`, `company`
- `quote`
- `rating` optional 1–5
- `is_featured` for homepage emphasis
- `created_at`, `updated_at`

Indexes:
- `idx_testimonials_featured (is_featured, created_at)`
- `idx_testimonials_company (company)`

### `case_studies`
Project / case-study entries for showcasing work.

Columns:
- `slug` unique identifier
- `title`, `summary`, `industry`, `stack`
- `challenge`, `solution`, `results`
- `is_published`, `published_at`
- `created_at`, `updated_at`

Indexes:
- `uq_case_studies_slug` unique
- `idx_case_studies_published (is_published, published_at)`

### `leads`
Contact form submissions.

Columns:
- `name`, `email`, `company`, `phone`
- `service_slug` optional hint about requested service
- `message`
- `source` optional tracking label (e.g., `website`)
- `status` (`new` by default)
- `ip`, `user_agent` optional (for backend to populate)
- `created_at`, `updated_at`

Indexes:
- `idx_leads_created (created_at)`
- `idx_leads_status_created (status, created_at)`
- `idx_leads_email (email)`
- `idx_leads_service_slug (service_slug)`

## DDL (single-line statements)

These are the statements that were applied to the running MySQL instance (kept single-line to match repo tooling conventions):

```sql
CREATE TABLE IF NOT EXISTS services (id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY, slug VARCHAR(100) NOT NULL, title VARCHAR(200) NOT NULL, short_desc VARCHAR(500) NOT NULL, body TEXT NULL, icon VARCHAR(50) NULL, sort_order INT NOT NULL DEFAULT 0, is_active TINYINT(1) NOT NULL DEFAULT 1, created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP, updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP, UNIQUE KEY uq_services_slug (slug), KEY idx_services_active_sort (is_active, sort_order)) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE IF NOT EXISTS testimonials (id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY, client_name VARCHAR(120) NOT NULL, client_title VARCHAR(120) NULL, company VARCHAR(160) NULL, quote TEXT NOT NULL, rating TINYINT UNSIGNED NULL, is_featured TINYINT(1) NOT NULL DEFAULT 0, created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP, updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP, KEY idx_testimonials_featured (is_featured, created_at), KEY idx_testimonials_company (company)) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE IF NOT EXISTS case_studies (id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY, slug VARCHAR(120) NOT NULL, title VARCHAR(220) NOT NULL, summary VARCHAR(600) NOT NULL, industry VARCHAR(120) NULL, stack VARCHAR(255) NULL, challenge TEXT NULL, solution TEXT NULL, results TEXT NULL, is_published TINYINT(1) NOT NULL DEFAULT 1, published_at DATETIME NULL, created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP, updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP, UNIQUE KEY uq_case_studies_slug (slug), KEY idx_case_studies_published (is_published, published_at)) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE IF NOT EXISTS leads (id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY, name VARCHAR(160) NOT NULL, email VARCHAR(254) NOT NULL, company VARCHAR(200) NULL, phone VARCHAR(50) NULL, service_slug VARCHAR(100) NULL, message TEXT NOT NULL, source VARCHAR(120) NULL, status VARCHAR(40) NOT NULL DEFAULT 'new', ip VARCHAR(45) NULL, user_agent VARCHAR(255) NULL, created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP, updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP, KEY idx_leads_created (created_at), KEY idx_leads_status_created (status, created_at), KEY idx_leads_email (email), KEY idx_leads_service_slug (service_slug)) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## Seed data (as applied)

Notes:
- `services` and `case_studies` use `ON DUPLICATE KEY UPDATE` keyed by their `slug` to remain idempotent.
- `testimonials` are inserted as plain rows (if you rerun them you may get duplicates; this is acceptable for early-stage seed content unless you want a unique constraint).

```sql
INSERT INTO services (slug, title, short_desc, body, icon, sort_order, is_active) VALUES ('embedded-software', 'Embedded Software Development', 'Firmware, drivers, and real-time applications for Linux and RTOS platforms.', 'We build production-grade embedded software across MCU and Linux-class SoCs—boot to application. Services include firmware architecture, BSP integration, device drivers, performance optimization, and long-term maintenance.', 'chip', 10, 1) ON DUPLICATE KEY UPDATE title=VALUES(title), short_desc=VALUES(short_desc), body=VALUES(body), icon=VALUES(icon), sort_order=VALUES(sort_order), is_active=VALUES(is_active);

INSERT INTO services (slug, title, short_desc, body, icon, sort_order, is_active) VALUES ('bsp-development', 'BSP Development', 'Board bring-up, bootloader, kernel, device tree, and peripheral enablement.', 'From early bring-up to stable production releases: U-Boot, Linux kernel, device tree, and peripheral drivers. We streamline onboarding with reproducible builds and clear documentation.', 'cpu', 20, 1) ON DUPLICATE KEY UPDATE title=VALUES(title), short_desc=VALUES(short_desc), body=VALUES(body), icon=VALUES(icon), sort_order=VALUES(sort_order), is_active=VALUES(is_active);

INSERT INTO services (slug, title, short_desc, body, icon, sort_order, is_active) VALUES ('networking', 'Networking Development (L2/L3)', 'Protocol implementation, acceleration, and diagnostics for high-throughput systems.', 'We deliver robust networking components: switching and routing features, packet processing pipelines, performance profiling, and interoperability testing. Expertise across L2/L3 protocol development and integration.', 'network', 30, 1) ON DUPLICATE KEY UPDATE title=VALUES(title), short_desc=VALUES(short_desc), body=VALUES(body), icon=VALUES(icon), sort_order=VALUES(sort_order), is_active=VALUES(is_active);

INSERT INTO services (slug, title, short_desc, body, icon, sort_order, is_active) VALUES ('gateway-platforms', 'Gateway Platforms (RDK-B / OpenWRT / prplOS)', 'Feature development and integration for modern broadband and IoT gateways.', 'Hands-on experience delivering gateway features, TR-181 integrations, Wi-Fi and LAN stack work, and platform customization across RDK-B, OpenWRT, and prplOS ecosystems.', 'router', 40, 1) ON DUPLICATE KEY UPDATE title=VALUES(title), short_desc=VALUES(short_desc), body=VALUES(body), icon=VALUES(icon), sort_order=VALUES(sort_order), is_active=VALUES(is_active);

INSERT INTO services (slug, title, short_desc, body, icon, sort_order, is_active) VALUES ('qa-engineering', 'QA Engineering', 'Automation, CI validation, and release-quality test strategy.', 'We build lean, effective quality systems: test plans, automation frameworks, hardware-in-the-loop where needed, and CI/CD integration to keep releases stable and predictable.', 'check', 50, 1) ON DUPLICATE KEY UPDATE title=VALUES(title), short_desc=VALUES(short_desc), body=VALUES(body), icon=VALUES(icon), sort_order=VALUES(sort_order), is_active=VALUES(is_active);

INSERT INTO testimonials (client_name, client_title, company, quote, rating, is_featured) VALUES ('Director of Engineering', 'Platform Lead', 'Broadband Gateway OEM', 'Their team ramped up quickly, stabilized our BSP, and delivered networking features on schedule. Communication was crisp and the code quality was consistently high.', 5, 1);

INSERT INTO testimonials (client_name, client_title, company, quote, rating, is_featured) VALUES ('Program Manager', 'Connectivity Programs', 'Consumer Electronics Brand', 'They diagnosed a tricky L2/L3 interoperability issue and provided a clear fix with regression tests. Our throughput improved and field issues dropped immediately.', 5, 0);

INSERT INTO testimonials (client_name, client_title, company, quote, rating, is_featured) VALUES ('QA Manager', 'Release Quality', 'IoT Solutions Provider', 'They built an automation suite that cut our release validation time by more than half and gave us confidence in every OTA release.', 5, 0);

INSERT INTO case_studies (slug, title, summary, industry, stack, challenge, solution, results, is_published, published_at) VALUES ('rdk-b-gateway-stabilization', 'RDK-B Gateway Stabilization & Feature Delivery', 'Stabilized an RDK-B based gateway platform, improved boot reliability, and delivered L2/L3 networking enhancements with CI-backed regression coverage.', 'Broadband / Gateways', 'RDK-B, Linux, U-Boot, Yocto, C/C++, Python, iptables/nftables', 'Intermittent boot failures, unstable Wi-Fi and LAN behavior under load, and slow release cycles due to manual validation.', 'Implemented bootloader and kernel fixes, improved device-tree configuration, optimized packet path components, and introduced automated smoke + throughput tests integrated into CI.', 'Boot reliability improved significantly, throughput stabilized under stress, and validation time reduced while enabling faster, safer releases.', 1, NOW()) ON DUPLICATE KEY UPDATE title=VALUES(title), summary=VALUES(summary), industry=VALUES(industry), stack=VALUES(stack), challenge=VALUES(challenge), solution=VALUES(solution), results=VALUES(results), is_published=VALUES(is_published), published_at=VALUES(published_at);
```
