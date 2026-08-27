# Red Empresarial Multisede
# 🔧 Network Infrastructure Project

<div align="center">

![Cisco](https://img.shields.io/badge/Cisco-IOS-blue.svg)
![OSPF](https://img.shields.io/badge/Routing-OSPF-green.svg)
![IPsec](https://img.shields.io/badge/VPN-IPsec%20IKEv2-red.svg)

*Diseño e implementación de una red empresarial multisede con VLANs, OSPF, HSRP, DHCP Relay y un túnel GRE sobre IPsec entre sedes*

</div>

---

## 📋 Tabla de Contenidos

- [Objetivo del Proyecto](#-objetivo-del-proyecto)
- [Características Principales](#-características-principales)
- [Capturas de Pantalla](#-capturas-de-pantalla)
- [Topología de Red](#-topología-de-red)
- [Tabla de Direccionamiento](#-tabla-de-direccionamiento)
- [Parámetros del Túnel WAN](#-parámetros-del-túnel-wan)
- [Verificación](#-verificación)

---

## 🎯 Objetivo del Proyecto

Diseñar e implementar una red empresarial de dos sedes (**Matriz** y **Sede STG**) interconectadas mediante un túnel cifrado sobre Internet, con segmentación por VLANs, redundancia de gateway, enrutamiento dinámico y asignación centralizada de direcciones IP, simulando un escenario corporativo real de nivel intermedio-avanzado.

## ⚙️ Características Principales

- 🧩 **VLANs + Trunking 802.1Q**: segmentación por departamento en ambas sedes
- 🔗 **EtherChannel**: agregación de enlaces entre switches
- 🔒 **Port Security**: control de acceso en puertos de usuario
- 🌐 **OSPF multi-área**: enrutamiento dinámico con autenticación MD5
- ⚡ **HSRP v2**: redundancia y failover automático del gateway por VLAN
- 📡 **DHCP + DHCP Relay**: asignación centralizada de IP a múltiples VLANs
- 🔐 **GRE sobre IPsec (IKEv2)**: túnel sitio a sitio cifrado entre sedes
- 🚪 **NAT/PAT**: salida a Internet en el router de borde

---

- Topología general de la red
  
<img width="359" height="140" alt="Screenshot 2026-08-26 164308" src="https://github.com/user-attachments/assets/4183c7c5-e87b-4c85-99dd-100323614a66" />


---
- Tabla de VLANs y direccionamiento IP

<img width="302" height="216" alt="Screenshot 2026-08-26 164643" src="https://github.com/user-attachments/assets/4249c3fa-286e-4696-8c3b-944e69124b16" />


---
- Configuración de VLANs con HSRP y autenticación OSPF

<img width="590" height="952" alt="Screenshot 2026-08-14 164020" src="https://github.com/user-attachments/assets/18d71836-fffe-48da-8cfe-8042a81a2949" />


---
- Tabla de rutas OSPF 

 <img width="931" height="532" alt="Screenshot 2026-08-14 164052" src="https://github.com/user-attachments/assets/91370bce-85df-4e9f-bb1e-da59b2a4c47c" />


---
- Configuración del servidor DHCP 

 <img width="526" height="903" alt="Screenshot 2026-08-14 163452" src="https://github.com/user-attachments/assets/c481e35a-d41a-4218-ac0d-edd77a4be9e3" />


---
- Estado de las SA de IKEv2 

  <img width="897" height="496" alt="Screenshot 2026-08-18 140303" src="https://github.com/user-attachments/assets/cd736416-c0e3-40ef-ad8a-6bf8f4908c45" />


---
- Socket criptográfico del túnel 

 <img width="899" height="491" alt="Screenshot 2026-08-18 140419" src="https://github.com/user-attachments/assets/2e1e9125-c306-41c3-9932-8be36ded5982" />


---
- Captura Wireshark con tráfico ESP cifrado
  
<img width="920" height="389" alt="Screenshot 2026-08-18 140759" src="https://github.com/user-attachments/assets/cd23e4a0-5e37-4f4c-aa4e-dde15c18c6bb" />

---
- Ping exitoso entre VPCs de distintas sedes a través del túnel

<img width="1708" height="497" alt="Screenshot 2026-08-18 140715" src="https://github.com/user-attachments/assets/0d7728ef-fcc9-4d7b-bb0c-9952c670683e" />

---

## 🗺️ Topología de Red

La red está formada por dos sedes conectadas por Internet a través de un túnel cifrado:

```
   Sede Principal (AREA 1 / AREA 0 / AREA 2)
        |
    [EDGE-RT] --203.45.67.90-- CLARO (ISP) --203.45.67.98-- [EDGE-STG]
        |            IPsec (IKEv2) + GRE — Tunnel 172.16.0.0/30           |
        |                                                              Sede STG
   Switches de distribución (HSRP + OSPF)                    Switches de distribución (HSRP + OSPF)
```

- **Matriz**: dividida en tres áreas OSPF, switches de distribución `SD-MT1`/`SD-MT2` en HSRP.
- **Sede STG**: sucursal remota con su propio esquema de VLANs y switch de distribución.
- **Interconexión**: túnel GRE cifrado con IPsec/IKEv2 entre los routers de borde `EDGE-RT` y `EDGE-STG`, que además llevan el tráfico OSPF entre ambas sedes.
- **Servidor DHCP** centralizado (Linux) atendiendo todas las VLANs mediante DHCP Relay.


---

## 🔐 Parámetros del Túnel WAN

| Parámetro | Valor |
|---|---|
| Peer local (EDGE-RT) | 203.45.67.90 |
| Peer remoto (EDGE-STG) | 203.45.67.98 |
| Red del túnel (Tu0) | 172.16.0.0/30 |
| Negociación | IKEv2 |
| Perfil IPsec | `GRE` |
| Cliente crypto | `TUNNEL SEC` |

---

## ✅ Verificación

| Comando / Herramienta | Qué confirma |
|---|---|
| `show crypto isakmp sa` | SAs de IKEv2 activas (`QM_IDLE` / `ACTIVE`) entre las IPs públicas |
| `show crypto socket` | Túnel `Tu0` con estado `Open` y perfil `GRE` |
| `show ip route ospf` | Rutas internas, inter-área y ruta por defecto vía OSPF |
| Wireshark (interfaz WAN) | Tráfico **ESP** entre routers de borde → confirma cifrado real |
| `ping` entre VPCs de distintas sedes | Conectividad end-to-end a través del túnel |

---

<div align="center">

**Proyecto de Infraestructura de Redes — Nivel Avanzado**

#                 Sr.Minyete

</div>
