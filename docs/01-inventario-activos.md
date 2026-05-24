# 01 — Inventario de Activos Críticos

```
╔══════════════════════════════════════════════════════════════════╗
║        ANÁLISIS DE RIESGOS — FINTECH BANKING DIGITAL             ║
║        Sección 2: Identificación de Activos                      ║
╚══════════════════════════════════════════════════════════════════╝
```

**Proyecto:** Fintech — Aplicación Móvil de Banca Digital  
**Documento:** `docs/01-inventario-activos.md`  
**Versión:** 1.0  
**Fecha:** Mayo 2026  
**Metodología de valoración:** Criterios CIA — Guía SGSI

---

## Criterios de Valoración CIA

| Nivel          | Confidencialidad                     | Integridad                                     | Disponibilidad             |
| -------------- | ------------------------------------ | ---------------------------------------------- | -------------------------- |
| **Bajo**       | Información pública                  | Pérdida sin impacto                            | Disponible en +1 semana    |
| **Medio**      | Uso interno                          | Influencia menor                               | Disponible en 1 día        |
| **Medio-Alto** | Acceso restringido                   | Influencia notable, debe evitarse              | Disponible en horas        |
| **Alto**       | Confidencial, autorización explícita | Pérdida con impacto importante, evitar siempre | Disponible en todo momento |

---

## 2.1 Activos de Información

### Grupo A — Datos de Usuarios y Cuentas

| ID  | Activo                                                                                                                   | Tipo                                          | Clasificación | Propietario        | C    | I    | D          |
| --- | ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------- | ------------- | ------------------ | ---- | ---- | ---------- |
| A01 | Datos de identidad (nombre, DNI/CUIT, fecha nac., dirección)                                                             | Información Personal                          | **Alto**      | Compliance / Legal | Alto | Alto | Medio-Alto |
| A02 | Datos de contacto (email, teléfono, dirección de notificación)                                                           | Información Personal                          | **Alto**      | Compliance / Legal | Alto | Alto | Medio-Alto |
| A03 | Credenciales de acceso (hash de contraseñas, PINs cifrados)                                                              | Autenticación                                 | **Alto**      | Backend Team       | Alto | Alto | Alto       |
| A04 | Tokens de sesión (JWT access tokens, refresh tokens)                                                                     | Autenticación                                 | **Alto**      | Backend Team       | Alto | Alto | Alto       |
| A05 | Datos de cuentas bancarias (CBU, alias, número de cuenta, saldo disponible en tiempo real, movimientos débito y crédito) | Financiero                                    | **Alto**      | Operaciones        | Alto | Alto | Alto       |
| A06 | Historial de movimientos (montos, fechas, destinatarios, conceptos)                                                      | Financiero                                    | **Alto**      | Operaciones        | Alto | Alto | Alto       |
| A07 | Datos de tarjetas — PAN tokenizado (CVV nunca en texto plano)                                                            | PCI-DSS (estándar internacional de seguridad) | **Alto**      | Pasarela de Pagos  | Alto | Alto | Medio-Alto |

### Grupo B — Datos de Transferencias

| ID  | Activo                                                                 | Tipo                     | Clasificación  | Propietario        | C          | I    | D          |
| --- | ---------------------------------------------------------------------- | ------------------------ | -------------- | ------------------ | ---------- | ---- | ---------- |
| A08 | Órdenes de transferencia (origen, destino, monto, concepto, timestamp) | Financiero               | **Alto**       | Operaciones        | Alto       | Alto | Alto       |
| A09 | Comprobantes de transferencia (ID transacción, firma digital)          | Financiero               | **Alto**       | Operaciones        | Alto       | Alto | Alto       |
| A10 | Registros de transferencias fallidas o rechazadas                      | Financiero / Auditoría   | **Medio-Alto** | Operaciones        | Medio-Alto | Alto | Medio      |
| A11 | Límites de transferencia por usuario y período                         | Configuración de negocio | **Medio-Alto** | Riesgo Operacional | Medio-Alto | Alto | Medio-Alto |

### Grupo C — Datos de Préstamos

| ID  | Activo                                                                  | Tipo                              | Clasificación | Propietario       | C    | I    | D          |
| --- | ----------------------------------------------------------------------- | --------------------------------- | ------------- | ----------------- | ---- | ---- | ---------- |
| A12 | Solicitudes de préstamo (monto, destino, estado, documentación adjunta) | Financiero                        | **Alto**      | Riesgo Crediticio | Alto | Alto | Alto       |
| A13 | Scoring crediticio del usuario (score, historial, capacidad de pago)    | Financiero / Información Personal | **Alto**      | Riesgo Crediticio | Alto | Alto | Medio-Alto |
| A14 | Contratos de préstamo firmados digitalmente                             | Legal / Financiero                | **Alto**      | Legal             | Alto | Alto | Alto       |
| A15 | Cuotas, tasas de interés y calendario de amortización                   | Financiero                        | **Alto**      | Operaciones       | Alto | Alto | Alto       |
| A16 | Historial de morosidad y deudas del usuario                             | Financiero / Información Personal | **Alto**      | Riesgo Crediticio | Alto | Alto | Medio-Alto |

### Grupo D — Datos de Inversiones

| ID  | Activo                                                            | Tipo                              | Clasificación  | Propietario              | C     | I    | D          |
| --- | ----------------------------------------------------------------- | --------------------------------- | -------------- | ------------------------ | ----- | ---- | ---------- |
| A17 | Portfolio de inversiones (fondos, montos, porcentajes, valuación) | Financiero                        | **Alto**       | Inversiones              | Alto  | Alto | Alto       |
| A18 | Órdenes de suscripción / rescate de fondos                        | Financiero                        | **Alto**       | Inversiones              | Alto  | Alto | Alto       |
| A19 | Valuaciones y rendimientos históricos de fondos                   | Financiero                        | **Medio-Alto** | Inversiones              | Medio | Alto | Alto       |
| A20 | Perfil de inversor (tolerancia al riesgo, objetivos financieros)  | Información Personal / Financiero | **Alto**       | Inversiones / Compliance | Alto  | Alto | Medio-Alto |

### Grupo E — Datos de Pagos de Servicios

| ID  | Activo                                                         | Tipo                              | Clasificación  | Propietario | C          | I    | D          |
| --- | -------------------------------------------------------------- | --------------------------------- | -------------- | ----------- | ---------- | ---- | ---------- |
| A21 | Servicios adheridos (luz, agua, telefonía, etc.)               | Información Personal              | **Medio-Alto** | Operaciones | Medio-Alto | Alto | Medio-Alto |
| A22 | Historial de pagos de servicios (montos, fechas, comprobantes) | Informacion Personal / Financiero | **Alto**       | Operaciones | Alto       | Alto | Alto       |
| A23 | Códigos de pago y referencias de servicios                     | Operacional                       | **Medio**      | Operaciones | Medio      | Alto | Alto       |

### Grupo F — Datos Operacionales y de Seguridad

| ID  | Activo                                                                       | Tipo                  | Clasificación | Propietario       | C     | I     | D          |
| --- | ---------------------------------------------------------------------------- | --------------------- | ------------- | ----------------- | ----- | ----- | ---------- |
| A24 | Logs de auditoría de operaciones (quién, qué, cuándo, desde dónde)           | Auditoría             | **Alto**      | CISO / Compliance | Medio | Alto  | Alto       |
| A25 | Logs de seguridad y eventos de autenticación (intentos fallidos, logins)     | Seguridad             | **Alto**      | CISO              | Medio | Alto  | Alto       |
| A26 | Tokens de dispositivos para push (FCM / APNs)                                | Técnico               | **Medio**     | Mobile Team       | Medio | Medio | Medio      |
| A27 | Claves criptográficas (TLS privadas, claves JWT, llaves cifrado BD)          | Criptográfico         | **Alto**      | DevSecOps         | Alto  | Alto  | Alto       |
| A28 | Secretos de infraestructura (API keys de terceros, vars de entorno, pass BD) | Técnico / Secretos    | **Alto**      | DevOps            | Alto  | Alto  | Alto       |
| A29 | Código fuente app móvil (iOS y Android)                                      | Propiedad Intelectual | **Alto**      | CTO               | Alto  | Alto  | Medio-Alto |
| A30 | Código fuente backend (microservicios, lógica de negocio)                    | Propiedad Intelectual | **Alto**      | CTO               | Alto  | Alto  | Medio-Alto |

---

## 2.2 Activos Tecnológicos

### Grupo T1 — Aplicación Móvil (iOS / Android)

| ID  | Activo                                                     | Tipo           | Criticidad | Descripción                                                                                                          |
| --- | ---------------------------------------------------------- | -------------- | ---------- | -------------------------------------------------------------------------------------------------------------------- |
| T01 | App móvil iOS (Swift / SwiftUI)                            | Software       | **Alta**   | Cliente nativo iPhone/iPad. Todas las operaciones bancarias. Tokens en Keychain. Certificate pinning activo.         |
| T02 | App móvil Android (Kotlin / Jetpack Compose)               | Software       | **Alta**   | Cliente nativo Android. Equivalente funcional a iOS. Tokens en EncryptedSharedPreferences + Keystore.                |
| T03 | SDK de biometría integrado (Face ID / Touch ID / Huella)   | Software (lib) | **Alta**   | Autenticación biométrica local. Los datos biométricos NO salen del dispositivo.                                      |
| T04 | SDK de pasarela de pagos en app (tokenización de tarjetas) | Software (lib) | **Alta**   | Biblioteca PCI-DSS para capturar y tokenizar PAN directamente en el cliente. El backend nunca recibe el número real. |
| T05 | Cliente de notificaciones push (FCM / APNs)                | Software (lib) | **Media**  | Recibe alertas de movimientos y eventos de seguridad. El payload no debe contener datos financieros.                 |

### Grupo T2 — API Backend (Microservicios)

| ID  | Activo                                       | Tipo                       | Criticidad | Descripción                                                                                                                                                             |
| --- | -------------------------------------------- | -------------------------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| T06 | API Gateway (REST)                           | Software / Infraestructura | **Alta**   | Punto de entrada único. Autenticación JWT, rate limiting, enrutamiento y TLS termination.                                                                               |
| T07 | WAF (Web Application Firewall)               | Infraestructura            | **Alta**   | Filtra tráfico malicioso. Reglas OWASP Top 10: SQLi, XSS, CSRF, DoS. Primera línea de defensa.                                                                          |
| T08 | Microservicio de Autenticación               | Software                   | **Alta**   | Login, registro, MFA, emisión/validación JWT, refresh tokens y blacklisting de sesiones. OAuth 2.0 + OIDC.                                                              |
| T09 | Microservicio de Cuentas y Saldos            | Software                   | **Alta**   | Consulta y actualización de saldos, historial de movimientos, resumen de cuenta.                                                                                        |
| T10 | Microservicio de Transferencias              | Software                   | **Alta**   | Procesamiento de transferencias propias y a terceros (CBU/alias). Validación de límites, fondos y antifraude. Lógica con bloqueo pesimista para evitar race conditions. |
| T11 | Microservicio de Préstamos                   | Software                   | **Alta**   | Evaluación de solicitudes, cálculo de scoring, aprobación/rechazo, cuotas y amortización.                                                                               |
| T12 | Microservicio de Inversiones                 | Software                   | **Alta**   | Suscripción y rescate de fondos, cálculo de rendimientos, integración con administradoras.                                                                              |
| T13 | Microservicio de Pagos de Servicios          | Software                   | **Alta**   | Adhesión de servicios, consulta de deuda, ejecución de pago, emisión de comprobante.                                                                                    |
| T14 | Microservicio de Notificaciones              | Software                   | **Media**  | Orquesta envío de push, emails y SMS ante eventos del sistema. No procesa datos financieros directamente.                                                               |
| T15 | Motor de antifraude (Fraud Detection Engine) | Software                   | **Alta**   | Análisis en tiempo real de transacciones. Detecta patrones sospechosos. Puede bloquear o escalar operaciones.                                                           |

### Grupo T3 — Base de Datos y Almacenamiento

| ID  | Activo                                    | Tipo                | Criticidad | Descripción                                                                                                                        |
| --- | ----------------------------------------- | ------------------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| T16 | PostgreSQL — Producción (primary)         | Software / Hardware | **Alta**   | Motor principal. Contiene todos los datos de usuarios, cuentas, transacciones, préstamos e inversiones. Cifrado en reposo AES-256. |
| T17 | PostgreSQL — Réplica de lectura           | Software / Hardware | **Alta**   | Réplica sincrónica para consultas de alto volumen y failover. Mismo nivel CIA que la primaria.                                     |
| T18 | Redis — Caché y sesiones activas          | Software            | **Alta**   | Almacena tokens JWT activos, caché de datos y rate limiting. Comprometido → session hijacking masivo.                              |
| T19 | Almacenamiento de objetos (S3-compatible) | Infraestructura     | **Alta**   | Contratos PDF, comprobantes, documentación KYC. Acceso por IAM con políticas restrictivas.                                         |

### Grupo T4 — Pasarela de Pagos e Integraciones Externas

| ID  | Activo                                               | Tipo             | Criticidad | Descripción                                                                                   |
| --- | ---------------------------------------------------- | ---------------- | ---------- | --------------------------------------------------------------------------------------------- |
| T20 | Pasarela de pagos externa (PCI-DSS Level 1)          | Servicio externo | **Alta**   | Procesamiento de pagos con tarjeta. Solo recibe tokens. Comunicación HTTPS + HMAC-SHA256.     |
| T21 | Integración red interbancaria (Red bancaria nacional/interbancaria / LINK / BCU) | Servicio externo | **Alta**   | Ejecución de transferencias CBU interbancarias. SLA crítico. Certificados mutuos (mTLS).      |
| T22 | Integración bureau de crédito (Veraz / Nosis)        | Servicio externo | **Alta**   | Consulta de scoring externo para préstamos. Datos confidenciales del usuario.                 |
| T23 | Integración administradoras de fondos                | Servicio externo | **Alta**   | Suscripción/rescate de cuotapartes. API REST con certificado cliente.                         |
| T24 | Proveedor SMS / OTP (Twilio / similar)               | Servicio externo | **Media**  | Envío de códigos MFA y alertas de seguridad. Vulnerable a SIM swapping si es el único factor. |
| T25 | Servicio de validación de identidad KYC DNIC         | Servicio externo | **Alta**   | Validación biométrica durante onboarding. Verificacion de de identidad por DNIC               |

### Grupo T5 — Infraestructura y DevSecOps

| ID  | Activo                                                     | Tipo            | Criticidad | Descripción                                                                                                                        |
| --- | ---------------------------------------------------------- | --------------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| T26 | VPC cloud (subnets, security groups, NACLs)                | Infraestructura | **Alta**   | Segmentación de red: zona pública (WAF/LB), privada (microservicios) y datos (BD).                                                 |
| T27 | Balanceador de carga (Load Balancer)                       | Infraestructura | **Alta**   | Distribución de tráfico entre instancias del API Gateway. TLS termination y health checks.                                         |
| T28 | CDN (Content Delivery Network)                             | Infraestructura | **Media**  | Distribución de assets estáticos y primera línea de mitigación DDoS.                                                               |
| T29 | Gestor de secretos (HashiCorp Vault / AWS Secrets Manager) | Software        | **Alta**   | Almacenamiento centralizado de credenciales, claves API, certificados. Rotación automática. Si es comprometido → compromiso total. |
| T30 | Pipeline CI/CD (GitHub Actions / GitLab CI)                | Software        | **Alta**   | Build, SAST, DAST y despliegue automatizado. Comprometido → inyección de código en producción.                                     |
| T31 | Sistema SIEM                                               | Software        | **Alta**   | Centralización y correlación de logs. Alertas en tiempo real ante anomalías.                                                       |
| T32 | Sistema de backup y DR                                     | Infraestructura | **Alta**   | Snapshots automáticos de PostgreSQL, retención 30 días, pruebas mensuales de restauración.                                         |
| T33 | Certificados TLS y PKI interna                             | Criptográfico   | **Alta**   | Certificados para cifrado externo e interno (mTLS entre microservicios).                                                           |

---

## 2.3 Activos Intangibles

| Activo                                                                              | Descripción                                                                                                                                      | Impacto si es comprometido                                                      |
| ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| Reputación de la fintech                                                            | Imagen ante usuarios, inversores y reguladores. En banca digital, la confianza es el producto.                                                   | Pérdida masiva de usuarios, caída de valuación, cobertura negativa en medios    |
| Confianza de los usuarios                                                           | Percepción de seguridad al delegar operaciones financieras. Se construye con años, se destruye en minutos.                                       | Churn acelerado, reviews negativas en stores, imposibilidad de retener clientes |
| Licencia operativa BCU                                                              | Habilitación del Banco Central del Uruguay para operar como Institución de Dinero Electrónico o Prestadora de Servicios de Pago bajo Ley 19.210. | Suspensión de operaciones, multas de hasta UI 2.000.000, revocación de licencia |
| Cumplimiento regulatorio BCU                                                        | HAdhesión a la Circular 2.244 del BCU (Marco de Ciberseguridad), Ley 18.331 (Protección de Datos Personales) y Ley 19.574 (Antilavado).          | Sanciones económicas, planes de remediación obligatorios, intervención del BCU  |
| Algoritmos de scoring crediticio                                                    | Modelos propietarios de riesgo crediticio desarrollados con datos propios.                                                                       | Pérdida de ventaja competitiva, posible uso por competidores                    |
| Datos de comportamiento financiero agregados                                        | Patrones de gasto, ahorro e inversión. Activo estratégico para productos personalizados.                                                         | Multas LGPD/GDPR, pérdida de activo estratégico                                 |
| Propiedad intelectual del código                                                    | Lógica de negocio, algoritmos y arquitectura de la plataforma.                                                                                   | Competencia desleal, clonación del producto                                     |
| Relaciones con socios bancarios                                                     | Acuerdos con bancos corresponsales, redes de pagos y administradoras.                                                                            | Pérdida de capacidad operativa si el socio termina el acuerdo por incidente     |
| Autorización de redes (Visa/Mastercard) para procesar tarjetas de débito y crédito. | Pérdida de capacidad operativa si el socio termina el acuerdo por incidente                                                                      | Multas USD 5.000–100.000/mes, inhabilitación para procesar pagos con tarjeta    |

---

## 2.4 Mapa de Criticidad — Resumen Visual

```
 CRITICIDAD DE ACTIVOS DE INFORMACIÓN
 ──────────────────────────────────────────────────────────────────
 🔴 MÁXIMA  │ A03 A04 A05 A06 A08 A09 A12 A14 A15 A17 A18 A27 A28
            │ Credenciales · Cuentas · Transferencias · Préstamos
            │ Inversiones · Claves criptográficas · Secretos
 ──────────────────────────────────────────────────────────────────
 🟠 ALTA    │ A01 A02 A07 A10 A11 A13 A16 A20 A22 A24 A25 A29 A30
            │ Información Personal · Tarjetas (token) · Scoring · Auditoría · IP
 ──────────────────────────────────────────────────────────────────
 🟡 MEDIA   │ A19 A21 A23 A26
            │ Valuaciones históricas · Servicios adheridos · Push tokens
 ──────────────────────────────────────────────────────────────────

 CRITICIDAD DE ACTIVOS TECNOLÓGICOS
 ──────────────────────────────────────────────────────────────────
 🔴 MÁXIMA  │ T06 T08 T10 T11 T12 T16 T18 T20 T21 T29 T30
            │ API GW · Auth · Transferencias · Préstamos · Inversiones
            │ PostgreSQL · Redis · Pasarela · Red interbancaria · Vault · CI/CD
 ──────────────────────────────────────────────────────────────────
 🟠 ALTA    │ T01 T02 T07 T09 T13 T15 T17 T19 T22 T23 T25 T26 T31 T32 T33
            │ Apps móviles · WAF · Cuentas · Pagos · Antifraude · KYC · SIEM
 ──────────────────────────────────────────────────────────────────
 🟡 MEDIA   │ T03 T04 T05 T14 T24 T27 T28
            │ Biometría local · SDK pagos · Push client · Notificaciones · CDN
 ──────────────────────────────────────────────────────────────────
```

---

## 2.5 Supuestos del Inventario

1. El PAN de tarjetas **nunca** se almacena en el backend; solo tokens PCI-DSS.
2. Las claves criptográficas residen en el gestor de secretos (Vault), **nunca** en repositorios Git.
3. La app móvil usa almacenamiento seguro del SO: Keychain (iOS) y Keystore (Android).
4. Los datos biométricos se procesan localmente y no se transmiten al servidor.
5. El código fuente está en repositorios privados con acceso por roles (mínimo privilegio).
6. Toda comunicación entre cliente y servidor usa TLS 1.3 como mínimo.
7. Los microservicios se comunican entre sí por mTLS dentro de la VPC privada.

---

## 2.6 Componentes Fuera de Alcance

| Componente                                  | Razón de exclusión                                               |
| ------------------------------------------- | ---------------------------------------------------------------- |
| Infraestructura física del data center      | Responsabilidad del proveedor cloud (responsabilidad compartida) |
| Dispositivos personales de empleados (BYOD) | Cubierto por política MDM separada                               |
| Panel de administración interno             | Threat model separado                                            |
| Sistemas legados del banco corresponsal     | Bajo control de tercero, no modificable                          |

---
