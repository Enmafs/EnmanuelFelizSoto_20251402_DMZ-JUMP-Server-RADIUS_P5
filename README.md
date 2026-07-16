# 🔐 Práctica 5 — RemoteApp (RDS/IIS) + AAA sobre RADIUS
                                                                                                                                                                 
**Estudiante:** Enmanuel Feliz Soto                                                                                                                   
**Matrícula:** 2025-1402                                                                                                                 
**Institución:** Instituto Tecnológico de Las Américas (ITLA)                                                                                  
**Curso:** Seguridad en Redes                                                                                                                              
**Sección:** 5, Lunes                                                                                                                                          
**Docente:** Jonathan Esteban Rondón Corniel                                                                                                                      

---

## 📋 Descripción

Publicación de una página web personalizada (IIS) como aplicación remota (RemoteApp) sobre Remote Desktop Services, asegurada con certificado SSL/TLS propio, y autenticación centralizada de acceso administrativo — tanto al portal web como a un router Cisco por SSH — mediante NPS/RADIUS integrado con Active Directory (AAA).

| Campo | Valor |
|-------|-------|
| **Tipo de despliegue** | RDS Quick Start (Session-Based) + RemoteApp |
| **Servicios** | IIS, RD Gateway, RD Web Access, RD Connection Broker, RD Licensing, NPS (RADIUS) |
| **Seguridad de transporte** | Certificado autofirmado SSL/TLS (SAN + EKU Server Authentication, 2048 bits) |
| **AAA** | NPS/RADIUS + Active Directory, grupos por nivel de privilegio (15 y 1) |
| **Dominio** | `NetSec.com` |
| **RemoteApp publicada** | Microsoft Edge → `http://localhost/laboratorio.html` |

---

## 🗺️ Topología

> 📸 <img width="1513" height="782" alt="Captura de pantalla 2026-07-16 163022" src="https://github.com/user-attachments/assets/b1fdfe9b-42e8-434d-bc48-9b5184e092ba" />


**Entorno:** PNetLab — Cisco IOL + Windows Server (RDS/IIS/NPS)
**Servidor RDS/IIS/NPS:** `WIN-BF2Q33KIN5K.NetSec.com`
**Router administrado vía RADIUS:** R1 (`14.2.10.1`)

### Tabla de Direccionamiento

| **Equipo** | **Rol** | **IP** | **Interfaz / Notas** |
|------------|---------|--------|------------------------|
| Servidor Windows Server | RDS + IIS + NPS + AD | 10.0.0.175 / 14.2.10.10 | `WIN-BF2Q33KIN5K.NetSec.com` |
| R1 | Cliente RADIUS (AAA) | 14.2.10.1 | Ethernet0/x, autenticación SSH vía NPS |
| ISP (SW1/SW2) | Interconexión LAN/WAN | USERS /25, SEREVR /24 | e0/0, e0/1, e0/2 según topología |
| Cliente Windows | Estación de prueba (admin15 / operator1) | DHCP interno | Acceso a RD Web Access y SSH |

### Dirección de Colección RemoteApp

| Recurso | Valor |
|----------|-------|
| Colección | `QuickSessionColection` |
| Aplicación publicada | Microsoft Edge → `laboratorio.html` |
| Certificado | CN=`win-bf2q33kin5k.netsec.com`, SAN + EKU Server Authentication |
| Zona horaria | SA Western Standard Time (UTC-4) |

---

## ⚙️ Configuración

El script y la documentación completa de configuración se encuentran en:
📄 [`EnmanuelFelizSoto_2025-1402_RDS_RemoteApp_RADIUS_P4.txt`](./EnmanuelFelizSoto_2025-1402_RDS_RemoteApp_RADIUS_P4.txt)

### Parámetros de Certificado / RADIUS

| Parámetro | Valor |
|-----------|-------|
| Longitud de clave | 2048 bits |
| EKU | Server Authentication (1.3.6.1.5.5.7.3.1) |
| Protocolo de autenticación NPS | PAP |
| Grupos AD | `GroupLevel15` (privilegio 15), `GroupLevel1` (privilegio 1) |
| Puertos RADIUS | 1812 (auth) / 1813 (acct) |

---

## ▶️ Procedimiento de Ejecución

### 1. Despliegue de roles (Windows Server)
```
# Orden de instalación:
# 1. Active Directory Domain Services + DNS
# 2. IIS
# 3. Remote Desktop Services (Quick Start)
# 4. Network Policy and Access Services (NPS)
```

### 2. Publicar el sitio y generar el certificado
```
# IIS Manager → Server Certificates → Create Self-Signed Certificate
# IIS Manager → Site Bindings → Add HTTPS (443) → seleccionar certificado
```

### 3. Asignar el certificado a los roles de RDS
```powershell
Get-RDCertificate
Set-RDCertificate -Role RDGateway,RDWebAccess,RDPublishing,RDRedirector -ImportPath cert.pfx
```

### 4. Configurar NPS/RADIUS y grupos AD
```
# Active Directory Users and Computers → New Group → GroupLevel15 / GroupLevel1
# NPS → RADIUS Clients → agregar R1 (14.2.10.1) con shared secret
# NPS → Network Policies → autenticación PAP
```

### 5. Configurar AAA en el router (R1)
```
aaa new-model
radius server NPS-NETSEC
 address ipv4 10.0.0.175 auth-port 1812 acct-port 1813
 key <clave-compartida>
aaa authentication login default group radius local
aaa authorization exec default group radius local
```

### 6. Verificar
```
show aaa servers
debug aaa authentication
debug radius
```
```
ssh admin15@14.2.10.1
ssh operator1@14.2.10.1
```
---

## 🔍 Análisis y Comparativa

### Ventajas de este esquema AAA centralizado
- Ver documentación técnica en el informe PDF/Word entregado.

### Incidente relevante resuelto
- Un desfase horario de ~7 horas entre servidor y cliente invalidaba silenciosamente la validación TLS del certificado (error "unexpected server authentication certificate"), sin relación aparente con el certificado en sí. Se corrigió sincronizando NTP en ambos extremos.
- Ver detalle completo en el informe técnico.

### Diferencias con otros labs
- Ver tabla comparativa en el README principal del repositorio.

---

## 📎 Recursos

| Recurso | Enlace |
|---------|--------|
| Repositorio Principal | [Enmafs/NetSec](https://github.com/Enmafs/NetSec) |
| Script / documentación de configuración | [`Script.txt`](https://github.com/Enmafs/EnmanuelFelizSoto_20251402_DMZ-JUMP-Server-RADIUS_P5/blob/main/EnmanuelFelizSoto_20251402_Script_P5) |
| Informe técnico (PDF) | [`Informe.PDF`](https://github.com/Enmafs/EnmanuelFelizSoto_20251402_DMZ-JUMP-Server-RADIUS_P5/blob/main/EnmanuelFelizSoto_20251402_Informe_P5.pdf) |
| Video demostración | 🎬 [Video](https://youtu.be/09i8jckBX54) |

---

> ⚠️ *Laboratorio realizado en entorno controlado (PNetLab / Windows Server de laboratorio). Fines exclusivamente académicos.*
