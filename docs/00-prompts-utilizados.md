# Registro de Prompts Utilizados

Este documento recopila los prompts utilizados como apoyo para:
- refinamiento visual,
- estructuración de documentación,
- generación de diagramas,
- mejora de redacción técnica.

La arquitectura, amenazas y decisiones de seguridad fueron definidas manualmente por el equipo y luego refinadas con asistencia de IA.

## Prompt — Mejora visual del diagrama de arquitectura

Tomar como base un diagrama de arquitectura fintech ya realizado manualmente y mejorarlo visualmente para una presentación universitaria de threat modeling.

Mantener:

* trust boundaries,
* arquitectura por capas,
* API Gateway,
* microservicios,
* capa de datos,
* integraciones externas.

Agregar:

* labels TB-1 a TB-6,
* colores suaves por zona,
* ícono de atacante externo,
* protocolos HTTPS/TLS y mTLS,
* mejor alineación visual,
* leyenda de trust boundaries.

Estilo:

* profesional,
* limpio,
* moderno,
* orientado a ciberseguridad y arquitectura cloud.

La arquitectura representa una app de banca digital/fintech con:

* apps móviles iOS y Android,
* WAF + CDN + Load Balancer,
* API Gateway,
* microservicios (Auth, Transferencias, Préstamos, Pagos, Inversiones),
* PostgreSQL,
* Redis,
* S3,
* integraciones externas de pagos, KYC y OTP.

El resultado debe verse claro, académico y fácil de explicar oralmente.
