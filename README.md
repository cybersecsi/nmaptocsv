[![Upload Python Package](https://github.com/cybersecsi/nmaptocsv/actions/workflows/python-publish.yml/badge.svg)](https://github.com/cybersecsi/nmaptocsv/actions/workflows/python-publish.yml)

NmaptoCSV
============

Description
-----------
A simple python script to convert Nmap output to CSV

Features
--------
* Support of Nmap version 5, 6, 7 normal format output (default format, `-oN` option)
* Support of Nmap any version Grepable format output (`-oG` option) and XML (`-oX`)
* Parsing main information : IP, FQDN, rDNS, MAC address and vendor, open ports, tcp/udp protocols, listening services and versions, OS, Number of hops to the target, and script output
* Custom output format following the main items

Usage
-----
Pass the Nmap output via stdin or from a specified file (`-i`).  
The processed dump can be collected at stdout or to a specified file (`-o`).

### Options
```
usage: nmaptocsv [-h] [-i INPUT] [-x XML_INPUT] [-o OUTPUT] [-f FORMAT] [-S]
                 [-d DELIMITER] [-n] [-s]

optional arguments:
  -h, --help            show this help message and exit

Mandatory parameters:
  -i INPUT, --input INPUT
                        Nmap scan output file in normal (-oN) or Grepable
                        (-oG) format (stdin if not specified)
  -x XML_INPUT, --xml-input XML_INPUT
                        Nmap scan output file in XML (-oX) format

Output parameters:
  -o OUTPUT, --output OUTPUT
                        CSV output filename (stdout if not specified)
  -f FORMAT, --format FORMAT
                        CSV column format { fqdn, rdns, hop_number, ip,
                        mac_address, mac_vendor, port, protocol, state, os, script,
                        service, version } (default: ip-fqdn-port-protocol-
                        service-version)
  -S, --script          Adds the script column in output, alias for -f "ip-
                        fqdn-port-protocol-service-version-script"
  -d DELIMITER, --delimiter DELIMITER
                        CSV output delimiter (default ";"). Ex: -d ","
  -n, --no-newline      Do not insert a newline between each host. By default,
                        a newline is added for better readability
  -s, --skip-header     Do not print the CSV header
```

### Nmap Normal format (default output format -oN)
```
$ python nmaptocsv.py -i test.nmap -f ip-fqdn-port-protocol-service-version-os
IP;FQDN;PORT;PROTOCOL;SERVICE;VERSION;OS
192.168.1.2;Test.lan;135;tcp;msrpc;Microsoft Windows RPC;Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
192.168.1.2;Test.lan;139;tcp;netbios-ssn;;Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
192.168.1.2;Test.lan;445;tcp;netbios-ssn;;Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
192.168.1.2;Test.lan;5357;tcp;http;Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP);Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)


$ python nmaptocsv.py -i test.nmap -f ip-fqdn-port-protocol-service-version -d ',' | csvlook
|--------------+----------+------+----------+-------------+------------------------------------------|
|  IP          | FQDN     | PORT | PROTOCOL | SERVICE     | VERSION                                  |
|--------------+----------+------+----------+-------------+------------------------------------------|
|  192.168.1.2 | Test.lan | 135  | tcp      | msrpc       | Microsoft Windows RPC                    |
|  192.168.1.2 | Test.lan | 139  | tcp      | netbios-ssn |                                          |
|  192.168.1.2 | Test.lan | 445  | tcp      | netbios-ssn |                                          |
|  192.168.1.2 | Test.lan | 5357 | tcp      | http        | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)  |
|  |
|--------------+----------+------+----------+-------------+------------------------------------------|

$ nmap -sV -p- localhost -oN - | python nmaptocsv.py 
IP;FQDN;PORT;PROTOCOL;SERVICE;VERSION
127.0.0.1;localhost;22;tcp;ssh;OpenSSH 7.6p1 Debian 2 (protocol 2.0)
127.0.0.1;localhost;80;tcp;http;Apache httpd 2.4.29 ((Debian))
127.0.0.1;localhost;5432;tcp;postgresql;PostgreSQL DB 9.5.4
127.0.0.1;localhost;5433;tcp;postgresql;PostgreSQL DB 9.6.0 or later
127.0.0.1;localhost;5434;tcp;postgresql;PostgreSQL DB 9.6.0 or later
```

### Nmap Grepable format (-oG)
```
$ cat scan.gnmap
# Nmap 6.01 scan initiated Thu Nov 22 11:28:15 2012 as: nmap -p- -sV -oA scan 10.0.0.0/24 
Host: 10.0.0.1 (test1.local)	Status: Up
Host: 10.0.0.1 (test1.local)	Ports: 23/open/tcp//telnet//Cisco router telnetd/	Ignored State: closed (65534)
Host: 10.0.0.2 (test2.local)	Status: Up
Host: 10.0.0.2 (test2.local)	Ports: 23/open/tcp//telnet//Cisco router telnetd/	Ignored State: closed (65534)
Host: 10.0.0.3 (test3.local)	Status: Up
Host: 10.0.0.3 (test3.local)	Ports: 23/open/tcp//telnet//Cisco router telnetd/	Ignored State: closed (65534)
Host: 10.0.0.50 (test50.local)	Status: Up
Host: 10.0.0.50 (test50.local)	Ports: 22/open/tcp//ssh//OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)/, 80/open/tcp//http//Apache httpd 1.3.33 ((Debian GNU|Linux) PHP|4.3.10-19)/, 111/open/tcp//rpcbind (rpcbind V2)/(rpcbind:100000*2-2)/2 (rpc #100000)/, 113/open/tcp//ident///, 684/open/tcp//status (status V1)/(status:100024*1-1)/1 (rpc #100024)/, 5432/open/tcp//postgresql//PostgreSQL DB (French)/	Ignored State: closed (65529)
Host: 10.0.0.100 (test100.local)	Status: Up
Host: 10.0.0.100 (test100.local)	Ports: 80/closed/tcp//http///, 5432/open/tcp//postgresql//PostgreSQL DB (French)/, 19999/filtered/tcp/////	Ignored State: closed (65532)

$ python nmaptocsv.py -i scan.gnmap -f ip-fqdn
IP;FQDN
10.0.0.1;test1.local

10.0.0.2;test2.local

10.0.0.3;test3.local

10.0.0.50;test50.local

10.0.0.100;test100.local


$ cat scan.gnmap | python nmaptocsv.py 
IP;FQDN;PORT;PROTOCOL;STATE;SERVICE;VERSION
10.0.0.1;test1.local;23;tcp;open;telnet;Cisco router telnetd

10.0.0.2;test2.local;23;tcp;open;telnet;Cisco router telnetd

10.0.0.3;test3.local;23;tcp;open;telnet;Cisco router telnetd

10.0.0.50;test50.local;22;tcp;open;ssh;OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
10.0.0.50;test50.local;80;tcp;open;http;Apache httpd 1.3.33 ((Debian GNU|Linux) PHP|4.3.10-19)
10.0.0.50;test50.local;111;tcp;open;rpcbind (rpcbind V2);(rpcbind:100000*2-2)/2 (rpc #100000)
10.0.0.50;test50.local;113;tcp;open;ident;
10.0.0.50;test50.local;684;tcp;open;status (status V1);(status:100024*1-1)/1 (rpc #100024)
10.0.0.50;test50.local;5432;tcp;open;postgresql;PostgreSQL DB (French)

10.0.0.100;test100.local;80;tcp;closed;http;
10.0.0.100;test100.local;5432;tcp;open;postgresql;PostgreSQL DB (French)
```

Dependencies and installation
-----------------------------
* A Python interpreter with version 2.7 or 3.X
  * Python 2.6 works with .nmap and .gnmap files but not with .xml ones
* The **easiest way** to setup everything: `pip install nmaptocsv` and then directly use `$ nmaptocsv`
  * Or git clone that repository and `pip install -r requirements.txt` and then `python nmaptocsv.py`

Changelog
---------
* version 1.7.1 - 15/07/2025: adding "State" clomun support on nmap XML input
* version 1.7 - 09/07/2025: Publishing package on PyPI and added "State" columns from nmap output
* version 1.6 - 06/13/2019: Python 3 support
* version 1.5 - 09/30/2018: few bugs fixed in XML parsing + script alias format added
* version 1.4 - 08/16/2018: few bugs fixed + XML parsing implementation
* version 1.3 - 08/13/2018: fixing the README layout on PyPI
* version 1.2 - 01/07/2018: script output parsing addition
* version 1.1 - 01/07/2018: Cleaning the code, fixing some parsing bugs, delimiter option addition and made newline default
* version 1.0 - from 2012 to 2013

Copyright and license
---------------------
Nmaptocsv is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

Nmaptocsv is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  

See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with nmaptocsv. 
If not, see http://www.gnu.org/licenses/.

Author
-------
* Thomas Debize < tdebize at mail d0t com >

Contact
-------
* Emanuele Galdi < emanuele.galdi at secsi d0t io >