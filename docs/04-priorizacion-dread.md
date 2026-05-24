# 04 — Priorización de Amenazas: Metodología DREAD

```
╔══════════════════════════════════════════════════════════════════╗
║        ANÁLISIS DE RIESGOS — FINTECH BANKING DIGITAL            ║
║        Sección 5: Priorización DREAD                            ║
╚══════════════════════════════════════════════════════════════════╝
```

**Proyecto:** Fintech — Aplicación Móvil de Banca Digital  
**Documento:** `docs/04-priorizacion-dread.md`  
**Versión:** 1.0  
**Fecha:** Mayo 2026  
**Base:** 38 amenazas identificadas en `docs/03-analisis-stride.md` (TH01–TH38)

---

## 5.1 Metodología DREAD

DREAD es un modelo cuantitativo de priorización de riesgos. Asigna un puntaje numérico a cada amenaza evaluando 5 criterios independientes. El resultado permite ordenar las amenazas objetivamente y decidir qué remediar primero.

### Criterios de evaluación — Escala 1 a 10

| Criterio            | Abrev. | Pregunta clave                                 | 1 (mínimo)                              | 5 (medio)                           | 10 (máximo)                                 |
| ------------------- | ------ | ---------------------------------------------- | --------------------------------------- | ----------------------------------- | ------------------------------------------- |
| **Damage**          | D      | ¿Qué tan grave es el daño si se explota?       | Sin impacto real                        | Afecta a algunos usuarios           | Sistema comprometido totalmente             |
| **Reproducibility** | R      | ¿Qué tan fácil es repetir el ataque?           | Muy difícil, rara vez funciona          | Funciona a veces con esfuerzo       | Siempre funciona, automatizable             |
| **Exploitability**  | E      | ¿Qué habilidad y recursos requiere?            | Requiere experto con recursos avanzados | Requiere conocimiento técnico medio | Sin habilidad especial, herramienta pública |
| **Affected Users**  | A      | ¿A cuántos usuarios impacta?                   | Pocos usuarios específicos              | Grupo significativo                 | Todos los usuarios de la plataforma         |
| **Discoverability** | Di     | ¿Qué tan fácil es descubrir la vulnerabilidad? | Muy difícil de encontrar                | Requiere análisis técnico           | Visible públicamente, documentada           |

**Fórmula:** `DREAD Score = (D + R + E + A + Di) / 5` → Rango **0 a 10**

> La suma de los 5 criterios (cada uno de 1 a 10) se divide entre 5 para obtener un score normalizado, siguiendo la metodología original de Microsoft documentada por la cátedra. En la matriz se muestra la suma sobre 50 y el score final sobre 10 para mayor claridad.

### Escala de severidad

| Score (0–10) | Suma (0–50) | Nivel           | Color | Acción requerida                       |
| ------------ | ----------- | --------------- | ----- | -------------------------------------- |
| 9.0 – 10.0   | 45 – 50     | **CRÍTICO**     | 🔴    | Remediar inmediatamente (< 24 horas)   |
| 7.0 – 8.9    | 35 – 44     | **ALTO**        | 🟠    | Remediar urgente (< 7 días)            |
| 5.0 – 6.9    | 25 – 34     | **MEDIO**       | 🟡    | Remediar en siguiente sprint (30 días) |
| 3.0 – 4.9    | 15 – 24     | **BAJO**        | 🔵    | Remediar programado                    |
| 0.0 – 2.9    | 0 – 14      | **INFORMATIVO** | ⚪    | Documentar y monitorear                |

---

## 5.2 Matriz de Priorización DREAD — 38 Amenazas

| ID   | Amenaza                                                 | Comp.      | D   | R   | E   | A   | Di  | **Suma** | **Score /10** | **Nivel**  |
| ---- | ------------------------------------------------------- | ---------- | --- | --- | --- | --- | --- | -------- | ------------- | ---------- |
| TH01 | Robo de JWT en dispositivo móvil inseguro               | App Móvil  | 8   | 6   | 6   | 6   | 5   | **31**   | **6.2**       | 🟡 MEDIO   |
| TH02 | Manipulación de parámetros de transferencia via MitM    | App Móvil  | 8   | 5   | 5   | 7   | 4   | **29**   | **5.8**       | 🟡 MEDIO   |
| TH03 | Exponer Datos sensibles en logs del sistema operativo   | App Móvil  | 5   | 5   | 5   | 5   | 5   | **25**   | **5.0**       | 🟡 MEDIO   |
| TH04 | Reverse engineering del APK/IPA                         | App Móvil  | 7   | 5   | 5   | 6   | 6   | **29**   | **5.8**       | 🟡 MEDIO   |
| TH05 | App ejecutada en dispositivo rooteado                   | App Móvil  | 6   | 5   | 4   | 6   | 5   | **26**   | **5.2**       | 🟡 MEDIO   |
| TH06 | DDoS volumétrico contra CDN/WAF                         | WAF/CDN    | 6   | 8   | 7   | 9   | 7   | **37**   | **7.4**       | 🟠 ALTO    |
| TH07 | Bypass del WAF con payloads codificados                 | WAF        | 8   | 6   | 6   | 8   | 5   | **33**   | **6.6**       | 🟡 MEDIO   |
| TH08 | Bypass JWT con `alg:none`                               | API GW     | 10  | 9   | 8   | 10  | 7   | **44**   | **8.8**       | 🟠 ALTO    |
| TH09 | Credential stuffing sin rate limiting                   | API GW     | 8   | 9   | 8   | 9   | 8   | **42**   | **8.4**       | 🟠 ALTO    |
| TH10 | IDOR: acceso a recursos de otro usuario                 | API GW     | 9   | 8   | 7   | 9   | 6   | **39**   | **7.8**       | 🟠 ALTO    |
| TH11 | Shadow APIs expuestas sin controles del Gateway         | API GW     | 7   | 6   | 6   | 8   | 6   | **33**   | **6.6**       | 🟡 MEDIO   |
| TH12 | Rate limiting ausente en endpoints costosos             | API GW     | 5   | 8   | 7   | 8   | 7   | **35**   | **7.0**       | 🟠 ALTO    |
| TH13 | Bypass de MFA por manipulación del callback             | Auth       | 9   | 6   | 6   | 9   | 5   | **35**   | **7.0**       | 🟠 ALTO    |
| TH14 | Fuerza bruta de códigos OTP sin bloqueo                 | Auth       | 7   | 8   | 7   | 9   | 7   | **38**   | **7.6**       | 🟠 ALTO    |
| TH15 | JWT con expiración excesiva o sin blacklist             | Auth       | 6   | 6   | 5   | 8   | 5   | **30**   | **6.0**       | 🟡 MEDIO   |
| TH16 | Race condition en transferencias (double spending)      | Transfer   | 9   | 8   | 7   | 8   | 5   | **37**   | **7.4**       | 🟠 ALTO    |
| TH17 | Manipulación de límites de transferencia                | Transfer   | 8   | 5   | 5   | 7   | 4   | **29**   | **5.8**       | 🟡 MEDIO   |
| TH18 | Ausencia de logging inmutable en transferencias         | Transfer   | 5   | 5   | 4   | 7   | 4   | **25**   | **5.0**       | 🟡 MEDIO   |
| TH19 | Manipulación del score crediticio en tránsito           | Loan       | 8   | 6   | 6   | 7   | 5   | **32**   | **6.4**       | 🟡 MEDIO   |
| TH20 | Exposición del historial de morosidad (insider)         | Loan       | 6   | 5   | 4   | 6   | 4   | **25**   | **5.0**       | 🟡 MEDIO   |
| TH21 | Contratos de préstamo sin timestamp TSA                 | Loan       | 4   | 4   | 3   | 5   | 3   | **19**   | **3.8**       | 🔵 BAJO    |
| TH22 | Manipulación de órdenes de inversión en tránsito        | Investment | 8   | 6   | 6   | 7   | 5   | **32**   | **6.4**       | 🟡 MEDIO   |
| TH23 | Acceso al portfolio de otro usuario (IDOR inversión)    | Investment | 7   | 6   | 6   | 6   | 5   | **30**   | **6.0**       | 🟡 MEDIO   |
| TH24 | Webhook de pagos sin validación HMAC                    | Payment    | 9   | 8   | 7   | 7   | 7   | **38**   | **7.6**       | 🟠 ALTO    |
| TH25 | Logging accidental de PAN (violación PCI-DSS)           | Payment    | 8   | 4   | 4   | 6   | 5   | **27**   | **5.4**       | 🟡 MEDIO   |
| TH26 | Notificaciones push con datos financieros sensibles     | Notif.     | 4   | 7   | 7   | 6   | 7   | **31**   | **6.2**       | 🟡 MEDIO   |
| TH27 | Evasión del motor antifraude (smurfing)                 | Fraud      | 6   | 6   | 5   | 6   | 4   | **27**   | **5.4**       | 🟡 MEDIO   |
| TH28 | SQL Injection en consultas de la API                    | PostgreSQL | 10  | 7   | 7   | 10  | 6   | **40**   | **8.0**       | 🟠 ALTO    |
| TH29 | Acceso directo a PostgreSQL bypasseando negocio         | PostgreSQL | 10  | 5   | 5   | 10  | 3   | **33**   | **6.6**       | 🟡 MEDIO   |
| TH30 | Eliminación maliciosa de tablas críticas en BD          | PostgreSQL | 10  | 5   | 5   | 10  | 3   | **33**   | **6.6**       | 🟡 MEDIO   |
| TH31 | Session hijacking masivo desde Redis                    | Redis      | 10  | 6   | 6   | 10  | 4   | **36**   | **7.2**       | 🟠 ALTO    |
| TH32 | Manipulación del rate limiting en Redis                 | Redis      | 7   | 7   | 6   | 8   | 5   | **33**   | **6.6**       | 🟡 MEDIO   |
| TH33 | Compromiso del gestor de secretos Vault                 | Vault      | 10  | 7   | 6   | 10  | 5   | **38**   | **7.6**       | 🟠 ALTO    |
| TH34 | Replay attack en webhook de pago ya procesado           | Pasarela   | 8   | 8   | 7   | 7   | 6   | **36**   | **7.2**       | 🟠 ALTO    |
| TH35 | Manipulación de transferencia interbancaria en tránsito | Interbank  | 9   | 6   | 6   | 8   | 4   | **33**   | **6.6**       | 🟡 MEDIO   |
| TH36 | Inyección de código en pipeline CI/CD                   | CI/CD      | 10  | 6   | 6   | 10  | 4   | **36**   | **7.2**       | 🟠 ALTO    |
| TH37 | Credenciales hardcodeadas en repositorio Git            | CI/CD      | 10  | 10  | 9   | 10  | 10  | **49**   | **9.8**       | 🔴 CRÍTICO |
| TH38 | Borrado de logs de auditoría para ocultar actividad     | SIEM       | 7   | 5   | 5   | 8   | 4   | **29**   | **5.8**       | 🟡 MEDIO   |

---

## 5.3 Ranking Consolidado por Prioridad

### 🔴 CRÍTICO (Score 9.0–10.0) — Remediar en menos de 24 horas

| Prioridad | ID   | Score   | Amenaza                          | Por qué es crítica                                                                                                                |
| --------- | ---- | ------- | -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **#1**    | TH37 | **9.8** | Credenciales hardcodeadas en Git | Daño máximo, reproducible en minutos, bots automatizados la descubren solos. Un solo commit expone toda la infraestructura cloud. |

> Solo TH37 alcanza el umbral CRÍTICO (≥ 9.0) según la escala de Microsoft. TH08 y TH09, aunque de altísima prioridad, quedan en ALTO (8.8 y 8.4 respectivamente) y deben remediarse en la misma semana.

---

### 🟠 ALTO (Score 7.0–8.9) — Remediar en menos de 7 días

| Prioridad | ID   | Score   | Amenaza                                            |
| --------- | ---- | ------- | -------------------------------------------------- |
| **#2**    | TH08 | **8.8** | Bypass JWT con `alg:none`                          |
| **#3**    | TH09 | **8.4** | Credential stuffing sin rate limiting              |
| **#4**    | TH28 | **8.0** | SQL Injection en consultas de la API               |
| **#5**    | TH10 | **7.8** | IDOR — acceso a recursos de otro usuario           |
| **#6**    | TH24 | **7.6** | Webhook de pagos sin validación HMAC               |
| **#7**    | TH14 | **7.6** | Fuerza bruta de OTP sin bloqueo                    |
| **#8**    | TH33 | **7.6** | Compromiso del gestor de secretos Vault            |
| **#9**    | TH16 | **7.4** | Race condition en transferencias (double spending) |
| **#10**   | TH06 | **7.4** | DDoS volumétrico contra CDN/WAF                    |
| **#11**   | TH34 | **7.2** | Replay attack en webhook de pago                   |
| **#12**   | TH31 | **7.2** | Session hijacking masivo desde Redis               |
| **#13**   | TH36 | **7.2** | Inyección de código en pipeline CI/CD              |
| **#14**   | TH13 | **7.0** | Bypass de MFA por manipulación del callback        |
| **#15**   | TH12 | **7.0** | Rate limiting ausente en endpoints costosos        |

---

### 🟡 MEDIO (Score 5.0–6.9) — Remediar en el próximo sprint (30 días)

| Prioridad | ID   | Score   | Amenaza                                     |
| --------- | ---- | ------- | ------------------------------------------- |
| **#16**   | TH29 | **6.6** | Acceso directo a PostgreSQL                 |
| **#17**   | TH35 | **6.6** | Manipulación de transferencia interbancaria |
| **#18**   | TH30 | **6.6** | Eliminación maliciosa de tablas críticas    |
| **#19**   | TH07 | **6.6** | Bypass del WAF con payloads codificados     |
| **#20**   | TH11 | **6.6** | Shadow APIs sin controles                   |
| **#21**   | TH32 | **6.6** | Manipulación del rate limiting en Redis     |
| **#22**   | TH19 | **6.4** | Manipulación del score crediticio           |
| **#23**   | TH22 | **6.4** | Manipulación de órdenes de inversión        |
| **#24**   | TH01 | **6.2** | Robo de JWT en dispositivo móvil            |
| **#25**   | TH26 | **6.2** | Notificaciones push con datos sensibles     |
| **#26**   | TH23 | **6.0** | IDOR en portfolio de inversión              |
| **#27**   | TH15 | **6.0** | JWT con expiración excesiva                 |
| **#28**   | TH38 | **5.8** | Borrado de logs de auditoría                |
| **#29**   | TH17 | **5.8** | Manipulación de límites de transferencia    |
| **#30**   | TH02 | **5.8** | Manipulación de parámetros via MitM         |
| **#31**   | TH04 | **5.8** | Reverse engineering APK/IPA                 |
| **#32**   | TH27 | **5.4** | Evasión del motor antifraude                |
| **#33**   | TH25 | **5.4** | Logging accidental de PAN (PCI-DSS)         |
| **#34**   | TH05 | **5.2** | App en dispositivo rooteado                 |
| **#35**   | TH20 | **5.0** | Exposición del historial de morosidad       |
| **#36**   | TH18 | **5.0** | Ausencia de logging inmutable               |
| **#37**   | TH03 | **5.0** | Datos sensibles en logs del SO              |

---

### 🔵 BAJO (Score 3.0–4.9) — Monitorear y planificar

| Prioridad | ID   | Score   | Amenaza                                 |
| --------- | ---- | ------- | --------------------------------------- |
| **#38**   | TH21 | **3.8** | Contratos de préstamo sin timestamp TSA |

---

## 5.4 Análisis Detallado de las 3 Amenazas de Mayor Prioridad

### #1 — TH37: Credenciales hardcodeadas en Git (Score: 49/50) 🔴 CRÍTICO

| Criterio             | Puntaje      | Justificación                                                                                                                                 |
| -------------------- | ------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Damage (D)           | **10**       | Un AWS key expuesto da acceso a toda la infraestructura: PostgreSQL, S3 con KYC, secrets de Vault. Compromiso total del sistema.              |
| Reproducibility (R)  | **10**       | Siempre funciona. La credencial es válida hasta que se rote. Bots automatizados la explotan en minutos de forma completamente confiable.      |
| Exploitability (E)   | **9**        | Solo se necesita `aws configure` con las keys encontradas. Herramientas públicas disponibles para todos. Sin habilidad especial requerida.    |
| Affected Users (A)   | **10**       | Afecta a todos los usuarios: si se accede a la BD, todos sus datos quedan expuestos. Si se compromete Vault, todo el sistema falla.           |
| Discoverability (Di) | **10**       | Bots como GitGuardian, trufflehog, gitleaks escanean GitHub en tiempo real. Patrón `AKIA` + 16 chars es trivialmente identificable con regex. |
| **TOTAL**            | **9.8 / 10** | **🔴 CRÍTICO — Remediar en < 24 horas**                                                                                                       |

---

### #2 — TH08: Bypass JWT con `alg:none` (Score: 44/50) 🟠 ALTO

| Criterio             | Puntaje      | Justificación                                                                                                                                                      |
| -------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Damage (D)           | **10**       | El atacante puede suplantar a cualquier usuario incluyendo administradores. Acceso total a cuentas, transferencias, préstamos e inversiones de toda la plataforma. |
| Reproducibility (R)  | **9**        | Siempre reproducible si la librería JWT no está configurada correctamente. El ataque es determinístico: modificar el header + eliminar la firma siempre funciona.  |
| Exploitability (E)   | **8**        | Requiere entender JWT (conocimiento técnico medio). Herramientas como jwt.io permiten decodificar y modificar tokens fácilmente en el browser.                     |
| Affected Users (A)   | **10**       | Cualquier usuario de la plataforma puede ser suplantado. Un atacante puede iterar sobre todos los user IDs y acceder a cada cuenta.                                |
| Discoverability (Di) | **7**        | Requiere probar activamente si el servidor acepta `alg:none`. Scanners de seguridad como Burp Suite tienen plugins específicos para detectar esta vulnerabilidad.  |
| **TOTAL**            | **8.8 / 10** | **🟠 ALTO — Remediar en < 7 días**                                                                                                                                 |

---

### #3 — TH09: Credential Stuffing sin Rate Limiting (Score: 42/50) 🟠 ALTO

| Criterio             | Puntaje      | Justificación                                                                                                                                                              |
| -------------------- | ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Damage (D)           | **8**        | Por cada cuenta comprometida, el atacante tiene acceso a saldo, movimientos, puede iniciar transferencias. El daño escala con la cantidad de cuentas afectadas.            |
| Reproducibility (R)  | **9**        | Con herramientas como Hydra o Burp Intruder, el ataque es completamente automatizable y reproducible. Solo necesita una lista de credenciales filtradas.                   |
| Exploitability (E)   | **8**        | Bases de datos de credenciales filtradas disponibles públicamente (Have I Been Pwned tiene 12 mil millones de registros). Herramientas de ataque gratuitas y documentadas. |
| Affected Users (A)   | **9**        | Puede comprometer decenas o cientos de cuentas simultáneamente. Sin rate limiting, el ataque escala indefinidamente hasta agotar la lista de credenciales.                 |
| Discoverability (Di) | **8**        | El endpoint de login está documentado en la API pública. Solo hace falta probarlo con múltiples intentos para confirmar que no hay bloqueo por intentos fallidos.          |
| **TOTAL**            | **8.4 / 10** | **🟠 ALTO — Remediar en < 7 días**                                                                                                                                         |

---

## 5.5 Distribución Visual de Riesgos

```
DISTRIBUCIÓN POR NIVEL DE SEVERIDAD
══════════════════════════════════════════════════════════════════

🔴 CRÍTICO  (9.0-10) │███░░░░░░░░│  3 amenazas  (7.9%)
🟠 ALTO     (7.0-8.9)│████████░░░│ 12 amenazas (31.6%)
🟡 MEDIO    (5.0-6.9)│████████████│ 22 amenazas (57.9%)
🔵 BAJO     (3.0-4.9)│█░░░░░░░░░░│  1 amenaza   (2.6%)
⚪ INFORM.   (0-2.9) │░░░░░░░░░░░│  0 amenazas  (0.0%)
                                   ──────────────────
                                   38 amenazas totales

CONCENTRACIÓN POR COMPONENTE
══════════════════════════════════════════════════════════════════

API Gateway    │████████│  5 amenazas — mayor superficie de ataque externo
PostgreSQL     │██████░░│  3 amenazas — mayor impacto potencial
Auth Service   │██████░░│  3 amenazas — punto crítico de acceso
App Móvil      │██████░░│  5 amenazas — TB-6 (dispositivo no controlado)
Transfer Svc   │████░░░░│  3 amenazas — riesgo financiero directo
CI/CD + Infra  │████░░░░│  3 amenazas — impacto sistémico total
Payment Svc    │████░░░░│  2 amenazas — riesgo PCI-DSS
Investment Svc │██░░░░░░│  2 amenazas
Redis          │██░░░░░░│  2 amenazas
Otros          │████████│  9 amenazas restantes
```

---

## 5.6 Mapa de Calor — Probabilidad vs Impacto

```
           IMPACTO
           Bajo    Medio    Alto    Crítico
         ┌────────┬────────┬────────┬─────────┐
  Alta   │        │  TH06  │  TH09  │ TH08    │
         │        │  TH12  │  TH14  │ TH37    │
         │        │        │  TH16  │         │
PROB.    ├────────┼────────┼────────┼─────────┤
  Media  │  TH21  │  TH27  │  TH10  │ TH28    │
         │        │  TH03  │  TH24  │ TH33    │
         │        │  TH26  │  TH34  │ TH38    │
         │        │        │  TH13  │         │
         ├────────┼────────┼────────┼─────────┤
  Baja   │        │  TH25  │  TH01  │ TH29    │
         │        │  TH20  │  TH35  │ TH30    │
         │        │  TH18  │  TH31  │         │
         └────────┴────────┴────────┴─────────┘

Las amenazas en la esquina superior derecha (ALTA probabilidad +
CRÍTICO impacto) son las de máxima prioridad → TH08 (8.8), TH09 (8.4), TH37 (9.8)
```

---

## 5.7 Supuestos del Análisis DREAD

1. Los puntajes se asignaron considerando el sistema **sin controles implementados** — es decir, el riesgo inherente antes de las mitigaciones.
2. La **Reproducibility** se evaluó asumiendo que el atacante tiene acceso a herramientas estándar de pentesting (Burp Suite, Metasploit, aws-cli).
3. La **Discoverability** se evaluó asumiendo que el atacante tiene acceso a la documentación OpenAPI pública de la API.
4. El **Affected Users** se evaluó sobre la totalidad de la base de usuarios de la plataforma, no sobre un subconjunto.
5. Los scores son relativos entre sí dentro del contexto de esta fintech — pueden variar si cambia la arquitectura o el volumen de usuarios.

---

## 5.8 Ajuste Regulatorio — Impacto BCU en el Criterio Damage (D)

> El criterio **Damage** en DREAD evalúa el daño potencial total si la vulnerabilidad es explotada. En el contexto de una fintech regulada por el **BCU (Banco Central del Uruguay)**, el daño no es solo técnico — incluye **multas, sanciones y pérdida de licencia**. Las siguientes amenazas tienen su score D ajustado al incorporar el impacto regulatorio.

| ID   | Amenaza                                  | D técnico | D + regulatorio | Motivo del ajuste                                                                                        | Exposición BCU       |
| ---- | ---------------------------------------- | --------- | --------------- | -------------------------------------------------------------------------------------------------------- | -------------------- |
| TH25 | Logging accidental de PAN (PCI-DSS)      | 8         | **10**          | Violación PCI-DSS Req. 3.4 → inhabilitación de pagos con tarjeta + USD 5.000–100.000/mes                 | Multa PCI-DSS + BCU  |
| TH28 | SQL Injection → exposición masiva de PII | 10        | **10**          | Activa Ley 18.331 + obligación de notificación en 72 hs al BCU + notificación a usuarios                 | UI 500.000–2.000.000 |
| TH06 | DDoS → caída del servicio > 4 horas      | 6         | **8**           | Falla en continuidad operacional viola Circular 2.244 Art. 15                                            | UI 100.000–500.000   |
| TH27 | Evasión del antifraude (smurfing)        | 6         | **8**           | No detección de operaciones sospechosas viola Ley 19.574 (Antilavado) — riesgo penal para directivos     | UI 200.000–1.000.000 |
| TH18 | Ausencia de logging inmutable            | 5         | **7**           | Sin trazabilidad = incumplimiento Circular 2.244 Art. 12 + imposibilidad de probar cumplimiento ante BCU | UI 50.000–200.000    |
| TH09 | Credential stuffing masivo               | 8         | **9**           | Compromiso masivo de cuentas activa notificación obligatoria al BCU + Ley 18.331                         | UI 500.000–2.000.000 |

### Obligaciones regulatorias BCU ante un incidente de seguridad

```
INCIDENTE DETECTADO
        │
        ▼ (inmediato)
Contención y evaluación del alcance
        │
        ▼ (< 72 horas)
Notificación al BCU con:
  · Descripción del incidente
  · Datos y sistemas afectados
  · Acciones tomadas hasta el momento
        │
        ▼ (< 72 horas)
Notificación a usuarios afectados
  · Qué datos fueron expuestos
  · Qué acciones tomar (cambiar contraseña, etc.)
        │
        ▼ (< 15 días)
Presentar Plan de Remediación al BCU
  · Causa raíz identificada
  · Controles implementados
  · Cronograma de remediación completa
        │
        ▼ (si el BCU lo requiere)
Auditoría externa independiente
  → Incumplir cualquier paso activa multas adicionales
```

**Referencia normativa:**

- Ley 19.210 — Inclusión Financiera (marco para IDE y PSP)
- Circular 2.244 — Marco de Ciberseguridad del BCU (2023)
- Ley 18.331 — Protección de Datos Personales (Uruguay)
- Ley 19.574 — Lavado de Activos y Financiamiento del Terrorismo
- Estándar PCI-DSS v4.0 — Para procesamiento de tarjetas

---
