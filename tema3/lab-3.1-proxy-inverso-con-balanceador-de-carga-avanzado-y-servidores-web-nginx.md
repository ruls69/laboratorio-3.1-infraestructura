SIS313 Lab 3.1: Proxy Inverso con Balanceador de Carga (Least Connection)Guía Completa Paso a Paso para 4 Personas - 7 VMsVersión: 3.0 (Actualizada: Sin hotspot, Red Interna al crear VMs)Fecha: 2026-04-19Red Interna: 172.16.100.0/27 (IPs Estáticas)ÍNDICEInformación del GrupoArquitectura de RedPreparación del EntornoCreación de VMs en VirtualBoxInstalación y Configuración Ubuntu (Proxy)Instalación y Configuración Alpine (WebServers)Configuración de IPs Estáticas (YAML + Interfaces)Instalación de NGINXConfiguración de Aplicaciones WebConfiguración del BalanceadorPruebas y ValidaciónDocumentación Final1. INFORMACIÓN DEL GRUPOIntegrantes#NombreRolLaptopVMs1[Nombre Persona 1]Proxy + BalanceadorLaptop 1VM12[Nombre Persona 2]WebServers PHPLaptop 2VM2, VM43[Nombre Persona 3]WebServers Node.jsLaptop 3VM3, VM54[Nombre Persona 4]WebServers MixtosLaptop 4VM6, VM7ResponsabilidadesPersona 1 (Proxy-LB):Crear VM1 (Proxy-LB) en Laptop 1Instalar Ubuntu 24.04 ServerConfigurar IPs estáticas con YAML Netplan (NAT + red interna)Habilitar IP Forwarding y NATInstalar y configurar NGINX como proxy inversoConfigurar algoritmo de balanceo: least_connPruebas de balanceo y distribuciónPersona 2 (WebServers PHP):Crear VM2 (webserver1) en Laptop 2Crear VM4 (webserver3) en Laptop 2Instalar Alpine Linux en ambasConfigurar IPs estáticas con /etc/network/interfacesInstalar NGINX + PHP-FPMCrear aplicación "Hola Mundo" en PHPPersona 3 (WebServers Node.js):Crear VM3 (webserver2) en Laptop 3Crear VM5 (webserver4) en Laptop 3Instalar Alpine Linux en ambasConfigurar IPs estáticas con /etc/network/interfacesInstalar NGINX + Node.jsCrear aplicación "Hola Mundo" en Node.jsPersona 4 (WebServers Mixtos):Crear VM6 (webserver5) en Laptop 4Crear VM7 (webserver6) en Laptop 4Instalar Alpine Linux en ambasConfigurar IPs estáticas con /etc/network/interfacesUna con PHP-FPM, otra con Node.jsCrear ambas aplicaciones Web2. ARQUITECTURA DE RED2.1 Diagrama GeneralPlaintextRED INTERNA VIRTUALBOX: 172.16.100.0/27 (Compartida entre Laptops)
═════════════════════════════════════════════════════════════════

      Laptop 1            Laptop 2            Laptop 3            Laptop 4
    ┌──────────┐        ┌──────────┐        ┌──────────┐        ┌──────────┐
    │          │        │          │        │          │        │          │
    │ ┌──────┐ │        │┌────────┐│        │┌────────┐│        │┌────────┐│
    │ │Proxy │ │        ││WS1-PHP ││        ││WS2-Node││        ││WS5-PHP ││
    │ │  LB  │ │        │├────────┤│        │├────────┤│        │├────────┤│
    │ └──┬───┘ │        ││172.16..2││        ││172.16..3││        ││172.16..6││
    │    │     │        │└────────┘│        │└────────┘│        │└────────┘│
    │    │     │        │          │        │          │        │          │
    │ ┌──┴───┐ │        │┌────────┐│        │┌────────┐│        │┌────────┐│
    │ │ NAT  │ │        ││WS3-PHP ││        ││WS4-Node││        ││WS6-Node││
    │ │172.. │ │        │├────────┤│        │├────────┤│        │├────────┤│
    │ └──────┘ │        ││172.16..4││        ││172.16..5││        ││172.16..7││
    └──────────┘        │└────────┘│        │└────────┘│        │└────────┘│
          │             └──────────┘        └──────────┘        └──────────┘
          │                   │                   │                   │
          └───────────────────┴───────────────────┴───────────────────┘
                         RED INTERNA VIRTUALBOX 172.16.100.0/27
                         ┌──────────────────────┐
                         │ Gateway: 172.16.100.1│
                         │ (Proxy-LB)           │
                         └──────────────────────┘
2.2 Tabla de Configuración de Red#VMHostnameIP InternaGatewayNetmaskLaptopSOTipo1Proxy-LBproxy-lb172.16.100.1N/A255.255.255.224L1Ubuntu 24.04Proxy2WebServer1webserver1172.16.100.2172.16.100.1255.255.255.224L2Alpine 3.22PHP3WebServer2webserver2172.16.100.3172.16.100.1255.255.255.224L3Alpine 3.22Node.js4WebServer3webserver3172.16.100.4172.16.100.1255.255.255.224L2Alpine 3.22PHP5WebServer4webserver4172.16.100.5172.16.100.1255.255.255.224L3Alpine 3.22Node.js6WebServer5webserver5172.16.100.6172.16.100.1255.255.255.224L4Alpine 3.22PHP7WebServer6webserver6172.16.100.7172.16.100.1255.255.255.224L4Alpine 3.22Node.js3. PREPARACIÓN DEL ENTORNO3.1 Requisitos Previos✓ 4 Laptops (Windows, macOS o Linux)✓ VirtualBox 7.2.6 o superior instalado en cada Laptop✓ ISOs descargadas:Ubuntu 24.04 Server (2.6 GB)Alpine Linux 3.22 (200 MB)3.2 Requisitos de AlmacenamientoTotal necesario por laptop:Laptop 1: 10 GB (Proxy-LB)Laptop 2: 12 GB (2x WebServers de 6 GB)Laptop 3: 12 GB (2x WebServers de 6 GB)Laptop 4: 12 GB (2x WebServers de 6 GB)4. CREACIÓN DE VMS EN VIRTUALBOX4.1 INFORMACIÓN IMPORTANTEVirtualBox 7.2.6: La red interna se crea al configurar cada VM, no en Preferencias.Cuando crees una VM, en la configuración de red selecciona:Red Interna (Internal Network)Nombre: SIS313-RedInternaVirtualBox creará automáticamente la red si no existe.4.2 LAPTOP 1: VM1 - PROXY-LB (Ubuntu 24.04)Paso 1: Crear la VMEn VirtualBox, haz clic en "Crear" (botón azul)Completa el formulario:PlaintextNombre: Proxy-LB 
Ruta de carpeta: (Dejar por defecto) 
Imagen ISO: ubuntu-24.04.3-live-server-amd64.iso 
Tipo: Linux 
Versión: Ubuntu (64-bit) 
Memoria base: 2048 MB 
Procesadores: 2 
Tamaño de disco: 10 GB 
Tipo de disco: VDI 
Almacenamiento: Dinámico
Paso 2: Configurar Adaptadores de RedAdaptador 1 (NAT):Conectado a: NATDejar valores por defectoAdaptador 2 (Red Interna):Conectado a: Red Interna (Internal Network)Nombre: SIS313-RedInternaPaso 3: FinalizarHaz clic en "Crear" y espera a que se cree la VM.4.3 LAPTOP 2: VM2 Y VM4 - WEBSERVER1 Y WEBSERVER3 (Alpine)VM2: WebServer1Haz clic en "Crear"PlaintextNombre: webserver1 
Imagen ISO: alpine-standard-3.22.1-x86_64.iso 
Tipo: Linux 
Versión: Other Linux (64-bit) 
Memoria base: 1024 MB 
Procesadores: 1 
Tamaño de disco: 6 GB 
Tipo de disco: VDI 
Almacenamiento: Dinámico
Adaptador de Red:Conectado a: Red Interna (Internal Network)Nombre: SIS313-RedInternaVM4: WebServer3Repite el proceso:PlaintextNombre: webserver3 
Imagen ISO: alpine-standard-3.22.1-x86_64.iso 
Tipo: Linux 
Versión: Other Linux (64-bit) 
Memoria base: 1024 MB 
Procesadores: 1 
Tamaño de disco: 6 GB 
Tipo de disco: VDI 
Almacenamiento: Dinámico
Adaptador de Red:Conectado a: Red Interna (Internal Network)Nombre: SIS313-RedInterna4.4 LAPTOP 3: VM3 Y VM5 - WEBSERVER2 Y WEBSERVER4 (Alpine)VM3: WebServer2PlaintextNombre: webserver2 
Imagen ISO: alpine-standard-3.22.1-x86_64.iso 
Tipo: Linux 
Versión: Other Linux (64-bit) 
Memoria base: 1024 MB 
Procesadores: 1 
Tamaño de disco: 6 GB 
Tipo de disco: VDI 
Almacenamiento: Dinámico
Adaptador de Red:Conectado a: Red Interna (Internal Network)Nombre: SIS313-RedInternaVM5: WebServer4PlaintextNombre: webserver4 
Imagen ISO: alpine-standard-3.22.1-x86_64.iso 
Tipo: Linux 
Versión: Other Linux (64-bit) 
Memoria base: 1024 MB 
Procesadores: 1 
Tamaño de disco: 6 GB 
Tipo de disco: VDI 
Almacenamiento: Dinámico
Adaptador de Red:Conectado a: Red Interna (Internal Network)Nombre: SIS313-RedInterna4.5 LAPTOP 4: VM6 Y VM7 - WEBSERVER5 Y WEBSERVER6 (Alpine)VM6: WebServer5PlaintextNombre: webserver5 
Imagen ISO: alpine-standard-3.22.1-x86_64.iso 
Tipo: Linux 
Versión: Other Linux (64-bit) 
Memoria base: 1024 MB 
Procesadores: 1 
Tamaño de disco: 6 GB 
Tipo de disco: VDI 
Almacenamiento: Dinámico
Adaptador de Red:Conectado a: Red Interna (Internal Network)Nombre: SIS313-RedInternaVM7: WebServer6PlaintextNombre: webserver6 
Imagen ISO: alpine-standard-3.22.1-x86_64.iso 
Tipo: Linux 
Versión: Other Linux (64-bit) 
Memoria base: 1024 MB 
Procesadores: 1 
Tamaño de disco: 6 GB 
Tipo de disco: VDI 
Almacenamiento: Dinámico
Adaptador de Red:Conectado a: Red Interna (Internal Network)Nombre: SIS313-RedInternaCHECKLIST PASO 4Laptop 1:[ ] VM1 (Proxy-LB) creada ✓[ ] Adaptador 1: NAT ✓[ ] Adaptador 2: Red Interna (SIS313-RedInterna) ✓Laptop 2:[ ] VM2 (webserver1) creada ✓[ ] VM4 (webserver3) creada ✓Laptop 3:[ ] VM3 (webserver2) creada ✓[ ] VM5 (webserver4) creada ✓Laptop 4:[ ] VM6 (webserver5) creada ✓[ ] VM7 (webserver6) creada ✓5. INSTALACIÓN Y CONFIGURACIÓN UBUNTU (PROXY)5.1 Instalar Ubuntu 24.04 en Proxy-LBEn Laptop 1:Inicia la VM Proxy-LBSelecciona ubuntu-24.04.3-live-server-amd64.isoSigue el instaladorEn Network configuration: Dejar valores por defecto (configuraremos después)Completa instalación con:Username: usuarioPassword: tu contraseñaInstall OpenSSH server: ✓ Marcar5.2 Post-Instalación UbuntuInicia la VM Proxy-LB:Bash# Login con usuario creado
login: usuario
password: (tu contraseña)

# Actualizar sistema
sudo apt update
sudo apt upgrade -y

# Instalar herramientas útiles
sudo apt install -y nano curl wget git htop net-tools

# Verificar interfaces de red
ip link show

# Ver configuración actual
ip addr show
6. INSTALACIÓN Y CONFIGURACIÓN ALPINE (WEBSERVERS)6.1 Instalar Alpine Linux (6 VMs)En cada WebServer Alpine (Laptops 2, 3, 4):Inicia la VMSelecciona alpine-standard-3.22.1-x86_64.isoLogin como root (sin contraseña)Ejecuta:Bashsetup-alpine
Sigue los pasos interactivos:PlaintextSelect keyboard layout [none]: us
Select variant: us
Hostname: webserver1 (cambiar según VM: webserver2, webserver3, etc.)

Interface: eth0
IP address for eth0? [dhcp]: dhcp
Do you want to do any manual network configuration? [n]: n

Root password: (ingresa tu contraseña)

Timezone: America/La_Paz
Proxy: none
NTP client: busybox
APK Mirror: 1

Setup a user? [no]: marcelo
Full name: Marcelo Quispe Ortega
Which ssh server? [openssh]: openssh

Which disk(s) would you like to use? [none]: sda
How would you like to use it? [?]: sys
WARNING: Erase the above disk(s) and continue? (y/n) [n]: y
Apaga la VM:Bashpoweroff
En VirtualBox: Configuración -> Almacenamiento -> Selecciona el CD -> Desconectar.6.2 Post-Instalación Alpine (6 VMs)Para cada VM Alpine:Bash# Login como root
# Instalar herramientas
apk add nano curl wget htop

# Instalar OpenSSH si no lo incluyó setup-alpine
apk add openssh
rc-service sshd start
rc-update add sshd

# Verificar interfaces
ip link show
7. CONFIGURACIÓN DE IPs ESTÁTICAS (YAML + Interfaces)7.1 PROXY-LB (Ubuntu 24.04) - Configuración YAML NetplanEn Laptop 1, en Proxy-LB:Bashsudo nano /etc/netplan/00-installer-config.yaml
Reemplazar con:YAMLnetwork:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
      dhcp4-overrides:
        use-dns: true
        use-ntp: true
      optional: true
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
Aplicar:Bashsudo netplan apply
ip addr show
7.2 WebServers Alpine (Ejemplo WebServer1)Repetir para cada uno cambiando la IP según la tabla de la sección 2.2.Bash# En WebServer1
nano /etc/network/interfaces
Contenido:Plaintextauto eth0
iface eth0 inet static
    address 172.16.100.2
    netmask 255.255.255.224
    gateway 172.16.100.1
    dns-nameservers 8.8.8.8 8.8.4.4
Reiniciar red:Bash/etc/init.d/networking restart
ping 172.16.100.1
8. INSTALACIÓN DE NGINX8.1 NGINX en Proxy-LB (Ubuntu)Bashsudo apt update
sudo apt install -y nginx
sudo systemctl enable --now nginx
8.2 NGINX en WebServers (Alpine) - 6 VMsBashapk add nginx
rc-update add nginx default
rc-service nginx start
9. CONFIGURACIÓN DE APLICACIONES WEB9.1 WebServers PHP (1, 3, 5)Bashapk add php php-fpm
rc-update add php-fpm83 default
rc-service php-fpm83 start
Editar /etc/nginx/http.d/default.conf para soportar PHP (usar bloque location ~ \.php$). Crear /var/www/localhost/htdocs/index.php con el código proporcionado en la guía.9.2 WebServers Node.js (2, 4, 6)Bashapk add nodejs npm
mkdir -p /var/www/nodejs
cd /var/www/nodejs
# Crear index.js y package.json
npm start > /tmp/nodejs.log 2>&1 &
Configurar NGINX como proxy local al puerto 3000 en /etc/nginx/http.d/default.conf.10. CONFIGURACIÓN DEL BALANCEADOR10.1 NGINX Least Connection (Proxy-LB)En Ubuntu /etc/nginx/sites-available/default:Nginxupstream backend {
    least_conn;
    server 172.16.100.2:80;
    server 172.16.100.3:80;
    server 172.16.100.4:80;
    server 172.16.100.5:80;
    server 172.16.100.6:80;
    server 172.16.100.7:80;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
Reiniciar: sudo systemctl restart nginx.10.2 Habilitar NAT en ProxyBashsudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
sudo apt install iptables-persistent
11. PRUEBAS Y VALIDACIÓNPing: Desde el Proxy a todos los backends.Curl: curl http://172.16.100.1/ repetidamente para ver el cambio de Hostname.Apache Bench: ab -n 100 -c 10 http://172.16.100.1/.Falla: Apagar una VM Alpine y verificar que el balanceador sigue funcionando con las restantes.12. DOCUMENTACIÓN FINALEl informe debe seguir la estructura Markdown detallada en la guía, incluyendo capturas de pantalla de las configuraciones de red, los archivos de configuración de NGINX y los resultados de las pruebas de carga.Documento Generado: 2026-04-19Estado: Completo y Listo para Usar en GitHub.
