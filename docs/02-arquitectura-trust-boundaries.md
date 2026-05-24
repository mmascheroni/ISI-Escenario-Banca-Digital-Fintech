# 02 — Arquitectura del Sistema y Trust Boundaries

```
╔══════════════════════════════════════════════════════════════════╗
║        ANÁLISIS DE RIESGOS — FINTECH BANKING DIGITAL            ║
║        Sección 3: Arquitectura del Sistema                      ║
╚══════════════════════════════════════════════════════════════════╝
```

**Proyecto:** Fintech — Aplicación Móvil de Banca Digital  
**Documento:** `docs/02-arquitectura-trust-boundaries.md`  
**Versión:** 1.0  
**Fecha:** Mayo 2026

---

## 3.1 Descripción General de la Arquitectura

El sistema está compuesto por una **arquitectura de microservicios** desplegada en la nube, organizada en zonas de confianza progresivamente más restringidas. El acceso externo pasa siempre por múltiples capas de seguridad antes de alcanzar los datos sensibles.

La arquitectura se divide en **5 zonas** separadas por Trust Boundaries:

| Zona                                        | Descripción                                           | Nivel de Confianza |
| ------------------------------------------- | ----------------------------------------------------- | ------------------ |
| **Zona 0** — Internet / Dispositivo cliente | Entorno del usuario final, completamente no confiable | ❌ Sin confianza   |
| **Zona 1** — Perímetro (CDN + WAF + LB)     | Primera línea de defensa, filtrado de tráfico         | ⚠️ Muy bajo        |
| **Zona 2** — API Gateway                    | Autenticación y enrutamiento, zona semipública        | 🔸 Bajo            |
| **Zona 3** — Microservicios (VPC privada)   | Lógica de negocio, comunicación interna mTLS          | 🔶 Medio           |
| **Zona 4** — Capa de datos                  | Bases de datos y almacenamiento, máxima restricción   | 🔒 Alto            |
| **Zona 5** — Integraciones externas         | Terceros de confianza contractual (pagos, BCU, KYC)   | 🔸 Bajo-Medio      |

---

## 3.2 Diagrama de Arquitectura con Trust Boundaries

```
╔═══════════════════════════════════════════════════════════════════════════════════╗
║  INTERNET — ZONA NO CONFIABLE                                                     ║
║                                                                                   ║
║   ┌──────────────┐    ┌──────────────┐    ┌─────────────────┐                    ║
║   │  Usuario     │    │  Atacante    │    │  Servicios      │                    ║
║   │  Final       │    │  Externo     │    │  Externos       │                    ║
║   │  📱 iOS/And  │    │  💀          │    │  (Pagos/BCU)   │                    ║
║   └──────┬───────┘    └──────┬───────┘    └────────┬────────┘                    ║
║          │ HTTPS/TLS 1.3     │ (bloqueado)          │ HTTPS+mTLS                  ║
╚══════════╪═══════════════════╪══════════════════════╪════════════════════════════╝
           │                   │                      │
╔══════════╪═══════════════════╪══════════════════════╪════════════════════════════╗
║ ◄──── TRUST BOUNDARY 1: Perímetro Externo ──────────────────────────────────► ║
║  CDN + WAF + Load Balancer                                                        ║
║                                                                                   ║
║   ┌──────▼───────────────────▼──────────────────────────────────┐                ║
║   │  CDN (CloudFront / similar)                                  │                ║
║   │  · Distribución de assets estáticos                         │                ║
║   │  · Absorción DDoS volumétrico (T28)                         │                ║
║   └──────────────────────┬───────────────────────────────────────┘                ║
║                          │                                                        ║
║   ┌──────────────────────▼───────────────────────────────────────┐                ║
║   │  WAF — Web Application Firewall (T07)                        │                ║
║   │  · Reglas OWASP Top 10                                       │                ║
║   │  · Bloqueo SQLi / XSS / CSRF                                 │                ║
║   │  · Rate limiting por IP                                      │                ║
║   │  · Geo-blocking configurable                                 │                ║
║   └──────────────────────┬───────────────────────────────────────┘                ║
║                          │                                                        ║
║   ┌──────────────────────▼───────────────────────────────────────┐                ║
║   │  Load Balancer (T27)                                         │                ║
║   │  · TLS termination (certificado público)                     │                ║
║   │  · Health checks                                             │                ║
║   │  · Distribución de tráfico                                   │                ║
║   └──────────────────────┬───────────────────────────────────────┘                ║
╚══════════════════════════╪════════════════════════════════════════════════════════╝
                           │
╔══════════════════════════╪════════════════════════════════════════════════════════╗
║ ◄──── TRUST BOUNDARY 2: API Gateway (Zona Semipública) ──────────────────────► ║
║                                                                                   ║
║   ┌──────────────────────▼───────────────────────────────────────┐                ║
║   │  API Gateway REST (T06)                                      │                ║
║   │  · Validación de JWT (firma RS256, expiración, claims)       │                ║
║   │  · Rate limiting por usuario y por endpoint                  │                ║
║   │  · Enrutamiento a microservicios                             │                ║
║   │  · Logging de todas las requests entrantes                   │                ║
║   │  · Validación estricta de JWT prevista mediante RS256        │                ║
║   └──────────────────────┬───────────────────────────────────────┘                ║
╚══════════════════════════╪════════════════════════════════════════════════════════╝
                           │ mTLS (certificados mutuos entre servicios)
╔══════════════════════════╪════════════════════════════════════════════════════════╗
║ ◄──── TRUST BOUNDARY 3: Microservicios — VPC Privada ────────────────────────► ║
║                                                                                   ║
║  ┌────────────────────────────────────────────────────────────────────────────┐  ║
║  │  VPC Privada — Subred de Aplicación                                        │  ║
║  │                                                                            │  ║
║  │  ┌─────────────────┐   ┌─────────────────┐   ┌────────────────────────┐   │  ║
║  │  │ Auth Service    │   │ Account Service  │   │ Transfer Service       │   │  ║
║  │  │ (T08)           │   │ (T09)            │   │ (T10)                  │   │  ║
║  │  │ · OAuth 2.0     │   │ · Consulta saldo │   │ · Validación fondos    │   │  ║
║  │  │ · JWT emisión   │   │ · Movimientos    │   │ · Estrategia de control concurrente en transferencias    │   │  ║
║  │  │ · MFA / TOTP    │   │ · Límites cuenta │   │ · Antifraude           │   │  ║
║  │  │ · Blacklist     │   └────────┬─────────┘   │ · Idempotency keys     │   │  ║
║  │  └────────┬────────┘            │              └───────────┬────────────┘   │  ║
║  │           │                     │                          │                │  ║
║  │  ┌────────▼────────┐   ┌────────▼─────────┐   ┌──────────▼────────────┐   │  ║
║  │  │ Loan Service    │   │ Investment Svc    │   │ Payment Service        │   │  ║
║  │  │ (T11)           │   │ (T12)             │   │ (T13)                  │   │  ║
║  │  │ · Scoring       │   │ · Suscripción     │   │ · Adhesión servicios   │   │  ║
║  │  │ · Aprobación    │   │ · Rescate fondos  │   │ · Pago y comprobante   │   │  ║
║  │  │ · Cuotas        │   │ · Rendimientos    │   │                        │   │  ║
║  │  └────────┬────────┘   └────────┬──────────┘   └──────────┬────────────┘   │  ║
║  │           │                     │                          │                │  ║
║  │  ┌────────▼─────────────────────▼──────────────────────────▼────────────┐  │  ║
║  │  │  Fraud Detection Engine (T15) · Notification Service (T14)           │  │  ║
║  │  └────────────────────────────────────────────────────────────────────┘  │  ║
║  └────────────────────────────────────────────────────────────────────────────┘  ║
╚══════════════════════════╪════════════════════════════════════════════════════════╝
                           │ Conexiones de BD cifradas (TLS + credenciales Vault)
╔══════════════════════════╪════════════════════════════════════════════════════════╗
║ ◄──── TRUST BOUNDARY 4: Capa de Datos ───────────────────────────────────────► ║
║                                                                                   ║
║  ┌──────────────────────────────────────────────────────────────────────────┐    ║
║  │  Subred de Datos — Acceso solo desde subred de aplicación                │    ║
║  │                                                                          │    ║
║  │  ┌───────────────────┐  ┌───────────────────┐  ┌─────────────────────┐  │    ║
║  │  │ PostgreSQL        │  │ PostgreSQL        │  │ Redis               │  │    ║
║  │  │ Primaria (T16)    │  │ Réplica (T17)     │  │ Caché+Sesiones(T18) │  │    ║
║  │  │ · AES-256 reposo  │  │ · Sync replicación│  │ · Tokens JWT activos│  │    ║
║  │  │ · mínimo privil.  │  │ · Failover auto   │  │ · Rate limit state  │  │    ║
║  │  └───────────────────┘  └───────────────────┘  └─────────────────────┘  │    ║
║  │                                                                          │    ║
║  │  ┌───────────────────────────────────────────────────────────────────┐   │    ║
║  │  │ Almacenamiento S3-compatible (T19) — Contratos, KYC, comprobantes│   │    ║
║  │  └───────────────────────────────────────────────────────────────────┘   │    ║
║  └──────────────────────────────────────────────────────────────────────────┘    ║
╚══════════════════════════════════════════════════════════════════════════════════╝

╔══════════════════════════════════════════════════════════════════════════════════╗
║ ◄──── TRUST BOUNDARY 5: Integraciones Externas (Terceros) ───────────────────► ║
║                                                                                  ║
║  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐  ┌──────────────┐  ║
║  │ Pasarela Pagos  │  │ Red Interbancaria│  │ Bureau       │  │ KYC          │  ║
║  │ PCI-DSS (T20)   │  │ SIIF/LINK(T21)│  │ Crédito(T22) │  │  DNIC(T25) │  ║
║  │ · Solo tokens   │  │ · mTLS mutual   │  │ · Scoring    │  │ · Biometría  │  ║
║  │ · HMAC-SHA256   │  │ · SLA crítico   │  │ externo      │  │ · DNI valid. │  ║
║  └─────────────────┘  └─────────────────┘  └──────────────┘  └──────────────┘  ║
╚══════════════════════════════════════════════════════════════════════════════════╝

╔══════════════════════════════════════════════════════════════════════════════════╗
║  INFRAESTRUCTURA TRANSVERSAL (acceso controlado desde todas las zonas)           ║
║                                                                                  ║
║  ┌──────────────────┐  ┌─────────────────┐  ┌──────────┐  ┌────────────────┐   ║
║  │ Vault / Secrets  │  │ CI/CD Pipeline  │  │   SIEM   │  │ Backup / DR    │   ║
║  │ Manager (T29)    │  │ (T30)           │  │  (T31)   │  │ (T32)          │   ║
║  │ · Claves y       │  │ · SAST / DAST   │  │ · Logs   │  │ · Snapshots    │   ║
║  │   credenciales   │  │ · Deploy seguro │  │ · Alertas│  │ · RTO < 4hs    │   ║
║  └──────────────────┘  └─────────────────┘  └──────────┘  └────────────────┘   ║
╚══════════════════════════════════════════════════════════════════════════════════╝
```

---

## 3.3 Flujos de Datos por Funcionalidad

### Flujo 1 — Autenticación de Usuario

```
📱 App Móvil
    │
    ├─[1]─ Ingresa usuario + contraseña + biometría local
    │       (biometría NO sale del dispositivo)
    │
    ├─[2]─ HTTPS POST /auth/login ──► WAF ──► API Gateway
    │                                          │
    │                                ┌─────────▼──────────┐
    │                                │   Auth Service     │
    │                                │   · Valida hash    │
    │                                │   · Solicita MFA   │
    │                                └─────────┬──────────┘
    │                                          │
    ├─[3]─ Ingresa TOTP / código OTP ──────────┤
    │                                          │
    │                                ┌─────────▼──────────┐
    │                                │   Auth Service     │
    │                                │   · Valida OTP     │
    │                                │   · Emite JWT      │
    │                                │     access (15min) │
    │                                │   + refresh (7d)   │
    │                                └─────────┬──────────┘
    │                                          │
    └─[4]─ Recibe JWT ◄────────────────────────┘
            (almacenado en Keychain/Keystore)

Datos en tránsito: HTTPS TLS 1.3
Datos en reposo (cliente): Keychain iOS / Keystore Android
Datos en reposo (servidor): PostgreSQL cifrado + Redis (tokens)
```

### Flujo 2 — Transferencia entre Cuentas

```
📱 App Móvil
    │
    ├─[1]─ Usuario ingresa: CBU destino + monto + concepto
    │
    ├─[2]─ HTTPS POST /transfers  {JWT en header}
    │       ──► WAF ──► API Gateway (valida JWT)
    │                       │
    │              ┌────────▼─────────────────────────┐
    │              │  Transfer Service (T10)           │
    │              │  [a] Valida formato CBU destino   │
    │              │  [b] SELECT FOR UPDATE saldo      │  ← Bloqueo pesimista
    │              │      (previene race condition)    │    en PostgreSQL
    │              │  [c] Verifica fondos suficientes  │
    │              │  [d] Verifica límites diarios     │
    │              │  [e] Consulta Fraud Engine (T15)  │
    │              │  [f] Ejecuta débito + crédito     │  ← Transacción ACID
    │              │  [g] Genera comprobante + ID único│
    │              └────────┬─────────────────────────┘
    │                       │
    │              ┌────────▼──────────────────────┐
    │              │  Red Interbancaria (T21)       │  ← Si es otro banco
    │              │  SIIF / LINK (si aplica)     │
    │              └────────┬──────────────────────┘
    │                       │
    │              ┌────────▼──────────────────────┐
    │              │  Notification Service (T14)    │
    │              │  Push al destinatario          │
    │              └───────────────────────────────┘
    │
    └─[3]─ Recibe comprobante con ID de transacción

Activos involucrados: A08, A09, A05, A06, T10, T16, T21
Trust Boundaries cruzados: TB1 → TB2 → TB3 → TB4 → TB5
```

### Flujo 3 — Solicitud de Préstamo

```
📱 App Móvil
    │
    ├─[1]─ Usuario completa formulario de solicitud
    │       (monto, plazo, destino del préstamo)
    │
    ├─[2]─ HTTPS POST /loans/apply  {JWT}
    │       ──► API Gateway ──► Loan Service (T11)
    │                               │
    │                    ┌──────────▼──────────────────────┐
    │                    │  Loan Service                    │
    │                    │  [a] Consulta historial interno  │ ← PostgreSQL
    │                    │  [b] Consulta bureau crédito     │ ← T22 (EQUIFAX Uruguay / Clearing)
    │                    │  [c] Calcula scoring propio      │
    │                    │  [d] Calcula capacidad de pago   │
    │                    │  [e] Aprueba / rechaza / escala  │
    │                    │  [f] Genera contrato PDF         │ ← S3 (T19)
    │                    │  [g] Firma digital del contrato  │
    │                    └──────────┬──────────────────────┘
    │                               │
    │                    ┌──────────▼──────────────────────┐
    │                    │  Notification Service (T14)      │
    │                    │  Email + Push con resultado      │
    │                    └─────────────────────────────────┘
    │
    └─[3]─ Recibe respuesta + enlace al contrato

Activos involucrados: A12, A13, A14, A15, A16, T11, T22, T19
```

### Flujo 4 — Pago con Tarjeta (PCI-DSS)

```
📱 App Móvil
    │
    ├─[1]─ Usuario ingresa datos de tarjeta
    │       → SDK PCI-DSS (T04) tokeniza en el dispositivo
    │         El PAN NUNCA llega al backend de la fintech
    │
    ├─[2]─ HTTPS POST /payments  {JWT + token_tarjeta}
    │       ──► API Gateway ──► Payment Service (T13)
    │                               │
    │                    ┌──────────▼──────────────────────┐
    │                    │  Payment Service                 │
    │                    │  [a] Valida token de tarjeta     │
    │                    │  [b] Reenvía token a pasarela    │
    │                    └──────────┬──────────────────────┘
    │                               │
    │                    ┌──────────▼──────────────────────┐
    │                    │  Pasarela de Pagos PCI-DSS (T20) │
    │                    │  · Procesa con el token          │
    │                    │  · Responde: aprobado/rechazado  │
    │                    │  · Envía webhook firmado (HMAC)  │
    │                    └──────────┬──────────────────────┘
    │                               │ Webhook con HMAC-SHA256
    │                    ┌──────────▼──────────────────────┐
    │                    │  Payment Service                 │
    │                    │  [c] Valida firma HMAC del wbhk  │ ← CRÍTICO
    │                    │  [d] Actualiza estado pago       │
    │                    │  [e] Emite comprobante           │
    │                    └─────────────────────────────────┘
    │
    └─[3]─ Recibe confirmación del pago

Activos involucrados: A07, A22, T04, T13, T20
TB cruzado: TB4 (cliente) → TB1 → TB2 → TB3 → TB5
```

### Flujo 5 — Inversión en Fondos

```
📱 App Móvil
    │
    ├─[1]─ Usuario elige fondo + monto a suscribir
    │
    ├─[2]─ HTTPS POST /investments/subscribe  {JWT}
    │       ──► API Gateway ──► Investment Service (T12)
    │                               │
    │                    ┌──────────▼──────────────────────┐
    │                    │  Investment Service              │
    │                    │  [a] Verifica fondos disponibles │ ← PostgreSQL
    │                    │  [b] Verifica perfil inversor    │ ← A20 (compliance)
    │                    │  [c] Envía orden a administradora│
    │                    └──────────┬──────────────────────┘
    │                               │
    │                    ┌──────────▼──────────────────────┐
    │                    │  Administradora de Fondos (T23)  │
    │                    │  · Acredita cuotapartes          │
    │                    │  · Confirma operación            │
    │                    └──────────┬──────────────────────┘
    │                               │
    │                    ┌──────────▼──────────────────────┐
    │                    │  Investment Service              │
    │                    │  [d] Actualiza portfolio (T17)   │
    │                    │  [e] Notifica resultado          │
    │                    └─────────────────────────────────┘
    │
    └─[3]─ Recibe confirmación de suscripción

Activos involucrados: A17, A18, A20, T12, T23
```

---

## 3.4 Actores del Sistema

| Actor                         | Descripción                                        | Nivel de Confianza  | Privilegios                          | Activos que accede          |
| ----------------------------- | -------------------------------------------------- | ------------------- | ------------------------------------ | --------------------------- |
| **Usuario final autenticado** | Cliente con JWT válido y MFA completado            | Bajo — verificado   | Solo sus propios recursos            | A01–A23, T01/T02            |
| **Administrador de sistema**  | Operador interno con acceso privilegiado a consola | Medio — auditado    | Configuración, gestión de incidentes | Mayoría de activos técnicos |
| **DevOps / SRE**              | Acceso a infraestructura y pipelines               | Medio — restringido | Deploy, monitoreo, infraestructura   | T26–T33                     |
| **Equipo de riesgo / fraude** | Analistas que revisan transacciones sospechosas    | Medio — acotado     | Datos de transacciones y alertas     | A06, A08, A24, A25          |
| **Pasarela de pagos**         | Servicio externo PCI-DSS                           | Medio — contractual | Solo datos de pago tokenizados       | A07 (token)                 |
| **Administradora de fondos**  | Servicio externo regulado                          | Medio — contractual | Órdenes de suscripción/rescate       | A17, A18                    |
| **Bureau de crédito**         | Servicio externo (EQUIFAX Uruguay / Clearing)      | Bajo — contractual  | Solo consultas de scoring            | A13                         |
| **Atacante externo**          | Actor malicioso sin acceso legítimo                | ❌ Ninguno          | Evaluado en STRIDE                   | Intenta acceder a todo      |
| **Insider malicioso**         | Empleado con acceso legítimo que actúa con malicia | ⚠️ Variable         | Según rol + accesos auditados        | Según área                  |

---

## 3.5 Trust Boundaries — Definición Formal

| ID       | Nombre                 | Ubicación en la Arquitectura                  | Controles en el límite                                                                                     | Activos que protege        |
| -------- | ---------------------- | --------------------------------------------- | ---------------------------------------------------------------------------------------------------------- | -------------------------- |
| **TB-1** | Perímetro externo      | Entre Internet y CDN/WAF/LB                   | TLS 1.3, WAF OWASP, DDoS protection, rate limiting por IP, geo-blocking                                    | Todos                      |
| **TB-2** | API Gateway            | Entre perímetro y microservicios              | Validación JWT (RS256), rate limiting por usuario/endpoint, rechazo alg:none, logging completo             | Todos los datos de negocio |
| **TB-3** | VPC privada            | Entre API Gateway y microservicios            | mTLS entre servicios, Network Policies, no exposición directa a internet, service mesh                     | A01–A30, T08–T15           |
| **TB-4** | Capa de datos          | Entre microservicios y bases de datos         | Credenciales desde Vault, TLS en conexión BD, mínimo privilegio por servicio, solo acceso desde subred app | A03–A30, T16–T19           |
| **TB-5** | Integraciones externas | Entre backend e integraciones de terceros     | mTLS mutual, validación criptográfica de webhooks, IP whitelist, timeout y circuit breaker                          | A07–A09, A12–A23           |
| **TB-6** | Dispositivo móvil      | Entre el dispositivo del usuario y el backend | Certificate pinning, biometría local, Keychain/Keystore, root detection, código ofuscado                   | A03, A04, A07              |

---

## 3.6 Puntos de Entrada (Attack Surface)

| #     | Punto de Entrada                           | Protocolo        | Autenticación        | Trust Boundary    | Riesgo Base |
| ----- | ------------------------------------------ | ---------------- | -------------------- | ----------------- | ----------- |
| PE-01 | App → API Gateway (`/auth/login`)          | HTTPS REST       | Ninguna (pre-auth)   | TB-1 → TB-2       | 🔴 Alto     |
| PE-02 | App → API Gateway (endpoints autenticados) | HTTPS REST + JWT | JWT RS256 + MFA      | TB-1 → TB-2       | 🟠 Medio    |
| PE-03 | Webhook pasarela de pagos → backend        | HTTPS + HMAC     | Firma HMAC-SHA256    | TB-1 → TB-5       | 🟠 Medio    |
| PE-04 | Backend → red interbancaria                | HTTPS + mTLS     | Certificado cliente  | TB-5              | 🟠 Medio    |
| PE-05 | Backend → bureau de crédito                | HTTPS REST       | API Key + TLS        | TB-5              | 🟡 Bajo     |
| PE-06 | Notificaciones push → dispositivo          | FCM / APNs       | Token de dispositivo | TB-6              | 🟡 Bajo     |
| PE-07 | CI/CD → entorno de producción              | SSH / API cloud  | Clave SSH + token    | Infra transversal | 🔴 Alto     |
| PE-08 | Admin panel                                | HTTPS            | MFA + IP whitelist   | TB-2              | 🔴 Alto     |
| PE-09 | Acceso directo a BD (riesgo insider)       | TCP PostgreSQL   | Usuario BD + TLS     | TB-4              | 🔴 Alto     |

---

## 3.7 Supuestos de Arquitectura

1. **mTLS interno:** Todos los microservicios se comunican entre sí con certificados mutuos dentro de la VPC, gestionados por un service mesh (Istio / Linkerd).
2. **Sin acceso directo a BD:** Ningún microservicio tiene usuario de BD con privilegios de superadmin; cada servicio tiene credenciales propias con acceso mínimo a sus tablas.
3. **Vault como fuente de verdad:** Ningún secreto reside en variables de entorno crudas en producción ni en repositorios Git.
4. **JWT de corta duración:** Access tokens con expiración de 15 minutos; refresh tokens de 7 días almacenados en Redis con blacklisting.
5. **Certificate pinning activo:** La app rechaza certificados no esperados para prevenir ataques MitM.
6. **Segregación de redes:** Las subredes de aplicación y datos están en subredes separadas con NACLs; la subred de datos no tiene ruta a Internet.

---
