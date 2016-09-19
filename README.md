Silk-Seeder
===========

Silk-Seeder crawls the Silk network in order to assemble a list
of reliable nodes via its own built-in DNS server.

Features:
* Regularly revisits known nodes to check their availability
* Bans nodes after enough failures, or bad behaviour
* Accepts nodes running, at the least, v1.0.0.0 to request new IP addresses from,
  but it only reports "good" nodes; defined as nodes that run the latest release, v1.0.0.0.
* Keeps statistics over (exponential) windows of 2 hours, 8 hours,
  1 day and 1 week, to base decisions on.
* Very low memory (a few tens of megabytes) and cpu requirements.
* Crawlers run in parallel (by default 96 threads simultaneously).

REQUIREMENTS
------------

$ sudo apt-get install build-essential libboost-all-dev libssl-dev php5-cli php5-curl

Assuming you want to run a dns seed on dnsseed.example.com, you will
need an authorative NS record in example.com's domain record, pointing
to for example vps.example.com:

$ dig -t NS dnsseed.example.com

;; ANSWER SECTION
dnsseed.example.com.   86400    IN      NS     vps.example.com.


COMPILING
---------
Compiling will require boost and ssl.  On debian systems, these are provided
by `libboost-dev` and `libssl-dev` respectively.

$ make

$ chmod +x dnsseed

$ ./dnsseed

OR

$ screen -dmS <subdomain> sh -c './dnsseed"

To initiate a screen session, detach the screen session with CTRL+A+D

$ crontab -e

$ php /path/to/Silk-Seeder/cf-php/cf.php

Have a cronjob run cf-php regularly. This will run the cf.php script every minute.


RUNNING AS NON-ROOT
-------------------

A DNS server typically listens on UPD port 53.  However, to bind to a priviledged port 
(i.e. a port less than 1024), it is necessary to do one of the following:

 1. Run as root (not recommended).
 2. Redirect port 53 to a non-priviledged port, such as with iptables.
 3. Use POSIX capabilities (in Linux with support) to allow binding.

To use an iptables rule (Linux only) to redirect it to a non-privileged port:

    iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-port 5353

If properly configured, this will allow you to run dnsseed in userspace, using
the `-p 5353` option.

Alternatively, non-root binding to privileged ports is possible on Linux supporting 
"POSIX capabilities".  If the `setcap` and `getcap` commands are available,

    sudo setcap cap_net_bind_service=+ep /path/to/dnsseed