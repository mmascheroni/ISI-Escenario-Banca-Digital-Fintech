# 06 — Plan de Mitigación y Matriz de Controles de Seguridad

```
╔══════════════════════════════════════════════════════════════════╗
║        ANÁLISIS DE RIESGOS — FINTECH BANKING DIGITAL            ║
║        Sección 7: Plan de Mitigación                            ║
╚══════════════════════════════════════════════════════════════════╝
```

**Proyecto:** Fintech — Aplicación Móvil de Banca Digital  
**Documento:** `docs/06-plan-mitigacion.md`  
**Versión:** 1.0  
**Fecha:** Mayo 2026  
**Base:** 38 amenazas STRIDE · Priorización DREAD · NIST SP 800-53 · ISO 27001 · Circular BCU 2.244

---

## 7.1 Estructura del Plan

Cada control se asigna siguiendo la prioridad DREAD. Los controles se clasifican en tres tipos:

| Tipo           | Descripción                                         | Cuándo actúa       |
| -------------- | --------------------------------------------------- | ------------------ |
| **Preventivo** | Evita que la amenaza ocurra                         | Antes del ataque   |
| **Detectivo**  | Identifica que la amenaza ocurrió o está ocurriendo | Durante el ataque  |
| **Correctivo** | Reduce el impacto y restaura el sistema             | Después del ataque |

---

## 7.2 Controles por Prioridad DREAD

### 🔴 CRÍTICO — Remediar en < 24 horas

---

#### C01 · TH37 — Credenciales hardcodeadas en repositorio Git (Score: 9.8)

| Campo               | Detalle                                                                               |
| ------------------- | ------------------------------------------------------------------------------------- |
| **Amenaza**         | Un developer comete credenciales en el repositorio. Bots las detectan en < 5 minutos. |
| **Tipo de control** | Preventivo + Detectivo                                                                |
| **Referencia NIST** | SC-12 (Gestión de claves), CM-7 (Funcionalidad mínima)                                |
| **Referencia ISO**  | A.10.1.2, A.12.6.2                                                                    |
| **Referencia BCU**  | Circular 2.244 Art. 8                                                                 |

**Controles a implementar:**

- Instalar `gitleaks` o `detect-secrets` como git hook pre-commit — bloquea el commit si detecta patrones de credenciales
- Escaneo retroactivo del historial completo del repositorio con `trufflehog`
- Todas las credenciales almacenadas exclusivamente en HashiCorp Vault — referenciadas por nombre, nunca en texto plano
- Política de rotación inmediata ante cualquier credencial expuesta
- `.gitignore` auditado para excluir `.env`, `*.pem`, `*.key`, `*.p12`
- Escaneo automático en cada PR como paso obligatorio del CI/CD

**Estado:** ⏳ Pendiente

---

### 🟠 ALTO — Remediar en < 7 días

---

#### C02 · TH08 — Bypass JWT con `alg:none` (Score: 8.8)

| Campo               | Detalle                                                                 |
| ------------------- | ----------------------------------------------------------------------- |
| **Amenaza**         | Atacante forja un JWT sin firma para suplantar a cualquier usuario      |
| **Tipo de control** | Preventivo                                                              |
| **Referencia NIST** | IA-2 (Identificación y autenticación), IA-5 (Gestión de autenticadores) |
| **Referencia ISO**  | A.9.4.2, A.9.4.3                                                        |
| **Referencia BCU**  | Circular 2.244 Art. 8 — Controles mínimos de autenticación              |

**Controles a implementar:**

- Configurar la librería JWT para aceptar **exclusivamente** `RS256` o `ES256` — rechazar cualquier otro algoritmo
- Agregar validación explícita que rechace `alg:none` con HTTP 401
- Test unitario obligatorio en CI que valide el rechazo de tokens con `alg:none`
- Rotación periódica del par de claves RSA (clave privada en Vault)
- Auditoría de todas las librerías JWT utilizadas en el proyecto

**Estado:** ⏳ Pendiente

---

#### C03 · TH09 — Credential stuffing sin rate limiting (Score: 8.4)

| Campo               | Detalle                                                                        |
| ------------------- | ------------------------------------------------------------------------------ |
| **Amenaza**         | Atacante prueba millones de credenciales filtradas contra el endpoint de login |
| **Tipo de control** | Preventivo + Detectivo                                                         |
| **Referencia NIST** | AC-7 (Bloqueo por intentos fallidos), SI-10 (Validación de entradas)           |
| **Referencia ISO**  | A.9.4.2                                                                        |
| **Referencia BCU**  | Circular 2.244 Art. 8 + Ley 18.331 (notificación ante compromiso masivo)       |

**Controles a implementar:**

- Rate limiting en `/auth/login`: máximo 5 intentos por IP cada 60 segundos
- Bloqueo temporal progresivo: 5 fallos → 15 min bloqueado; 10 fallos → 24 hs bloqueado
- MFA obligatorio para todos los usuarios (TOTP / biometría)
- Alertas SIEM ante más de 50 intentos fallidos por IP en 5 minutos
- Integración con bases de datos de credenciales comprometidas (Have I Been Pwned API)
- CAPTCHA en el formulario de login tras 3 intentos fallidos

**Estado:** ⏳ Pendiente

---

#### C04 · TH10 — IDOR: acceso a recursos de otro usuario (Score: 7.8)

| Campo               | Detalle                                                                                     |
| ------------------- | ------------------------------------------------------------------------------------------- |
| **Amenaza**         | Atacante manipula IDs en la URL para acceder a cuentas, préstamos o inversiones de terceros |
| **Tipo de control** | Preventivo                                                                                  |
| **Referencia NIST** | AC-3 (Cumplimiento de control de acceso), AC-2 (Gestión de cuentas)                         |
| **Referencia ISO**  | A.9.4.1                                                                                     |

**Controles a implementar:**

- Validar en cada endpoint que el `userId` del JWT coincide con el recurso solicitado
- Nunca exponer IDs secuenciales — usar UUIDs aleatorios no predecibles
- Tests automáticos de autorización (OWASP ZAP, Burp Suite) en el pipeline CI/CD
- Code review obligatorio enfocado en validación de ownership antes de cada release
- Logs de auditoría para cada acceso a recursos — detectar patrones de enumeración

**Estado:** ⏳ Pendiente

---

#### C05 · TH12 — Rate limiting ausente en endpoints costosos (Score: 7.0)

| Campo               | Detalle                                                                                  |
| ------------------- | ---------------------------------------------------------------------------------------- |
| **Amenaza**         | Abuso de endpoints de scoring, transferencias o préstamos que consumen recursos críticos |
| **Tipo de control** | Preventivo                                                                               |
| **Referencia NIST** | SC-5 (Protección contra DoS)                                                             |
| **Referencia ISO**  | A.13.1.1                                                                                 |

**Controles a implementar:**

- Rate limiting por usuario y por endpoint en el API Gateway:
    - `/transfers`: máx 10 requests/minuto por usuario
    - `/loans/apply`: máx 3 requests/hora por usuario
    - `/investments/subscribe`: máx 20 requests/hora por usuario
- Rate limiting global por IP para todos los endpoints
- Circuit breaker ante picos de tráfico anómalos
- Alertas automáticas cuando un usuario supera el 80% del límite

**Estado:** ⏳ Pendiente

---

#### C06 · TH13 — Bypass de MFA por manipulación del callback (Score: 7.0)

| Campo               | Detalle                                                                                     |
| ------------------- | ------------------------------------------------------------------------------------------- |
| **Amenaza**         | Atacante manipula la respuesta del callback de autenticación para saltear el segundo factor |
| **Tipo de control** | Preventivo                                                                                  |
| **Referencia NIST** | IA-2 (MFA), IA-5                                                                            |
| **Referencia ISO**  | A.9.4.2                                                                                     |
| **Referencia BCU**  | Circular 2.244 Art. 8                                                                       |

**Controles a implementar:**

- Validar el resultado del MFA exclusivamente del lado servidor — nunca confiar en la respuesta del cliente
- El JWT solo se emite tras completar exitosamente ambos factores en el servidor
- Implementar PKCE (Proof Key for Code Exchange) en el flujo OAuth 2.0
- Logging de todos los intentos de autenticación con resultado y timestamp
- Alertas ante intentos repetidos de acceder al callback sin MFA completado

**Estado:** ⏳ Pendiente

---

#### C07 · TH14 — Fuerza bruta de OTP sin bloqueo (Score: 7.6)

| Campo               | Detalle                                                                            |
| ------------------- | ---------------------------------------------------------------------------------- |
| **Amenaza**         | Atacante prueba las 1.000.000 combinaciones posibles de un código OTP de 6 dígitos |
| **Tipo de control** | Preventivo                                                                         |
| **Referencia NIST** | AC-7, IA-5                                                                         |
| **Referencia ISO**  | A.9.4.2                                                                            |

**Controles a implementar:**

- Bloqueo tras 3 intentos fallidos de OTP — invalidar el código y requerir uno nuevo
- Tiempo de vida del OTP máximo 5 minutos (TOTP estándar: 30 segundos)
- Preferir TOTP (Google Authenticator / Authy) sobre SMS-OTP para evitar SIM swapping
- Rate limiting específico en el endpoint de validación de OTP: máx 3 intentos por sesión
- Alertas ante múltiples intentos fallidos de OTP para el mismo usuario

**Estado:** ⏳ Pendiente

---

#### C08 · TH16 — Race condition en transferencias / double spending (Score: 7.4)

| Campo               | Detalle                                                          |
| ------------------- | ---------------------------------------------------------------- |
| **Amenaza**         | Múltiples requests simultáneos debitan más dinero del disponible |
| **Tipo de control** | Preventivo                                                       |
| **Referencia NIST** | SC-28 (Protección de información en reposo), AU-12               |
| **Referencia ISO**  | A.14.2.5                                                         |
| **Referencia BCU**  | Ley 19.574 — operaciones sospechosas                             |

**Controles a implementar:**

- `SELECT FOR UPDATE` en PostgreSQL al leer el saldo antes de cualquier débito
- Transacciones ACID con nivel de aislamiento `SERIALIZABLE`
- Idempotency key única por solicitud de transferencia — deduplicación en Redis con TTL 24hs
- Rate limiting en `/transfers`: máx 3 transferencias simultáneas por usuario
- Tests de carga con concurrencia alta (JMeter / k6) antes de cada release en producción

**Estado:** ⏳ Pendiente

---

#### C09 · TH06 — DDoS volumétrico contra CDN/WAF (Score: 7.4)

| Campo               | Detalle                                                              |
| ------------------- | -------------------------------------------------------------------- |
| **Amenaza**         | Ataque de volumen que satura el perímetro dejando la app inaccesible |
| **Tipo de control** | Preventivo + Correctivo                                              |
| **Referencia NIST** | SC-5 (Protección contra DoS)                                         |
| **Referencia ISO**  | A.13.1.1                                                             |
| **Referencia BCU**  | Circular 2.244 Art. 15 — Continuidad operacional                     |

**Controles a implementar:**

- Activar protección DDoS avanzada en CloudFront/Cloudflare (capas 3, 4 y 7)
- Geo-blocking configurable para países con alto riesgo de ataque
- Rate limiting global por IP en el WAF: máx 1.000 requests/minuto
- Plan de contingencia documentado con RTO < 4 horas (exigido por BCU)
- Pruebas de resiliencia ante DDoS al menos 1 vez al año
- Alertas automáticas al SIEM cuando el tráfico supere el triple del promedio diario

**Estado:** ⏳ Pendiente

---

#### C10 · TH24 — Webhook de pasarela de pagos sin validación HMAC (Score: 7.6)

| Campo               | Detalle                                                           |
| ------------------- | ----------------------------------------------------------------- |
| **Amenaza**         | Atacante envía webhooks falsos para acreditar pagos no realizados |
| **Tipo de control** | Preventivo                                                        |
| **Referencia NIST** | SC-8 (Confidencialidad en tránsito), AU-2                         |
| **Referencia ISO**  | A.13.2.3                                                          |
| **Referencia BCU**  | PCI-DSS Req. 6.4 + Ley 19.574                                     |

**Controles a implementar:**

- Validar la firma HMAC-SHA256 del header `X-Signature` en cada webhook recibido
- Rechazar con HTTP 401 cualquier webhook sin firma o con firma inválida
- IP whitelist de los servidores de la pasarela de pagos — rechazar cualquier otra fuente
- Re-consultar el estado del pago directamente a la API de la pasarela antes de acreditar fondos
- Registro inmutable de todos los webhooks recibidos (válidos e inválidos) con timestamp
- Implementar nonce único por webhook para prevenir replay attacks

**Estado:** ⏳ Pendiente

---

#### C11 · TH28 — SQL Injection en consultas de la API (Score: 8.0)

| Campo               | Detalle                                                         |
| ------------------- | --------------------------------------------------------------- |
| **Amenaza**         | Inyección SQL que expone o destruye datos de todos los usuarios |
| **Tipo de control** | Preventivo + Detectivo                                          |
| **Referencia NIST** | SI-10 (Validación de entradas), SC-28                           |
| **Referencia ISO**  | A.14.2.5, A.12.2.1                                              |
| **Referencia BCU**  | Circular 2.244 + Ley 18.331 + PCI-DSS Req. 6.3                  |

**Controles a implementar:**

- Usar **exclusivamente** prepared statements o parameterized queries en toda la capa de acceso a datos
- ORM con escape automático (SQLAlchemy, Hibernate, Sequelize) — prohibir SQL dinámico concatenado
- SAST obligatorio en CI/CD con reglas específicas de SQLi (Semgrep, SonarQube)
- WAF con reglas de detección de SQLi como segunda capa de defensa
- Principio de mínimo privilegio en BD: cada microservicio tiene usuario propio con acceso solo a sus tablas
- Auditoría periódica de todos los queries de la aplicación

**Estado:** ⏳ Pendiente

---

#### C12 · TH31 — Session hijacking masivo desde Redis (Score: 7.2)

| Campo               | Detalle                                                                   |
| ------------------- | ------------------------------------------------------------------------- |
| **Amenaza**         | Acceso a Redis sin autenticación permite robar todas las sesiones activas |
| **Tipo de control** | Preventivo + Detectivo                                                    |
| **Referencia NIST** | IA-5, SC-28, AC-17                                                        |
| **Referencia ISO**  | A.9.4.3, A.10.1.1                                                         |

**Controles a implementar:**

- Autenticación obligatoria en Redis: `requirepass` fuerte + ACLs por microservicio
- Acceso a Redis exclusivamente desde la subred privada de aplicación (Network Policy en K8s)
- Cifrado TLS en la conexión entre microservicios y Redis
- Credenciales de Redis gestionadas y rotadas automáticamente por Vault
- Alertas SIEM ante cualquier conexión a Redis desde IPs fuera de la subred esperada
- TTL agresivo en tokens: access 15 min, refresh 7 días con blacklist activo

**Estado:** ⏳ Pendiente

---

#### C13 · TH33 — Compromiso del gestor de secretos Vault (Score: 7.6)

| Campo               | Detalle                                                          |
| ------------------- | ---------------------------------------------------------------- |
| **Amenaza**         | Si Vault es comprometido, toda la infraestructura queda expuesta |
| **Tipo de control** | Preventivo + Detectivo                                           |
| **Referencia NIST** | SC-12, SC-28, AC-2                                               |
| **Referencia ISO**  | A.10.1.2                                                         |

**Controles a implementar:**

- MFA obligatorio para cualquier acceso humano a Vault
- Políticas de mínimo privilegio en Vault: cada servicio accede solo a sus propios secretos
- Seal/unseal automático con AWS KMS — sin clave maestra manual
- Auditoría de todos los accesos a Vault con alertas en SIEM
- Rotación automática de todos los secretos almacenados en Vault
- Snapshots periódicos de Vault cifrados y almacenados en ubicación separada

**Estado:** ⏳ Pendiente

---

#### C14 · TH34 — Replay attack en webhook de pago (Score: 7.2)

| Campo               | Detalle                                                                  |
| ------------------- | ------------------------------------------------------------------------ |
| **Amenaza**         | Reenvío de un webhook ya procesado para acreditar fondos por segunda vez |
| **Tipo de control** | Preventivo                                                               |
| **Referencia NIST** | SC-8, AU-2                                                               |
| **Referencia ISO**  | A.13.2.3                                                                 |

**Controles a implementar:**

- Implementar nonce único en cada webhook — almacenar en Redis con TTL 24hs para deduplicación
- Verificar timestamp del webhook: rechazar si tiene más de 5 minutos de antigüedad
- Idempotency check por `payment_id`: si ya fue procesado, responder 200 sin re-acreditar
- Registro de todos los `payment_id` procesados en la BD para consulta rápida

**Estado:** ⏳ Pendiente

---

#### C15 · TH36 — Inyección de código en pipeline CI/CD (Score: 7.2)

| Campo               | Detalle                                                                              |
| ------------------- | ------------------------------------------------------------------------------------ |
| **Amenaza**         | Código malicioso inyectado en el pipeline se despliega automáticamente en producción |
| **Tipo de control** | Preventivo + Detectivo                                                               |
| **Referencia NIST** | SA-11 (Pruebas de seguridad), SR-4 (Verificación de integridad), CM-7                |
| **Referencia ISO**  | A.14.2.8, A.15.1.1                                                                   |

**Controles a implementar:**

- Approval manual obligatorio de mínimo 2 personas para deploy a producción
- SAST (Semgrep, SonarQube) y análisis de dependencias (Snyk, OWASP Dependency Check) en cada PR
- Firma de artefactos de build con checksum SHA-256 verificado antes del deploy
- Principio de mínimo privilegio en runners de GitHub Actions
- Lockfile de dependencias commiteado (`package-lock.json`, `poetry.lock`) y auditado
- Escaneo de vulnerabilidades en imágenes Docker antes del push al registry (Trivy)

**Estado:** ⏳ Pendiente

---

### 🟡 MEDIO — Remediar en < 30 días

---

#### C16 · TH11 — Shadow APIs expuestas (Score: 6.6)

**Controles:** Inventario completo de todos los endpoints; API Gateway como único punto de entrada; eliminar o deshabilitar endpoints no documentados; revisión periódica con OWASP ZAP.
**NIST:** CM-7 | **ISO:** A.12.6.2 | **Estado:** ⏳ Pendiente

---

#### C17 · TH07 — Bypass del WAF con payloads codificados (Score: 6.6)

**Controles:** Actualizar reglas WAF para detectar doble encoding y fragmentación HTTP; activar modo de inspección profunda de paquetes; pruebas de penetración trimestrales contra el WAF.
**NIST:** SC-5, SI-3 | **ISO:** A.12.2.1 | **Estado:** ⏳ Pendiente

---

#### C18 · TH29 — Acceso directo a PostgreSQL bypasseando negocio (Score: 6.6)

**Controles:** Network Policy en K8s que restringe acceso a PostgreSQL solo desde pods autorizados; usuarios de BD con permisos mínimos por microservicio; alertas SIEM ante conexiones directas a la BD desde IPs no esperadas.
**NIST:** AC-3, SC-28 | **ISO:** A.9.4.1 | **Estado:** ⏳ Pendiente

---

#### C19 · TH30 — Eliminación maliciosa de tablas críticas (Score: 6.6)

**Controles:** Snapshots automáticos diarios de PostgreSQL con retención de 30 días; privilegios de DROP TABLE eliminados de todos los usuarios de aplicación; alertas ante DDL ejecutado en producción; pruebas de restauración mensuales.
**NIST:** CP-9 (Backup), CP-10 (Recuperación) | **ISO:** A.12.3.1 | **BCU:** RTO < 4hs exigido | **Estado:** ⏳ Pendiente

---

#### C20 · TH32 — Manipulación del rate limiting en Redis (Score: 6.6)

**Controles:** ACLs en Redis que impidan escritura sobre claves de rate limiting desde cualquier servicio que no sea el API Gateway; validación de integridad de contadores de rate limiting.
**NIST:** SC-5, AC-3 | **ISO:** A.9.4.1 | **Estado:** ⏳ Pendiente

---

#### C21 · TH35 — Manipulación de transferencia interbancaria en tránsito (Score: 6.6)

**Controles:** mTLS con certificados mutuos en la integración con COELSA/LINK; validación de firma de cada instrucción de transferencia; re-confirmación del CBU destino antes de ejecutar.
**NIST:** SC-8, SC-12 | **ISO:** A.13.2.3 | **Estado:** ⏳ Pendiente

---

#### C22 · TH19 — Manipulación del score crediticio en tránsito (Score: 6.4)

**Controles:** mTLS en la comunicación con el bureau de crédito (Veraz/Nosis); validar checksum de la respuesta antes de usar el score; logging inmutable de cada consulta de scoring.
**NIST:** SC-8, AU-2 | **ISO:** A.13.2.3 | **Estado:** ⏳ Pendiente

---

#### C23 · TH22 — Manipulación de órdenes de inversión en tránsito (Score: 6.4)

**Controles:** mTLS en la comunicación con administradoras de fondos; firma digital de cada orden de suscripción/rescate; re-confirmación del usuario antes de ejecutar órdenes superiores a umbral definido.
**NIST:** SC-8, IA-5 | **ISO:** A.13.2.3 | **Estado:** ⏳ Pendiente

---

#### C24 · TH01 — Robo de JWT en dispositivo móvil (Score: 6.2)

**Controles:** Almacenar tokens exclusivamente en Keychain (iOS) y EncryptedSharedPreferences + Keystore (Android); access token con TTL de 15 minutos; invalidación del token ante detección de dispositivo nuevo o cambio de IP.
**NIST:** SC-28, IA-5 | **ISO:** A.10.1.1 | **Estado:** ⏳ Pendiente

---

#### C25 · TH26 — Notificaciones push con datos financieros sensibles (Score: 6.2)

**Controles:** Eliminar montos, saldos y CBU del payload de las notificaciones push; usar solo texto genérico como "Tenés un nuevo movimiento en tu cuenta"; los datos reales se obtienen al abrir la app autenticada.
**NIST:** SC-28 | **ISO:** A.13.2.3 | **Estado:** ⏳ Pendiente

---

#### C26 · TH23 — IDOR en portfolio de inversión (Score: 6.0)

**Controles:** Validar que el `userId` del JWT coincide con el portfolio solicitado en cada endpoint del Investment Service; tests automatizados de autorización cruzada entre usuarios.
**NIST:** AC-3 | **ISO:** A.9.4.1 | **Estado:** ⏳ Pendiente

---

#### C27 · TH15 — JWT con expiración excesiva o sin blacklist (Score: 6.0)

**Controles:** Access token TTL máximo 15 minutos; refresh token TTL 7 días con blacklist en Redis; invalidación inmediata del refresh token ante cierre de sesión o cambio de contraseña; endpoint `/auth/logout` que invalida el token en Redis.
**NIST:** AC-12 (Terminación de sesión), IA-5 | **ISO:** A.9.4.2 | **Estado:** ⏳ Pendiente

---

#### C28 · TH17 — Manipulación de límites de transferencia (Score: 5.8)

**Controles:** Los límites de transferencia se leen exclusivamente desde la BD — nunca desde el cliente; validación del límite en el Transfer Service antes de ejecutar; alertas ante modificaciones de límites fuera del horario laboral.
**NIST:** AC-3, AU-2 | **ISO:** A.9.4.1 | **Estado:** ⏳ Pendiente

---

#### C29 · TH02 — Manipulación de parámetros via MitM (Score: 5.8)

**Controles:** Certificate pinning en ambas apps (iOS y Android); TLS 1.3 como versión mínima; validar todos los parámetros críticos (monto, CBU) del lado servidor — nunca confiar en el cliente.
**NIST:** SC-8, SC-12 | **ISO:** A.13.2.3 | **Estado:** ⏳ Pendiente

---

#### C30 · TH04 — Reverse engineering del APK/IPA (Score: 5.8)

**Controles:** Ofuscación del código con ProGuard (Android) y SwiftShield (iOS); detección de root/jailbreak con restricción de funciones; ninguna clave o secreto hardcodeado en el binario.
**NIST:** SA-11, SC-28 | **ISO:** A.14.2.8 | **Estado:** ⏳ Pendiente

---

#### C31 · TH38 — Borrado de logs de auditoría (Score: 5.8)

**Controles:** Logs de auditoría escritos en almacenamiento inmutable (S3 con Object Lock o WORM); los logs se envían al SIEM en tiempo real — una copia siempre fuera del sistema comprometido; alertas ante gaps en la secuencia de logs.
**NIST:** AU-9 (Protección de auditoría), AU-12 | **ISO:** A.12.4.2 | **BCU:** Circular 2.244 Art. 12 | **Estado:** ⏳ Pendiente

---

#### C32 · TH25 — Logging accidental de PAN (Score: 5.4)

**Controles:** Implementar scrubbing automático de logs que detecte y enmascare patrones de PAN (regex 16 dígitos); auditoría inmediata de logs existentes; prohibir el logging de datos de tarjeta en cualquier nivel.
**NIST:** SC-28, AU-2 | **ISO:** A.10.1.1 | **BCU:** PCI-DSS Req. 3.4 | **Estado:** ⏳ Pendiente

---

#### C33 · TH27 — Evasión del motor antifraude (Score: 5.4)

**Controles:** Actualizar umbrales de detección para incluir patrones de smurfing (transacciones pequeñas frecuentes); correlación temporal entre transacciones del mismo usuario; alerta ante N transacciones en ventana de tiempo T; revisión trimestral de reglas del motor.
**NIST:** SI-3, AU-12 | **ISO:** A.12.2.1 | **BCU:** Ley 19.574 (Antilavado) | **Estado:** ⏳ Pendiente

---

#### C34 · TH05 — App en dispositivo rooteado (Score: 5.2)

**Controles:** Detección de root (Android) y jailbreak (iOS) al iniciar la app; mostrar advertencia y deshabilitar funciones críticas (transferencias, pagos) en dispositivos comprometidos.
**NIST:** SC-28, AC-17 | **ISO:** A.6.2.2 | **Estado:** ⏳ Pendiente

---

#### C35 · TH20 — Exposición del historial de morosidad (Score: 5.0)

**Controles:** Principio de mínimo privilegio: solo Riesgo Crediticio accede al historial de morosidad; logs de auditoría de cada acceso; revisión trimestral de permisos de acceso.
**NIST:** AC-2, AC-6 | **ISO:** A.9.2.3 | **Estado:** ⏳ Pendiente

---

#### C36 · TH18 — Ausencia de logging inmutable en transferencias (Score: 5.0)

**Controles:** Logging obligatorio con timestamp, userId, monto, CBU origen y destino para cada transferencia; logs enviados al SIEM en tiempo real; firma digital del log para garantizar no repudio.
**NIST:** AU-2, AU-9 | **ISO:** A.12.4.1 | **BCU:** Circular 2.244 Art. 12 | **Estado:** ⏳ Pendiente

---

#### C37 · TH03 — Datos sensibles en logs del SO (Score: 5.0)

**Controles:** Auditoría del código móvil para eliminar cualquier `print()`, `NSLog()` o `Log.d()` que imprima datos sensibles; implementar log scrubbing antes de escribir en el sistema de logging del SO.
**NIST:** AU-2, SC-28 | **ISO:** A.12.4.1 | **Estado:** ⏳ Pendiente

---

### 🔵 BAJO — Remediar programado

---

#### C38 · TH21 — Contratos de préstamo sin timestamp TSA (Score: 3.8)

**Controles:** Integrar un servicio de sellado de tiempo (TSA — RFC 3161) al generar contratos firmados digitalmente; el timestamp garantiza no repudio temporal ante disputas legales.
**NIST:** AU-9 | **ISO:** A.18.1.3 | **BCU:** Ley 18.600 (Documento Electrónico Uruguay) | **Estado:** ⏳ Pendiente

---

## 7.3 Controles por Categoría

### Controles Preventivos

- C01 (Vault + gitleaks), C02 (JWT RS256), C03 (Rate limiting + MFA), C04 (IDOR), C05 (Rate limiting endpoints), C06 (MFA server-side), C07 (OTP bloqueo), C08 (SELECT FOR UPDATE), C09 (DDoS), C10 (HMAC webhooks), C11 (SQLi), C12 (Redis auth), C13 (Vault MFA), C14 (Replay nonce), C15 (CI/CD approval), C24 (JWT móvil), C25 (Push sin datos), C29 (Certificate pinning), C30 (Ofuscación APK)

### Controles Detectivos

- C01 (Escaneo Git), C03 (Alertas SIEM credential stuffing), C04 (Logs IDOR), C09 (Alertas tráfico), C11 (WAF SQLi), C12 (Alertas Redis), C13 (Auditoría Vault), C31 (Logs inmutables), C33 (Antifraude smurfing)

### Controles Correctivos

- C09 (Plan contingencia DDoS / BCP), C19 (Backup y restauración BD), C36 (Logs no repudio transferencias)

---

## 7.4 Matriz de Controles NIST SP 800-53 / ISO 27001

| Control ID | Familia NIST         | Descripción                                     | Referencia NIST | ISO 27001 | Amenazas que cubre     |
| ---------- | -------------------- | ----------------------------------------------- | --------------- | --------- | ---------------------- |
| AC-2       | Acceso               | Gestión de cuentas y roles                      | NIST AC-2       | A.9.2.1   | TH10, TH20, TH29       |
| AC-3       | Acceso               | Cumplimiento de políticas de acceso             | NIST AC-3       | A.9.4.1   | TH10, TH18, TH29       |
| AC-6       | Acceso               | Mínimo privilegio                               | NIST AC-6       | A.9.2.3   | TH20, TH29, TH33       |
| AC-7       | Acceso               | Bloqueo ante intentos fallidos                  | NIST AC-7       | A.9.4.2   | TH09, TH14             |
| AC-12      | Acceso               | Terminación automática de sesión                | NIST AC-12      | A.9.4.2   | TH15                   |
| AU-2       | Auditoría            | Registro de eventos de seguridad                | NIST AU-2       | A.12.4.1  | TH18, TH25, TH36, TH38 |
| AU-9       | Auditoría            | Protección de registros de auditoría            | NIST AU-9       | A.12.4.2  | TH31, TH38             |
| AU-12      | Auditoría            | Generación de registros                         | NIST AU-12      | A.12.4.1  | TH18, TH33             |
| CP-9       | Contingencia         | Backup del sistema de información               | NIST CP-9       | A.12.3.1  | TH30                   |
| CP-10      | Contingencia         | Recuperación del sistema                        | NIST CP-10      | A.12.3.1  | TH30                   |
| IA-2       | Identidad            | MFA para acceso al sistema                      | NIST IA-2       | A.9.4.2   | TH08, TH09, TH13       |
| IA-5       | Identidad            | Gestión de autenticadores (tokens, contraseñas) | NIST IA-5       | A.9.4.3   | TH01, TH09, TH15       |
| SA-11      | Adquisición          | Pruebas de seguridad en desarrollo (SAST/DAST)  | NIST SA-11      | A.14.2.8  | TH04, TH11, TH15       |
| SC-5       | Comunicaciones       | Protección contra DoS                           | NIST SC-5       | A.13.1.1  | TH06, TH12             |
| SC-8       | Comunicaciones       | Confidencialidad en tránsito (TLS)              | NIST SC-8       | A.13.2.3  | TH02, TH10, TH19       |
| SC-12      | Comunicaciones       | Gestión de claves criptográficas                | NIST SC-12      | A.10.1.2  | TH37, TH33             |
| SC-28      | Comunicaciones       | Protección de información en reposo             | NIST SC-28      | A.10.1.1  | TH01, TH11, TH28       |
| SI-3       | Integridad           | Protección contra código malicioso              | NIST SI-3       | A.12.2.1  | TH36                   |
| SI-10      | Integridad           | Validación de entradas                          | NIST SI-10      | A.14.2.5  | TH02, TH28             |
| SR-4       | Cadena de suministro | Verificación de integridad de componentes       | NIST SR-4       | A.15.1.1  | TH36                   |

---

## 7.5 Cronograma de Implementación

```
SEMANA 1 — CRÍTICO (< 24 horas)
├── C01: Instalar gitleaks + rotar credenciales expuestas
└── Notificación inmediata al equipo de seguridad

SEMANA 1-2 — ALTO MÁXIMA PRIORIDAD
├── C02: Parche JWT — forzar RS256, rechazar alg:none
├── C03: Rate limiting en login + MFA obligatorio
├── C11: Auditar queries — migrar a prepared statements
└── C10: Validación HMAC en todos los webhooks

SEMANA 2-3 — ALTO
├── C04: Fix IDOR en todos los endpoints
├── C05: Rate limiting por endpoint en API Gateway
├── C06: Validar MFA del lado servidor
├── C07: Bloqueo OTP tras 3 intentos
├── C08: SELECT FOR UPDATE en Transfer Service
├── C09: Activar protección DDoS avanzada
├── C12: Autenticación y TLS en Redis
├── C13: MFA y auditoría en Vault
├── C14: Nonce anti-replay en webhooks
└── C15: Approval manual para deploys a producción

MES 2 — MEDIO
├── C16–C30: Controles de nivel medio por sprint
└── Pruebas de penetración externa

MES 3 — BAJO + VALIDACIÓN
├── C38: Integrar TSA en contratos de préstamo
├── Auditoría completa de controles implementados
└── Re-evaluación del modelo de amenazas
```

---

## 7.6 Resumen Ejecutivo de Controles

| Nivel DREAD | Amenazas    | Controles        | SLA         |
| ----------- | ----------- | ---------------- | ----------- |
| 🔴 Crítico  | 1 (TH37)    | C01              | < 24 horas  |
| 🟠 Alto     | 14 amenazas | C02–C15          | < 7 días    |
| 🟡 Medio    | 22 amenazas | C16–C37          | < 30 días   |
| 🔵 Bajo     | 1 (TH21)    | C38              | Planificado |
| **Total**   | **38**      | **38 controles** |             |

---
