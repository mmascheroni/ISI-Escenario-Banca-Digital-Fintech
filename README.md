# Fintech Banking Digital — Threat Modeling

## Integrantes
- Mauro Mascheroni
- Ximena Gonzalez

## Escenario
Sistema de Banking Digital (Escenario 1)
Análisis de riesgos y modelado de amenazas aplicado a una plataforma fintech de banca digital.

El sistema permite:
- Consulta de saldos y movimientos
- Transferencias entre cuentas
- Pagos de servicios
- Solicitud de préstamos online
- Inversión en fondos

El alcance del análisis incluye:
- Aplicación móvil iOS/Android
- API Backend REST
- Microservicios
- Base de datos PostgreSQL
- Integraciones externas
- Pasarela de pagos
- Sistema de notificaciones push

---

## Objetivo

Aplicar metodologías de análisis de riesgos y threat modeling sobre una arquitectura fintech moderna, identificando amenazas relevantes, priorizando riesgos y definiendo controles de mitigación alineados con estándares de seguridad reconocidos.

---

## Metodologías Utilizadas

- STRIDE
- DREAD
- MITRE ATT&CK
- NIST SP 800-53
- ISO 27001

---

## Contenido del Repositorio

```text
docs/
├── 01-inventario-activos.md
├── 02-arquitectura-trust-boundaries.md
├── 03-analisis-stride.md
├── 04-priorizacion-dread.md
├── 05-mapa-mitre-attack.md
├── 06-plan-mitigacion.md
└── 07-riesgos-residuales.md

diagrams/
├── arquitectura-fintech-mejorado.png
└── arquitectura-fintech.drawio.png

prompts/
├── 00-memory-bank.md
└── 01-diagrama_arquitectura.md
```
---

## Herramientas utilizadas
- VSC
- draw.io
- GitHub

## IA utilizada
- ChatGPT para asistencia en documentación y refinamiento visual

## Referencias
- OWASP Threat Modeling
- MITRE ATT&CK
- NIST SP 800-30
- NIST SP 800-53
- ISO 27001
- OWASP Top 10