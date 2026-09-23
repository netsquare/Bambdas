# Bad HTTP Request Fuzzer - Burp Suite Custom Action (Bambda Script)

---

## 📌 Overview

**Bad HTTP Request Fuzzer** is a robust Burp Suite custom action (Bambda script) designed to test web server resilience, HTTP parser edge cases, and error-handling behavior.

By taking a baseline HTTP request and generating **56 intentionally malformed variations**, this script helps you observe how target servers, reverse proxies, and Web Application Firewalls (WAFs) respond to malformed syntax, structural anomalies, and oversized headers.

---

## ⚙️ Features

* ✅ **56 Targeted Test Cases:** Injects raw null bytes, massive Content-Lengths (integer overflows), malformed Host headers, invalid HTTP versions, CRLF variations, missing delimiters, and oversized payloads.
* 🛡️ **Scope Protection:** Automatically verifies the target is in-scope before execution to prevent accidental or unauthorized testing.
* 🚦 **Smart Organizer Highlighting:** Saves all requests/responses to the Burp Organizer, automatically applying color highlights based on the HTTP response code to speed up manual review.
* ⚠️ **HTTP/2 Detection:** Automatically logs a warning if the baseline request is using HTTP/2, as raw string manipulations (like CRLF fuzzing) are most effective over HTTP/1.1.
* 📊 **Detailed Analytics:** Aggregates status codes, calculates response lengths, tracks request duration (ms), and provides execution statistics in the output console.
* 🛡️ **Fault Tolerance & Exception Isolation:** Wraps each test case in strict `try-catch` blocks. If a payload causes a catastrophic failure or API exception, the script safely logs it and continues executing the remaining tests without crashing.
* 🧠 **Null-Pointer Protection:** Safely handles dropped connections and empty responses to ensure the script never crashes when calculating response body lengths.
* ⚖️ **WAF-Safe Sequential Execution:** Runs test cases synchronously and respects standard timeouts. This avoids overwhelming the server, prevents rate-limiting/IP bans, and keeps evidence in perfect chronological order.

---

## 🎨 Organizer Color Guide

To simplify reviewing results, the script assigns color tags to items sent to the **Burp Organizer**. 

> **Note:** Colors group responses by status code behavior and do not automatically indicate security vulnerabilities.

| Color | Status Code | Context |
| :--- | :--- | :--- |
| 🔴 **Red** | `500+` |  **Server Error:** The server responded with a 5xx error. This could mean a backend crash, an unhandled exception, or a security control intentionally blocking the malformed request and returning a 500. |
| 🔵 **Cyan** | `200` / `201` | **2xx Success:** The server processed the fundamentally broken request and returned a success code. |
| 🟡 **Yellow** | `0` | **Dropped / Timeout:** The server dropped the connection entirely or the request timed out waiting for a response. |
| 🟦 **Blue** | `1337` | **Headerless / HTTP 0.9 Response:** Burp Suite injects `HTTP/0.9 1337 No response headers received` when a server returns a body but fails to return valid HTTP headers. This mock status ensures the raw response is still viewable in the Message Editor. |
| ⚪ **Gray** | `4xx` (Default) | **Client Error:** Standard rejection (e.g., `400 Bad Request`, `405 Method Not Allowed`), indicating the server properly detected malformed syntax. |

---

## 🚀 Usage

1. Clone or download this repository.
2. Open **Burp Suite**.
3. Navigate to **Extensions → Bambda Library**.
4. Click **Import** and select the Bambda script file: **BadHTTPRequestFuzzer.bambda**.
5. Go to the **Repeater** tab.
6. Select an HTTP/1.1 request (recommended over HTTP/2 for best results).
7. Open **Custom Actions**.
8. Click **Load** and select: **BadHTTPRequestFuzzer**.
9. **Run** the custom action.
10. Check the **Output Panel** for live execution logs and the **Organizer** tab for color-coded evidence.

---

## 🧪 Example Output

```text
----- Bad HTTP Request Fuzzer Started -----
[Info] Target: example.com
[1/56]: Send empty request (send nothing) -> No HTTP response received | Duration: 20004ms
[2/56]: Send request containing only CRLF -> No HTTP response received | Duration: 20008ms
[3/56]: Send request containing only HTTP method -> Status: 400 | Length: 266 | Duration: 3ms
[4/56]: Send request containing only HTTP method and URI -> Status: 1337 | Length: 191 | Duration: 5ms
...
...
[54/56]: Send request without header-body separator CRLF -> Status: 408 | Length: 261 | Duration: 21260ms
[55/56]: Send request using only LF instead of CRLF -> Status: 400 | Length: 266 | Duration: 2ms
[56/56]: Send request with tabs instead of spaces -> Status: 400 | Length: 266 | Duration: 2ms
---
[Info] Status Code Statistics:
Status 0 (Dropped/Timeout): 2 occurrences
Status 200: 7 occurrences
Status 400: 41 occurrences
Status 408: 3 occurrences
Status 501: 3 occurrences
[Info] Total Cases: 56, Executed: 56, Failed to Execute: 0
[Info] Refer to Organizer for saved requests & responses for further analysis.
----- Bad HTTP Request Fuzzer Completed -----

```

## 🤝 Contributing

Suggestions for improvements, additional test cases, or bug fixes are welcome. Please submit pull requests or open issues on the GitHub repository.

## ⚠️ Disclaimer

This tool is intended for authorized security testing and educational purposes only. Intentionally sending malformed packets can cause unexpected backend behavior. Use only against systems you own or have explicit permission to test.

## 👤 Author

Nikunj Sapara — [github.com/nikunjsapara](https://github.com/nikunjsapara/)
