# Chopper WebShell ChopShop module

The Chopper Web shell communicates over TCP using HTTP POST requests. Network traffic analysis of Chopper packets can reveal the commands and files sent during an attacker's session. Because Chopper generates a POST request for each command, manual analysis can get tedious if the attacker is very active. Another challenge is when Chopper is deployed on a Web server behind SSL, causing all traffic generated by Chopper to be encrypted. The ChopShop network analysis framework allows us to develop specific decode modules to parse and decode PCAPs without having to deal with parsing underlying protocols eg. ssl, http, etc. The Chopper decode module I’ve written for the ChopShop Framework is designed to be chained with the ‘chop_ssl’ and  ‘http’ modules. To decode SSL traffic, the ‘chop_ssl’ module requires the server’s private key in RSA format.

```
webshell_chopper_decode (0.1) -- requires ChopLib 4.0 or greater:
Extract Chopper Webshell commands and output from HTTP traffic. Requires 'http' parent module.
Usage: webshell_chopper_decode [options]

Options:
  -h, --help           show this help message and exit
  -d, --dict_output    Formats output to sets of dicts
  -c, --commands_only  Only output chopper commands
  -o, --outputs_only   Only output chopper responses
  -x, --extract_pe     Attempts to extract pe files from session

```

## Installion instructions 
* Grab and Set up MITRE's ChopShop network decoder framework from https://github.com/MITRECND/chopshop
* Chopshop's HTTP module requires Python library htpy, you can grab it on MITRE's Github https://github.com/MITRECND/htpy 
* Installing modules are simple, just copy over the ChopShop module `webshell_chopper_decode.py` file to ChopShop's modules directory. 
* Note although the `chop_ssl.py` SSL decoder module is included in the default ChopShop installation, I've had some issues running it aganist certain PCAPs that had SSL reconds spanning multiple TCP segments. I've included a modified verison of the "chop_ssl.py" module with minor changes to ensure it bypasses these errors.

## Usage Examples:

Run webshell_chopper_decode module for ssl traffic and save output.
```
./chopshop –f chopper_traffic_ssl.pcap "chop_ssl -k privatekeyrsa.key | http | webshell_chopper_decode" > decoded_commands.txt
```
Run webshell_chopper_decode module and carve out any PE files seen in sessions.
```
./chopshop -s . –f chopper_traffic_ssl.pcap "chop_ssl -k privatekeyrsa.key | http | webshell_chopper_decode –x"
```
Run webshell_chopper_decode module for plain http Chopper traffic and only show commands.
```
./chopshop –f chopper_traffic_http.pcap "http | webshell_chopper_decode -c"
```

* Private Key should be in RSA format eg.:
```
-----BEGIN RSA PRIVATE KEY-----
MIICXQIBAAKBgQCt7k7fAoX2xvmRIDndTUXZXtKQJxjsFlPJgUDyD577rn4zfhcg
LYKnzyfv+yp1XkN9avaZ8ih4Ug8oSsU4aopVweqLYqvvD1ZJq+7r2xxME10S2rMn
kRvdjWGQ+4EDFSmobtsqWVfsNAi5E0ZpyQEJO485oMqTDxSco0mzgUGwUwIDAQAB
AoGAUaM+f+xmRruEHns6zcXcWDfqq2C+kOm18Cnr+vIcFFQFxlOTtTXHUs6oFzsT
5b0V/oM7Nmz0U+1oUegug9l2Dh5kIc59l2kXwAZba9PRgck+E+ub61Hj91QhMLd9
BKmJnJqdP78Xam7qRhn3lkEfzGXmsAfh2VZ5tK/Cd8U5xfkCQQDZwy+CmFk6aubM
J+qxUhMibE0WPuDF0/plf/D5/ko4/mX7MiISACdWoxWDCWzvcU43ru1KxY8IdnkK
3Nh6SwGnAkEAzHjQsG/QIayNbL5sPCmhuFeZwPXQBVWTWEdy4Ixqj1X+N0rFlN8R
QHV7+x1nKy92hpWXErxC0HkULcQq/355dQJAA8BaDCzltJzs1u2FHILmc3xcI5r3
slDBiogWtafMzYiMZzRo49h+1P5AO56o8sMH7uujiNs4aJPp5+cAD7NFFwJBAMXd
9DWJPmQX0xP0glEGCJWXUBbGyXMgCOJY4fYia8whb0yacvFJnCxAhKXRIlFMMOq0
P+nFfPK4+KoBN4rfHTECQQC5H3vc/SeWj8Q7WBO/R83u4wpiAb15/Pii3ponQP1S
R1o8ZiQSDB6T8xWPx2M+EV4QNW7jBVVtzV024dJZa3AO
-----END RSA PRIVATE KEY-----
```

* Sample Output:

```
Starting ChopShop (Created by MITRE)
Initializing Modules ...
	Initializing module 'chop_ssl'
	Initializing module 'http'
	Initializing module 'webshell_chopper_decode'
Running Modules ...
[COMMAND] at 2015-02-24 06:50:56 UTC
[Z0 Parameter] - Q1|
[Z1 Parameter] - C:\\Program Files\\Microsoft\\Exchange Server\\V14\\ClientAccess\\owa\\auth\\
[RESPONSE] at 2015-02-24 06:50:56 UTC
[Ouput] - ->|error.aspx	2013-02-05 23:00:32	8363	32
error2.aspx	2013-02-05 23:00:32	1888	32
expiredpassword.aspx	2013-02-05 23:00:32	7290	32
exppw.dll	2014-08-06 20:24:58	73392	32
logoff.aspx	2013-02-05 23:00:32	6067	32
logon.aspx	2013-02-05 23:00:32	13479	32
owaauth.dll	2014-08-06 20:23:10	104600	32
|<-
[COMMAND] at 2015-02-24 06:51:34 UTC
[Z0 Parameter] - Q1|
[Z1 Parameter] - /ccmd
[Z2 Parameter] - cd /d "C:\Program Files\Microsoft\Exchange Server\V14\ClientAccess\Owa\auth\"&dir C:\windows\temp\cmd.exe&echo [S]&cd&echo [E]
[RESPONSE] at 2015-02-24 06:51:34 UTC
[Ouput] - ->| Volume in drive C has no label.
 Volume Serial Number is 74D4-1648

 Directory of C:\windows\temp

11/20/2010  05:24 AM           345,088 cmd.exe
               1 File(s)        345,088 bytes
               0 Dir(s)  87,548,649,472 bytes free
[S]
C:\Program Files\Microsoft\Exchange Server\V14\ClientAccess\Owa\auth
[E]
|<-
[COMMAND] at 2015-02-24 07:17:41 UTC
[Z0 Parameter] - Q1|
[Z1 Parameter] - /cC:\windows\temp\cmd.exe
[Z2 Parameter] - cd /d "c:\Windows\Temp\"&net use&echo [S]&cd&echo [E]
[RESPONSE] at 2015-02-24 07:17:41 UTC
[Ouput] - ->|New connections will be remembered.


Status       Local     Remote                    Network

-------------------------------------------------------------------------------
Disconnected           \\ACMEPC\IPC$            Microsoft Windows Network
The command completed successfully.

[S]
c:\Windows\Temp
[E]
|<-
```

## Author
```
William Tan
william.tan@crowdstrike.com
```

## References
* http://blog.crowdstrike.com/chopping-packets-decoding-china-chopper-web-shell-traffic-over-ssl/
* https://github.com/MITRECND/chopshop
* https://github.com/MITRECND/htpy
* http://www.mitre.org/capabilities/cybersecurity/overview/cybersecurity-blog/decrypting-ssl-with-chopshop
* https://www.fireeye.com/content/dam/legacy/resources/pdfs/fireeye-china-chopper-report.pdf
* http://informationonsecurity.blogspot.com/2012/11/china-chopper-webshell.html
