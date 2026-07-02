# JWT Attacker - Custom Action (Bambda Script)

---

## 📌 Overview

**JWT Attacker** is a Burp Suite custom action (Bambda script) that extracts a JSON Web Token from the
`Authorization: Bearer` header of the current Repeater request and runs the standard JWT tampering
test-suite against the endpoint — automatically analyzing each response against a baseline and routing
every attempt to the **Organizer** tab.

Unlike a blind fuzzer, it flags results: a tampered token that _should_ be rejected but comes back with
a baseline-like success response is highlighted **RED** as a likely broken-verification finding.

## ⚙️ Features

- ✅ Decodes and logs the JWT header and payload (alg, kid, exp, claims)
- 🧪 Runs seven JWT attack modules in one click (see below)
- 📊 Baseline comparison with automatic anomaly detection (status + length)
- 🎨 Colour-coded results in Organizer — RED = likely vulnerable, YELLOW = anomaly
- 🔓 Offline HMAC weak-secret cracking against a customizable wordlist
- 🗂 Every request/response stored in Burp Organizer with descriptive notes
- 🧾 Detailed execution logging in the output panel
- 🧩 Dependency-free — pure JDK crypto (`javax.crypto.Mac`, `java.util.Base64`)

## 🎯 Attack Modules

| #   | Module                                                                        | What it checks                                   |
| --- | ----------------------------------------------------------------------------- | ------------------------------------------------ |
| 1   | **alg:none** (`none`/`None`/`NONE`/`nOnE`)                                    | Server accepts an unsigned token                 |
| 2   | **Signature stripping**                                                       | Server accepts a token with no signature segment |
| 3   | **Signature tampering** (blank / flipped)                                     | Server does not verify the signature             |
| 4   | **Claim tampering** (`exp` bump + `role`/`admin`/`isAdmin`/`user` escalation) | Privilege escalation / expiry bypass             |
| 5   | **kid header injection** (path traversal, SQLi, OOB)                          | Key-lookup injection / SSRF via `kid`            |
| 6   | **Algorithm confusion** (RS256 → HS256)                                       | RSA public key accepted as an HMAC secret        |
| 7   | **Weak-secret cracking** (offline)                                            | HMAC signing secret is guessable                 |

## 🔧 Configuration (top of the script)

| Variable          | Purpose                                                                  |
| ----------------- | ------------------------------------------------------------------------ |
| `GLOBAL_DELAY_MS` | Delay between requests (default `200`)                                   |
| `hostname`        | Burp Collaborator / callback host used by the `kid` OOB payloads         |
| `KNOWN_SECRET`    | If set, forges validly-signed claim-tampered tokens                      |
| `PUBLIC_KEY_PEM`  | Server's RSA public key — required to run the RS256→HS256 confusion test |
| `SECRET_WORDLIST` | Wordlist for the offline HMAC secret cracker                             |

## 🚀 Usage

1. Clone or download this repository.
2. Open **Burp Suite**.
3. Navigate to **Extensions → Bambda Library**.
4. Click **Import** and select the Bambda script file: **JWTAttacker.bambda**.
5. Go to the **Repeater** tab and open a request that carries an `Authorization: Bearer <jwt>` header.
6. Open **Custom Actions**, click **Load**, and select **JWTAttacker**.
7. (Optional) Fill in `hostname`, `KNOWN_SECRET`, `PUBLIC_KEY_PEM`, or extend `SECRET_WORDLIST`.
8. **Run** the custom action and review the **Organizer** tab — RED entries are the highest priority.

> Notes:
>
> - Only the `Authorization: Bearer` header is inspected. Cookie/body-based JWTs are out of scope in this version.
> - The `kid` OOB payloads require you to watch your Collaborator/callback host for interactions.
> - This is not a replacement for a dedicated JWT scanner; it automates the common manual checks from Repeater.

## 👤 Author

**Sourav Banerjee** — [github.com/ns-sourav](https://github.com/ns-sourav)
