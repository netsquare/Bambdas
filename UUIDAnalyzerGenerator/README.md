# UUID Analyzer & Generator - Burp Suite Custom Action (Bambda Script)
![Version](https://img.shields.io/badge/version-1.0.0-blue) ![Burp Suite](https://img.shields.io/badge/Burp_Suite-Bambda-FF6633)

## 📌 Overview
**UUID Analyzer & Generator** is a Burp Suite Repeater custom action that serves a dual purpose: it automatically generates RFC 9562 structurally valid UUID test values to replace predefined placeholders in your requests, and it passively scans the HTTP request & response for UUID-related vulnerabilities (such as information disclosure and predictable tokens).

This is highly useful for penetration testers looking to test UUID-based IDORs, session forgery, or bypass strict backend format validations dynamically.

## 📖 Table of Contents
- [Features](#-features)
- [Supported Placeholders](#-supported-placeholders)
- [Limitations & Notes](#-limitations--notes)
- [Troubleshooting](#-troubleshooting)
- [Usage](#-usage)
- [Example](#-example)

## ⚙️ Features
- ✅ **Dynamic UUID Generation:** Generates secure random (v4) and structurally valid (v1, v6, v7, v8) UUIDs.
- ✅ **Deterministic Hashing:** True MD5 (v3) and SHA-1 (v5) generation using custom seed strings.
- 🔍 **Passive Vulnerability Scanner:** Analyzes requests and responses for UUIDs and identifies:
  - **Nil & Max UUIDs** (Risk of auth bypass or bounds issues).
  - **Predictable Tokens** (v1, v2).
  - **Information Disclosure** (Extracts MAC addresses, Local UIDs, and exact Generation Timestamps natively).
  - **Smart Filtering:** Automatically ignores standard RFC predefined namespaces (like NameSpace_DNS, NameSpace_URL, NameSpace_OID, NameSpace_X500) and explicitly excludes the UUIDs it just generated from its own scan results to prevent false positives.
- 🎯 **Burp Dashboard Integration:** Passive scanner findings are injected directly into the Burp Suite **Target Site Map & Issue Dashboard** as Burp Site Map Audit Issues created by the custom action, complete with severity ratings and remediation advice for easy reporting.
- 🧠 **Smart Deduplication Engine:** Prevents dashboard flooding during heavy testing. The scanner intelligently deduplicates issues by analyzing the exact endpoint path and the underlying vulnerability (e.g., logging a new issue only if a *different* MAC address is leaked on the same URL).
- 🛡️ **Fail-Safe Execution & Crash Protection:** 
  - **Smart Input Validation:** Safely intercepts empty request panes, missing Host/Port configurations, and malformed HTTP requests, preventing unhandled `MalformedRequestException` errors and keeping the output console clean.
  - **Environment Adaptability:** Includes a seamless fallback to manual bitwise generation for UUIDv7 if run on older Java environments lacking modern standard library support.
  - **Input Safety:** Safely catches and logs malformed namespaces without breaking the tool.
- 🔄 **In-Memory Replacement:** Replaces placeholders directly in the Repeater request pane in a single optimized pass.
- 🧾 **Detailed Logging:** Outputs highly structured, formatted vulnerability reports and generation logs directly to the output console. 
- 🔗 **State Retention for Complex Workflows:** Using the exact same placeholder (e.g., `<UUIDv4>`) multiple times in a single request generates the *same* UUID for all instances. This allows you to easily map matching identifiers across JSON bodies, headers, and URL parameters in a single execution.
- 🧲 **Support for Special Cases:** Using the placeholders `<UUID:nil>` & `<UUID:max>` will generate and replace the RFC 9562 special-case Nil & Max UUIDs.
- 🔢 **Universal Indexing for Distinct UUIDs:** Append an index suffix (e.g., `_1`, `_2`) to any non-deterministic placeholder (e.g., `<UUIDv4_1>`, `<UUIDv4_2>`, `<UUIDv7_1>`, `<UUIDv7_2>`) to generate a distinct, unique UUID for each specific index within a single request. *(Note: While index suffixes are syntactically accepted on `v3` and `v5` placeholders, deterministic versions always hash the `namespace:name` pair—producing distinct UUIDs for `v3`/`v5` requires changing the `namespace` or `name` seed).*
- 🔠 **Case Sensitivity & Output Casing:** Placeholders are strictly case-sensitive and control output casing. Using uppercase `<UUID...>` generates an **uppercase** UUID, while lowercase `<uuid...>` generates a **lowercase** UUID (except `<UUID:nil>`, which remains all zeros). Using `<UUIDv4>` and `<uuidv4>` in the same request will be treated as two distinct placeholders and will generate two *different* UUIDs.

## 🧩 Supported Placeholders
| Placeholder | Description |
|---|---|
| `<UUID:nil>` | Replaces with the RFC 9562 special-case Nil (`00000000-0000-0000-0000-000000000000`) placeholder. |
| `<UUID:max>` / `<uuid:max>` | Replaces with the RFC 9562 special-case Max (`FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF` for `<UUID:max>`, or lowercase `ffffffff-ffff-ffff-ffff-ffffffffffff` for `<uuid:max>`) placeholder. |
| `<UUID>` | Auto-detects the first RFC UUID version found in the request/response and replaces it, defaulting to v4 if no UUIDs are detected. **Note:** If v3 or v5 is detected, the script will log an error to the console; you must manually replace it with the explicit `<UUIDv3:namespace:name>` or `<UUIDv5:namespace:name>` syntax). |
| `<UUIDv1>`, `<UUIDv6>`, `<UUIDv7>`, `<UUIDv8>` | Replaces with a structurally valid "fake" UUID of the requested version using bitwise manipulation. |
| `<UUIDv3:namespace:name>` | Replaces with a true deterministic MD5-based v3 UUID. Requires explicit `namespace:name` syntax. **Note:** The `namespace` can be a strictly formatted UUID, or one of the built-in RFC aliases: `DNS`, `URL`, `OID`, or `X500`. You can leave the name empty to hash the namespace root (e.g., `<UUIDv3:DNS:admin>`, `<UUIDv3:URL:>`,`<UUIDv3:6ba7b810-9dad-11d1-80b4-00c04fd430c8:admin>` , `<UUIDv3:6ba7b810-9dad-11d1-80b4-00c04fd430c8:>`). |
| `<UUIDv4>` | Replaces with a cryptographically secure, random v4 UUID. |
| `<UUIDv5:namespace:name>` | Replaces with a true deterministic SHA-1-based v5 UUID. Requires explicit `namespace:name` syntax. **Note:** The `namespace` can be a strictly formatted UUID, or one of the built-in RFC aliases: `DNS`, `URL`, `OID`, or `X500` (e.g., `<UUIDv5:URL:https://example.com>`,`<UUIDv5:6ba7b810-9dad-11d1-80b4-00c04fd430c8:admin>`). |
| `<UUIDvX_Y>` | Indexed placeholders used to generate *distinct* UUIDs within a single request for random/time-based versions (e.g., `<UUID_1>`, `<UUIDv1_2>`, `<UUIDv4_1>`). If used with deterministic versions (`<UUIDv3_1:namespace:name>`, `<UUIDv5_1:namespace:name>`), the index is ignored during hashing and the output depends strictly on `namespace:name`. |

## ⚠️ Limitations & Notes
* **UUIDv3 / UUIDv5 Name Encoding:** The Bambda encodes the supplied name component as UTF-8 bytes. Namespace-specific canonicalization is not automatically performed.
* **UUIDv2 Excluded:** UUIDv2 generation is excluded because its DCE Security definition is outside RFC 9562 and requires local UID/GID bindings. It is still supported by the passive scanner.
* **Time-Based OPSEC:** Time-based placeholders (v1, v6) generate mathematically valid timestamps for the *current* execution time. However, the Node ID (MAC Address) is securely randomized using the multicast bit to prevent leaking the penetration tester's physical hardware address.
* **"Fake" Structural Generation:** Java's standard UUID class only includes built-in generators for v3 and v4. To generate RFC 9562-compliant UUIDs for v1, v6, v7, and v8, this script constructs the exact bit layouts (60-bit/48-bit timestamps, version bits, RFC variant bits, and CSPRNG payload/multicast node bits) directly. **For UUIDv7**: The script attempts to generate it natively using modern Java's `UUID.ofEpochMillis()` first, and only falls back to "fake" bitwise generation if the underlying Java environment is too old and throws an error.
* **Content-Length Header:** Because replacing placeholders changes the size of your request body, ensure that Burp's "Update Content-Length" setting is enabled in the Repeater top menu so your request doesn't fail.

> **Note:** Because the time-based versions (v1, v6, v7) use manipulated Node IDs, they will pass Regex and basic format checks, but may fail strict cryptographic validation on backends that verify MAC address manufacturer codes for v1 & v6.

## 🐛 Troubleshooting
| Issue | Solution |
|-------|----------|
| Placeholders not replaced | Check Custom Actions -> Load selected script |
| Null Pointer Error | Ensure target URL configured in Repeater |
| No scan results appearing | Generated UUIDs are intentionally excluded |
| Server returns 400 Bad Request or times out. | Ensure "Update Content-Length" is checked in the Repeater top menu. |

## 🚀 Usage
1. Clone or download this repository.
2. Open **Burp Suite**.
3. Navigate to **Extensions → Bambda Library**.
4. Click **Import** and select the Bambda script file: **UUIDAnalyzerGenerator.bambda**.
5. Open the **Repeater** tab.
6. Add one of the supported placeholders inside the request (e.g., `<UUIDv1>`).
7. Open **Custom Actions**.
8. Click Load and select: **UUID Analyzer & Generator**.
9. **Run** the custom action.
10. The placeholders will automatically be replaced with generated UUIDs, and the passive scan results will print in the output console.

## 🧪 Example
*(Notice how the lowercase `<uuid>` placeholder generates a lowercase string, while the uppercase `<UUIDv4>` generates an uppercase string!)*

### Before
```http
POST /api/users HTTP/1.1
Host: example.com

{
  "uid": "132a66aa-b1cb-11f1-8042-325096b39f47",
  "Uuid": "<uuid>",
  "new_user_id": "<UUIDv4>",
  "tracking_id": "<UUIDv5:6ba7b810-9dad-11d1-80b4-00c04fd430c8:admin>",
  "item_1": "<UUIDv4_1>",
  "item_2": "<UUIDv4_2>"
}
```

### After
```http
POST /api/users HTTP/1.1
Host: example.com

{
  "uid": "132a66aa-b1cb-11f1-8042-325096b39f47",
  "Uuid": "708106b0-df39-11f1-aa8d-7132b0b5e4f5",
  "new_user_id": "DFE53785-11D6-4FCE-A245-C2784B926F9F",
  "tracking_id": "BACE0701-15E3-5144-97C5-47487D543032",
  "item_1": "CEBC3C16-AFF0-4214-BD46-250C39BA753C",
  "item_2": "2CFC5912-5B8B-49D4-B6B1-C2594386C8B1"
}
```

### Console Output
```text
----- UUID Analyzer & Generator Started -----
[-] Replaced <uuid> (Version: 1) with: 708106b0-df39-11f1-aa8d-7132b0b5e4f5
[-] Replaced <UUIDv4> (Version: 4) with: DFE53785-11D6-4FCE-A245-C2784B926F9F
[-] Replaced <UUIDv5:6ba7b810-9dad-11d1-80b4-00c04fd430c8:admin> (True Hash) with: BACE0701-15E3-5144-97C5-47487D543032
[-] Replaced <UUIDv4_1> (Version: 4) with: CEBC3C16-AFF0-4214-BD46-250C39BA753C
[-] Replaced <UUIDv4_2> (Version: 4) with: 2CFC5912-5B8B-49D4-B6B1-C2594386C8B1

[-] Passive Analysis Results (Generated placeholder UUIDs were excluded):
[INFO] Total UUIDs Detected: 1
[INFO] New Issues Created: 1
[-] UUID: 132a66aa-b1cb-11f1-8042-325096b39f47 (v1)
    Severity: [LOW]
    Issue:    Predictable token & Information Disclosure
    Location: Request JSON Parameter [uid]
    Leak:     MAC Address / Node ID -> 32:50:96:b3:9f:47
    Leak:     Timestamp   -> 2026-09-16T12:35:18.321Z
    Leak:     Clock Sequence -> 66

[INFO] Script executed in 5ms
----- UUID Analyzer & Generator Completed -----
```

## 🤝 Contributing
Suggestions for improvements or bug fixes are welcome. Please submit pull requests or open issues on the GitHub repository.

## ⚖️ Disclaimer
This script is designed strictly for educational purposes and authorized penetration testing. The author(s) are not responsible for any misuse, damage, or unauthorized access caused by utilizing this script against systems you do not have explicit permission to test.

## 👤 Author
Nikunj Sapara — [github.com/nikunjsapara](https://github.com/nikunjsapara/)
