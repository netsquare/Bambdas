# Timestamp Generate & Replace - Burp Suite Custom Action (Bambda Script)

## 📌 Overview
**Timestamp Generate & Replace** is a Burp Suite custom action (Bambda script) that automatically generates epoch timestamps and replaces predefined placeholders in HTTP requests.

This is useful for testing APIs or applications that require dynamic timestamp values in requests.

## ⚙️ Features
- ✅ Automatically generates:
  - Epoch timestamp in seconds
  - Epoch timestamp in milliseconds
- 🔄 Replaces placeholders directly in requests:
  - `<EPOCH>`
  - `<EPOCHMILLI>`
- 🧾 Logs replacement activity in the output panel
- ⚡ Quickly updates requests without manual timestamp generation

## 🧩 Supported Placeholders
| Placeholder | Description |
|---|---|
| `<EPOCH>` | Replaced with epoch timestamp in seconds |
| `<EPOCHMILLI>` | Replaced with epoch timestamp in milliseconds |

## 🚀 Usage
1. Clone or download this repository.
2. Open **Burp Suite**.
3. Navigate to **Extensions → Bambda Library**.
4. Click **Import** and select the Bambda script file: **TimestampGenerateReplace.bambda**.
5. Open the **Repeater** tab.
6. Add one of the supported placeholders inside the request:
   - `<EPOCH>`
   - `<EPOCHMILLI>`
7. Open **Custom Actions**.
8. Click Load and select: **TimestampGenerateReplace**.
9. **Run** the custom action.
10. The placeholders will automatically be replaced with generated timestamps.

## 🧪 Example
### Before
```
POST /api/test HTTP/1.1

{
  "timestamp": "<EPOCH>",
  "timestamp_ms": "<EPOCHMILLI>"
}
```

### After
```
POST /api/test HTTP/1.1

{
  "timestamp": "1755216000",
  "timestamp_ms": "1755216000000"
}
```

## 🤝 Contributing
Suggestions for improvements or bug fixes are welcome. Please submit pull requests or open issues on the GitHub repository.
