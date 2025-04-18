# Outlook Account Checker

<img src="https://i.imgur.com/mBpk4e0.png" width="1000px">

This tool is a multithreaded Outlook login checker written in Python. It sends structured HTTP requests to Outlook's login endpoints to validate the credentials of provided accounts.

---

## ⚙️ How It Works

- Uses `tls_client` to simulate a Chrome-based TLS session for realistic requests.
- Initializes session via `https://outlook.live.com/owa/?nlp=1` to obtain required login tokens (PPFT, uaid, etc).
- Sends pre-auth and login requests to Outlook's endpoints.
- If login is successful, credentials are saved to `success.txt`.
- Supports concurrent checking with up to 200 threads for high performance.

---

## 📁 Setup

### 1. Requirements

Install required libraries using pip:
```python
pip install tls-client loguru
```

### 2. Account List

Prepare a file named `accounts.txt` in the same directory as the script. Format:
```
email:password
```

---

## 🚀 Usage

Simply run the Python script:
- Successful logins will be saved in `success.txt`.

---

## 🛡️ Notes

- The checker can be configured to use proxies by uncommenting the relevant lines in the `__init__` method.
- Random user-agent and TLS extension order are used to minimize detection.
- Handles exceptions and failed attempts gracefully.

---

### ⚠️ Disclaimer  

This project has been developed for educational and research purposes only. Unauthorized access to any service or system is illegal and strictly prohibited. The developer is not responsible for any misuse of this tool.
