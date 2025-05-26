Servidor Samba con Autenticación (Ubuntu Server 22.04 + VMware)
Guía paso a paso para configurar un recurso compartido seguro con autenticación por usuario/contraseña, accesible desde Windows, macOS y Linux.

📋 Requisitos



Componente
Especificación



Hipervisor
VMware Workstation 14 (Pro) o superior


ISO Ubuntu
Ubuntu Server 22.04 LTS (x64)


Recursos VM
2 vCPUs, 1 GB RAM, 20 GB disco


Red
LAN con router DHCP (para modo puente)



🛠️ Instrucciones de Configuración
1. Crear y Configurar la Máquina Virtual

En VMware, selecciona Archivo > Nueva Máquina Virtual > Típica.
Usa la ISO ubuntu-server-22.04.iso y configura el sistema operativo como Ubuntu 64-bit.
Personaliza el hardware: Configura el Adaptador de Red en Modo Puente.
En Editar > Editor de Red Virtual, asigna VMnet0 al adaptador físico de tu PC (Wi-Fi/Ethernet).

2. Instalar Ubuntu Server
Durante la instalación:

Usuario: gleb
Contraseña: Establece una contraseña segura
Instalar OpenSSH: Sí

3. Configurar IP Estática con Netplan

Identifica tu interfaz de red:ip link  # Ejemplo: ens33


Edita la configuración de Netplan:sudo nano /etc/netplan/01-samba.yaml

Añade lo siguiente:network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
      addresses: [192.168.32.220/24]
      gateway4: 192.168.32.1
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]


Aplica los cambios:sudo netplan apply



4. Instalar Samba
sudo apt update && sudo apt install samba smbclient -y

5. Crear un Usuario Exclusivo para Samba
sudo useradd -M -s /usr/sbin/nologin alfonso
sudo smbpasswd -a alfonso  # Establece una contraseña para alfonso

6. Configurar el Recurso Compartido Seguro

Crea el directorio compartido:sudo mkdir -p /srv/compartido
sudo chown alfonso /srv/compartido


Edita la configuración de Samba:sudo nano /etc/samba/smb.conf

Añade al final:[compartido]
    path        = /srv/compartido
    browseable  = yes
    read only   = no
    guest ok    = no
    valid users = alfonso


(Recomendado) En la sección [global], añade:server min protocol = SMB2


Reinicia el servicio de Samba:sudo systemctl restart smbd



7. Configurar el Cortafuegos
sudo ufw allow OpenSSH
sudo ufw allow Samba
sudo ufw enable


🧪 Pruebas del Funcionamiento
8.1. Probar desde la VM
smbclient //localhost/compartido -U alfonso
smb:\> put /etc/hosts hosts.txt

8.2. Probar desde Windows

Presiona Win + R e introduce:\\192.168.32.220\compartido


Usa las credenciales:
Usuario: alfonso
Contraseña: (la definida en el paso 5)



8.3. Probar desde Linux/macOS
smbclient //192.168.32.220/compartido -U alfonso


🔧 Solución de Problemas



Problema
Comando de Ayuda



Credenciales rechazadas
sudo pdbedit -L (lista usuarios de Samba)


Recurso no visible
testparm (valida smb.conf)


Conexión lenta
sudo ethtool -K ens33 tx off (desactiva offloading de checksum TX)



📜 Licencia
MIT License © 2025 Gleb Ursol
