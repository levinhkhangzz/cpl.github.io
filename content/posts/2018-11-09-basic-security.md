---
title: "Good security practices"
description: "How to keep yourself, your data and your day to day business private and secure."
keywords: [technical, tutorial, security, privacy]
date: 2018-11-09
---

## Passwords

### Password Tips

* Do not store your password in **plain text** or on **paper**
* Do not share your credentials with other people
    * This includes your **parents**, **significant other**, **pets**, **colleagues**, **boss/superior**
* Do not use the same password everywhere
    * Or the same password with small changes
    * For example
        * `pass1` `pass2` `pass3`
        * `alexGmail` `alexReddit`
* Do not use personal details in your password
    * This makes it vulnerable to **dictionary attacks**, or **social engineering** attacks
    * For example:
        * Name
        * Date of birth
        * Family members
        * Favourite animal
        * Pet names
        * Place of birth
        * Username
        * Favourite numbers
        * Or words in general
* Passwords should be more than **16 characters** and should contain a varied number of **digits**, **symbols** and optional **extended ASCII**
* This is how a good password looks like:
    * ```7})k6L7+K>jFA(oY-cM)x/4m*pQr*&`X```
    * `$;Kf&*c=pYCm]tosQF5JVK>eg@"HB%^8`
    * `F4cR¯i°TL+CØD2·S»3ÝÇÜm²ÃÜºçèºÐ¬Ó`
        * This one contains **extended ASCII**
* Use special software (or hardware) to generate your passwords. (eg: password managers)
    * This ensures true entropy is used
    * Passwords are managed and encrypted

### Password Managers

#### Definition

A password manager is typically a software application or a hardware device that is used to store and manage a person's passwords and strong passwords. Typically all stored passwords are **encrypted**, requiring the user to create a **master password** to access all managed passwords.

#### Instructions

1. Pick a program from the [list](#software) below
2. You generate a **masater password** (a very very strong and super super secret password)
    * Which you should not forget! If you forget your master password there is no *forgot password* button. The database is encrypted so you permanently loose access.
    * Some password managers allow the generation of a **key** (which is stored **as a file**) and can be used to unlock the database
        * The key file can also be **requiered** to unlock the database (This means **password** + **key**, in order to unlock)
3. Create groups and/or tags to order your credentials
![KeePassXC, groups used by me](/assets/images/pmgroups.png)
4. Create new entries in the password manager
    * You can store a **title**, **username**, **website**
    * Which can be used for autocompletion by using a shortcut, `Cmd` + `V`
![](/assets/images/pmentries.png)
5. Use the password generator provided by the manager
![](/assets/images/pmgenerator.png)
6. You can add extra fields or attach files to your entries
    * These will be encrypted aswell

---

* Always make sure you **lock** your password manager
* Always make sure to **save** (`Cmd` + `S`)
* Master password
    * Do not share your master password!
    * Do not forget your master password!
    * Do not write down your master password!
* Try and backup your password database as often as possible!

#### Software

* [KeePass](https://keepass.info/)
    * Perosnal recommendation
* [LastPass](https://www.lastpass.com/)
    * Botnet (enjoy beeing tracked)
* [1Password](https://1password.com/)
    * I think it's not free?
* [PassGo](https://github.com/ejcx/passgo)
    * Maybe don't

## Online Security

### HTTPS

When logging in (or signing up) using a web service, always check for the **green lock** and `https://` in the front of the `URL`. Not `http` (which stands for **Hyper-Text-Transfer-Protocol**), but `https`, with the extra `s` at the end (which stands for HTTP **Secure**).

**HTTPS** allows for Client-Server encrypted communication, while HTTP transfers everything in plain-text allowing "intruders" to sniff in on your traffic and steal your credentials. Even worse, HTTP allows for [Man in the middle](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) attacks.

Traffic over **HTTPS** is encrypted using SSL Certificates. You can read more about SSL [here](https://en.wikipedia.org/wiki/Transport_Layer_Security).

Example: 
![](/assets/images/httpslock.png)

Web Extension that tries to use HTTPS on all websites:
* [Chrome](https://chrome.google.com/webstore/detail/https-everywhere/gcbommkclmclpchllfjekcdonpmejbdp?hl=en)
* [Firefox](https://addons.mozilla.org/en-GB/firefox/addon/https-everywhere/)
* [Opera](https://addons.opera.com/en-gb/extensions/details/https-everywhere/)


### AdBlockers

They generally block ads, but can also block:
* Crytocurrency miners that use JavaScript to mine on your CPU
    * Eg: [Coinhive](https://coinhive.com/) which mines [Monero](https://getmonero.org/)
* Malicious content which can harm your computer or compromise your privacy and security
* Tracking software

Web Extension that blocks ads:
* [Chrome](https://chrome.google.com/webstore/detail/ublock-origin/cjpalhdlnbpafiamejdnhcphjbkeiagm?hl=en)
* [Firefox](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/)
* [Opera](https://addons.opera.com/en-gb/extensions/details/ublock/)

### 2FA (2 Factor Auth)

Some websites (most large websites) offer the option of **2 factor auth**. This means that after logging in using your password, the website will ask for a code (usually 6 digits). This code will either:
* Be sent in an email (not that secure)
* Be sent in a SMS (not that secure, better than email)
* Be generated by a **2FA** app
    * Eg: Google Authenticator
    ![](/assets/images/2fa.jpeg)

The codes can be time based (they expire after 30-60 seconds), or count based (only valid in some order).

###### Why use 2FA?

In case your password gets compromised, 2FA will stop attackers from gaining access to your account without having access to your 2FA device (this is why an app is prefered, or dedicated hardware for code generation (eg: bank tokens)).

2FA will save your accounts security in case the websites password database gets leaked, your password gets stolen, your password gets cracked.


## Keys

### RSA/ECDSA

RSA/ECDSA/(or other) keys are usually used to replace passwords. Keys provide protection agains brute force/dicionary attacks. Keys can also be used to "sign" (verify that something comes from you) a "message" (or file, or anything).

In order to generate a key:
```shell
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (~/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```
This will create a RSA key pair for you (in **~/.ssh**). Do not share the `id_rsa` file ever! Only share the public key, found in `id_rsa.pub`.

```
# My public RSA key
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCTuTjOy984o3SuXoGfsnuaOuqET8wzmE+B0oDTDcl/Hz3SkNvHuKwYrXx0oHi2JUkKSwUx7XZtil0TN+U3mZ63gsfJ3ITazzsQ4hb39seajUiLK5Tcfgx1XnAevXRb9Bp+6LyEws4KbNbHv2bruYDYdoypkdTTfRJKZVjP0t4YxTkE69ImsW4K/Wi8f8WVa9EZecqEs3TvbVc4iuiJ9Fm2qkRCgD+kOmYf7+YNkLcgvuYDx0m7zRNqJyGs2r31qm8f/BMgpVZdN8o0441zotalDqLUHFlITxspKfiQyMr4NHQ/YuJZcAe5zhjutbEqi6FNGOMCK1YgYSUeywlCpxar

# My public ECDSA key
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAT4L1J12pZm3Ga/NHOvC0mudTRz5bv2UON/2Z294Z/ZCEy4wLqkIip4GnrJEbt9pJwG721fheVHP2PqBKwXyZU=
```

### GPG

1. Install GPG (OS dependant, should be easy by googling it)
2. Check [this](https://help.github.com/articles/generating-a-new-gpg-key/) guide for creating GPG keys.
3. For step 14 share your public key with us.

## SSH

### Security

**S**ecure **SH**ell, or **SSH** for short is a protocol which allows a users machine (**client**) to connect to another machine (**host**), allowing the user to run commands on the host. The link between the two is secure (no tampering is possible or sniffing).

A **SSH** server allows multiple auth methods (or any combination of these):
* Password (weak)
* Key (prefered)
* 2FA (overkill)

If the server does not have a **firewall** setup, brute force attacks (or dicionary attacks) are possible on the admin accounts. The best practice for security is disabling password auth, admin auth and force key only login over SSH.

### Client Config

Clients can find their **SSH** configurations, keys and known hosts in `~/.ssh`. The `.ssh/known_hosts` contains a list of all previously connected machines, their "id" (used to determine the authenticity of the host).

The default directory for `ssh-keygen` will be `.ssh/id_*` where `*` will be the format used (ecdsa, rsa, ...).

Never share your private keys! (`id_rsa`). Your public key is the only one you should ever share (`id_rsa.pub`).

###### `.ssh/config`

```
# This will show a nice ascii art image on connection
# allowing you to identify a host/key used.
VisualHostKey=yes
```
![](/assets/images/sshimgart.png)


```
# If you have a Mac, you might want to turn this on
# for all hosts.
Host	*
  UseKeychain yes
```

```
# Example of a host config
Host    gst.io
    HostName gstechnologies.io
    User example
    Port 22
```
`gst.io` is an alias in this case. Allowing you to run `ssh gst.io` instead of `ssh example@gstechnologies.io`

```
# You can send environment variables to your host
Host    *
    SendEnv VARNAME
```

###### GS Technologies SSH Config

If you add this to your `.ssh/config`, you can `ssh gstech`
```
Host    gstech
    HostName gstechnologies.io
    User YOUR_USERNAME
    Port 22
    IdentityFile $HOME/.ssh/YOUR_KEY
```

### Server Config

```
LoginGraceTime 1m
PermitRootLogin no
MaxAuthTries 3

PasswordAuthentication no
```
This enforces key only auth, and some more severe auth.

## Offline Security

### Your Device

* Do not leave yourself logged in!
* Set your devices to auto-lock when closed.
* Set your devices to auto-close after short amounts of inactivity.
* Do not use crap pins:
    * 1234
    * 0000
    * 2580
    * The year you were born in
* Do not browse private data in public areas.
    * Always look over your shoulder.

### Encryption

You can encrypt files using your GPG key.
You can also sign files (prove that they are coming from you) using GPG.

### Checksum

You can run `sum FILE_NAME` to get a number which represents the *version* of the file. This can tell you if the file you obtained was modified or replaced.
