# Setup guide: PowerDNS serup

### PowerDNS module **[WHMCS](https://puqcloud.com/link.php?id=77)** 

#####  [Order now](https://puqcloud.com/index.php?rp=/store/whmcs-module-powerdns) | [Download](https://download.puqcloud.com/WHMCS/servers/PUQ_WHMCS-PowerDNS/) | [FAQ](https://faq.puqcloud.com/)

<p class="callout info">**Disclaimer:** This guide is intended for informational purposes only and provides a basic example of how to enable the API in PowerDNS. It is strongly recommended to refer to the official PowerDNS documentation for comprehensive and accurate instructions. Following official guidelines ensures that your setup is secure, reliable, and fully supported. This example may not cover all security considerations or configurations required for your specific environment. Use this guide at your own risk.</p>

## Install PowerDNS

***Update the System***

It is always safe to work with a system that is up-to-date. Updating your Debian system can be done using the simple command:

```wp-block-code
sudo apt update && sudo apt upgrade
```

Install the required tools:

```wp-block-code
sudo apt install curl vim git libpq-dev -y
```

Once all the packages have been updated to their latest stable versions, proceed with the below steps.

### 1 – Install PowerDNS Relational Database

PowerDNS supports innumerable database backends such as MySQL, PostgreSQL, Oracle e.t.c. Here, we will use the MariaDB as backend storage for PowerDNS zone files.

Install MariaDB on Debian using the below steps:

First, install the required tools:

```wp-block-code
sudo apt install software-properties-common gnupg2 -y
```

Then proceed and the MariaDB 10.6 repository on the system.

```wp-block-code
curl -LsS -O https://downloads.mariadb.com/MariaDB/mariadb_repo_setup
sudo bash mariadb_repo_setup
```

Update your package index and install MariaDB.

```wp-block-code
sudo apt update
sudo apt install mariadb-server mariadb-client
```

Once the installation is complete, start and enable MariaDB.

```wp-block-code
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

Login to the shell using the *root* user

```wp-block-code
sudo mysql -u root
```

Now create a PowerDNS database.

```wp-block-code
CREATE DATABASE powerdns;
GRANT ALL ON powerdns.* TO 'powerdns_user'@'%' IDENTIFIED BY 'Strongpassword';
FLUSH PRIVILEGES;
EXIT
```

Remember the password set for the user should ***not contain special characters*** since PowerDNS doesn’t like this and will cause the error “**Access denied for user ‘powerdns\_user’@’localhost’ (using password: YES)**“

### 2 – Install PowerDNS on Debian

We will begin by disabling the **systemd-resolved** service. This service runs on port ***53*** providing network name resolution used to load applications but now we want to use PowerDNS.

Stop and disable ***systemd-resolved*** using the commands:

```wp-block-code
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
```

Proceed and remove the symbolic link for the file.

```wp-block-code
$ ls -lh /etc/resolv.conf 
-rw-r--r-- 1 root root 49 Feb 23 04:53 /etc/resolv.conf
$ sudo unlink /etc/resolv.conf
```

Update the ***resolv.conf*** file.

```wp-block-code
$ sudo vim /etc/resolv.conf
nameserver 8.8.8.8
```

After the above adjustments, you can install PowerDNS from the default APT repositories using the command:

```wp-block-code
sudo apt install pdns-server pdns-backend-mysql
```

Install the latest release of PowerDNS available on the official [PowerDNS release](https://doc.powerdns.com/authoritative/changelog/) page. As of this guide, the stable release was at 4.6. The repository for this release can be added to the system as below.

```wp-block-code
sudo vim /etc/apt/sources.list.d/pdns.list
```

For ***Debian 12***

```wp-block-code
deb [arch=amd64] http://repo.powerdns.com/debian bookworm-auth-46 main
```

For ***Debian 11***

```wp-block-code
deb [arch=amd64] http://repo.powerdns.com/debian bullseye-auth-46 main
```

For ***Debian 10***

```wp-block-code
deb [arch=amd64] http://repo.powerdns.com/debian buster-auth-46 main
```

Import the GPG key signing for the repository.

```wp-block-code
curl -fsSL https://repo.powerdns.com/FD380FBB-pub.asc | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/pdns.gpg
```

Set the APT preferences.

```wp-block-code
$ sudo vim /etc/apt/preferences.d/pdns
Package: pdns-*
Pin: origin repo.powerdns.com
Pin-Priority: 600
```

Update your APT package index.

```wp-block-code
sudo apt update
```

Now install the PowerDNS server and the MySQL backend as below.

```wp-block-code
sudo apt install pdns-server pdns-backend-mysql
```

### 3 – Configure the PowerDNS Database

Now that we have the PowerDNS database already created on MariaDB, we will proceed and import the database schemas to it. This normally saved under the ***/usr/share/pdns-backend-mysql/schema/*** as a ***schema.mysql.sql*** file.

Now import this schema to the created database(**powerdns**) in step 1.

```wp-block-code
mysql -u powerdns_user -p powerdns < /usr/share/pdns-backend-mysql/schema/schema.mysql.sql
```

You can then verify schema import as below.

```wp-block-code
sudo mysql -u root
use powerdns;
show tables;
```

After the schema has been imported, we will now configure the PowerDNS connection details to the database.

This can be done by creating the file below.

```wp-block-code
sudo vim /etc/powerdns/pdns.d/pdns.local.gmysql.conf 
```

In the opened file, edit the lines:

```wp-block-code
# MySQL Configuration
# Launch gmysql backend
launch+=gmysql
# gmysql parameters
gmysql-host=127.0.0.1
gmysql-port=3306
gmysql-dbname=powerdns
gmysql-user=powerdns_user
gmysql-password=Strongpassword
gmysql-dnssec=yes
# gmysql-socket=
```

Set the appropriate permissions for the file.

```wp-block-code
sudo chown pdns: /etc/powerdns/pdns.d/pdns.local.gmysql.conf
sudo chmod 640 /etc/powerdns/pdns.d/pdns.local.gmysql.conf
```

You can now verify the database connection.

```wp-block-code
sudo systemctl stop pdns.service
sudo pdns_server --daemon=no --guardian=no --loglevel=9
```

With the above output, the database connection is successful. Restart and enable the PowerDNS service.

```wp-block-code
sudo systemctl restart pdns
sudo systemctl enable pdns
```

Verify the port 53 is open for DNS.

```wp-block-code
sudo ss -alnp4 | grep pdns
```

Output:

```wp-block-code
udp   UNCONN 0      0            0.0.0.0:53         0.0.0.0:*    users:(("pdns_server",pid=18530,fd=5))
tcp   LISTEN 0      128          0.0.0.0:53         0.0.0.0:*    users:(("pdns_server",pid=18530,fd=7))
```

You can also check if PowerDNS is responding to requests.

```wp-block-code
$ dig @127.0.0.1

; <<>> DiG 9.16.22-Debian <<>> @127.0.0.1
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 4882
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;.				IN	NS

;; Query time: 4 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Wed Feb 23 06:03:49 EST 2022
;; MSG SIZE  rcvd: 28
```

## To enable the API in PowerDNS

### 1 – Edit the PowerDNS Configuration File

The configuration file for PowerDNS is usually located at `/etc/powerdns/pdns.conf`. Open it for editing:

<div id="bkmrk-sudo-nano-%2Fetc%2Fpower"><div>`sudo nano /etc/powerdns/pdns.conf`</div></div>### 2 – Enable the API

Find and modify the following lines, or add them if they are not present:

<div id="bkmrk-api%3Dyeswebserver%3Dyes">```
api=yes<br></br>webserver=yes<br></br>webserver-address=0.0.0.0 <br></br>webserver-port=8081
```

</div>- `api=yes`: Enables the API.
- `webserver=yes`: Enables the web server for accessing the API.
- `webserver-address=0.0.0.0`: Configures the server to listen on all IP addresses. If you want to restrict access to a specific IP, specify that IP address here.
- `webserver-port=8081`: Specifies the port on which the API web server will be available (default is 8081).

### 3 – Configure Access from Another Server

To allow access to the API from another server, set up authentication by adding the following line in `pdns.conf`:

<div id="bkmrk-api-key%3Dyour_api_key">```ini
`api-key=your_api_key_here`
```

</div>- `api-key=your_api_key_here`: Set the API key that will be used to authenticate requests to the API. Replace `your_api_key_here` with a strong, secure key.

### 4 – Restart PowerDNS

After making these changes, restart PowerDNS to apply them:

<div id="bkmrk-sudo-systemctl-resta-0">```bash
`sudo systemctl restart pdns`
```

</div>### 5 – Test the API

From another server, test the API by making a request using the API key, for example:

<div id="bkmrk-curl--x-get--h-%27x-ap">```bash
`curl -X GET -H 'X-API-Key: your_api_key_here' http://ip_address_of_pdns_server:8081/api/v1/servers`
```

</div>Replace `your_api_key_here` with your API key and `ip_address_of_pdns_server` with the IP address of the server where PowerDNS is installed.