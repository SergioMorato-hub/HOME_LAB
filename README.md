# 🧪 Home Lab

> Un laboratorio casero para aprender redes, virtualización, seguridad y automatización, con infraestructura real y documentación versionada.

![Estado](https://img.shields.io/badge/estado-en%20construcci%C3%B3n-orange)
![Switch](https://img.shields.io/badge/switch-Cisco%20CBS250-049fd9)
![Firmware](https://img.shields.io/badge/firmware-v3.6.0.3-success)
![Licencia](https://img.shields.io/badge/licencia-MIT-blue)

---

## 🗺️ Topología

```
                    ┌──────────────────┐
   Internet ────────┤  USG Pro 4 (*)   │
                    └────────┬─────────┘
                             │ trunk (VLAN 10, 20, 30)
                    ┌────────┴─────────┐
                    │  SW-HOME-LAB     │
                    │  Cisco CBS250    │
                    └─┬─────────┬────┬─┘
                      │         │    │
            ┌─────────┴──┐  ┌───┴────┐ ┌┴───────────┐
            │ VLAN 20    │  │ VLAN 30│ │ VLAN 1     │
            │ LAB        │  │ IOT    │ │ MNGT       │
            │ gi1–gi4    │  │ gi5–gi8│ │ gi10       │
            └────────────┘  └────────┘ └────────────┘
```

> 📸 *Captura: diagrama de topología (`docs/topology.png`)*

(*) Equipo previsto, en proceso de configuración.

---

## 🧰 Stack

| Capa | Tecnología | Función |
|---|---|---|
| Conmutación | Cisco CBS250 (`SW-HOME-LAB`) | VLANs, trunk y acceso por puerto |
| Enrutamiento y firewall | Ubiquiti USG Pro 4 | Segmentación y políticas entre VLANs |
| Computación | Raspberry Pi | Servicios ligeros y pruebas |
| Servidor | Debian | Servicios y virtualización |
| Acceso remoto | WireGuard | VPN para entrar al laboratorio |
| Monitorización | Grafana | Métricas y paneles |
| Automatización | Bash y PowerShell | Backups y comprobaciones |

---

## 🌐 Segmentación por VLANs

| VLAN | Nombre | Uso |
|:---:|---|---|
| 1 | `MNGT` | Gestión de equipos |
| 20 | `LAB` | Servidores y pruebas |
| 30 | `IOT` | Dispositivos IoT, aislados del resto |

> 📸 *Captura: salida de `show vlan`*

---

## 📁 Estructura del repositorio

```
homelab/
├── config/        # Configuraciones de equipos (sin secretos)
├── docs/          # Arquitectura, decisiones, runbooks
├── scripts/       # Backups y comprobaciones automáticas
├── diagrams/      # Topología (drawio y svg)
└── site/          # Página interactiva (GitHub Pages)
```

---

## 🚀 Puesta en marcha

1. Clona el repositorio:
```bash
   git clone https://github.com/TU_USUARIO/homelab.git
```
2. Revisa `docs/runbooks/backup-restore.md` antes de tocar cualquier equipo.
3. Ejecuta el backup del switch:
```bash
   ./scripts/backup-switch.sh
```

---

## 🛡️ Seguridad

- Las contraseñas y claves **nunca** se suben al repositorio.
- Los backups de configuración se guardan fuera de Git.
- El acceso administrativo al switch se hace por SSH.

---

## 🗺️ Roadmap

- [x] Switch con VLANs `LAB` e `IOT`
- [x] Firmware del switch actualizado
- [x] Backups de configuración del switch
- [ ] Integración del USG Pro 4 como router y firewall
- [ ] Reglas de firewall entre VLANs
- [ ] Acceso remoto con WireGuard
- [ ] Monitorización con Grafana
- [ ] Servicios desplegados en el servidor Debian

---

## 📸 Galería

| | |
|:---:|:---:|
| *Captura 1: `show version`* | *Captura 2: `show interfaces status`* |
| *Captura 3: panel de Grafana* | *Captura 4: página del proyecto* |

---

## 📜 Licencia

Distribuido bajo licencia MIT. Consulta el archivo [LICENSE](LICENSE).

---

<p align="center">
  Hecho con ☕, cables y muchas pruebas en <b>Madrid</b>.
</p>