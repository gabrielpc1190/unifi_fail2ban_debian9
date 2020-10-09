# This is a simple way of using Fail2ban with an Unifi server on the cloud on a Debian 9 server. Totally needed in my case, because I'm the only user and I usually save my passwords on my password manager, so retrying several times is not going to happen from my side.

# Install Fail2ban
```
sudo apt-get install fail2ban -y
```
# Create a filter for unifi on the faul2ban filter.d folder
```
sudo nano /etc/fail2ban/filter.d/unifi.conf
```
Paste the following code:
```
[INCLUDES]
before = common.conf
[Definition]
failregex = ^(.*)Failed .* login .* <HOST>*\s*$
ignoreregex =
```
# Now create the jail.local file and setup the filters for the services, in this case ssh and unifi
```
sudo nano /etc/fail2ban/jail.local
```
Paste the following code:
```
[DEFAULT]
ignoreip = xxx.yyy.zzz.hhh/32
bantime  = 345600
findtime  = 300
maxretry = 4
[sshd]
enabled = true

[unifi]
ignoreip = xxx.yyy.zzz.hhh/32
enabled = true
port = 8080,8443
filter = unifi
logpath = /var/log/unifi/server.log
maxretry = 4
bantime = 3600
findtime = 300
action = iptables-multiport[name="unifi", port="8443,8080"]
```
Check the ignoreip, that's an IP that will be ignored by the fail2ban, it will never be blocked.
Make sure to edit the fields based on your specific case, on my case, I want to block any IP address failing more than 4 times in 300 seconds (5 minutes) for 3600 seconds (1 hour)

# Restart Fail2ban to apply changes above.
```
sudo service fail2ban restart
```

# Testing
Now, from a mobile connection or any other IP you may use to try the block, try to login 4 times with incorrect credentials. You can see on this file the block being executed after the 4 try.
```
sudo tail -f /var/log/fail2ban.log
```
```
2020-10-09 02:29:11,154 fail2ban.jail           [3211]: INFO    Creating new jail 'unifi'
2020-10-09 02:29:11,154 fail2ban.jail           [3211]: INFO    Jail 'unifi' uses pyinotify {}
2020-10-09 02:29:11,156 fail2ban.jail           [3211]: INFO    Initiated 'pyinotify' backend
2020-10-09 02:29:11,159 fail2ban.filter         [3211]: INFO    Added logfile: '/var/log/unifi/server.log' (pos = 5593389, hash = d61baa573c241caf565d6201c5f5b8aee6280857)
2020-10-09 02:29:11,159 fail2ban.filter         [3211]: INFO      encoding: UTF-8
2020-10-09 02:29:11,160 fail2ban.filter         [3211]: INFO      maxRetry: 4
2020-10-09 02:29:11,160 fail2ban.filter         [3211]: INFO      findtime: 900
2020-10-09 02:29:11,160 fail2ban.actions        [3211]: INFO      banTime: 3600
2020-10-09 02:29:11,162 fail2ban.jail           [3211]: INFO    Jail 'sshd' started
2020-10-09 02:29:11,163 fail2ban.jail           [3211]: INFO    Jail 'unifi' started
2020-10-09 02:29:56,734 fail2ban.filter         [3211]: INFO    [unifi] Found xx1.x1X.xxx.125 - 2020-10-09 02:29:56
2020-10-09 02:30:16,209 fail2ban.filter         [3211]: INFO    [unifi] Found xx1.x1X.xxx.125 - 2020-10-09 02:30:15
2020-10-09 02:30:20,558 fail2ban.filter         [3211]: INFO    [unifi] Found xx1.x1X.xxx.125 - 2020-10-09 02:30:20
2020-10-09 02:30:22,898 fail2ban.filter         [3211]: INFO    [unifi] Found xx1.x1X.xxx.125 - 2020-10-09 02:30:22
2020-10-09 02:30:23,253 fail2ban.actions        [3211]: NOTICE  [unifi] Ban xx1.x1X.xxx.125
```
Now you will see that from that IP address the connection is rejected, so it banned the IP succesfully 🙂

# Want to unban an ip address? Sure! Anyone can get banned when doing testing! Hopefully you will have SSH access to your server!
```
sudo fail2ban-client set sshd unbanip xxx.yyy.zzz.hhh
```

The original idea was found on this script. Thanks for the ideas!
https://raw.githubusercontent.com/miketabor/unifi-autoinstall/master/install.sh
