# Servidor Samba con Autenticación (Ubuntu Server 22.04 + VMware)

Guía paso a paso para configurar un recurso compartido seguro con autenticación por usuario/contraseña, accesible desde Windows, macOS y Linux.

---

## 📋 Requisitos

| Componente | Especificación |
|------------|---------------|
| **Hipervisor** | VMware Workstation 14 (Pro) o superior |
| **ISO Ubuntu** | Ubuntu Desktop 24.04.2 LTS |
| **Recursos VM** | 2 vCPUs, 1 GB RAM, 20 GB disco |
| **Red** | LAN con router DHCP (para modo puente) |

---

## 🛠️ Instrucciones de Configuración

### 1. Crear y Configurar la Máquina Virtual
1. En VMware, selecciona **Archivo > Nueva Máquina Virtual > Típica**.
2. Usa la ISO **ubuntu-server-22.04.iso** y configura el sistema operativo como **Ubuntu 64-bit**.
3. Personaliza el hardware: Configura el **Adaptador de Red** en **Modo Puente**.
4. En **Editar > Editor de Red Virtual**, asigna **VMnet0** al adaptador físico de tu PC (Wi-Fi/Ethernet).

### 2. Instalar Ubuntu Server
Durante la instalación:
- **Usuario**: `gleb`
- **Contraseña**: Establece una contraseña segura
- **Instalar OpenSSH**: Sí

### 3. Configurar IP Estática con Netplan
1. Identifica tu interfaz de red:
   ```bash
   ip link  # Ejemplo: ens33
   ```
2. Edita la configuración de Netplan:
   ```bash
   sudo nano /etc/netplan/01-samba.yaml
   ```
   Añade lo siguiente:
   ```yaml
   network:
     version: 2
     ethernets:
       ens33:
         dhcp4: no
         addresses: [192.168.32.220/24]
       routes:                
         - to: default
           via: 192.168.32.1  
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
   ```
3. Aplica los cambios:
   ```bash
   sudo netplan apply
   ```

### 4. Instalar Samba
```bash
sudo apt update && sudo apt install samba smbclient -y
```

### 5. Crear un Usuario Exclusivo para Samba
```bash
sudo useradd -M -s /usr/sbin/nologin alfonso
sudo smbpasswd -a alfonso  # Establece una contraseña para alfonso
```

### 6. Configurar el Recurso Compartido Seguro
1. Crea el directorio compartido:
   ```bash
   sudo mkdir -p /srv/compartido
   sudo chown alfonso /srv/compartido
   ```
2. Edita la configuración de Samba:
   ```bash
   sudo nano /etc/samba/smb.conf
   ```
   Añade al final:
   ```ini
   [compartido]
       path        = /srv/compartido
       browseable  = yes
       read only   = no
       guest ok    = no
       valid users = alfonso
   ```
3. (Recomendado) En la sección `[global]`, añade:
   ```ini
   server min protocol = SMB2
   ```
4. Reinicia el servicio de Samba:
   ```bash
   sudo systemctl restart smbd
   ```

### 7. Configurar el Cortafuegos
```bash
sudo ufw allow ssh
sudo ufw allow Samba
sudo ufw enable
```

---

## 🧪 Pruebas del Funcionamiento

### 8.1. Probar desde la VM
```bash
smbclient //localhost/compartido -U alfonso
smb:\> put /etc/hosts hosts.txt
```

### 8.2. Probar desde Windows
1. Presiona `Win + R` e introduce:
   ```
   \\192.168.32.220\compartido
   ```
2. Usa las credenciales:
   - **Usuario**: `alfonso`
   - **Contraseña**: (la definida en el paso 5)

### 8.3. Probar desde Linux/macOS
```bash
smbclient //192.168.32.220/compartido -U alfonso
```

---

## 🔧 Solución de Problemas

| Problema | Comando de Ayuda |
|----------|------------------|
| **Credenciales rechazadas** | `sudo pdbedit -L` (lista usuarios de Samba) |
| **Hay conneccion abierta** | `net use * /delete /y` (Cerrar conexiones SMB abiertas) |

---

## 📜 Licencia
MIT License © 2025 Gleb Ursol
