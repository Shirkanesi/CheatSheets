# DNS-Setup
You need at least the following two DNS-records to send mails to the outside world:
- MX-record for `yourdomain.com` with the value `mail.yourdomain.com` (set priority to `2`)
- TXT-record for `yordomain.com` with the value `v=spf1 mx -all`

You should also add the following record, however this is not required
- MX-record for `mail.yourdomain.com` with the value `mail.yourdomain.com` (set priority to `2`)

# Postfix (SMTP)
Based on [this Tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-postfix-on-ubuntu-20-04-de)

## Installing postfix
Run `$ sudo apt update` to update your package-cache.

Install postfix with `$ sudo apt install postfix`. This might bring up a configuration-interface, if not run `$ sudo dpkg-reconfigure postfix` afterwards.

In the configuration you want `Internet Site`, 
then your domain name, e.g. `shirkanesi.com`. (Note: even if your server is only reachable by `mail.yourdomain.com`, you want to use `yourdomain.com` here.).
In the next box just enter the username you're currently loggen in with.

In the next screen, you'll be asked for optional hostnames, it is recommendet to add here `mail.yourdomain.com`, however not necessarry.

For the next option just hit `No`

Local networks is a bit tricky. Basially it askes you for the IP-range from which you'll be abled to send mails from the server. Even if it sounds counterintuitive, leave it `127.0.0.1/8`, we'll fix this issue later.

For the rest just use the default values (`0`, `+`, `all`).

## Configuring postfix
Run `$ sudo postconf -e 'home_mailbox= Maildir/'` to set everyones maildir to `~/Maildir/.

Next type `$ sudo postconf -e 'virtual_alias_maps= hash:/etc/postfix/virtual'`  to specify the location of your virtual address-mapping. 

Create the virtual address-mapping-file by `$ sudo nano /etc/postfix/virtual`
In this file, you can now set, what email-addresses will be mapped to which user:
```
# EMAIL-address     linux-user
mail@shirkanesi.com shirkanesi
info@shirkanesi.com shirkanesi
```

Now restart postfix to apply the settings: `$ sudo systemctl restart postfix`

## Preparing accounts for email
Run `$ echo 'export MAIL=~/Maildir' | sudo tee -a /etc/bash.bashrc | sudo tee -a /etc/profile.d/mail.sh` to make sure the MAIL-Variable is set correctly all the time.
To actually set the variable in your current session, you can either relogin or run `source /etc/profile.d/mail.sh`

# Dovecot (IMAP)
Taken from [this tutorial](https://www.arubacloud.com/tutorial/how-to-configure-a-pop3-imap-mail-server-with-dovecot-on-ubuntu-18-04.aspx)

Install Dovecot by running 

`$ sudo apt update && sudo apt install dovecot-core dovecot-pop3d dovecot-imapd`

Open `/etc/dovecot/dovecot.conf` and set the following values:
- `protocols = imap pop3`
- `listen = *, ::`
- `mail_location = maildir:~/Maildir`

For combatibility with old Outlook, add also 
- `pop3_uidl_format = %08Xu%08Xv`
- `pop3_client_workarounds = outlook-no-nuls oe-ns-eoh`

Now just restart dovecot with `$ sudo systemctl restart dovecot` and you're done.

# OpenDMARC (DMARC)
## Install
Taken from [this tutorial](https://www.linuxbabe.com/mail-server/opendmarc-postfix-ubuntu)

Run `$ sudo apt install opendmarc`. In case an installer appears, just hit `No`.

To autostart OpenDMARC, run `$ sudo systemctl enable opendmarc` (optional but very recommended!)

Open the configuration-file `$ sudo nano /etc/opendmarc.conf`

Change `# AuthservID name` to `AuthservID OpenDMARC`

Add `TrustedAuthservIDs mail.yourdomain.com` (replace with your domain)

Replace `# RejectFailures false` with `RejectFailures true`

Add `IgnoreAuthenticatedClients true` and `RequiredHeaders true` and `SPFSelfValidate true``

Now replace `Socket local:/var/run/opendmarc/opendmarc.sock` with `Socket local:/var/spool/postfix/opendmarc/opendmarc.sock`

Now you just need to create a spooler dir and an opendmarc user:
```bash
sudo mkdir -p /var/spool/postfix/opendmarc
sudo chown opendmarc:opendmarc /var/spool/postfix/opendmarc -R
sudo chmod 750 /var/spool/postfix/opendmarc/ -R
sudo adduser postfix opendmarc
sudo systemctl restart opendmarc
```
In your postfix-config (`/etc/postfix/main.cf`) you need to add
```
# Milter configuration
milter_default_action = accept
milter_protocol = 6
smtpd_milters = local:opendkim/opendkim.sock,local:opendmarc/opendmarc.sock
non_smtpd_milters = $smtpd_milters
```
Now restart postfix `$ sudo systemctl restart postfix` and everything should work just fine.

Note: you can alway take a look at `/var/log/mail.log` to see, if anything goes wrong.

# Preventing mail-users from logging in via SSH
(_Hint: make sure not to do a stupid mistake by disabling SSH for every account. If you do, you'll be locked out of the system :c_)

Add a new group named "mailusers": `$ sudo groupadd mailusers`

Open `/etc/ssh/sshd_config` and add `DenyGroups mailusers`

Restart your ssh-deamon: `$ sudo systemctl restart sshd`

Now add each user you don't want to log in via ssh to this group:

`$ sudo usermod -aG mailusers <username>` (Important: do not forget the `a` in `-aG`. The a makes sure you do not remove all other groups!)

## Re-enabling SSH for a user
`$ sudo deluser <username> mailusers`  (Note: this will neither delete the user nor the group. It only deltes the membership of the user)

# Daily house-ceeping
## Add a new user
```
sudo useradd john -m
sudo passwd john
sudo usermod -aG mailusers john         // Disables ssh for john
```