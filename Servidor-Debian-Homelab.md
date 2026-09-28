---
tags: [homelab, debian, servidor, lpic1, docker]
fecha: 2026-08-29
---

# Servidor Debian - Homelab + IA local

## Hardware

- **CPU**: AMD Ryzen 5 5600G (sin gráfica dedicada por ahora)
- **RAM**: 16 GB
- **Almacenamiento**: NVMe Kingston 250GB + HDD Toshiba 1TB
- **Fuente**: 750W
- **Objetivo**: servidor dedicado para IA local (Ollama), procesamiento de datos y homelab, practicando de cara a la LPIC-1

---

## 1. Instalación de Debian 13

### Descarga e imagen
- ISO: **netinst amd64** desde https://www.debian.org/distrib/netinst
- No existe una "edición server" separada como en Ubuntu — la diferencia se hace en tasksel durante la instalación
- Grabado del USB con Rufus / `dd` / balenaEtcher

### Particionado (manual, dos discos)

**NVMe (250GB) — sistema:**

| Partición | Tamaño | FS | Punto de montaje |
|---|---|---|---|
| #1 | 1.0 GB | ESP | (arranque EFI) |
| #2 | 236.1 GB | ext4 | `/` |
| #3 | 12.9 GB | swap | intercambio |

**HDD (1TB) — datos:**

| Partición | Tamaño | FS | Punto de montaje |
|---|---|---|---|
| #1 | 1.0 TB | ext4 (etiqueta: data) | `/srv` |

> Nota: Windows eliminado por completo, disco usado al 100% para Debian. Boot configurado directo a Debian en BIOS (sin dual boot).

### Selección de software (tasksel)
Solo se marcó:
- ✅ SSH server
- ✅ standard system utilities

Sin entorno gráfico (Debian desktop environment desmarcado).

### Incidencias durante la instalación
- Aviso "no se determinó el sistema de ficheros raíz" y "no se encontró partición EFI" → se resolvió revisando que las particiones `/` y ESP tuvieran correctamente asignado su punto de montaje / tipo, y continuando el asistente (se resolvió solo al re-confirmar el particionado).

---

## 2. Configuración post-instalación

### sudo
Debian no trae `sudo` preinstalado por defecto. Instalación y configuración:

```bash
su -
apt update
apt install sudo -y
usermod -aG sudo tu_usuario
exit
```

Cerrar sesión y volver a entrar para aplicar el cambio de grupo. Verificar con:
```bash
groups tu_usuario
sudo whoami   # debe devolver "root"
```

### Actualización del sistema
```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y curl wget git vim htop net-tools
```

---

## 3. SSH

### Comprobación del servicio
```bash
sudo systemctl status ssh
```

### IP del servidor
```bash
ip a
```

### Acceso por clave (desde el cliente Ubuntu)
```bash
ssh-keygen -t ed25519 -C "servidor-homelab"
ssh-copy-id tu_usuario@IP_DEL_SERVIDOR
```

### Hardening (`/etc/ssh/sshd_config`)
```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

⚠️ No desactivar `PasswordAuthentication` hasta confirmar que el login por clave funciona.

```bash
sudo systemctl restart ssh
```

### Firewall
```bash
sudo apt install ufw -y
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status
```

### Fail2ban
```bash
sudo apt install fail2ban -y
sudo systemctl enable --now fail2ban
```

---

## 4. Docker

### Instalación (repo oficial)
```bash
sudo apt remove docker docker-engine docker.io containerd runc

sudo apt update
sudo apt install -y ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Verificación
```bash
sudo docker run hello-world
```

### Uso sin sudo
```bash
sudo usermod -aG docker tu_usuario
# cerrar sesión y volver a entrar
docker run hello-world
```

### Arranque automático
```bash
sudo systemctl enable docker
```

### Ubicación de datos
- Motor de Docker en NVMe (`/var/lib/docker`) por velocidad
- Volúmenes de datos pesados (modelos IA, bases de datos grandes) → `/srv` (HDD)

---

## 5. Portainer

```bash
docker volume create portainer_data

docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

Acceso: `https://IP_DEL_SERVIDOR:9443` (certificado autofirmado, aviso normal del navegador)

```bash
sudo ufw allow 9443/tcp
sudo ufw allow 8000/tcp
```

---
