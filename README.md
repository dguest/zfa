Zero-factor Authentication
==========================

Two Factor Authentication (TFA) is a lot
of things: it's cool, it's ubiquitous, it's practically
unhackable. It's like having Fort Knox in your computer and your
computer surrounded by landmines. Who wouldn't want that?

It turns out everyone wouldn't want that. TFA is a raging pain in the
ass. What was wrong with good-ol--ssh-keys? Do you actually know
anyone who had their account compromised before TFA? This
overcomplicated setup is just begging for someone to come up with a
workaround far less secure than what we were doing before.

Well, beg no longer, that workaround is here! I present to you: zero-factor authentication. This **reduces** your security in several ways:
- hardcode your credentials (as plain text) into your home area
- blindly execute some random OTP generator I found on the interweb
- janky expect scripting to hold it all together.

What could possibly go wrong?[^1] Oh... a lot you say? Well don't say
I didn't warn you.

---


[^1]: I do care about security. If you can think of an obvious attack vector please just raise an issue or figure out another way to contact me.

This repo contains some scripts to automate connecting to
`lxplus`. The minimum things you'll need are:

* Kerberos authentication (`kinit`)
* One-time password (2FA) via `totp-cli`

There are two methods below; I recommend the second. I'm leaving the
first method because it can forward _all_ your web traffic,
not just SSH, to `lxplus`. As a bonus it heaps `sshuttle` onto this
Rube-Goldberg monstrosity, and then slaps on some root password (also in
plain text, naturally) for extra sheen.

---

## Method 1 Requirements

Seriously don't use this method, skip to method 2 (which will also
tell you to come back here; if you can't read a convoluted manual you
probably shouldn't be using this).

- **sshuttle** installed (unless you're using method 2)
- **Expect** installed
- **Kerberos client** (`kinit`) configured
- **TOTP generator** (`totp-cli`) installed

### Setup

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

(note that you don't need the `.sudopass` for method 2 and shouldn't
need it for method 1)

3. **Store your token in `totp-cli`

Use `totp-cli add-token cern lxplus` and then enter the token when
prompted.

4. **Set the TOTP environment variable**:

```bash
export TOTP_PASS="your_totp_seed_here"
```

---

### Usage

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

### Notes

* Uses per-user Kerberos ticket cache: `/tmp/krb5cc_$USER_lxplus`.
* Routes **all network traffic**, including DNS, through the tunnel (`0/0`).
* Logging is controlled via the `log_enabled` variable in the script.

## Method 2: Control Master (recommended)

Setup like method 1, but you don't need `sshuttle`.

You'll also need something like this in your `.ssh/config`:

```
host lxplus*.cern.ch lxplus
     user gianotti
     Hostname lxplus.cern.ch
     GSSAPIAuthentication yes
     GSSAPIDelegateCredentials yes
     PasswordAuthentication no
     ControlPath ~/.ssh/cm-%r@%h:%p
     ControlMaster auto
     ControlPersist 20h
     DynamicForward 8090

```

Then run the `zfa-cm` script. It will launch `ssh` in the
background. Note that it might stick around for a while. Find any
orphaned remnants with `pgrep -l ssh` and kill them with `pkill`.
