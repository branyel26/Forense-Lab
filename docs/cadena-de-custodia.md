# Cadena de Custodia — Formulario Oficial
### Ministerio Público · Caso No. 2026-001

**Instituto Tecnológico de las Américas (ITLA)**  
Asignatura: Informática Forense · Profesor: Cándido Noel Ramírez  
Integrantes: Branyel Pérez · Francis Vidal · Elian Leonardo  
Fecha de entrega: 24/04/2026

---

## Información del Caso

| Campo | Detalle |
|---|---|
| Número de caso | 2026-001 |
| Delito | Acceso ilícito a la red de la empresa |
| Oficiales | Elian Leonardo (2023-0961) · Branyel Pérez (2024-1489) · Francis Vidal (2024-1183) |
| Víctima | Path Secure SRL |
| Sospechoso | No identificado |
| Fecha / Hora | 23 de abril de 2026 — 11:45 AM |
| Lugar | Oficinas de la empresa, Santo Domingo Este |

---

## Descripción de la Evidencia

| Objeto | Cantidad | Descripción |
|---|---|---|
| Volcado de Memoria RAM | — | Volcado de memoria y paginación — RAM de 84 GB DDR4, velocidad 4800 MHz, formato DIMM |
| Disco Duro | 1 | Western Digital, Modelo WD10EZEX, 1 Terabyte, color plateado |

---

## Registro de Transferencia

| Objeto | Fecha / Hora | Entrega (Firma / ID) | Recibe (Firma / ID) | Comentarios |
|---|---|---|---|---|
| Volcado RAM | 23/04/2026 — 2:58 PM | Elian Leonardo · 2023-0961 | Francis Vidal · 2024-1183 | Archivo `memoria_ram.mem` — MD5 y SHA1 verificados con HashCalc |
| Disco Duro | 23/04/2026 — 2:58 PM | Branyel Pérez · 2024-1489 | Francis Vidal · 2024-1183 | Imagen E01 adquirida con FTK Imager 4.7.3.81 — MD5/SHA1 MATCH |

---

## Hashes de Integridad

| Evidencia | MD5 | SHA1 |
|---|---|---|
| Imagen disco E01 | `9ff28a94e47dfa529faf4c37f23a0d83` | `6343b0e37c873f564b728860d726fa1103e47196` |
| Volcado RAM (`memoria_ram.mem`) | `0b0c8d77c430517fce4e50dc6f7cef9d` | `1bc79849f5e82fc267bcd3be8f3c5f17f7cf3e7b` |

> Los hashes fueron calculados de forma independiente con **HashCalc** y verificados por **FTK Imager 4.7.3.81** al momento de la adquisición, confirmando que las imágenes no fueron alteradas durante el proceso.

---

*Documento elaborado por el Grupo 6 — Informática Forense · ITLA · Santo Domingo, República Dominicana*
