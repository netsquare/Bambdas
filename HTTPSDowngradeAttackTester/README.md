# HTTPS Downgrade Attack Tester - Custom Action (Bambda Script)
---
## 📌 Overview
**HTTPS Downgrade Attack Tester** is a Burp Suite custom action (Bambda script) designed to quickly test whether a target domain is vulnerable to HTTPS downgrade attacks by comparing HTTP and HTTPS responses.

The script automatically converts HTTPS requests into plain HTTP requests and analyzes the server behavior to determine whether HTTPS enforcement is properly implemented.

## ⚙️ Features
- ✅ Automatic HTTPS to HTTP downgrade testing
- 🔄 HTTP vs HTTPS response comparison
- 📊 Response analysis (status code, length, headers)
- 🧠 Built-in vulnerability detection logic, Categorizes results as Vulnerable, Secure, or Neutral
- 🗂 Automatically stores requests and responses in Burp Organizer
- 🧾 Detailed execution logging in the output panel
- ⚡ Supports both default(port 80) and custom ports

## 🚀 Usage
1. Clone or download this repository.
2. Open **Burp Suite**.
3. Navigate to **Extensions → Bambda Library**.
4. Click **Import** and select the Bambda script file: **HTTPSDowngradeAttackTester.bambda**.
5. Go to the **Repeater** tab.
6. Open **Custom Actions**.
7. Click **Load** and select: **HTTPSDowngradeAttackTester**.
8. **Run** the custom action to test for HTTPS downgrade vulnerabilities.

## 🔒 Recommendations
If a target is identified as vulnerable:
- Enforce HTTPS using `301` or `302` redirects.
- Enable HSTS (`Strict-Transport-Security`).
- Disable HTTP entirely whenever possible.

## 🤝 Contributing
Suggestions for improvements or bug fixes are welcome. Please submit pull requests or open issues on the GitHub repository.

## ⚠️ Disclaimer
This tool is intended for authorized security testing and educational purposes only. Use only against systems you own or have explicit permission to test.
