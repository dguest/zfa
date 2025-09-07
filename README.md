Zero-factor Authentication
==========================

Automates connecting to `lxplus.cern.ch` via `sshuttle` with:

* Kerberos authentication (`kinit`)
* Local sudo password for sshuttle
* SSH password for lxplus
* One-time password (2FA) via `totp-cli`
* DNS tunneling
* Persistent interactive session

---

## Setup

1. **Clone the repository**:

```bash
git clone <repo-url>
cd <repo-directory>
```

2. **Create password files** (permissions must be 600):

```bash
echo "your_sudo_password" > ~/.sudopass
echo "your_ssh_password" > ~/.lxpluspass
chmod 600 ~/.sudopass ~/.lxpluspass
```

3. **Set the TOTP environment variable**:

```bash
export TOTP_PASS="your_totp_seed_here"
```

---

## Usage

Run the script:

```bash
./zfa-lxplus
```

* The script will:

  1. Refresh or request your Kerberos ticket (`kinit`).
  2. Generate a one-time password using `totp-cli g cern lxplus`.
  3. Connect to `lxplus.cern.ch` via `sshuttle`.
  4. Supply sudo, SSH, and OTP passwords automatically.
  5. Print:

```
Tunnel established successfully. Session is now interactive.
```

* The session remains open indefinitely for the VPN tunnel.

---

## Notes

* Uses per-user Kerberos ticket cache: `/tmp/krb5cc_$USER_lxplus`.
* Routes **all network traffic**, including DNS, through the tunnel (`0/0`).
* Logging is controlled via the `log_enabled` variable in the script.

