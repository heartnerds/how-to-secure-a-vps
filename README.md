## **How to secure a VPS ?**

### **Creating a user with restricted rights**

Create a new user with the following command:
```sh
$ adduser myuser
```

The new user may want to execute `sudo` commands. To do so, you can add the user to the sudo group:
```sh
$ usermod -aG sudo myuser
```

### **Setup SSH for the new user**

Switch to the new user with the following command:
```sh
$ su - myuser
```

Create a `.ssh` folder in the user's home directory:
```sh
$ mkdir ~/.ssh
```

Restrict access to this user only. Let's change the folder's permission so only this user is allowed to read and write to it:
```sh
$ chmod 700 ~/.ssh
```

In this folder, we'll create a file named `authorized_keys` which will contain your public key:
```sh
$ vim ~/.ssh/authorized_keys
```

Restrict access to this user only, same operation as above, but this time it's for the file `authorized_keys`:
```sh
$ chmod 600 ~/.ssh/authorized_keys
```

For the changes to take effect, we'll need to restart `sshd` with the following command: 
```sh
$ systemctl restart sshd.service
```

### **Disable password and root login**

It's good practice to disable password login and protect your VPS from brute force attacks.

<table>
<tr>
<th>
In /etc/ssh/sshd_config
</th>
</tr>

<tr>
<td>

Change 
```
PermitRootLogin no
PasswordAuthentication no
```

</td>
</tr>
</table>

For the changes to take effect, we'll need to restart `sshd` with the following command: 
```sh
$ systemctl restart sshd.service
```

### **Configuring the firewall**

GNU/Linux distributions come with a firewall service named iptables. By default, this service does not have any active rules.

This script is a secure and minimal configuration, you need to modify it to suit your needs:
```sh
#!/bin/sh

# Empty existing rules
iptables -t filter -F
iptables -t filter -X

# Rejects all connections
iptables -t filter -P INPUT DROP
iptables -t filter -P FORWARD DROP
iptables -t filter -P OUTPUT DROP

# Allow connections already established
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

# Allows loop-back (localhost)
iptables -t filter -A INPUT -i lo -j ACCEPT
iptables -t filter -A OUTPUT -o lo -j ACCEPT

# Allow NTP 
iptables -t filter -A OUTPUT -p udp --dport 123 -j ACCEPT

# Allow pinging
iptables -t filter -A INPUT -p icmp -j ACCEPT
iptables -t filter -A OUTPUT -p icmp -j ACCEPT

# Allow SSH
iptables -t filter -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 22 -j ACCEPT

# Allow DNS (Use with HTTP)
iptables -t filter -A OUTPUT -p tcp --dport 53 -j ACCEPT
iptables -t filter -A OUTPUT -p udp --dport 53 -j ACCEPT

# Allow HTTP
iptables -t filter -A OUTPUT -p tcp --dport 80 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 443 -j ACCEPT
```

Iptables rules need to be saved, by default the rules are reset when the VPS is restarted. Root is needed to saves the rules:
```sh
$ su
```

Save iptables rules with ***Arch linux***:
```sh
$ iptables-save > /etc/iptables/iptables.rules
$ ip6tables-save > /etc/iptables/ip6tables.rules
```

Save iptables rules with ***Debian***:

*You need to install the package `iptables-persistent`*.

```sh
$ iptables-save > /etc/iptables/rules.v4
$ ip6tables-save > /etc/iptables/rules.v6
```

### **Disable IPv6**

IPv6 has several advantages over IPv4, but it’s unlikely that you’re using it, few people are.

<table>
<tr>
<th>
In /etc/default/grub
</th>
</tr>

<tr>
<td>

Add `ipv6.disable=1` on this two lines:
```
GRUB_CMDLINE_LINUX_DEFAULT="... ipv6.disable=1 ..."
GRUB_CMDLINE_LINUX="... ipv6.disable=1 ..."
```

</td>
</tr>
</table>

Update the GRUB with ***Arch linux***:
```sh
$ grub-mkconfig -o /boot/grub/grub.cfg
```

Update the GRUB with ***Debian***:
```sh
$ update-grub
```
