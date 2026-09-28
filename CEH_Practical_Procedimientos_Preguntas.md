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


Aquí el procedimiento para cada uno:

# 1) Hash SHA224 de ejecutable ELF64 "angel"

file angel             # confirmar que es ELF 64-bit
sha224sum angel

Toma los últimos 4 caracteres del hash impreso.

# 2) RDP + descifrar forger.cfe + SHA1 de imagen

Descubrir hosts con RDP (puerto 3389):
nmap -p 3389 --open 10.10.55.0/24
Crackear credenciales de Jones:
hydra -l Jones -P /usr/share/wordlists/rockyou.txt rdp://[IP]
Conectarte y localizar forger.cfe:
xfreerdp /u:Jones /p:<password_encontrada> /v:[IP]
.cfe suele ser un contenedor cifrado (CryptoForge). Extrae/descifra usando la contraseña de Jones (CryptoForge Decrypt, por GUI o su CLI si está instalado en la máquina).
Una vez obtenida la imagen descifrada:
sha1sum imagen_descifrada.<ext>

Toma los últimos 6 caracteres (formato NNaaNN).

# 3) Esteganografía en .bmp de dispositivo móvil

Accede al dispositivo (vía ADB si hay debug habilitado, o vía el RAT ya desplegado si aplica):
adb connect [IP]:5555
adb shell
adb pull /sdcard/<ruta>/imagen.bmp
Analiza el BMP en busca de datos ocultos. Prueba primero herramientas comunes de esteganografía usadas en los labs de EC-Council:
steghide extract -sf imagen.bmp

Si steghide falla (no siempre funciona con BMP), prueba OpenStego o inspección con exiftool / binwalk:

binwalk imagen.bmp
exiftool imagen.bmp

El texto extraído es el secret code (formato AaaaaANa).

# 4) Base64 en archivos subidos por DVWA

Login en DVWA (admin/password) y revisa los archivos subidos en:
C:\wamp64\www\DVWA\SecureWeb\prod\

(por RDP/acceso a la máquina, o vía un LFI/path traversal si el objetivo es explotar la subida).
2. Identifica cuál archivo contiene texto base64 (ábrelos con type o cat).
3. Decodifica cada candidato:

echo "<cadena_base64>" | base64 -d

El que produzca texto legible es el mensaje original (formato AaaN*aNaN).

# 5) CVE de menor severidad tras escaneo de vulnerabilidades

Escaneo con OpenVAS (o Nessus si está disponible en el lab):
docker run -d -p 443:443 --name openvas mikesplain/openvas
Añade el target 192.168.44.32, ejecuta el scan.
En el reporte, ordena resultados por severidad ascendente y toma el CVE con el score más bajo (formato AAA-NNNN-NNNN, ej. CVE-2017-1234).
Alternativa rápida con nmap:
nmap --script vuln 192.168.44.32

# 6) pixelpioneer.txt — extracción con "password" como clave
El archivo probablemente es un contenedor esteganográfico o cifrado (no texto plano). Como sabes que la clave es literalmente password:

steghide extract -sf pixelpioneer.txt -p password

Si no es steghide sino un archivo cifrado directo (OpenSSL), prueba:

openssl enc -d -aes-256-cbc -in pixelpioneer.txt -out output.txt -k password

El contenido resultante es la credencial de 9 caracteres alfanuméricos (formato ANaa*aANaNa — ojo, el formato dado tiene más de 9, revisa si piden el resultado completo del archivo, no solo la credencial).

# 7) Static malware analysis — Image Version de Wildfire.exe
Usa PEStudio o Detect It Easy (DIE), herramientas típicas de análisis estático en los labs de CEH:

Abre Wildfire.exe en PEStudio.
Ve a la sección Header / Optional Header.
Busca el campo Image Version (o "Major/Minor Image Version").
CLI alternativa con pefile en Python:
python
import pefile
pe = pefile.PE("Wildfire.exe")
print(pe.OPTIONAL_HEADER.MajorImageVersion, pe.OPTIONAL_HEADER.MinorImageVersion)

Formato N*N sugiere algo como 6.3.

# 8) Contar archivos en carpeta "Honeywell" vía RAT

Conéctate a la sesión activa del RAT ya desplegado en la máquina objetivo (según el lab, suele ser Metasploit/Meterpreter o njRAT/AndroRAT):
msfconsole
sessions -i [id]
Dentro de la sesión Meterpreter:
cd C:\\Users\\<usuario>\\...\\Honeywell
ls
Cuenta el número de archivos listados — esa es tu respuesta (formato N).
Si el RAT es distinto, usa su comando equivalente de listado de directorio (dir, ls, list files según el cliente).

---

