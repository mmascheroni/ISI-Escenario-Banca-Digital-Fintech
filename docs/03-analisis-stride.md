# 03 — Análisis de Amenazas: Metodología STRIDE

```
╔══════════════════════════════════════════════════════════════════╗
║        ANÁLISIS DE RIESGOS — FINTECH BANKING DIGITAL            ║
║        Sección 4: Análisis de Amenazas STRIDE                   ║
╚══════════════════════════════════════════════════════════════════╝
```

**Proyecto:** Fintech — Aplicación Móvil de Banca Digital  
**Documento:** `docs/03-analisis-stride.md`  
**Versión:** 1.0  
**Fecha:** Mayo 2026

---

## 4.1 Metodología Aplicada

STRIDE es un framework de modelado de amenazas que categoriza los ataques en 6 tipos. Se aplica **por componente** cruzando cada trust boundary identificado en el Paso 2.

| Letra | Categoría              | Pregunta que responde                                           | Propiedad CIA vulnerada |
| ----- | ---------------------- | --------------------------------------------------------------- | ----------------------- |
| **S** | Spoofing               | ¿Alguien puede suplantar la identidad de un usuario o servicio? | Autenticación           |
| **T** | Tampering              | ¿Alguien puede modificar datos en tránsito o en reposo?         | Integridad              |
| **R** | Repudiation            | ¿Puede alguien negar haber realizado una acción?                | No repudio              |
| **I** | Information Disclosure | ¿Se expone información sensible a quien no debe verla?          | Confidencialidad        |
| **D** | Denial of Service      | ¿Puede alguien dejar el sistema o un servicio inaccesible?      | Disponibilidad          |
| **E** | Elevation of Privilege | ¿Puede alguien obtener más permisos de los que le corresponden? | Autorización            |

**Criterio de cobertura:** Se analiza cada componente del sistema en cada Trust Boundary (TB-1 a TB-6), priorizando los activos de criticidad máxima (🔴) y alta (🟠) identificados en el inventario.

---

## 4.2 Matriz de Amenazas por Componente

| ID   | Componente                 | TB    | STRIDE | Amenaza                                                                                                      | Activos Afectados          |
| ---- | -------------------------- | ----- | ------ | ------------------------------------------------------------------------------------------------------------ | -------------------------- |
| TH01 | App Móvil (iOS/Android)    | TB-6  | **S**  | Robo de JWT almacenado de forma insegura en el dispositivo                                                   | A03, A04, A05              |
| TH02 | App Móvil (iOS/Android)    | TB-6  | **T**  | Manipulación de parámetros de transferencia via proxy MitM (monto, CBU destino)                              | A08, A05                   |
| TH03 | App Móvil (iOS/Android)    | TB-6  | **I**  | Exponer datos sensibles en logs del sistema operativo o caché sin cifrar                                     | A03, A05, A06              |
| TH04 | App Móvil (iOS/Android)    | TB-6  | **T**  | Reverse engineering del APK/IPA para extraer claves hardcodeadas o lógica de negocio                         | A27, A28, A29              |
| TH05 | App Móvil (iOS/Android)    | TB-6  | **E**  | Ejecución de la app en dispositivo rooteado/jailbroken que anula controles de seguridad                      | A03, A04                   |
| TH06 | WAF / CDN                  | TB-1  | **D**  | Ataque DDoS que satura el CDN/WAF y deja la API inaccesible                                                  | T06, T07, T28              |
| TH07 | WAF / CDN                  | TB-1  | **T**  | Bypass del WAF mediante payloads codificados (doble encoding, fragmentación HTTP)                            | T07, T06                   |
| TH08 | API Gateway                | TB-2  | **S**  | Bypass de autenticación JWT con header `alg:none` (sin verificación de firma)                                | A03, A04, A05–A23          |
| TH09 | API Gateway                | TB-2  | **S**  | Credential stuffing con bases de datos filtradas de otros servicios (sin rate limiting)                      | A03, A04                   |
| TH10 | API Gateway                | TB-2  | **E**  | IDOR: acceso a recursos (cuenta, préstamo, inversión) de otro usuario por manipulación de IDs                | A05, A06, A12, A17         |
| TH11 | API Gateway                | TB-2  | **I**  | Exposición de endpoints no documentados (Shadow APIs) que no pasan por los controles del Gateway             | A05–A23                    |
| TH12 | API Gateway                | TB-2  | **D**  | Rate limiting ausente o mal configurado que permite abuso de endpoints costosos (scoring, transferencias)    | T08, T10, T11              |
| TH13 | Auth Service (T08)         | TB-3  | **S**  | Bypass de MFA por manipulación de la respuesta de autenticación (parameter tampering en el callback)         | A03, A04                   |
| TH14 | Auth Service (T08)         | TB-3  | **S**  | Fuerza bruta de códigos OTP de 6 dígitos sin bloqueo por intentos (1M combinaciones posibles)                | A03, A04                   |
| TH15 | Auth Service (T08)         | TB-3  | **I**  | Tokens JWT con tiempo de expiración excesivo o sin mecanismo de revocación (blacklist)                       | A04                        |
| TH16 | Transfer Service (T10)     | TB-3  | **T**  | Race condition: envío simultáneo de múltiples transferencias antes de actualizar el saldo (double spending)  | A05, A08, A09              |
| TH17 | Transfer Service (T10)     | TB-3  | **T**  | Manipulación de límites de transferencia diarios mediante llamadas directas al servicio interno              | A11, A08                   |
| TH18 | Transfer Service (T10)     | TB-3  | **R**  | Ausencia de logging inmutable en transferencias: el usuario niega haber iniciado la operación                | A08, A09, A24              |
| TH19 | Prestamo Service (T11)     | TB-3  | **T**  | Manipulación del score crediticio en tránsito entre el bureau externo y el servicio de préstamos             | A13, A12                   |
| TH20 | Prestamo Service (T11)     | TB-3  | **I**  | Exposición del historial de morosidad y scoring a empleados sin necesidad legítima (exceso de privilegio)    | A13, A16                   |
| TH21 | Prestamo Service (T11)     | TB-3  | **R**  | Contratos de préstamo firmados digitalmente sin timestamp de autoridad de sellado (TSA) verificable          | A14                        |
| TH22 | Investment Service (T12)   | TB-3  | **T**  | Manipulación de órdenes de suscripción/rescate en tránsito para alterar monto o fondo destino                | A18, A17                   |
| TH23 | Investment Service (T12)   | TB-3  | **I**  | Acceso al portfolio de otro usuario por falla en el control de autorización a nivel de servicio              | A17, A20                   |
| TH24 | Payment Service (T13)      | TB-3  | **T**  | Webhook de la pasarela de pagos sin validación de firma HMAC: confirmación de pago falsa                     | A22, A07                   |
| TH25 | Payment Service (T13)      | TB-3  | **I**  | Logging accidental del PAN o datos de tarjeta en texto plano en logs del backend (violación PCI-DSS)         | A07, A22                   |
| TH26 | Notification Service (T14) | TB-3  | **I**  | Notificaciones push con datos financieros sensibles (saldo, monto) visibles en pantalla bloqueada            | A05, A06, A26              |
| TH27 | Fraud Engine (T15)         | TB-3  | **E**  | Evasión del motor antifraude mediante transacciones graduales por debajo del umbral de detección             | A05, A08                   |
| TH28 | PostgreSQL (T16/T17)       | TB-4  | **I**  | SQL Injection en parámetros de búsqueda o filtrado que expone datos de otros usuarios                        | A01–A23, T16               |
| TH29 | PostgreSQL (T16/T17)       | TB-4  | **T**  | Acceso directo a la base de datos bypasseando la capa de negocio (insider o credencial comprometida)         | Todos los activos de datos |
| TH30 | PostgreSQL (T16/T17)       | TB-4  | **D**  | Eliminación o cifrado malicioso de tablas críticas (accounts, transactions) sin backup reciente              | A05, A06, A08, T16         |
| TH31 | Redis (T18)                | TB-4  | **S**  | Robo de tokens JWT activos desde Redis (sin autenticación o con credencial débil) → session hijacking masivo | A04, todos los usuarios    |
| TH32 | Redis (T18)                | TB-4  | **T**  | Manipulación del estado de rate limiting en Redis para anular los límites del API Gateway                    | T06, T08                   |
| TH33 | Vault / Secrets (T29)      | TB-4  | **E**  | Compromiso del gestor de secretos: acceso a todas las claves, API keys y credenciales del sistema            | A27, A28, todos            |
| TH34 | Pasarela de Pagos (T20)    | TB-5  | **T**  | Replay attack: reenvío de un webhook de pago ya procesado para acreditar fondos por segunda vez              | A22, A07                   |
| TH35 | Red Interbancaria (T21)    | TB-5  | **T**  | Manipulación de instrucciones de transferencia interbancaria en tránsito (CBU destino, monto)                | A08, A09                   |
| TH36 | CI/CD Pipeline (T30)       | Infra | **T**  | Inyección de código malicioso en el pipeline de build: backdoor deployado en producción                      | A29, A30, todos            |
| TH37 | CI/CD Pipeline (T30)       | Infra | **I**  | Credenciales de cloud o secretos hardcodeados en repositorio Git (público o privado comprometido)            | A27, A28, T26              |
| TH38 | SIEM (T31)                 | Infra | **T**  | Manipulación o borrado de logs de auditoría para ocultar actividad maliciosa (insider)                       | A24, A25                   |

**Total: 38 amenazas identificadas** en 15 componentes a través de 6 Trust Boundaries.

---

## 4.3 Detalle de Amenazas Críticas

A continuación se desarrollan las amenazas de mayor impacto potencial sobre el sistema fintech.

---

### TH08 — Bypass de autenticación JWT con `alg:none`

```
Categoría STRIDE : Spoofing
Componente       : API Gateway (T06)
Trust Boundary   : TB-2
Activos afectados: A03 (Credenciales), A04 (Tokens), A05–A23 (todos los datos de negocio)
Probabilidad     : Media (depende de la librería JWT y su configuración)
Impacto          : CRÍTICO — acceso total al sistema como cualquier usuario
```

**Descripción:**
El estándar JWT permite especificar el algoritmo de firma en el header (`"alg": "RS256"`). Si el servidor no valida ni restringe este campo, un atacante puede modificar el header a `"alg": "none"`, eliminando la firma por completo. El servidor acepta el token falsificado como válido, permitiendo suplantar a cualquier usuario —incluyendo administradores— sin conocer ninguna credencial.

**Vector de ataque:**

```
1. Atacante obtiene cualquier JWT válido (propio o interceptado)
2. Decodifica el header: {"alg": "RS256", "typ": "JWT"}
3. Lo modifica a:        {"alg": "none", "typ": "JWT"}
4. Modifica el payload:  {"sub": "admin", "role": "superuser"}
5. Elimina la firma (deja el JWT con punto final vacío)
6. Envía el token modificado → el servidor lo acepta si no valida el alg
```

**Técnicas MITRE ATT&CK:**

- T1078 — Valid Accounts: el atacante empieza con una cuenta propia legítima para obtener un JWT real. MITRE documenta que los atacantes usan cuentas válidas como punto de partida .
- T1550.001 — Use Alternate Authentication Material: Application Access Token - luego manipula ese token de aplicación para elevar sus privilegios sin conocer la contraseña de admin. MITRE registró esta técnica porque atacantes reales la usaron contra APIs con JWT mal configurados.

**Controles recomendados:**

- Configurar la librería JWT para aceptar **exclusivamente** `RS256` o `ES256`
- Rechazar explícitamente cualquier token con `alg: none` o `alg: HS256` (si el sistema usa asimétrico)
- Test unitario obligatorio en CI que verifique el rechazo de `alg:none`
- Rotación periódica del par de claves RSA usadas para firma

---

### TH16 — Race condition en transferencias (double spending)

```
Categoría STRIDE : Tampering
Componente       : Transfer Service (T10)
Trust Boundary   : TB-3
Activos afectados: A05 (Saldo cuenta), A08 (Órdenes de transferencia), A09 (Comprobantes)
Probabilidad     : Media-Alta (error de implementación común en sistemas concurrentes)
Impacto          : ALTO — pérdida financiera directa y cuantificable
```

**Descripción:**
Un usuario malicioso envía múltiples solicitudes de transferencia simultáneas (ej. 10–20 requests en paralelo) antes de que el sistema actualice el saldo en la base de datos. Si la lectura del saldo y su actualización no son una operación atómica protegida por bloqueo, todas las solicitudes "leen" el mismo saldo disponible y se aprueban, debitando más dinero del que existe.

**Vector de ataque:**

```
Saldo inicial: $10.000
Atacante lanza 10 requests simultáneos de transferencia de $9.000

Sin lock pesimista:
  Thread 1: Lee saldo = $10.000 → aprueba
  Thread 2: Lee saldo = $10.000 → aprueba (antes de que Thread 1 escriba)
  ...todos aprueban
  Resultado: se transfieren $90.000 con saldo de $10.000

Con SELECT FOR UPDATE:
  Thread 1: Adquiere lock → lee $10.000 → aprueba → escribe $1.000 → libera
  Thread 2: Espera lock → lee $1.000 → rechaza (fondos insuficientes)
```

**Técnicas MITRE ATT&CK:**

- T1499.004 — Application or System Exploitation: l atacante explota vulnerabilidades en la lógica de la aplicación — como condiciones de carrera (race conditions) — para causar que el sistema procese operaciones de forma incorrecta, generando estados inválidos que benefician al atacante.

**Controles recomendados:**

- `SELECT FOR UPDATE` en PostgreSQL al leer el saldo antes de debitar
- Transacciones ACID con aislamiento `SERIALIZABLE` o `REPEATABLE READ`
- Idempotency keys por solicitud de transferencia (deduplicación en Redis)
- Test de carga con concurrencia alta antes de cada release

---

### TH24 — Webhook de pasarela de pagos sin validación HMAC

```
Categoría STRIDE : Tampering
Componente       : Payment Service (T13) / Pasarela de Pagos (T20)
Trust Boundary   : TB-5
Activos afectados: A07 (PAN tokenizado), A22 (Historial de pagos)
Probabilidad     : Alta (error de implementación muy frecuente)
Impacto          : ALTO — fraude financiero directo (servicios sin cobrar)
```

**Descripción:**
La pasarela de pagos notifica el resultado de una transacción mediante webhooks HTTP. Si el backend no valida la firma HMAC-SHA256 incluida en el header del webhook, un atacante puede enviar webhooks falsos indicando que un pago fue aprobado cuando en realidad fue rechazado, obteniendo servicios sin pagar.

**Vector de ataque:**

```
1. Usuario inicia un pago (intencionalmente con tarjeta rechazada)
2. Atacante intercepta la URL del webhook (expuesta o adivinada)
3. Envía manualmente: POST /webhooks/payment
   Body: {"status": "approved", "payment_id": "pay_123", "amount": 5000}
4. Si el backend no valida la firma HMAC → acredita el pago como exitoso
5. Usuario obtiene el servicio/préstamo sin haber pagado
```

**Técnicas MITRE ATT&CK:**

- T1565.001 — Stored Data Manipulation: El atacante no destruye datos ni bloquea el sistema. En cambio, inserta o modifica datos específicos para que el sistema tome decisiones incorrectas que lo benefician. Los datos quedan almacenados como si fueran legítimos.

**Controles recomendados:**

- Validar siempre la firma HMAC-SHA256 incluida en el header `X-Signature`
- Rechazar webhooks sin firma o con firma inválida con HTTP 401
- IP whitelist de los servidores de la pasarela de pagos
- Re-consultar el estado del pago directamente a la API de la pasarela antes de acreditar
- Registro de todos los webhooks recibidos (válidos e inválidos) con timestamp

---

### TH28 — SQL Injection en consultas de la API

```
Categoría STRIDE : Information Disclosure / Tampering
Componente       : API Backend → PostgreSQL (T16)
Trust Boundary   : TB-4
Activos afectados: A01–A23 (potencialmente toda la base de datos)
Probabilidad     : Media (controlable con buenas prácticas, pero frecuente en equipos sin formación)
Impacto          : CRÍTICO — exposición masiva de datos financieros y PII
```

**Descripción:**
Si algún endpoint de la API construye consultas SQL concatenando parámetros del usuario sin sanitizar, un atacante puede inyectar código SQL para extraer datos de tablas arbitrarias, modificar registros, o eliminar información crítica. En un sistema financiero, esto representa una violación catastrófica de PCI-DSS e ISO 27001.

**Vector de ataque:**

```
GET /api/movements?accountId=1 OR 1=1--
→ Retorna movimientos de TODOS los usuarios

GET /api/users?search='; DROP TABLE transactions;--
→ Elimina la tabla de transacciones

GET /api/loans?userId=1 UNION SELECT username,password,null FROM users--
→ Extrae credenciales
```

**Técnicas MITRE ATT&CK:**

- T1190 — Exploit Public-Facing Application: El atacante encuentra y explota una vulnerabilidad en una aplicación expuesta a internet — en este caso, los endpoints REST de la API. El objetivo es entrar al sistema o extraer datos a través de esa puerta pública.
- T1005 — Data from Local System: Una vez dentro, el atacante recolecta información sensible almacenada en el sistema comprometido. En una fintech, "el sistema local" es PostgreSQL — el atacante extrae datos de cuentas, movimientos, préstamos e identidad de todos los usuarios.

**Controles recomendados:**

- Usar **exclusivamente** prepared statements o parameterized queries en toda la capa de acceso a datos
- ORM con escape automático (SQLAlchemy, Hibernate, Sequelize)
- SAST en el pipeline CI/CD con reglas específicas de SQLi
- WAF con reglas de detección de SQLi como segunda capa
- Principio de mínimo privilegio en usuarios de BD: cada microservicio solo tiene acceso a sus tablas

---

### TH31 — Session hijacking masivo desde Redis

```
Categoría STRIDE : Spoofing
Componente       : Redis (T18)
Trust Boundary   : TB-4
Activos afectados: A04 (Tokens JWT activos), todos los usuarios de la plataforma
Probabilidad     : Baja (requiere acceso a la red interna), Impacto: CRÍTICO
Impacto          : CRÍTICO — compromiso de TODAS las sesiones activas simultáneamente
```

**Descripción:**
Redis almacena los JWT activos y el estado de las sesiones. Si Redis no tiene autenticación configurada (o tiene la contraseña por defecto vacía), cualquier actor con acceso a la red interna puede conectarse directamente, extraer todos los tokens activos y suplantar a cualquier usuario de la plataforma en paralelo. A diferencia de comprometer una cuenta, esto afecta a **todos los usuarios simultáneamente**.

**Vector de ataque:**

```
1. Atacante obtiene acceso a la VPC (insider, CI/CD comprometido, lateral movement)
2. redis-cli -h redis-internal -p 6379
   → Si no hay AUTH configurado, acceso inmediato
3. KEYS session:* → lista todos los tokens activos
4. GET session:user_12345 → obtiene JWT de ese usuario
5. Usa JWT para llamar a la API como ese usuario
6. Puede iterar sobre todos los usuarios activos
```

**Técnicas MITRE ATT&CK:**

- T1539 — Steal Web Session Cookie
- T1078 — Valid Accounts

**Controles recomendados:**

- Autenticación obligatoria en Redis (`requirepass` + ACLs por usuario)
- Acceso a Redis exclusivamente desde la subred de aplicación (network policy)
- Cifrado TLS en la conexión a Redis
- Rotación automática de la contraseña de Redis via Vault
- Alertas SIEM ante accesos directos a Redis fuera de los microservicios esperados

---

### TH38 — Inyección de código en pipeline CI/CD

```
Categoría STRIDE : Tampering
Componente       : CI/CD Pipeline (T30)
Trust Boundary   : Infraestructura transversal
Activos afectados: A29, A30 (código fuente), todos los activos del sistema en producción
Probabilidad     : Media (requiere comprometer el repo o las credenciales del pipeline)
Impacto          : CRÍTICO — backdoor en producción, acceso total al sistema
```

**Descripción:**
El pipeline de CI/CD tiene acceso privilegiado a producción. Si un atacante compromete una dependencia del proyecto (supply chain attack), las credenciales del pipeline, o los permisos de un PR, puede insertar código malicioso que se despliega automáticamente en producción sin detección. Un backdoor en el backend puede exfiltrar datos, crear cuentas ocultas o manipular transacciones.

**Vector de ataque:**

```
Escenario A — Supply chain:
1. Atacante publica versión maliciosa de dependencia npm/pip usada por el backend
2. `npm update` en CI incorpora la dependencia maliciosa
3. Build pasa sin errores → deploy a producción
4. El paquete malicioso exfiltra secretos del entorno al iniciar

Escenario B — Credenciales del runner:
1. Atacante obtiene el token del runner de GitHub Actions
2. Modifica el workflow para ejecutar código arbitrario con permisos cloud
3. Extrae credenciales, modifica infraestructura, crea backdoor

Escenario C — PR malicioso:
1. Atacante abre PR aparentemente legítimo con cambio pequeño
2. El cambio incluye lógica oculta en código de bajo nivel
3. Code review no detecta → merge → deploy
```

**Técnicas MITRE ATT&CK:**

- T1059 — Command and Scripting Interpreter: El atacante ejecuta comandos o scripts maliciosos usando intérpretes legítimos del sistema — bash, Python, PowerShell, Node.js. En el contexto de CI/CD, ejecuta código arbitrario dentro del pipeline aprovechando que los runners tienen acceso privilegiado a producción.
- T1195.002 — Supply Chain Compromise: Compromise Software Supply Chain: El atacante compromete una dependencia de terceros (paquete npm, pip, librería) antes de que llegue al proyecto. La fintech instala sin saber el paquete malicioso, que se ejecuta en el build. Es el ataque más difícil de detectar porque viene de fuentes aparentemente confiables.
- T1078.004 — Valid Accounts: Cloud Accounts: El atacante roba o filtra credenciales de cuentas cloud (AWS, GCP, Azure) — claves de acceso, tokens de servicio, credenciales de runners. Con una cuenta cloud válida puede modificar infraestructura, acceder a bases de datos, escalar privilegios y moverse lateralmente sin levantar alarmas.

**Controles recomendados:**

- Approval manual obligatorio para deploy a producción (mínimo 2 personas)
- SAST y análisis de dependencias (OWASP Dependency Check, Snyk) en cada PR
- Firma de artefactos de build con checksum verificado antes del deploy
- Principio de mínimo privilegio en runners: solo los permisos mínimos necesarios
- Revisión de permisos de terceras partes con acceso al repositorio
- Lockfile de dependencias (`package-lock.json`, `poetry.lock`) commiteado y auditado

---

### TH39 — Credenciales hardcodeadas en repositorio Git

```
Categoría STRIDE : Information Disclosure / Elevation of Privilege
Componente       : CI/CD / Repositorio de código (T30)
Trust Boundary   : Infraestructura transversal
Activos afectados: A27 (claves criptográficas), A28 (secretos de infra), toda la infraestructura
Probabilidad     : Alta (error humano muy frecuente)
Impacto          : CRÍTICO — compromiso total de la infraestructura
```

**Descripción:**
Un desarrollador comete por error una API key de AWS, una contraseña de base de datos o una clave privada en el repositorio Git. Bots automatizados escanean repositorios públicos y privados comprometidos en busca de patrones de credenciales (ej. `AKIA...` para AWS). En minutos desde el commit, el atacante puede tener acceso completo a la infraestructura cloud, bases de datos y todos los activos del sistema.

**Vector de ataque:**

```
1. Desarrollador hace commit con: DB_PASSWORD=supersecret123 en .env
2. Bot de GitHub scanning detecta el patrón en < 5 minutos
3. Atacante usa la credencial: psql -h prod-db.fintech.com -U admin
4. Acceso directo a PostgreSQL de producción con todos los datos
→ También aplica para: AWS keys, tokens JWT signing keys, API keys de terceros
```

**Técnicas MITRE ATT&CK:**

- T1552.001 — Unsecured Credentials: Credentials In Files: El atacante busca credenciales almacenadas en texto plano dentro de archivos del sistema — código fuente, archivos de configuración, variables de entorno, scripts. En el contexto de Git, busca dentro del historial de commits y repositorios expuestos.
- T1078.004 — Valid Accounts: Cloud Accounts: Una vez que el atacante obtiene las credenciales de nube (AWS keys, tokens de servicio), las usa como cuentas válidas para autenticarse directamente en la infraestructura cloud. No necesita hackear nada más — las credenciales son legítimas.

**Controles recomendados:**

- Git hooks pre-commit: `detect-secrets`, `gitleaks` que bloquean el commit si hay secretos
- Escaneo retroactivo del historial del repositorio con `trufflehog`
- Política de rotación **inmediata** de cualquier credencial que haya sido commiteada
- Todas las credenciales en Vault (T29), referenciadas por nombre, nunca en texto plano
- Revisión periódica de `.gitignore` para garantizar exclusión de `.env`, `*.pem`, `*.key`

---

## 4.4 Resumen por Categoría STRIDE

| Categoría                      | Cant. | Amenazas principales                                                               |
| ------------------------------ | ----- | ---------------------------------------------------------------------------------- |
| **S — Spoofing**               | 8     | TH01, TH08, TH09, TH13, TH14, TH15, TH31, (implícito TH33)                         |
| **T — Tampering**              | 13    | TH02, TH04, TH07, TH16, TH17, TH19, TH22, TH24, TH29, TH32, TH34, TH35, TH38, TH40 |
| **R — Repudiation**            | 3     | TH18, TH21, TH40                                                                   |
| **I — Information Disclosure** | 8     | TH03, TH11, TH15, TH20, TH23, TH25, TH26, TH28, TH39                               |
| **D — Denial of Service**      | 3     | TH06, TH12, TH30                                                                   |
| **E — Elevation of Privilege** | 5     | TH05, TH10, TH27, TH29, TH33                                                       |

> Algunas amenazas cubren múltiples categorías STRIDE (ej. TH29 es T + E, TH33 es E + I, TH40 es T + R).

---

## 4.5 Distribución de Amenazas por Trust Boundary

| Trust Boundary                | Amenazas | IDs                          |
| ----------------------------- | -------- | ---------------------------- |
| TB-6 — Dispositivo móvil      | 5        | TH01, TH02, TH03, TH04, TH05 |
| TB-1 — Perímetro externo      | 2        | TH06, TH07                   |
| TB-2 — API Gateway            | 5        | TH08, TH09, TH10, TH11, TH12 |
| TB-3 — Microservicios         | 15       | TH13–TH27                    |
| TB-4 — Capa de datos          | 6        | TH28–TH33                    |
| TB-5 — Integraciones externas | 2        | TH34, TH35                   |
| Infra transversal             | 3        | TH38, TH39, TH40             |

> **La zona de microservicios (TB-3) concentra la mayor cantidad de amenazas**, lo que refleja la complejidad de la lógica de negocio financiera: transferencias, préstamos, inversiones y pagos.

---

## 4.6 Supuestos del Análisis

1. Se asume que el atacante externo tiene acceso a la documentación pública de la API (Swagger/OpenAPI).
2. Se asume que pueden existir insiders con acceso legítimo a partes del sistema que actúan de forma maliciosa.
3. Los ataques de ingeniería social (phishing a empleados) están fuera del alcance de este documento pero se consideran vectores de entrada para amenazas internas.
4. Las amenazas de hardware físico (robo de servidores) están fuera del alcance por responsabilidad del proveedor cloud.

---
