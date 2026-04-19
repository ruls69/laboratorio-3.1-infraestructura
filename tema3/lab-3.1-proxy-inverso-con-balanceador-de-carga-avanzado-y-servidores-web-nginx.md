# SIS313 Lab 3.1: Proxy Inverso con Balanceador de Carga (Least Connection)
## Guía Completa Paso a Paso para 4 Personas - 7 VMs

**Versión:** 3.0 (Actualizada: Sin hotspot, Red Interna al crear VMs)  
**Fecha:** 2026-04-19  
**Red Interna:** 172.16.100.0/27 (IPs Estáticas)  

---

## ÍNDICE

1. [Información del Grupo](#1-información-del-grupo)
2. [Arquitectura de Red](#2-arquitectura-de-red)
3. [Preparación del Entorno](#3-preparación-del-entorno)
4. [Creación de VMs en VirtualBox](#4-creación-de-vms-en-virtualbox)
5. [Instalación y Configuración Ubuntu (Proxy)](#5-instalación-y-configuración-ubuntu-proxy)
6. [Instalación y Configuración Alpine (WebServers)](#6-instalación-y-configuración-alpine-webservers)
7. [Configuración de IPs Estáticas (YAML + Interfaces)](#7-configuración-de-ips-estáticas-yaml--interfaces)
8. [Instalación de NGINX](#8-instalación-de-nginx)
9. [Configuración de Aplicaciones Web](#9-configuración-de-aplicaciones-web)
10. [Configuración del Balanceador](#10-configuración-del-balanceador)
11. [Pruebas y Validación](#11-pruebas-y-validación)
12. [Documentación Final](#12-documentación-final)

---

## 1. INFORMACIÓN DEL GRUPO

### Integrantes

| # | Nombre | Rol | Laptop | VMs |
|---|--------|-----|--------|-----|
| 1 | [Nombre Persona 1] | Proxy + Balanceador | Laptop 1 | VM1 |
| 2 | [Nombre Persona 2] | WebServers PHP | Laptop 2 | VM2, VM4 |
| 3 | [Nombre Persona 3] | WebServers Node.js | Laptop 3 | VM3, VM5 |
| 4 | [Nombre Persona 4] | WebServers Mixtos | Laptop 4 | VM6, VM7 |

### Responsabilidades

**Persona 1 (Proxy-LB):**
- Crear VM1 (Proxy-LB) en Laptop 1
- Instalar Ubuntu 24.04 Server
- Configurar IPs estáticas con YAML Netplan (NAT + red interna)
- Habilitar IP Forwarding y NAT
- Instalar y configurar NGINX como proxy inverso
- Configurar algoritmo de balanceo: `least_conn`
- Pruebas de balanceo y distribución

**Persona 2 (WebServers PHP):**
- Crear VM2 (webserver1) en Laptop 2
- Crear VM4 (webserver3) en Laptop 2
- Instalar Alpine Linux en ambas
- Configurar IPs estáticas con `/etc/network/interfaces`
- Instalar NGINX + PHP-FPM
- Crear aplicación "Hola Mundo" en PHP

**Persona 3 (WebServers Node.js):**
- Crear VM3 (webserver2) en Laptop 3
- Crear VM5 (webserver4) en Laptop 3
- Instalar Alpine Linux en ambas
- Configurar IPs estáticas con `/etc/network/interfaces`
- Instalar NGINX + Node.js
- Crear aplicación "Hola Mundo" en Node.js

**Persona 4 (WebServers Mixtos):**
- Crear VM6 (webserver5) en Laptop 4
- Crear VM7 (webserver6) en Laptop 4
- Instalar Alpine Linux en ambas
- Configurar IPs estáticas con `/etc/network/interfaces`
- Una con PHP-FPM, otra con Node.js
- Crear ambas aplicaciones Web

---

## 2. ARQUITECTURA DE RED

### 2.1 Diagrama General


RED INTERNA VIRTUALBOX: 172.16.100.0/27 (Compartida entre Laptops)
═════════════════════════════════════════════════════════════════

Laptop 1 Laptop 2 Laptop 3 Laptop 4
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│ │ │ │ │ │ │ │
│ ┌──────┐│ │┌────────┐│ │┌────────┐│ │┌────────┐│
│ │Proxy ││ ││WS1-PHP ││ ││WS2-Node││ ││WS5-PHP ││
│ │ LB ││ │├────────┤│ │├────────┤│ │├────────┤│
│ └──┬───┘│ ││172.16..2││ ││172.16..3││ ││172.16..6││
│ │ │ │└────────┘│ │└────────┘│ │└────────┘│
│ │ │ │ │ │ │ │ │
│ ┌──┴───┐│ │┌────────┐│ │┌────────┐│ │┌────────┐│
│ │ NAT ││ ││WS3-PHP ││ ││WS4-Node││ ││WS6-Node││
│ └──────┘│ │├────────┤│ │├────────┤│ │├────────┤│
│ │ ││172.16..4││ ││172.16..5││ ││172.16..7││
│ │ │└────────┘│ │└────────┘│ │└────────┘│
└──────────┘ └──────────┘ └──────────┘ └──────────┘


---

### 2.2 Tabla de Configuración de Red

| # | VM | Hostname | IP Interna | Gateway | Netmask | Laptop | SO | Tipo |
|---|----|----|-----------|---------|---------|--------|-----|------|
| 1 | Proxy-LB | proxy-lb | 172.16.100.1 | N/A | 255.255.255.224 | L1 | Ubuntu 24.04 | Proxy |
| 2 | WebServer1 | webserver1 | 172.16.100.2 | 172.16.100.1 | 255.255.255.224 | L2 | Alpine 3.22 | PHP |
| 3 | WebServer2 | webserver2 | 172.16.100.3 | 172.16.100.1 | 255.255.255.224 | L3 | Alpine 3.22 | Node.js |
| 4 | WebServer3 | webserver3 | 172.16.100.4 | 172.16.100.1 | 255.255.255.224 | L2 | Alpine 3.22 | PHP |
| 5 | WebServer4 | webserver4 | 172.16.100.5 | 172.16.100.1 | 255.255.255.224 | L3 | Alpine 3.22 | Node.js |
| 6 | WebServer5 | webserver5 | 172.16.100.6 | 172.16.100.1 | 255.255.255.224 | L4 | Alpine 3.22 | PHP |
| 7 | WebServer6 | webserver6 | 172.16.100.7 | 172.16.100.1 | 255.255.255.224 | L4 | Alpine 3.22 | Node.js |

---

## 3. PREPARACIÓN DEL ENTORNO

### 3.1 Requisitos Previos

✓ 4 Laptops  
✓ VirtualBox 7.2.6 o superior  
✓ ISOs descargadas:  
- Ubuntu 24.04 Server  
- Alpine Linux 3.22  

---

## 5. INSTALACIÓN Y CONFIGURACIÓN UBUNTU (PROXY)

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y nano curl wget git htop net-tools
6. INSTALACIÓN Y CONFIGURACIÓN ALPINE (WEBSERVERS)
setup-alpine
7. CONFIGURACIÓN DE IPs ESTÁTICAS
Ubuntu (Netplan)
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
        - 172.16.100.1/27
Alpine
auto eth0
iface eth0 inet static
    address 172.16.100.X
    netmask 255.255.255.224
    gateway 172.16.100.1
8. INSTALACIÓN DE NGINX
Ubuntu
sudo apt install nginx
Alpine
apk add nginx
10. CONFIGURACIÓN DEL BALANCEADOR
upstream backend {
    least_conn;
    server 172.16.100.2;
    server 172.16.100.3;
    server 172.16.100.4;
    server 172.16.100.5;
    server 172.16.100.6;
    server 172.16.100.7;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
11. PRUEBAS Y VALIDACIÓN
curl http://172.16.100.1
ab -n 100 -c 10 http://172.16.100.1/
12. DOCUMENTACIÓN FINAL
CHECKLIST FINAL
 VMs creadas
 IPs configuradas
 NGINX funcionando
 Balanceador activo
 Pruebas realizadas
