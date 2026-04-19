SIS313 Lab 3.1: Proxy Inverso con Balanceador de Carga (Least Connection)
Guía Completa Paso a Paso para 4 Personas - 7 VMs

Versión: 3.0 (Actualizada: Sin hotspot, Red Interna al crear VMs)
Fecha: 2026-04-19
Red Interna: 172.16.100.0/27 (IPs Estáticas)

================================================================================
1. INFORMACIÓN DEL GRUPO
================================================================================

INTEGRANTES

# | Nombre               | Rol                        | Laptop    | VMs
1 | [Nombre Persona 1]   | Proxy + Balanceador        | Laptop 1  | VM1
2 | [Nombre Persona 2]   | WebServers PHP             | Laptop 2  | VM2, VM4
3 | [Nombre Persona 3]   | WebServers Node.js         | Laptop 3  | VM3, VM5
4 | [Nombre Persona 4]   | WebServers Mixtos          | Laptop 4  | VM6, VM7

RESPONSABILIDADES

Persona 1 (Proxy-LB):
- Crear VM1 (Proxy-LB) en Laptop 1
- Instalar Ubuntu 24.04 Server
- Configurar IPs estáticas con YAML Netplan (NAT + red interna)
- Habilitar IP Forwarding y NAT
- Instalar y configurar NGINX como proxy inverso
- Configurar algoritmo de balanceo: least_conn
- Pruebas de balanceo y distribución

Persona 2 (WebServers PHP):
- Crear VM2 (webserver1) en Laptop 2
- Crear VM4 (webserver3) en Laptop 2
- Instalar Alpine Linux en ambas
- Configurar IPs estáticas con /etc/network/interfaces
- Instalar NGINX + PHP-FPM
- Crear aplicación "Hola Mundo" en PHP

Persona 3 (WebServers Node.js):
- Crear VM3 (webserver2) en Laptop 3
- Crear VM5 (webserver4) en Laptop 3
- Instalar Alpine Linux en ambas
- Configurar IPs estáticas con /etc/network/interfaces
- Instalar NGINX + Node.js
- Crear aplicación "Hola Mundo" en Node.js

Persona 4 (WebServers Mixtos):
- Crear VM6 (webserver5) en Laptop 4
- Crear VM7 (webserver6) en Laptop 4
- Instalar Alpine Linux en ambas
- Configurar IPs estáticas con /etc/network/interfaces
- Una con PHP-FPM, otra con Node.js
- Crear ambas aplicaciones Web

================================================================================
2. ARQUITECTURA DE RED
================================================================================

TABLA DE CONFIGURACIÓN

# | VM         | Hostname    | IP Interna    | Gateway      | Netmask        | Laptop | SO            | Tipo
1 | Proxy-LB   | proxy-lb    | 172.16.100.1  | N/A          | 255.255.255.224| L1     | Ubuntu 24.04  | Proxy
2 | WebServer1 | webserver1  | 172.16.100.2  | 172.16.100.1 | 255.255.255.224| L2     | Alpine 3.22   | PHP
3 | WebServer2 | webserver2  | 172.16.100.3  | 172.16.100.1 | 255.255.255.224| L3     | Alpine 3.22   | Node.js
4 | WebServer3 | webserver3  | 172.16.100.4  | 172.16.100.1 | 255.255.255.224| L2     | Alpine 3.22   | PHP
5 | WebServer4 | webserver4  | 172.16.100.5  | 172.16.100.1 | 255.255.255.224| L3     | Alpine 3.22   | Node.js
6 | WebServer5 | webserver5  | 172.16.100.6  | 172.16.100.1 | 255.255.255.224| L4     | Alpine 3.22   | PHP
7 | WebServer6 | webserver6  | 172.16.100.7  | 172.16.100.1 | 255.255.255.224| L4     | Alpine 3.22   | Node.js

RED: 172.16.100.0/27 (Compartida entre Laptops)
GATEWAY: 172.16.100.1 (Proxy-LB)

================================================================================
3. PREPARACIÓN DEL ENTORNO
================================================================================

REQUISITOS PREVIOS

✓ 4 Laptops (Windows, macOS o Linux)
✓ VirtualBox 7.2.6 o superior instalado en cada Laptop
✓ ISOs descargadas:
  - Ubuntu 24.04 Server (2.6 GB)
  - Alpine Linux 3.22 (200 MB)

REQUISITOS DE ALMACENAMIENTO

- Laptop 1: 10 GB (Proxy-LB)
- Laptop 2: 12 GB (2x WebServers de 6 GB)
- Laptop 3: 12 GB (2x WebServers de 6 GB)
- Laptop 4: 12 GB (2x WebServers de 6 GB)

================================================================================
4. CREACIÓN DE VMS EN VIRTUALBOX
================================================================================

INFORMACIÓN IMPORTANTE - VirtualBox 7.2.6

La red interna se crea al configurar cada VM, NO en Preferencias.
Cuando crees una VM, en la configuración de red selecciona:
- Red Interna (Internal Network)
- Nombre: SIS313-RedInterna
VirtualBox creará automáticamente la red si no existe.

---

LAPTOP 1: VM1 - PROXY-LB (UBUNTU 24.04)

Paso 1: Crear la VM

En VirtualBox, haz clic en "Crear"

Completa el formulario:
  Nombre: Proxy-LB
  Ruta de carpeta: (Dejar por defecto)
  Imagen ISO: ubuntu-24.04.3-live-server-amd64.iso
  Tipo: Linux
  Versión: Ubuntu (64-bit)
  Memoria base: 2048 MB
  Procesadores: 2
  Tamaño de disco: 10 GB
  Tipo de disco: VDI
  Almacenamiento: Dinámico

Paso 2: Configurar Adaptadores de Red

Adaptador 1 (NAT):
  - Conectado a: NAT
  - Dejar valores por defecto

Adaptador 2 (Red Interna):
  - Conectado a: Red Interna (Internal Network)
  - Nombre: SIS313-RedInterna

Paso 3: Finalizar
  Haz clic en "Crear"

---

LAPTOP 2: VM2 Y VM4 - WEBSERVER1 Y WEBSERVER3 (ALPINE)

VM2: WebServer1

Haz clic en "Crear"

Completa:
  Nombre: webserver1
  Imagen ISO: alpine-standard-3.22.1-x86_64.iso
  Tipo: Linux
  Versión: Other Linux (64-bit)
  Memoria base: 1024 MB
  Procesadores: 1
  Tamaño de disco: 6 GB
  Tipo de disco: VDI
  Almacenamiento: Dinámico

Adaptador de Red:
  - Conectado a: Red Interna (Internal Network)
  - Nombre: SIS313-RedInterna

VM4: WebServer3

Repite el proceso:
  Nombre: webserver3
  (Resto igual a WebServer1)

---

LAPTOP 3: VM3 Y VM5 - WEBSERVER2 Y WEBSERVER4 (ALPINE)

VM3: WebServer2
  Nombre: webserver2
  (Resto igual a WebServer1)

VM5: WebServer4
  Nombre: webserver4
  (Resto igual a WebServer1)

---

LAPTOP 4: VM6 Y VM7 - WEBSERVER5 Y WEBSERVER6 (ALPINE)

VM6: WebServer5
  Nombre: webserver5
  (Resto igual a WebServer1)

VM7: WebServer6
  Nombre: webserver6
  (Resto igual a WebServer1)

================================================================================
5. INSTALACIÓN Y CONFIGURACIÓN UBUNTU (PROXY)
================================================================================

INSTALAR UBUNTU 24.04 EN PROXY-LB

En Laptop 1:

1. Inicia la VM Proxy-LB
2. Selecciona ubuntu-24.04.3-live-server-amd64.iso
3. Sigue el instalador
4. En Network configuration: Dejar valores por defecto
5. Completa instalación con:
   - Username: usuario
   - Password: tu contraseña
   - Install OpenSSH server: Marcar

POST-INSTALACIÓN UBUNTU

Login con usuario creado:
  login: usuario
  password: (tu contraseña)

Actualizar sistema:
  sudo apt update
  sudo apt upgrade -y

Instalar herramientas:
  sudo apt install -y nano curl wget git htop net-tools

Verificar interfaces de red:
  ip link show
  ip addr show

================================================================================
6. INSTALACIÓN Y CONFIGURACIÓN ALPINE (WEBSERVERS)
================================================================================

INSTALAR ALPINE LINUX (6 VMs)

En cada WebServer Alpine (Laptops 2, 3, 4):

1. Inicia la VM
2. Selecciona alpine-standard-3.22.1-x86_64.iso
3. Login como root (sin contraseña)
4. Ejecuta: setup-alpine

PASOS DE CONFIGURACIÓN setup-alpine

Select keyboard layout [none]: us
Select variant: us
Hostname: webserver1 (cambiar según VM: webserver2, webserver3, etc.)

Interface:
  Which one do you want to initialize? [eth0]: eth0
  IP address for eth0? [dhcp]: dhcp
  Do you want to do any manual network configuration? [n]: n

Root password:
  New password: (ingresa tu contraseña)
  Retype password: (confirma)

Timezone:
  Which timezone are you in? [UTC]: America/La_Paz

Proxy:
  HTTP/FTP proxy URL? [none]: none

Network Time Protocol:
  Which NTP client to run? [busybox]: busybox

APK Mirror:
  Enter mirror number or URL: [1]: 1

User:
  Setup a user? [no]: marcelo
  Full name for user marcelo [marcelo]: Marcelo Quispe Ortega
  New password: (contraseña)
  Retype password: (confirma)
  Enter ssh key or URL for marcelo [none]: none
  Which ssh server? [openssh]: openssh

Disk & Install:
  Which disk(s) would you like to use? [none]: sda
  How would you like to use it? [?]: sys
  WARNING: Erase the above disk(s) and continue? [n]: y

Espera a que termine... "Installation is complete. Please reboot."

POST-INSTALACIÓN

Apaga la VM:
  poweroff

En VirtualBox:
  - Clic derecho en VM → Configuración
  - → Almacenamiento
  - Selecciona "IDE Secundario (CD/DVD)"
  - Click en el ícono de CD/DVD y selecciona "Desconectar"

Reinicia la VM y ejecuta:

  login: root
  password: (contraseña que configuraste)

Instalar herramientas:
  apk add nano curl wget htop

Instalar OpenSSH:
  apk add openssh
  rc-service sshd start
  rc-update add sshd

Verificar interfaces:
  ip link show
  ip addr show

================================================================================
7. CONFIGURACIÓN DE IPS ESTÁTICAS (YAML + INTERFACES)
================================================================================

PROXY-LB (UBUNTU 24.04) - YAML NETPLAN

En Laptop 1, en Proxy-LB:

Editar configuración:
  sudo nano /etc/netplan/00-installer-config.yaml

Borrar TODO y reemplazar con:

---START---
network:
  version: 2
  renderer: networkd
  ethernets:
    # Adaptador 1: NAT (acceso a internet)
    enp0s3:
      dhcp4: true
      dhcp4-overrides:
        use-dns: true
        use-ntp: true
      optional: true
    
    # Adaptador 2: Red Interna VirtualBox (IP 172.16.100.1)
    enp0s8:
      dhcp4: false
      addresses:
        - 172.16.100.1/27
      routes:
        - to: 172.16.100.0/27
          via: 172.16.100.1
          on-link: true
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
---END---

Aplicar configuración:
  sudo netplan validate
  sudo netplan apply
  ip addr show

Resultado esperado:
  enp0s3: inet 10.0.2.X/24 (NAT)
  enp0s8: inet 172.16.100.1/27 (Red Interna)

Ver rutas:
  ip route show

Verificar conectividad:
  ping 8.8.8.8

---

WEBSERVER1 (ALPINE) - IP: 172.16.100.2 - LAPTOP 2

En Laptop 2, en WebServer1:

  login: root
  nano /etc/network/interfaces

Reemplazar TODO con:

---START---
auto eth0
iface eth0 inet static
    address 172.16.100.2
    netmask 255.255.255.224
    gateway 172.16.100.1
    dns-nameservers 8.8.8.8 8.8.4.4
---END---

Aplicar cambios:
  /etc/init.d/networking restart
  ip addr show
  ping 172.16.100.1

---

WEBSERVER2 (ALPINE) - IP: 172.16.100.3 - LAPTOP 3

En Laptop 3, en WebServer2:

  login: root
  nano /etc/network/interfaces

Reemplazar TODO con:

---START---
auto eth0
iface eth0 inet static
    address 172.16.100.3
    netmask 255.255.255.224
    gateway 172.16.100.1
    dns-nameservers 8.8.8.8 8.8.4.4
---END---

Aplicar:
  /etc/init.d/networking restart
  ip addr show
  ping 172.16.100.1

---

WEBSERVER3 (ALPINE) - IP: 172.16.100.4 - LAPTOP 2

En Laptop 2, en WebServer3:

Reemplazar IP 172.16.100.4 (resto igual a WebServer1)

---

WEBSERVER4 (ALPINE) - IP: 172.16.100.5 - LAPTOP 3

En Laptop 3, en WebServer4:

Reemplazar IP 172.16.100.5 (resto igual a WebServer2)

---

WEBSERVER5 (ALPINE) - IP: 172.16.100.6 - LAPTOP 4

En Laptop 4, en WebServer5:

Reemplazar IP 172.16.100.6 (resto igual a WebServer1)

---

WEBSERVER6 (ALPINE) - IP: 172.16.100.7 - LAPTOP 4

En Laptop 4, en WebServer6:

Reemplazar IP 172.16.100.7 (resto igual a WebServer2)

================================================================================
8. INSTALACIÓN DE NGINX
================================================================================

NGINX EN PROXY-LB (UBUNTU)

En Laptop 1, en Proxy-LB:

  sudo apt update
  sudo apt install -y nginx
  sudo systemctl start nginx
  sudo systemctl enable nginx
  sudo systemctl status nginx

---

NGINX EN WEBSERVERS (ALPINE) - 6 VMS

En cada WebServer Alpine:

  apk add nginx
  rc-update add nginx default
  rc-service nginx start
  rc-service nginx status

================================================================================
9. CONFIGURACIÓN DE APLICACIONES WEB
================================================================================

WEBSERVERS PHP (1, 3, 5)

En cada WebServer PHP (webserver1, webserver3, webserver5):

1. Habilitar repositorio community:
  nano /etc/apk/repositories
  (Descomentar la línea de community)

2. Actualizar:
  apk update

3. Instalar PHP-FPM:
  apk add php php-fpm
  php -v

4. Agregar a arranque:
  rc-update add php-fpm83

5. Iniciar:
  rc-service php-fpm83 start
  rc-service php-fpm83 status

CONFIGURAR NGINX PARA PHP

Editar configuración NGINX:
  nano /etc/nginx/http.d/default.conf

Reemplazar TODO con:

---START---
server {
    listen 80;
    server_name _;
    root /var/www/localhost/htdocs;

    location / {
        index index.php index.html index.htm;
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
---END---

CREAR APLICACIÓN PHP

Crear directorio:
  mkdir -p /var/www/localhost/htdocs

Crear archivo index.php:
  nano /var/www/localhost/htdocs/index.php

Copiar contenido (ver archivo apps/php/index.php en repositorio)

REINICIAR SERVICIOS

  rc-service php-fpm83 restart
  rc-service nginx restart
  curl http://localhost

---

WEBSERVERS NODE.JS (2, 4, 6)

En cada WebServer Node.js (webserver2, webserver4, webserver6):

1. Instalar Node.js y npm:
  apk add nodejs npm
  node -v
  npm -v

2. Crear directorio de la aplicación:
  mkdir -p /var/www/nodejs
  cd /var/www/nodejs

3. Crear package.json:
  nano package.json

Copiar:

---START---
{
  "name": "hola-mundo-app",
  "version": "1.0.0",
  "description": "Aplicación simple Hola Mundo",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  }
}
---END---

4. Crear archivo index.js:
  nano /var/www/nodejs/index.js

Copiar contenido (ver archivo apps/nodejs/index.js en repositorio)

INSTALAR Y EJECUTAR

  npm install
  npm start > /tmp/nodejs.log 2>&1 &
  ps aux | grep node

CONFIGURAR NGINX COMO PROXY

Editar configuración NGINX:
  nano /etc/nginx/http.d/default.conf

Reemplazar TODO con:

---START---
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
---END---

REINICIAR

  rc-service nginx restart
  curl http://localhost

================================================================================
10. CONFIGURACIÓN DEL BALANCEADOR
================================================================================

CONFIGURAR NGINX COMO PROXY INVERSO CON LEAST CONNECTION

En Laptop 1, en Proxy-LB:

Editar configuración:
  sudo nano /etc/nginx/sites-available/default

Reemplazar TODO con:

---START---
# Upstream: Define los servidores backend
upstream backend {
    # Algoritmo de balanceo: Least Connection (Requerido)
    least_conn;
    
    # 3 WebServers PHP
    server 172.16.100.2:80;   # WebServer1-PHP
    server 172.16.100.4:80;   # WebServer3-PHP
    server 172.16.100.6:80;   # WebServer5-PHP
    
    # 3 WebServers Node.js
    server 172.16.100.3:80;   # WebServer2-Node
    server 172.16.100.5:80;   # WebServer4-Node
    server 172.16.100.7:80;   # WebServer6-Node
}

# Server block: Escucha en puerto 80
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    server_name _;

    # Toda solicitud se envía al upstream backend
    location / {
        proxy_pass http://backend;

        # Headers necesarios para que los backends sepan la IP real
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
---END---

VALIDAR Y APLICAR

  sudo nginx -t

Salida esperada:
  nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
  nginx: configuration file /etc/nginx/nginx.conf test is successful

Reiniciar:
  sudo systemctl restart nginx
  sudo systemctl status nginx

---

HABILITAR IP FORWARDING Y NAT EN PROXY

En Laptop 1, en Proxy-LB:

Habilitar IP Forwarding:
  sudo sysctl -w net.ipv4.ip_forward=1

Hacer persistente:
  echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
  sudo sysctl -p

Verificar:
  cat /etc/sysctl.conf | grep ip_forward

CONFIGURAR IPTABLES PARA NAT

  sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

Instalar iptables-persistent:
  sudo apt install -y iptables-persistent

Guardar reglas (Selecciona SÍ cuando pregunta):
  sudo netfilter-persistent save

Verificar:
  sudo iptables -t nat -L -v

================================================================================
11. PRUEBAS Y VALIDACIÓN
================================================================================

TEST DE CONECTIVIDAD BÁSICA

Desde Proxy (172.16.100.1):

  for i in 2 3 4 5 6 7; do
      echo "=== Ping a 172.16.100.$i ==="
      ping -c 2 172.16.100.$i
      echo ""
  done

Salida esperada: 6 respuestas exitosas

Desde cada WebServer:

  ping -c 2 172.16.100.1

Salida esperada: 2 respuestas exitosas

---

TEST HTTP A CADA WEBSERVER (DIRECTAMENTE)

Desde Proxy:

  for i in 2 3 4 5 6 7; do
      echo "=== HTTP a 172.16.100.$i ==="
      curl -s http://172.16.100.$i/ | grep -o "<h1>.*</h1>"
      echo ""
  done

Salida esperada: 6 respuestas con "¡Hola Mundo desde..."

---

TEST DE BALANCEO CON LEAST CONNECTION

Desde Proxy:

Instalar Apache Bench:
  sudo apt install -y apache2-utils

Ejecutar prueba de carga (100 peticiones, 10 concurrentes):
  ab -n 100 -c 10 http://172.16.100.1/

Ver resultado: distribución equitativa entre 6 servidores

VER DISTRIBUCIÓN EN TIEMPO REAL

Abre nuevas terminales SSH en cada WebServer y ejecuta:

  tail -f /var/log/nginx/access.log

---

TEST DE SIMULACIÓN DE FALLA

Apagar WebServer2-Node (172.16.100.3):
  sudo poweroff

Apagar WebServer4-Node (172.16.100.5):
  sudo poweroff

Desde Proxy, verificar balanceo con 20 peticiones:

  for i in {1..20}; do
      echo "Petición $i:"
      curl -s http://172.16.100.1/ | grep -o "<h1>.*</h1>"
      sleep 0.5
  done

Salida esperada: Solo respuestas de 4 servidores activos
(Sin webserver2 ni webserver4)

================================================================================
12. DOCUMENTACIÓN FINAL
================================================================================

CHECKLIST FINAL

Instalación de SOs:
  ☐ Ubuntu 24.04 instalado en Proxy-LB
  ☐ 6x Alpine Linux 3.22 instalados en WebServers
  ☐ OpenSSH instalado en todos
  ☐ Todos los SOs actualizados (apt/apk update)

Configuración de IPs Estáticas (YAML + Interfaces):
  ☐ Proxy enp0s3: DHCP desde NAT
  ☐ Proxy enp0s8: 172.16.100.1/27 (YAML)
  ☐ WebServer1: 172.16.100.2/27 (Interfaces)
  ☐ WebServer2: 172.16.100.3/27 (Interfaces)
  ☐ WebServer3: 172.16.100.4/27 (Interfaces)
  ☐ WebServer4: 172.16.100.5/27 (Interfaces)
  ☐ WebServer5: 172.16.100.6/27 (Interfaces)
  ☐ WebServer6: 172.16.100.7/27 (Interfaces)

Servicios Instalados:
  ☐ NGINX instalado y funcionando en Proxy
  ☐ NGINX instalado y funcionando en 6x WebServers
  ☐ PHP-FPM funcionando en WebServer1, WebServer3, WebServer5
  ☐ Node.js funcionando en WebServer2, WebServer4, WebServer6

Configuración de Aplicaciones Web:
  ☐ Página "Hola Mundo" PHP responde correctamente
  ☐ Página "Hola Mundo" Node.js responde correctamente
  ☐ Todas las páginas muestran hostname e IP

Configuración del Balanceador:
  ☐ NGINX proxy configurado con upstream backend
  ☐ Algoritmo least_conn habilitado
  ☐ Los 6 WebServers agregados al upstream
  ☐ IP Forwarding habilitado en Proxy
  ☐ NAT configurado con iptables

Pruebas Ejecutadas:
  ☐ Ping desde Proxy a todos los WebServers
  ☐ Ping desde WebServers al Proxy
  ☐ HTTP GET a cada WebServer directamente
  ☐ HTTP GET a través del balanceador
  ☐ Apache Bench ejecutado
  ☐ Distribución de carga verificada
  ☐ Simulación de falla completada

Documentación:
  ☐ Informe markdown creado y completo
  ☐ Todas las capturas de pantalla incluidas
  ☐ Configuraciones documentadas en anexos
  ☐ Conclusiones individuales completadas

================================================================================
PROBLEMAS COMUNES Y SOLUCIONES
================================================================================

Problema: Las VMs no se conectan entre sí
Solución:
1. Verifica que todas las VMs estén conectadas a la red interna "SIS313-RedInterna"
2. Verifica IPs con: ip addr show
3. Reinicia las interfaces de red

Problema: NGINX no responde
Solución:
  sudo nginx -t
  sudo systemctl status nginx
  tail -f /var/log/nginx/error.log
  sudo systemctl restart nginx

Problema: PHP no funciona
Solución:
  rc-service php-fpm83 status
  rc-service php-fpm83 restart
  tail -f /var/log/nginx/error.log

Problema: Balanceo no distribuye equitativamente
Solución:
  sudo nano /etc/nginx/sites-available/default
  (Verificar que "least_conn;" esté presente)
  sudo nginx -t
  sudo systemctl restart nginx

================================================================================

Documento Generado: 2026-04-19
Versión: 3.0 (Sin hotspot, Red Interna en VMs)
Estado: Completo y Listo para Usar

FIN DE LA GUÍA
