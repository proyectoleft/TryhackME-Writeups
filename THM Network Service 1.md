# SMB Enumeration

## SMB (Server Message Block)
**Server Message Block (SMB)** is a network communication protocol based on a client-server architecture. Its primary function is to facilitate shared and authenticated access to files, printers, serial ports, and other resources within a local network.

At a technical level, SMB is characterized by the following:

* **Communication Model:** It operates under a **request-response** scheme. The client sends specific commands (called SMBs) and the server processes the request to return the response or the requested resource.
* **Network Layer:** It functions at the **Application Layer (Layer 7 of the OSI model)**, managing resource transactions.
* **Transport:** Modern SMB implementations operate directly over **TCP/IP** using **port 445**. Historically, it relied on NetBIOS over TCP/IP (NBT) using ports 137, 138, and 139.
* **Ecosystem:** While it is the native standard for resource sharing in Microsoft Windows environments, its use has become universal thanks to open-source implementations like **Samba**, which allow full interoperability with Unix and Linux operating systems.

---

## SMB Enumeration
Enumeration is the process of gathering information about a target to identify potential attack vectors and assist in exploitation.

The first step in enumeration is performing a **port scan** to obtain as much information as possible about the services, applications, structure, and operating system of the target machine.

### Port Scan
A full port scan was conducted to identify open services:

```bash
nmap -n -Pn -sV -sC -p- --min-rate 3000 10.66.181.13
```



