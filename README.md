# 🛡️ ADCS Discover (`ESC1–ESC8` Detection)

This tool scans IPs for Active Directory Certificate Services (ADCS) and detects known misconfigurations or exposures from the **ESC1 to ESC8** series, as documented by SpecterOps.

It supports **unauthenticated HTTP/HTTPS scanning** and **optional authenticated LDAP** for deeper inspection of certificate templates.

---

## 📌 Key Capabilities

- 🔍 Detects ADCS servers using:
  - `--method ldap`: Queries LDAP port 389 for `pKIEnrollmentService` objects
  - `--method http`: Probes common ADCS endpoints (like `/certsrv/`) via HTTP or HTTPS
- ✅ Detects the full suite of **ESC1 to ESC8** misconfigurations
- ⚡ Uses **multithreaded scanning** (via ThreadPoolExecutor)
- 🔐 Supports **optional LDAP authentication** to unlock deeper inspection of certificate templates
- 📄 Outputs IPs of ADCS servers found and associated ESC vulnerabilities

---

## 📥 Requirements

- Python 3.7+
- Dependencies:

```bash
pip install ldap3 requests
```

---

## 🚀 Usage

```bash
python3 ADCS-Discover.py ip_list.txt [options]
```

### 🔧 Command-Line Options

| Option              | Description |
|---------------------|-------------|
| `ip_file`           | **(Required)** File with one IP per line |
| `--method`          | Scan method: `ldap` (default) or `http` |
| `--timeout`         | Timeout for connection attempts (default: `5` sec) |
| `--workers`         | Max concurrent scan threads (default: `50`) |
| `--https`           | Use HTTPS instead of HTTP (only applies if `--method http` is set) |
| `--username`        | LDAP username (`domain\\user`) — only required if checking ESC1–ESC6 and ESC8 |
| `--password`        | LDAP password — used with `--username` |

---

## 🔐 Auth Requirements by Check

| Check     | LDAP Bind Required? | Protocol |
|-----------|---------------------|----------|
| ADCS Detection (`--method ldap`) | ❌ (uses anonymous bind) | LDAP |
| ADCS Detection (`--method http`) | ❌                        | HTTP/HTTPS |
| ESC1–ESC6, ESC8                  | ✅ (requires `--username` and `--password`) | LDAP |
| ESC7                             | ❌                        | HTTP/HTTPS |

> 🔸 Without a username/password, the script can still identify ADCS servers and test for **ESC7** using unauthenticated web probes.

---

## 🧠 What the Script Does

1. **Reads IPs** from a file and validates them
2. **Scans each IP** for ADCS via chosen method (LDAP or HTTP)
3. If an ADCS server is found:
   - Checks for **ESC7**: unauthenticated access to `/certsrv/`
   - If LDAP is authenticated:
     - Queries **certificate templates**
     - Looks for misconfigurations indicating **ESC1–ESC6, ESC8**

---

## 🧪 Example Commands

**Basic scan using unauthenticated LDAP (for ADCS detection only + ESC7):**
```bash
python3 ADCS-Discover.py targets.txt --method ldap
```

**HTTP-based ADCS scan with HTTPS (still unauthenticated):**
```bash
python3 ADCS-Discover.py targets.txt --method http --https
```

**Full scan with LDAP authentication (for ESC1–ESC6 & ESC8):**
```bash
python3 ADCS-Discover.py targets.txt --username "corp\\admin" --password "S0mePassw0rd1"
```

---

## ⚠️ What Are ESC1–ESC8?

| ESC ID | Description |
|--------|-------------|
| **ESC1** | Enrollee can specify Subject Alternative Name (SAN) |
| **ESC2** | EKU includes Any Purpose or is missing |
| **ESC3** | Dangerous application policies like Smartcard Logon |
| **ESC4** | Low-privileged users have write access to templates |
| **ESC5** | Template allows auto-enrollment |
| **ESC6** | No approval required for certs with dangerous usage |
| **ESC7** | Web enrollment accessible without authentication |
| **ESC8** | Enrollment agents can request on behalf of others unsafely |

> See [SpecterOps’ whitepaper](https://posts.specterops.io/certified-pre-owned-d95910965cd2) for in-depth details on each.

---

## 🧾 Sample Output

```
2025-04-10 12:34:56 - INFO - ADCS server found at 192.168.1.100 (LDAP): CA01.corp.local
ESC1 on 192.168.1.100: Template 'User' allows enrollee-supplied SAN
ESC3 on 192.168.1.100: Template 'WebServer' allows dangerous application policies
ESC7 on 192.168.1.100: Unauthenticated web enrollment accessible at http://192.168.1.100/certsrv/
```

---

## 🛠 Limitations

- ⚠️ **ESC4**: Detection is limited to presence of a security descriptor; **no full DACL parsing**
- ⚠️ **ESC8**: Simplified detection based on presence of enrollment agent EKU
- ✳️ This tool **does not exploit** vulnerabilities — it is read-only
