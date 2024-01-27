# ROUTER + FW + SMTP + IMAP + HTTP + SSH

# Konfigurace síťových karet na linuxu
- eth0: 192.168.60.200+X/24 kde X je číslo Vašeho počítače, maska 255.255.255.0, brána 192.168.60.254, DNS 192.168.60.165 a 192.168.60.166
- eth1: nastavit staticky na 192.168.111.1/24

Yes, on AlmaLinux, which is a Red Hat Enterprise Linux (RHEL) fork, network interfaces are typically configured using configuration files located in `/etc/sysconfig/network-scripts/`. Here's how you can do it:

1. Navigate to the directory containing the network script files.

```sh
cd /etc/sysconfig/network-scripts/
```

2. You will need to edit the configuration files for each network interface. These files are usually named `ifcfg-eth0` for the first Ethernet interface, `ifcfg-eth1` for the second, and so on. 

For `eth0`, the file would be `/etc/sysconfig/network-scripts/ifcfg-eth0`, and you can create or edit it with a text editor such as `vi` or `nano`.

Here's a template for `eth0`:

```plaintext
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.60.200X  # replace X with the number of your computer
PREFIX=24
GATEWAY=192.168.60.254
DNS1=192.168.60.165
DNS2=192.168.60.166
```

And here's a template for `eth1`:

```plaintext
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
NAME=eth1
DEVICE=eth1
ONBOOT=yes
IPADDR=192.168.111.1
PREFIX=24
```

3. After you've edited or created these files, you'll need to restart the network service for the changes to take effect:

```sh
sudo nmcli con reload
sudo systemctl restart NetworkManager
```

Alternatively, if you are not using NetworkManager, you can bring the interfaces down and up:

```sh
sudo ifdown eth0 && sudo ifup eth0
sudo ifdown eth1 && sudo ifup eth1
```

Remember to replace `eth0` and `eth1` with the actual interface names on your system if they are different. You can check the names of your interfaces by using the `ip a` or `nmcli d` command. 

Please ensure that you have root or sudo privileges to make these changes, and always be careful when modifying network configurations to avoid connectivity issues.

# Konfigurace síťových karet na windows:
- nastavit statickou adresu 192.168.111.2, masku 255.255.255.0, brána 192.168.111.1, DNS 192.168.60.165 a 192.168.60.166

To configure a network card with a static IP address on a Windows operating system, follow these steps:

**Part 2: Configure Network Interface on Windows**

1. Open the Control Panel.

2. Navigate to "Network and Sharing Center."

3. Click on "Change adapter settings" on the left sidebar.

4. Right-click on the network interface you want to configure (often named "Ethernet" or "Local Area Connection") and select "Properties."

5. In the list, select "Internet Protocol Version 4 (TCP/IPv4)" and then click the "Properties" button.

6. In the new window, select "Use the following IP address:"

7. Enter the following details:
   - IP address: `192.168.111.2`
   - Subnet mask: `255.255.255.0`
   - Default gateway: `192.168.111.1`

8. Under "Use the following DNS server addresses:", enter:
   - Preferred DNS server: `192.168.60.165`
   - Alternate DNS server: `192.168.60.166`

9. Click "OK" to close the IPv4 properties window.

10. Click "OK" again to close the network interface properties window.

Your Windows network interface should now be configured with the static IP settings you've provided. It's a good idea to test the configuration by pinging another device on the same network, or by attempting to access the internet or a local network resource.

# FW
- povolení forwarding packetů na linux routeru
- zapnout NAT pro 192.168.111.0/24
- otestování na stanici

To enable packet forwarding and set up NAT (Network Address Translation) on a Linux router, you would typically need to perform the following steps:

**Part 3: Enable Packet Forwarding and NAT on a Linux Router**

1. **Enable IP forwarding**:

   Edit the `sysctl.conf` file to permanently enable IP forwarding:

   ```sh
   sudo nano /etc/sysctl.conf
   ```

   Add or uncomment the following line:

   ```
   net.ipv4.ip_forward=1
   ```

   Apply the changes by running:

   ```sh
   sudo sysctl -p
   ```

2. **Set up NAT for the 192.168.111.0/24 network**:

   Use the `iptables` command to set up NAT. This will typically involve masquerading, which allows for source IP address translation:

   ```sh
   sudo iptables -t nat -A POSTROUTING -o eth0 -s 192.168.111.0/24 -j MASQUERADE
   ```

   In the above command, replace `eth0` with the actual interface connected to the internet or the external network.

3. **Save the iptables rules**:

   The method to save `iptables` rules varies by distribution. On systems with `iptables-persistent`:

   ```sh
   sudo netfilter-persistent save
   ```

   On other systems, you might need to use a different method to ensure the rules persist after reboot.

4. **Test the configuration**:

   From a client machine on the 192.168.111.0/24 network, attempt to access an external resource, like the internet, to ensure that forwarding and NAT are working correctly.

   ```sh
   ping google.com
   ```

   If the ping is successful, your Linux router is forwarding packets and performing NAT as expected.

Remember to replace interface names and IP ranges with those appropriate for your specific network setup. Also, be aware that using `iptables` requires root privileges, and making changes to your network configuration can disrupt connectivity. Always proceed with caution and ensure you have a way to revert changes if necessary.

# konfigurace firewallu na linux:
- povolit prichozi komunikaci pro sluzby SMTP, HTTP, IMAP a 30332 TCP
- zakazat port 22 TCP

To configure the firewall on a Linux system to allow incoming traffic for specific services and to block a port, you would typically use `iptables`, the user-space utility program that allows a system administrator to configure the IP packet filter rules of the Linux kernel firewall.

Here's how you can do it:

**Part 4: Configure Firewall on Linux**

1. **Allow incoming traffic for SMTP, HTTP, IMAP, and port 30332 TCP:**

   SMTP typically uses port 25, HTTP uses port 80, and IMAP uses port 143. You can allow incoming traffic on these ports with the following `iptables` commands:

   ```sh
   sudo iptables -A INPUT -p tcp --dport 25 -j ACCEPT  # SMTP
   sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT  # HTTP
   sudo iptables -A INPUT -p tcp --dport 143 -j ACCEPT # IMAP
   sudo iptables -A INPUT -p tcp --dport 30332 -j ACCEPT # Custom TCP port 30332
   ```

2. **Block port 22 TCP (commonly used by SSH):**

   To block incoming connections on port 22, use the following command:

   ```sh
   sudo iptables -A INPUT -p tcp --dport 22 -j DROP
   ```

3. **Save the iptables rules:**

   As mentioned previously, saving `iptables` rules so they persist across reboots can vary depending on your Linux distribution. If you have `iptables-persistent` installed, you can save the rules like this:

   ```sh
   sudo netfilter-persistent save
   ```

   Otherwise, you may need to use a service or script that runs at startup to restore the rules.

4. **Apply the configuration:**

   After adding the rules, ensure they are applied correctly:

   ```sh
   sudo iptables -L -v
   ```

   This command lists all active rules along with their packet and byte counts, which can help verify that the rules are set up as intended.

Make sure to adjust the commands if your system uses a different port for these services or if your firewall configuration utility is not `iptables` (e.g., `firewalld` on newer distributions). Always check the current rules before making changes to avoid locking yourself out of the system, especially when modifying SSH rules.

# SMTP
 - konfigurace SMTP služby pro doménu mojefirma.cz na linux (jméno serveru mx.mojefirma.cz)
 - doručování mailů do uživatelského Maildiru, klienti se připojí z interní sítě
 - root mail přesměrovat na účet apribyl
 - zaslat mail na účet aprybyl a root

Configuring an SMTP service for a domain and setting up email delivery requires several steps and configurations, which typically include installing and configuring an MTA (Mail Transfer Agent) like Postfix, setting up user accounts, and redirecting emails. Here's a general outline of the steps for Part 5:

**Part 5: Configure SMTP Service and Email Delivery**

1. **Install and Configure Postfix (or another MTA)**:

   - Install Postfix:

     ```sh
     sudo apt-get update
     sudo apt-get install postfix
     ```

   - During installation, you will be prompted to choose a configuration type. Select "Internet Site".

   - When prompted for the system mail name, enter `mx.mojefirma.cz`.

2. **Configure Postfix for Your Domain**:

   - Edit the Postfix configuration file:

     ```sh
     sudo nano /etc/postfix/main.cf
     ```

   - Set the domain and hostnames:

     ```
     mydomain = mojefirma.cz
     myhostname = mx.mojefirma.cz
     myorigin = $mydomain
     inet_interfaces = all
     home_mailbox = Maildir/
     ```

   - Configure Postfix to handle emails for your domain:

     ```
     mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
     ```

   - Add configurations for user mail delivery to Maildir:

     ```
     mailbox_command = 
     virtual_alias_maps = hash:/etc/postfix/virtual
     ```

3. **Set up Email Redirection for root**:

   - Edit or create the virtual alias file:

     ```sh
     sudo nano /etc/postfix/virtual
     ```

   - Add the redirection rule:

     ```
     root@mojefirma.cz apribyl@mojefirma.cz
     ```

   - Update the virtual alias database:

     ```sh
     sudo postmap /etc/postfix/virtual
     ```

4. **Reload Postfix Configuration**:

   ```sh
   sudo systemctl reload postfix
   ```

5. **Test Email Delivery**:

   - Install `mailutils` to send a test email:

     ```sh
     sudo apt-get install mailutils
     ```

   - Send a test email to `apribyl`:

     ```sh
     echo "Test email to apribyl" | mail -s "Test Email" apribyl@mojefirma.cz
     ```

   - Send a test email to `root` (which should be redirected to `apribyl`):

     ```sh
     echo "Test email to root" | mail -s "Root Email" root@mojefirma.cz
     ```

Please ensure you have the necessary DNS records set up for `mx.mojefirma.cz`, including an MX record pointing to your SMTP server's IP address. Also, ensure that any firewall and/or SELinux configurations are adjusted to allow SMTP traffic (typically on port 25).

Adjust paths, domain names, and user accounts as necessary for your actual configuration. Keep in mind that managing a mail server also involves handling aspects like spam filtering and security, which are beyond the scope of this basic setup.

# IMAP 
- Instalace a konfigurace IMAP serveru pro domenu mojefirma.cz na linux
- demonstarace na aprybyl účtu z thunderbird ze stanice

  For Part 6, you will be installing and configuring an IMAP server, which is used for accessing email stored on your mail server. Dovecot is a popular open-source IMAP and POP3 server for Unix-like operating systems. After setting up the IMAP server, you will then demonstrate accessing the email using Thunderbird, a widely used email client.

**Part 6: Install and Configure IMAP Server**

1. **Install Dovecot**:

   ```sh
   sudo apt-get update
   sudo apt-get install dovecot-core dovecot-imapd
   ```

2. **Configure Dovecot**:

   - Edit the Dovecot configuration file to ensure it's listening on all interfaces and supports IMAP:

     ```sh
     sudo nano /etc/dovecot/dovecot.conf
     ```

     Add or ensure these lines exist:

     ```
     listen = *
     ```

   - Configure mailboxes and user authentication. Dovecot should work with the Maildir setup by default, but you can verify it in the following file:

     ```sh
     sudo nano /etc/dovecot/conf.d/10-mail.conf
     ```

     And ensure this setting is present:

     ```
     mail_location = maildir:~/Maildir
     ```

   - You may also need to configure SSL, depending on your requirements, in the file `/etc/dovecot/conf.d/10-ssl.conf`. For a simple setup without SSL (not recommended for production), you can set:

     ```
     ssl = no
     ```

3. **Reload Dovecot to apply changes**:

   ```sh
   sudo systemctl restart dovecot
   ```

4. **Demonstrate Using Thunderbird**:

   - On a client machine, open Thunderbird.

   - Go to the account settings and set up a new email account.

   - Enter the details for the `apribyl` account and specify the IMAP server settings:
     - Server hostname: `mx.mojefirma.cz` (or the IP address of your mail server)
     - Port: `143` for non-SSL or `993` for SSL
     - SSL: Off for non-SSL or SSL/TLS for SSL
     - Authentication: Normal password (if SSL is off)

   - Thunderbird will try to connect to the Dovecot IMAP server and retrieve emails for the `apribyl` account.

Please ensure you have configured your DNS correctly and have the necessary records for `mojefirma.cz`. For production environments, it is crucial to secure the IMAP server with SSL/TLS. You would typically obtain a certificate from a Certificate Authority (CA) and configure Dovecot to use it for secure connections.

# HTTP
- instalace web serveru s php
- vytvořit vlastní výchozí php stránku, 
- povolit uživatelské weby 
- předvést na aprybyl účtu s vlastni HTML strankou

For Part 7, we'll install a web server with PHP support, create a default PHP page, enable user directories so that users can host their own web pages, and then demonstrate this with an account.

**Part 7: Install Web Server with PHP and Set Up User Web Pages**

1. **Install Apache and PHP**:

   ```sh
   sudo apt-get update
   sudo apt-get install apache2 php libapache2-mod-php
   ```

   This will install Apache 2 and PHP, along with the module to integrate PHP with Apache.

2. **Create a Default PHP Page**:

   - Navigate to the web server's root directory (usually `/var/www/html`) and create a default PHP file:

     ```sh
     cd /var/www/html
     sudo nano index.php
     ```

   - Add the following PHP code:

     ```php
     <?php
     phpinfo();
     ?>
     ```

   - Save and exit the text editor.

3. **Enable User Websites**:

   - Enable the `userdir` module which allows users to host their web pages in their home directories:

     ```sh
     sudo a2enmod userdir
     ```

   - Restart Apache to apply the changes:

     ```sh
     sudo systemctl restart apache2
     ```

   - By default, Apache on Debian-based systems will look for user content in the `public_html` directory within the user's home directory.

4. **Create a User HTML Page for 'apribyl'**:

   - Navigate to the user's home directory and create the `public_html` directory:

     ```sh
     cd /home/apribyl
     mkdir public_html
     ```

   - Change the ownership of the `public_html` directory to the user:

     ```sh
     sudo chown apribyl:apribyl public_html
     ```

   - Navigate into this directory and create an HTML file:

     ```sh
     cd public_html
     nano index.html
     ```

   - Add some HTML content:

     ```html
     <!DOCTYPE html>
     <html lang="en">
     <head>
       <meta charset="UTF-8">
       <title>apribyl's Page</title>
     </head>
     <body>
       <h1>Welcome to apribyl's Homepage</h1>
       <p>This is a custom HTML page.</p>
     </body>
     </html>
     ```

   - Save and exit the text editor.

5. **Demonstrate the User Website**:

   - Open a web browser and navigate to `http://[Server_IP]/~apribyl`. Replace `[Server_IP]` with the actual IP address of your server.

   - You should see the custom HTML page served from the `apribyl` user's `public_html` directory.

Please replace `apribyl` with the actual username, and ensure the user has permissions to edit the contents of their `public_html` directory. For security and performance reasons, you should also consider additional configurations, such as enabling PHP opcache, configuring proper permissions, and setting up HTTPS.

# SSH
PART 8:
- zmena vychoziho portu SSH na 30332 a jeji zprovozneni na tomto portu na linux 
- zakazat TCP forwardingu
- povolit pristup jenom uctu aprybyl

For Part 8, you'll change the default SSH port to 30332, disable TCP forwarding, and restrict access to the SSH service to only the `apribyl` user account on a Linux system.

**Part 8: Change Default SSH Port, Disable TCP Forwarding, and Restrict User Access**

1. **Change the SSH Port**:

   - Open the SSH daemon configuration file:

     ```sh
     sudo nano /etc/ssh/sshd_config
     ```

   - Find the line that says `#Port 22` (the hash symbol indicates a comment) and change it to:

     ```
     Port 30332
     ```

   - If there's no `Port` line, you can add one.

2. **Disable TCP Forwarding**:

   - In the same `sshd_config` file, ensure that TCP forwarding is disabled by setting or adding the following lines:

     ```
     AllowTcpForwarding no
     X11Forwarding no
     ```

3. **Restrict SSH Access to Specific User**:

   - Further down in the `sshd_config` file, you can restrict SSH access to the `apribyl` user only by adding:

     ```
     AllowUsers apribyl
     ```

   - This line ensures that only the user `apribyl` can log in via SSH.

4. **Apply the Changes**:

   - Save the changes to the `sshd_config` file and exit the text editor.

   - Restart the SSH service to apply the new configuration:

     ```sh
     sudo systemctl restart sshd
     ```

     or

     ```sh
     sudo service ssh restart
     ```

     depending on your Linux distribution.

5. **Test the New Configuration**:

   - Try logging in with the `apribyl` account using the new SSH port to ensure everything is configured correctly:

     ```sh
     ssh apribyl@your-server-ip -p 30332
     ```

   - Remember to replace `your-server-ip` with your server's actual IP address.

**Note**: Changing the SSH port and restricting user access are significant changes. If you're doing this remotely, ensure you maintain your current session until you confirm the new setup works to prevent being locked out of your system. Also, remember to update any firewall rules to allow incoming connections on the new SSH port (30332).

---------------------------------------------------------------------------
