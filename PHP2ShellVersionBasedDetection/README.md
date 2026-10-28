# PHP2Shell - Version Based Detection - Burp Suite Custom Action (Bambda Script)

---

## 📌 Overview

**PHP2Shell - Version Based Detection** is a Burp Suite custom action (Bambda script) that passively inspects selected HTTP response headers for a disclosed PHP version and compares it with configured version thresholds associated with PHP2Shell advisories.
- CVE-2026-109618 ([GHSA-w8hw-w9fx-j79w](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w))
- CVE-2026-109619 ([GHSA-6wmf-4rv5-p3rp](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp))

The script does **not** exploit the target or confirm that a vulnerable code path is reachable. It uses version information exposed in response headers to identify versions that may require investigation.

---

## ⚙️ Features

* 🔎 **Passive Header Inspection:** Checks `X-Powered-By`, `Server`, and `X-PHP-Version` response headers.
* 🧩 **Version Parsing:** Recognizes version strings such as `PHP/8.2.34` and `PHP 8.5.12`.
* ⚠️ **Legacy Branch Reporting:** Identifies PHP versions older than the configured supported-branch range for separate vendor-support review.
* 🛡️ **Threshold-Based Classification:** Compares full major, minor, and patch versions against the configured fixed-version thresholds.
* 🧾 **Remediation Guidance:** Includes an upgrade recommendation for versions below the configured threshold.
* 🧯 **Basic Error Handling:** Checks for a missing HTTP response and handles numeric parsing errors.
* 🧪 **Non-Intrusive Detection:** Does not send additional requests or payloads; it examines the response already available to the custom action.

---
### 🕳️ Affected / fixed versions

| **Branch** | **Affected** | **Fixed** |
| ---------- | ----------------------- | ------------------------- |
| 8.2 | < 8.2.35 | 8.2.35 |
| 8.3 | < 8.3.36 | 8.3.36 |
| 8.4 | < 8.4.27 | 8.4.27 |
| 8.5 | < 8.5.12 | 8.5.12 |
| < 8.2 | EOL - No upstream patch | Verify vendor backports |
---

## 🧭 Detection Logic

The script checks the following response headers:

* `X-Powered-By`
* `Server`
* `X-PHP-Version`

It searches their values for a PHP version and classifies a complete version according to the rules below.

| Detected version | Script classification | Guidance |
| :--- | :--- | :--- |
| PHP earlier than 8.2 | `Legacy PHP Branch` | Verify the distribution/vendor's support status and any backported fixes. |
| PHP 8.2.x below `8.2.35` | `Vulnerable PHP Version Detected (PHP2Shell)` | Upgrade to `8.2.35` or a verified vendor-patched equivalent. |
| PHP 8.3.x below `8.3.36` | `Vulnerable PHP Version Detected (PHP2Shell)` | Upgrade to `8.3.36` or a verified vendor-patched equivalent. |
| PHP 8.4.x below `8.4.27` | `Vulnerable PHP Version Detected (PHP2Shell)` | Upgrade to `8.4.27` or a verified vendor-patched equivalent. |
| PHP 8.5.x below `8.5.12` | `Vulnerable PHP Version Detected (PHP2Shell)` | Upgrade to `8.5.12` or a verified vendor-patched equivalent. |
| PHP 8.2.35+, 8.3.36+, 8.4.27+, or 8.5.12+ within the respective branch | `At or Above Known Fixed Threshold` | Confirm that the installed vendor build includes the applicable security fixes. |
| Other versions not covered by these rules | `PHP Version Not Classified` | Check the relevant PHP security advisory and vendor package information. |
| Version disclosed without a patch number | `Incomplete PHP version disclosed` | Obtain the full version before comparing patch thresholds. |

> **Important:** The thresholds above are the values configured in this script. Confirm them against the official PHP security advisory before relying on them. A response header is only a version fingerprint; it does not prove that the runtime is vulnerable or patched. Vendors may backport fixes without changing the upstream version string.

---

## 🚀 Usage

1. Clone or download this repository.
2. Open **Burp Suite**.
3. Navigate to **Extensions → Bambda Library**.
4. Click **Import** and select the Bambda script file: **PHP2ShellVersionBasedDetection.bambda**.
5. Open a request that has an HTTP response available (for example, in **Repeater**).
6. Open **Custom Actions**.
7. Click **Load** and select: **PHP2ShellVersionBasedDetection**.
8. **Run** the action.
9. Review the **Output Panel** for any version classifications and remediation guidance.
---

## 🧪 Example Output

A version below a configured threshold may produce output similar to:

```text
----- PHP2Shell - Version Based Detection -----
Vulnerable PHP Version Detected (PHP2Shell)
    Version: 8.5.11
    Header: Server
    Fix: Upgrade to PHP 8.5.12 or the vendor's equivalent patched build.
```

A version at or above the configured threshold may produce:

```text
----- PHP2Shell - Version Based Detection -----
[INFO] At or Above Known Fixed Threshold
    Version: 8.5.12
    Header: Server
    Note: Verify vendor backports and applicable security advisories.
```

Exact output depends on the response headers and the version rules in the script.

---

## ⚠️ Limitations and False Positives

* The script only examines the three headers listed above. If PHP is not disclosed in those headers, the script may not detect it.
* A server header may contain a PHP version string that does not reliably represent the PHP runtime serving the application.
* Version strings can be hidden, altered, or customized.
* The script's legacy-branch check is not proof of PHP2Shell vulnerability.
* A version at or above a configured threshold is not proof that a vendor build contains the relevant fix.
* The script does not test exploitability, reachability, or vulnerable application behavior.
* In the current implementation, headers with no matching PHP version are silently skipped; no “PHP not detected” summary is printed.

---

### References
- CVE-2026-109618 ([GHSA-w8hw-w9fx-j79w](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w))
- CVE-2026-109619 ([GHSA-6wmf-4rv5-p3rp](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp))
- [PHP2Shell](https://www.php2shell.com/)
- [PHP Patch - Commit 3584a98](https://github.com/php/php-src/commit/3584a98bda)
- [PHP Security advisory](https://www.php.net/archive/2026.php#2026-10-21-5)
---

## ⚠️ Disclaimer

This tool is intended for authorized security testing and educational purposes only. Run it only against systems you own or have explicit permission to assess. The output is based on passive version disclosure and should be validated against authoritative advisories and vendor package information.

---

## 👤 Author

Nikunj Sapara — [github.com/nikunjsapara](https://github.com/nikunjsapara/)
