# Florida Tech: BIND9 DNS Cheat Sheet

This markdown is to be used is a cheatsheet for installing, troubleshooting, and searching for artifacts related to Dr. Abdullah's DNS Research at Florida Tech.

## DISCLAIMER:

This cheatsheet is meant to provide a quick and easy to digest list of commands and resources to bridge the knowledge gap of the existing implementation/ plans.
If any information such as detailed descriptions and definitions are needed, please use online resources such as ChatGPT, Official Documentation, or the professor if necessary.

## Quick Links

1) [BIND9](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#bind9)
2) [CACHING](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#caching)
3) [Configuration Files/ Directories](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#configuration-files-directories)
4) [DNSPerf](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#dnsperf)
5) [DNSSEC](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#dnssec)
6) [DOH (Incomplete)](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#doh-incomplete)
7) [DOT](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#dot)
8) [FIU 5G Testbed](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#fiu-5g-testbed)
9) [Virt-Manager](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#virt-manager)
10) [Additional Troubleshooting Tips](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#additional-troubleshooting-tips)



## BIND9

Installation

``` 
sudo apt update
sudo apt install bind9

```

Test install using any CLI dns query tool such as dig or nslookup,
Note you must give it your machine ip address or localhost example:

```
nslookup google.com 127.0.0.1
```

Bind 9 firewall:
This may not be a problem in most cases but if needed, to allow bind9 through firewall run the following commmand:

```
sudo ufw allow bind9
```


Modify Bind9 configuration files:

```
sudo nano /etc/bind/named.conf.options

```

Replace any contents in the file with:

```
options {
    directory "/var/cache/bind";
    recursion yes;
    listen-on { any; };
    allow-query { any; };
    forwarders {
            8.8.8.8;
            8.8.4.4;
    };
};

```



```
sudo nano /etc/bind/named.conf.local
```

Replace any contents in the file with:

```
zone "domain-name.com" {
    type master;
    file "/etc/bind/db.domain-name.com";
    allow-transfer { xx.xx.xx.xx; };
    also-notify { xx.xx.xx.xx; };
};
```


Make a directory to store all zone files

```
mkdir zones
```


Create a file for your zone

```
nano db.domain-name.com
```

Paste the following in the file. Modify this template as necessary.


```
$TTL    604800
@       IN      SOA     ns1.domain-name.com. root.domain-name.com. (
                  3       ; Serial
             604800     ; Refresh
              86400     ; Retry
            2419200     ; Expire
             604800 )   ; Negative Cache TTL
;
; name servers - NS records
     IN      NS      ns1.domain-name.com.

; name servers - A records
ns1.domain-name.com.         IN      A      172.20.0.2

host1.domain-name.com.       IN      A      172.20.0.4
host2.domain-name.com.       IN      A      172.20.0.5
host3.domain-name.com.       IN      A      172.20.0.6
host3.domain-name.com.       IN      A      172.20.0.7
host4.domain-name.com.       IN      A      172.20.0.8

```


To check if bind9 configuration files are error free run the following command and then restart bind9 to ensure changes are made

```
sudo named-checkconf
systemctl restart bind9
```

The following commands are also useful related to bind9.
You can stop the service, start/restart or check the status for errors:


```
systemctl stop bind9
systemctl start bind9
systemctl restart bind9
systemctl status bind9
```



## Caching

I never really dug too deep into how exactly the bind9 cache works but based on what I've seen:


The directory the cache is stored is set in the bind9 named.conf.options file in the following line:

```
directory "/var/cache/bind";
```

In order to actually get the cache file if you need to inspect it run:

```
sudo rndc dumpdb -all
```

To delete that file run

```
sudo rm named_dump.db
```

To actually clear the cache run

```
sudo rndc flush
```
I usually run it a few times just to be sure.




## Configuration Files/ Directories 

The following contains a list of directories that might help.


### /etc/bind
All bind9 related config files

### /etc/bind/named.conf.options
Main configuration file for bind9 

### /etc/bind/named.conf.local
Bind9 configuration for zone file and local changes.

### /etc/bind/zones
Where zones are currently stored

### /var/cache/bind
Bind9 dumps the cache here
The keys generated for DNSSEC are also there


### /etc/stunnel/stunnel.conf
Stunnel configuration file used for DOT set up.


### /etc/nginx/sites-available/default
This file stores the nginx configuration used in the DOH set up. 

### cd ~/UERANSIM/config
Configuration files for UERANSIM used on gNB and UE for the 5g set up.

### /install/etc/open5gs
Configuration files used on CP and UP for the 5g set up.



## DNSPerf
DNSperf is used in testing when multiple queries need to be sent easily. It also generates performance data such as minimum, maximum and average latency.

To install run:

```
sudo apt-get update
sudo apt-get install -y build-essential libtool automake autoconf libpcap-dev libssl-dev
wget https://www.dns-oarc.net/files/dnsperf/dnsperf-2.3.2.tar.gz

```

Extract the files and change directory

```
tar -xvf dnsperf-2.3.2.tar.gz
cd dnsperf-2.3.2

```

These dependencies may also be missing from the system:

```
sudo apt-get install -y libbind-dev
sudo apt-get install -y libkrb5-dev
```

Build

```
./configure
make && sudo make install
dnsperf -v
```


Dnsperf can be used differently depending on what needs to be done so for specific please use online resources, but the following is an example:

```
dnsperf -d file.txt -s 127.0.0.1 -c 10 -Q 10000 -l 10
```

This command is telling the dnsperf tool to send 10,000 DNS queries to a local DNS server running on the same machine, using 10 concurrent clients, and to run the test for 10 seconds. The tool will read the list of DNS queries from the file.txt input file.


If testing DNSSEC include "-D"

for example:

```
dnsperf -D -d file.txt -s 127.0.0.1 -c 10 -Q 10000 -l 10
```

If testing DOT set the mode to tls by including "-m tls"

for example:

```
dnsperf -m tls -d file.txt -s 127.0.0.1 -c 10 -Q 10000 -l 10
```

If testing both DNSSEC + DOT combined, just include both

for example:

```
dnsperf -D -m tls -d file.txt -s 127.0.0.1 -c 10 -Q 10000 -l 10
```

## DNSSEC

Install [bind](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#bind9)

Configure bind options to allow for dnssec

```
cd /etc/bind/
sudo nano named.conf.options
```

Add the following to the existing options

```
options {
   
             ... ;
             ... ;
             ... ;
             dnssec-enable yes;
	     dnssec-validation yes;
	     dnssec-lookaside auto;
}
```

Make sure there are no issues with configuration file

```
named-checkconf
```


Generate Keys

#### Disclaimer: zone name much match zone name specified in /etc/bind/named.conf.local
Replace all names accoridngly

```
cd /var/cache/bind
```

Create zone signing key (ZSK): 
 
 ```
 sudo dnssec-keygen -a NSEC3RSASHA1 -b 2048 -n ZONE jerrel-ns.com
 ```
 
Create key signing key (KSK): 

```
sudo dnssec-keygen -f KSK -a NSEC3RSASHA1 -b 4096 -n ZONE jerrel-ns.com
```
  
Note: 
Kxxxxxx.key are public keys
Kxxxxxx.private are private keys
      
Public keys must be added as records in zone file.

Use script to add public keys to zone file

```
nano migrate.sh
```

Save the following into the file

```
for key in `ls Kjerrel-ns.com*.key`
do
echo "\$INCLUDE $key">> /etc/bind/zones/db.jerrel-ns.com
done
```

make it executable 

```
chmod +x file.sh
```

modify the script to match the patterns of your key file names ... 
my key files in this example are:

Kjerrel-ns.com.+007+28483.key      Kjerrel-ns.com.+007+41844.private 
Kjerrel-ns.com.+007+41844.key Kjerrel-ns.com.+007+28483.private


run file: 
```
sudo ./file.sh
```



Check to make sure the entries got added. Change to wherever ur zone file is stored.

```
cat etc/bind/zones/db.jerrel-ns.com
```



```
-----------------------FILE-----------------------------
.
.
.
; name servers - A records
ns2.jerrel-ns.com.          IN      A      192.168.222.5

host1.jerrel-ns.com.        IN      A      10.0.2.15
host2.jerrel-ns.com.        IN      A      10.0.2.6
host3.jerrel-ns.com.        IN      A      10.0.2.7
host3.jerrel-ns.com.        IN      A      10.0.2.8
host4.jerrel-ns.com.        IN      A      10.0.2.9
$INCLUDE Kjerrel-ns.com.+007+28483.key
$INCLUDE Kjerrel-ns.com.+007+41844.key

-----------------------FILE ----------------------------
```


Sign the files 

```
sudo dnssec-signzone -A -N INCREMENT -o jerrel-ns.com -t /etc/bind/zones/db.jerrel-ns.com
````

Output should look like:

```
Verifying the zone using the following algorithms: NSEC3RSASHA1.
Zone fully signed:
Algorithm: NSEC3RSASHA1: KSKs: 1 active, 0 stand-by, 0 revoked
                         ZSKs: 1 active, 0 stand-by, 0 revoked
/etc/bind/zones/db.jerrel-ns.com.signed
Signatures generated:                       15
Signatures retained:                         0
Signatures dropped:                          0
Signatures successfully verified:            0
Signatures unsuccessfully verified:          0
Signing time in seconds:                 0.020
Signatures per second:                 750.000
Runtime in seconds:                      0.036
```


Check to make sure a new zonefile was created. it should be in the same directory as your original zone file. with ".signed" appended to the name

```
ls /etc/bind/zones
```


Inspect the new file

Update /etc/bind/named.conf.local with the new signed file.

reload bind 

```
service bind9 reload
```

Run to test 

```
dig DNSKEY jerrel-ns.com. @localhost +multiline
```

```
dig A jerrel-ns.com. @localhost +noadditional +dnssec +multiline
```

```
kdig @192.168.222.5 +dnssec bbc.co.uk
```

[DNSSEC](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#dnssec) + [DOT](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#dot)

```
kdig @192.168.222.5 +tls +dnssec bbc.co.uk
```


## DOH (Incomplete)


To implement DNS-over-HTTPS (DOH) on your Bind9 server, you can follow these general steps:

> Install a web server, such as Apache or Nginx, on your Bind9 server.

> Install Certbot, a free and open-source tool, to obtain an SSL/TLS certificate for your web server. Certbot can automatically configure your web server to use HTTPS and keep your certificate up-to-date.

> Install the "mod_proxy" module for your web server. This module enables your web server to act as a reverse proxy, which can forward incoming DOH requests to your Bind9 server.

> Configure your web server to act as a reverse proxy for DOH traffic. For example, in Nginx, you can add the following location block to your configuration file


Install [bind](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#bind9)

Install Web server: Nginx

```
sudo apt-get update
sudo apt-get install nginx
```

Install Certbot for certificates

```
sudo apt-get update
sudo apt-get install certbot python3-certbot-nginx
```


Configure your web server to use HTTPS and obtain a certificate from Let's Encrypt

Certbot can automatically configure your web server to use HTTPS and obtain a certificate from Let's Encrypt, a free certificate authority that provides trusted SSL/TLS certificates. To use Certbot with Nginx, run the following command:
    
```    
sudo certbot --nginx
```

Install the mod_proxy module for your web server
The mod_proxy module is required to enable your web server to act as a reverse proxy for DOH traffic. To install mod_proxy for Nginx, run the following command:

```
sudo apt-get install nginx-extras
```


Configure your web server to act as a reverse proxy for DOH traffic
Note, the port in this file, needs to correspond with the port that you specify in your Bind9 configurations
In this example we are using port 8053


```
sudo nano /etc/nginx/sites-available/default
```

```
server {
    ...
    location /dns-query {
        proxy_pass http://127.0.0.1:8053/dns-query;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
    }
    ...
}
```


Configure your Bind9 server to listen on the specified port for DOH requests
Add the following options to your Bind9 configuration file (/etc/bind/named.conf.options) to listen on port 8053 for DOH requests:
Note the same port 8053 from step 5

```
sudo nano /etc/bind/named.conf.options
```


```
options {
    ...
    listen-on-v6 { any; };
    listen-on port 8053 { any; };
    ...
};
```

Restart Bind9 and Nginx

```
sudo systemctl restart nginx
sudo systemctl restart bind9
```


Additional Notes:

To debug:

```
sudo systemctl status nginx
sudo systemctl status bind9
```

#### To disable DOH:

First stop the nginx service

```
sudo systemctl stop nginx
```


Then remove the changes from step 5 and revert to the original
In this case just removing port 8053 would be fine.

```
sudo nano /etc/bind/named.conf.options
```

FROM 

```
options {
    ...
    listen-on-v6 { any; };
    listen-on port 8053 { any; };
    ...
};
```

TO

```
options {
    ...
    listen-on-v6 { any; };
    listen-on { any; };
    ...
};
```

Reminder:
You can you ```sudo named-checkconf``` to see if your configuration files are good


Test using curl following this format

```
curl -H 'accept: application/dns-json' 'https://8.8.8.8/resolve?name=https://www.gov.tt'
```

This example uses the google DNS. DOH is incomplete since curl does not work, but the above steps can be used to begin troubleshooting



## DOT

Install [bind](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#bind9)

Open the main configuration file.

```
sudo nano /etc/bind/named.conf.options
```

Add the following to the Bind configuration

```
acl trusted {
    127.0.0.0/8;
};


options {
    directory "/var/cache/bind";
    recursion yes;
    forwarders {
            8.8.8.8;
            8.8.4.4;
    };

    // DOT with stunnel
    listen-on-v6 { none; };
    listen-on { 127.0.0.1; };
    allow-query { trusted; };
};
```


Set up DOT using stunnel

```
sudo apt update
sudo apt install stunnel4
sudo openssl req -new -x509 -nodes -out /etc/stunnel/stunnel.pem -keyout /etc/stunnel/stunnel.pem -days 3650
sudo nano /etc/stunnel/stunnel.conf
```

Paste the following into the file


```
sudo nano /etc/stunnel/stunnel.conf
```

```
------ file    ---------
pid = /run/stunnel4/stunnel.pid
cert = /etc/stunnel/stunnel.pem
options = NO_SSLv2
options = NO_SSLv3
options = NO_TLSv1
options = NO_TLSv1.1
options = SINGLE_ECDH_USE
options = SINGLE_DH_USE
ciphers = HIGH:!aNULL:!eNULL:!SSLv2:!SSLv3
[dot]
accept = 853
connect = 127.0.0.1:53
------ file ---------
```


Restart Bind9 and Stunnel

```
sudo systemctl restart bind9
sudo systemctl restart stunnel4
```


To test:

```
kdig @192.168.222.5 +tls bbc.co.uk
```

[DNSSEC](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#dnssec) + [DOT](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#dot)

```
kdig @192.168.222.5 +tls +dnssec bbc.co.uk
```



If changing the conf makes BIND9 status = failed
Try rerunning

```
sudo systemctl stop bind9
sudo systemctl start bind9
sudo systemctl restart bind9
sudo systemctl status bind9
```

If it still fails try restarting without stopping

```
sudo systemctl restart bind9
sudo systemctl status bind9
```



## FIU 5G Testbed


#### NOTE: DNS server can be set up either on gNB directly, or on a separate vm, as long as the controller is set up on gNB.

* To set up the testbed use the following youtube link
* To set up flows, use Ricardo's script. It is currently located on the Desktop of gNB.
* If Installation steps are needed, request permission from Ricardo for his GitHub.
* For anything else related to the 5G set up, please contact Diana or Ricardo.



## OVS

Install:

```
sudo apt install openvswitch-switch -y 
```

Create a switch

```
sudo ovs-vsctl add-br br0    : 
```

```
sudo apt install net-tools -y 
```


Assigns ip address to switch/ bridge

```
sudo ifconfig br0 192.168.233.1/24   
```

```
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -s 192.168.233.0/24 -o ens160 -j MASQUERADE
```
ens160 just represents the management port. It may be eth0..ens…en0….etc etc



creates ports for each vm to connect to (gNB, UP, CP, UE etc)

```
sudo ovs-vsctl add-port br0 cp -- set Interface cp type=internal    : 
sudo ovs-vsctl add-port br0 up -- set Interface up type=internal  
sudo ovs-vsctl add-port br0 ue -- set Interface ue type=internal  
sudo ovs-vsctl add-port br0 gnb -- set Interface gnb type=internal  
```


## Virt-Manager

Install

```
sudo apt install virt-manager 
```

This is a gui, so should be fairly intuitive.
Note: There were plans to use the CLI "virsh" in conjunction with virt-manager to get more NFV features



--------------------------------------------------------------------------------------------------------------------------------------------------------------------


## Additional Troubleshooting Tips

* Sometimes the vms may be disconnected from the internet and you may be unsure why. running this on the main workstation/ whereever your switch is fixes it usually.

```
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -s 192.168.233.0/24 -o ens160 -j MASQUERADE
```
Change the IP accoridngly 



* When making changes to bind, sometimes it breaks. The following commands can help you narrow down the problem:

```
sudo systemctl status bind9
sudo named-checkconf
```

* Clear [cache](https://github.com/jerrelgordon/fit-dns-research-cheatsheet/blob/main/README.md#caching) before experiments for external DNS records. Might have to clear after each query for that record.

```
sudo rndc flush
```

* If debugging nginx for DOH set up use

```
sudo systemctl status nginx
```

