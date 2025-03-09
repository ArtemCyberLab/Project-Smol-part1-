Smol is a project I decided to explore as part of a CTF challenge. It is a machine of medium difficulty, hosting a vulnerable WordPress site with two plugins that serve as the primary attack vectors. One of the plugins, jsmol2wp, is vulnerable to arbitrary file reading, and the second contains a backdoor that leads to remote code execution (RCE).

I started by scanning all TCP ports using Nmap to identify the services running on the machine. The scan revealed that both SSH and HTTP services were active. I decided to focus on the HTTP service since it was hosting a WordPress site. I added the domain www.smol.thm to my /etc/hosts file to avoid issues with domain resolution during the scan.

Next, I used WPScan for more detailed scanning of the site, which revealed that the site was using the jsmol2wp plugin version 1.07, which was vulnerable to arbitrary file reading. I quickly found that this vulnerability was documented in CVE-2018-20463. Using this vulnerability, I was able to read the wp-config.php file, which contained important details, including the credentials for the wpuser account. I used the following payload to achieve this:
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-config.php
After obtaining the credentials, I logged into the WordPress admin panel using the wpuser credentials. While exploring the admin panel, I came across a page related to webmaster tasks, which contained the code for the Hello Dolly plugin. This plugin is typically disabled because it serves no real purpose, but in this case, the code was available on the page. Upon closer inspection, I noticed a line involving the eval() function, which is a potential vulnerability for executing arbitrary code. I realized that this was a backdoor, and the code was obfuscated using base64. I used CyberChef to decode the base64-encoded string and discovered that it was executing arbitrary PHP code.

After understanding how the backdoor worked, I used it to execute arbitrary code on the server, allowing me to continue the exploitation process. In the next stage, I leveraged various Linux utilities for privilege escalation and ultimately gained root access. Overall, the project was interesting and challenging, combining classic web application vulnerabilities and providing great opportunities for privilege escalation. The backdoor in the Hello Dolly plugin played a crucial role in advancing the exploitation process.
