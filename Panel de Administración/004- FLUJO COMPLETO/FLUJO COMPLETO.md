
1. EL DOCTOR INGRESA AL SISTEMA.
2. VER LISTA DE PACIENTES.
3. HACER CLIC EN UNO (DETALLES).
4. REVISIÓN DE INFORMACIÓN DEL PACIENTE.
5. CONSULTAR HISTORIAL MÉDICO COMPLETO (OPCIONAL).
6. IR A NUEVA CONSULTA.
7. REGISTRAR DIAGNÓSTICO.
8. GENERAR RECETA.
9. ENVIAR RECETA.
10. MARCAR PACIENTE COMO ATENDIDO (FT).
11. OPCIÓN DE CONFIRMAR CAMBIOS (S/N).
12. LISTA DE PACIENTES (PANTALLA INICIAL).



# Análisis de Requerimientos — PharmaBot

> **Versión:** 1.0 | **Fecha:** Abril 2026 | **Estado:** Draft para revisión

---

## 1. Contexto General del Sistema

PharmaBot es un sistema distribuido que conecta tres capas tecnológicas independientes pero altamente integradas:

|Capa|Tecnología|Actor Principal|
|---|---|---|
|**App Móvil (React Native / Expo)**|Paciente|Búsqueda, compra y retiro de medicamentos|
|**Panel Web (Admin / Médico)**|Doctor / Farmacéutico / Técnico|Emisión de recetas, gestión de pacientes, inventario|
|**Expendedora Física (IoT / Firmware)**|Sistema autónomo supervisado|Dispensación física del medicamento|

El flujo central del sistema es: **el médico emite una receta digital → el paciente la visualiza en la app → el paciente compra el medicamento → la máquina lo dispensa físicamente.**

### 1.1 Marco Legal Aplicable

|Ley / Norma|Impacto directo en el sistema|
|---|---|
|**Ley 81/2019** — Protección de Datos Personales|Consentimiento explícito, cifrado AES-256, derecho al olvido, ROPA, no compartir PII con terceros|
|**Ley 1/2001** — Farmacia y Medicamentos|Trazabilidad Rx, supervisión farmacéutica remota, verificación de idoneidad del médico|
|**Ley 83/2012** — Medicamentos Genéricos|Mostrar alternativa genérica obligatoriamente antes de confirmar compra de marca|
|**Reglamentos CSS** — Controlados|Audit log inmutable 5 años, foto al dispensar, límite diario, MFA obligatorio|
|**Tribunal Electoral**|Verificación de cédula panameña obligatoria para compras con receta|

---

## 2. Análisis de los Módulos Identificados

### 2.1 Módulos del Panel Web (Médico / Admin)

Los documentos definen un flujo médico claro pero con algunos vacíos. Se identifican los siguientes módulos:

| Módulo                 | Estado en documentación | Observaciones                                                  |
| ---------------------- | ----------------------- | -------------------------------------------------------------- |
| **Login médico**       | Bien definido           | RUC como identificador único, OTP, bloqueo por fuerza bruta    |
| **Lista de pacientes** | Bien definido           | Búsqueda en tiempo real, filtros por estado, card por paciente |
| **Panel del paciente** | Parcialmente definido   | Resumen general + historial clínico                            |
| **Nueva consulta**     | Bien definido           | Formulario de consulta, guardar + generar receta               |
| **Exámenes**           | Parcialmente definido   | Listado, detalle, adjuntar resultados                          |
| **Receta electrónica** | Bien definido           | Editor, vista previa, envío a app del paciente                 |

**Vacío crítico identificado:** No está definido el mecanismo exacto de cómo la receta emitida en el panel web llega a la app móvil del paciente. Se asume: backend genera la receta → la asocia al `user_id` del paciente → el paciente la visualiza en su app.

### 2.2 Módulos de la App Móvil (Paciente)

|Módulo|Estado en documentación|Observaciones|
|---|---|---|
|**Autenticación**|Bien definido|Registro con cédula, OTP, JWT RS256|
|**Home / Inicio**|Bien definido|Recetas activas, dispensadores frecuentes, búsqueda rápida|
|**Búsqueda de medicamentos**|Bien definido|Debounce 300ms, filtros, sugerencia de genérico|
|**Mapa de expendedoras**|Bien definido|Estado en tiempo real, telemetría, modo offline degradado|
|**Compra / Checkout**|Bien definido (crítico)|Saga pattern, QR HMAC, OTC vs Rx vs Controlado|
|**Recetas**|Bien definido|Listado por estado, detalle, historial de dispensaciones|
|**Perfil**|Bien definido|Datos personales, perfil médico, métodos de pago, configuración|

### 2.3 Módulo de Expendedora (IoT)

|Función|Estado|Observaciones|
|---|---|---|
|**Telemetría MQTT**|Referenciado|Cada 30 segundos, alerta sin señal > 90s|
|**Lectura de QR**|Bien definido|Envía token al backend para verificación|
|**Dispensación**|Bien definido|Atómica, confirmación ACK, modo offline degradado|
|**Cámara (controlados)**|Bien definido|Foto obligatoria, cifrada en Storage|
|**Modo offline**|Bien definido|Solo OTC, máximo 50 tokens cacheados|

---

## 3. Requerimientos Funcionales Generales del Sistema

Estos son los requerimientos que aplican al sistema completo como unidad.

### RF-GEN-001 — Sistema de Autenticación Unificado

El sistema debe gestionar dos tipos de usuarios con autenticación diferenciada: pacientes (app móvil, cédula/correo) y personal médico/admin (panel web, RUC). Ambos flujos deben producir JWT firmados con RS256 y soporte para MFA.

### RF-GEN-002 — Canal Seguro App ↔ Backend ↔ Expendedora

Toda comunicación debe ocurrir bajo TLS 1.3. La comunicación con la expendedora debe usar mTLS con certificado único por dispositivo sobre el broker MQTT (HiveMQ).

### RF-GEN-003 — Ciclo Completo de Receta Digital

El sistema debe soportar el ciclo completo: médico emite receta en panel web → backend la firma y la asocia al paciente → paciente la visualiza en app → paciente inicia compra → expendedora la dispensa → historial actualizado en todas las capas.

### RF-GEN-004 — Trazabilidad Integral e Inmutable

Toda dispensación debe quedar registrada en un audit log de solo INSERT. Para medicamentos controlados, el registro debe incluir identidad verificada, foto del retiro, cantidad, médico prescriptor y timestamp. Retención mínima: 5 años.

### RF-GEN-005 — Cumplimiento con Tribunal Electoral

El sistema debe integrarse con la API del Tribunal Electoral de Panamá para verificar cédulas. Esta verificación es obligatoria en el registro de pacientes y en compras con receta.

### RF-GEN-006 — Gestión de Catálogo con Restricciones Legales

El catálogo de medicamentos debe clasificar cada producto con: `requiere_receta`, `es_controlado`, `generico_equivalente_id`. Un medicamento controlado implica obligatoriamente `requiere_receta = TRUE`. La alternativa genérica debe mostrarse antes de confirmar compra de marca.

### RF-GEN-007 — Disponibilidad en Tiempo Real

El estado de las expendedoras (stock, operatividad, telemetría) debe reflejarse en tiempo real en todas las interfaces vía Supabase Realtime. Latencia máxima: 5 segundos desde el evento hasta la visualización.

### RF-GEN-008 — Modo Offline Degradado Seguro

Sin conexión MQTT por más de 60 segundos, la expendedora debe operar en modo degradado: solo OTC con QR pre-cacheados. Medicamentos Rx y controlados deben quedar bloqueados en firmware. Este comportamiento es no negociable.

### RF-GEN-009 — Gestión de Reembolsos Automáticos

Ante cualquier fallo en la cadena pago → dispensación, el sistema debe iniciar un reembolso automático con reintentos exponenciales. Si el reembolso no se completa en 24 horas, escala a cola manual. El dinero del paciente nunca se descarta silenciosamente.

### RF-GEN-010 — Separación de Roles y Datos

El sistema debe implementar RBAC en dos capas independientes (middleware backend + RLS en base de datos). Los roles son: Paciente, Médico, Técnico, Admin Farmacia, Regulador MINSA. Cada rol solo accede a los datos que su función legítima requiere.

---

## 4. Requerimientos Funcionales Específicos por Módulo

---

### 4.1 MÓDULO: Autenticación y Registro (App Móvil)

|ID|Requerimiento|Prioridad|Normativa|
|---|---|---|---|
|RF-AUTH-001|Registro con cédula panameña verificada contra API del Tribunal Electoral (≤ 5s)|Crítica|Ley 81, BR-M01|
|RF-AUTH-002|Consentimiento explícito y granular antes de activar la cuenta, con timestamp y versión registrados|Crítica|Ley 81 Art.5, BR-M02|
|RF-AUTH-003|Verificación de identidad mediante OTP de un solo uso (TTL 10 min, máx 3 intentos) enviado a correo o SMS|Alta|BR-T02|
|RF-AUTH-004|Generación de JWT RS256 con rol `patient`, `user_id` y expiración de 24h, almacenado en Expo SecureStore|Crítica|RS-006, RS-005|
|RF-AUTH-005|Bloqueo de cuenta por 15 minutos tras 5 intentos fallidos de login, con registro de IP en audit log|Alta|BR-T02|
|RF-AUTH-006|Recuperación de contraseña con token HMAC-SHA256 de un solo uso (TTL 30 min), sin confirmar si el email existe|Alta|Ley 81|
|RF-AUTH-007|Activación de MFA (TOTP o SMS OTP) obligatoria para compras de medicamentos controlados|Crítica|BR-M06, CSS|
|RF-AUTH-008|Refresh token silencioso antes de redirigir al login cuando el JWT expira|Media|UX + seguridad|

---

### 4.2 MÓDULO: Login Panel Web (Médico / Admin)

|ID|Requerimiento|Prioridad|Normativa|
|---|---|---|---|
|RF-WEB-LOGIN-001|Autenticación mediante RUC del médico y contraseña verificados contra base de datos|Alta|RF-001 Panel|
|RF-WEB-LOGIN-002|Segundo factor obligatorio (OTP por correo o SMS) tras validar contraseña correctamente|Alta|RF-002 Panel|
|RF-WEB-LOGIN-003|Bloqueo temporal de cuenta tras 5 intentos fallidos de inicio de sesión|Alta|RF-003 Panel|
|RF-WEB-LOGIN-004|Generación de JWT con rol específico (médico, técnico, admin) y tiempo de expiración definido|Alta|RF-004 Panel|

---

### 4.3 MÓDULO: Lista de Pacientes (Panel Web)

|ID|Requerimiento|Prioridad|Normativa|
|---|---|---|---|
|RF-PAC-001|Cargar y mostrar la lista de pacientes asignados al médico activo para el día en curso|Alta|RF-005 Panel|
|RF-PAC-002|Búsqueda en tiempo real por nombre completo o cédula sin recargar la página|Alta|RF-006 Panel|
|RF-PAC-003|Filtrar pacientes por estado: En espera, En consulta, Atendido, Urgencias|Alta|RF-007 Panel|
|RF-PAC-004|Al seleccionar un paciente, navegar al expediente completo con transición fluida|Media|RF-008 Panel|
|RF-PAC-005|Cada tarjeta de paciente muestra: nombre, edad, hora de cita, motivo de consulta y estado actual|Media|Distribución general|

---

### 4.4 MÓDULO: Panel del Paciente — Información General (Panel Web)

|ID|Requerimiento|Prioridad|Normativa|
|---|---|---|---|
|RF-INFO-001|Mostrar resumen del paciente: nombre, apellido, sexo, edad, motivo de consulta actual|Alta|RF-001 Panel paciente|
|RF-INFO-002|Mostrar datos sensibles del paciente: alergias, condiciones crónicas, tipo de sangre (solo para roles autorizados)|Alta|RF-002, Ley 81|
|RF-INFO-003|Mostrar historial clínico completo en lista cronológica con fecha, diagnóstico, tratamiento y médico|Alta|RF-003, RF-004|
|RF-INFO-004|Permitir seleccionar una consulta específica del historial para ver su detalle sin recargar la página|Media|RF-005|
|RF-INFO-005|Restringir visualización de datos sensibles exclusivamente a roles autorizados (médico tratante)|Crítica|RF-006, Ley 81, BR-A10|

---

### 4.5 MÓDULO: Nueva Consulta (Panel Web)

|ID|Requerimiento|Prioridad|Normativa|
|---|---|---|---|
|RF-CONS-001|Formulario de consulta con campos: motivo, síntomas, diagnóstico, tratamiento, notas adicionales|Alta|RF-001 consulta|
|RF-CONS-002|Validar campos obligatorios (motivo, diagnóstico, tratamiento) antes de permitir el guardado|Alta|RF-002 consulta|
|RF-CONS-003|Guardar la consulta de forma segura en la base de datos asociada al paciente y al médico activo|Alta|RF-003 consulta|
|RF-CONS-004|Opción de guardar consulta y generar receta electrónica en un mismo flujo continuo|Alta|RF-004 consulta|
|RF-CONS-005|Confirmación visual explícita tras registro exitoso de la consulta|Media|RF-005 consulta|
|RF-CONS-006|Permitir editar cualquier campo antes de confirmar el guardado definitivo|Media|RF-006 consulta|

---

### 4.6 MÓDULO: Receta Electrónica (Panel Web)

|ID|Requerimiento|Prioridad|Normativa|
|---|---|---|---|
|RF-RX-WEB-001|Editor de receta con campos: medicamento, dosis, frecuencia, duración del tratamiento, indicaciones adicionales|Alta|RF-002 receta|
|RF-RX-WEB-002|Validar que medicamento, dosis y frecuencia estén completos antes de generar la receta|Alta|RF-003 receta|
|RF-RX-WEB-003|El backend firma criptográficamente el contenido de la receta usando credenciales del médico|Crítica|BR-A02, RS-002|
|RF-RX-WEB-004|Verificar que el médico emisor tenga idoneidad activa antes de permitir la emisión|Crítica|BR-A01, Ley 1/2001|
|RF-RX-WEB-005|Vista previa de la receta antes del envío para validación por parte del médico|Media|RF-004 receta|
|RF-RX-WEB-006|Enviar la receta firmada digitalmente a la app del paciente asociado|Crítica|RF-005 receta|
|RF-RX-WEB-007|Confirmar visualmente cuando la receta haya sido enviada exitosamente|Media|RF-006 receta|
|RF-RX-WEB-008|Almacenar la receta generada en el historial clínico del paciente|Alta|RF-008 receta|

---

### 4.7 MÓDULO: Inicio / Home (App Móvil)

|ID|Requerimiento|Prioridad|Normativa|
|---|---|---|---|
|RF-HOME-001|Mostrar listado de máquinas expendedoras cercanas ordenadas por distancia GPS, con indicador de estado|Alta|RFI-001|
|RF-HOME-002|Barra de búsqueda rápida de medicamentos por nombre comercial o principio activo|Alta|RFI-002|
|RF-HOME-003|Mostrar tarjetas de recetas activas/parciales con badge de alerta si vencen en menos de 48h|Alta|RFI-005, RF-041|
|RF-HOME-004|Notificaciones: nueva receta emitida, receta próxima a vencer, stock disponible en máquina cercana|Alta|RFI-004, BR-M10|
|RF-HOME-005|Acceso rápido a última compra con botón de repetir pedido si el medicamento sigue disponible|Media|RFI-003|

---

### 4.8 MÓDULO: Búsqueda de Medicamentos (App Móvil)

|ID|Requerimiento|Prioridad|Normativa|
|---|---|---|---|
|RF-BUSQ-001|Buscar por nombre comercial, principio activo o categoría terapéutica con debounce de 300ms|Alta|RF-006|
|RF-BUSQ-002|Mostrar en cada resultado: nombre, badge (Venta Libre / Requiere Receta), precio, cantidad de máquinas cercanas con stock|Alta|RF-007|
|RF-BUSQ-003|Filtros: Venta Libre / Con Receta / Disponible ahora / Por distancia|Alta|RF-008|
|RF-BUSQ-004|Mostrar alternativa genérica equivalente debajo de cada medicamento de marca (Ley 83/2012)|Alta|RF-009, BR-M04|
|RF-BUSQ-005|Mostrar alerta visible si el usuario tiene alergia registrada relacionada con el principio activo|Alta|RF-013, BR-M11|
|RF-BUSQ-006|Para medicamentos Rx, verificar si el usuario tiene recetas activas para ese medicamento antes de habilitar compra|Crítica|RF-022, BR-M07|
|RF-BUSQ-007|Solo mostrar máquinas con stock > 0 del medicamento buscado; ocultar o atenuar las demás|Media|BR-M14|

---

### 4.9 MÓDULO: Mapa de Expendedoras (App Móvil)

|ID|Requerimiento|Prioridad|Normativa|
|---|---|---|---|
|RF-MAPA-001|Mapa interactivo centrado en la ubicación GPS del usuario con marcadores de expendedoras|Alta|RF-01, RF-02|
|RF-MAPA-002|Marcadores con código de color: Verde (Activa), Rojo (Inactiva), Naranja (Mantenimiento)|Alta|RF-04|
|RF-MAPA-003|Al tocar un marcador, mostrar tarjeta con: nombre, dirección, estado, distancia, última conexión e inventario|Alta|RF-034, RF-035|
|RF-MAPA-004|Botón "Cómo llegar" que abre la app de navegación nativa con coordenadas de la máquina|Media|RF-036|
|RF-MAPA-005|Botón "Comprar aquí" habilitado solo si la máquina está Activa y tiene stock. Guarda `machine_id` en state global|Crítica|RF-037, BR-L08|
|RF-MAPA-006|Filtros: por provincia, por estado operativo, por favoritas|Media|RF-06|
|RF-MAPA-007|Mostrar alerta en tarjeta si la telemetría no se actualizó en más de 15 minutos|Media|RF-034|
|RF-MAPA-008|En modo offline degradado, mostrar banner de advertencia y bloquear Rx/controlados en inventario|Crítica|RF-038, BR-I04|

---

### 4.10 MÓDULO: Compra / Checkout (App Móvil + Expendedora)

|ID|Requerimiento|Prioridad|Normativa|
|---|---|---|---|
|RF-COMP-001|Pantalla de resumen obligatoria antes del pago: medicamento, máquina, cantidad y precio total fijado|Alta|RF-09, BR-L06|
|RF-COMP-002|Mostrar sugerencia de genérico equivalente con comparativo de precio antes de confirmar pago de marca|Crítica|RF-09, Ley 83, BR-M04|
|RF-COMP-003|Iniciar pago con Yappy o tarjeta tokenizada. Timeout de respuesta: 10 segundos|Alta|RF-015, BR-M12|
|RF-COMP-004|Generar QR con UUID v4 firmado HMAC-SHA256 solo tras confirmación de pago. TTL: 15 minutos|Crítica|RF-017, BR-I01|
|RF-COMP-005|Mostrar QR con temporizador de cuenta regresiva y nombre de la máquina destino|Alta|RF-017|
|RF-COMP-006|Para compras Rx: verificar identidad contra Tribunal Electoral antes del pago|Crítica|RF-020, BR-M05|
|RF-COMP-007|Para medicamentos controlados: MFA obligatorio (TOTP o SMS), límite 1 receta/día/paciente, foto en retiro|Crítica|RF-021, BR-M06, BR-M13|
|RF-COMP-008|La máquina envía el token al backend para verificación; el backend envía comando MQTT firmado para dispensar|Crítica|BR-I02|
|RF-COMP-009|Tras dispensación exitosa (ACK del firmware), mostrar pantalla de confirmación y actualizar stock en tiempo real|Alta|RF-018, BR-I06|
|RF-COMP-010|Si el QR expira sin escanear, iniciar reembolso automático con reintentos y escalar si supera 24h|Crítica|RF-019, BR-L09|
|RF-COMP-011|Reserva lógica de stock al generar el token; liberación si el token expira o falla|Alta|BR-L05|
|RF-COMP-012|Idempotency key único por operación de pago para prevenir doble cobro|Crítica|BR-L12|

---

### 4.11 MÓDULO: Recetas (App Móvil)

|ID|Requerimiento|Prioridad|Normativa|
|---|---|---|---|
|RF-REC-001|Listado de recetas clasificadas por estado: Activas, Parciales, Agotadas, Vencidas, con contador por tab|Alta|RF-039|
|RF-REC-002|Cada tarjeta muestra: medicamento, dosis, médico emisor, fecha de vencimiento, unidades disponibles/prescritas|Alta|RF-040|
|RF-REC-003|Badge de alerta en recetas que vencen en menos de 48 horas|Alta|RF-041, BR-L10|
|RF-REC-004|Búsqueda por nombre de medicamento dentro del listado en tiempo real|Media|RF-042|
|RF-REC-005|Detalle completo: principio activo, dosis, frecuencia, cantidad prescrita/dispensada/restante|Alta|RF-043|
|RF-REC-006|Mostrar información del médico emisor: nombre, especialidad, número de idoneidad|Alta|RF-044|
|RF-REC-007|Historial de dispensaciones: fecha, máquina utilizada, cantidad dispensada en cada uso|Alta|RF-045|
|RF-REC-008|Botón "Usar esta receta" visible solo si la receta está activa o parcial Y hay stock disponible en máquina cercana|Crítica|RF-046, BR-M07|
|RF-REC-009|Mostrar QR de presentación para farmacia física (modo solo visual, sin valor para la expendedora)|Media|RF-047|

---

### 4.12 MÓDULO: Perfil (App Móvil)

|ID|Requerimiento|Prioridad|Normativa|
|---|---|---|---|
|RF-PERF-001|Mostrar: foto de perfil, nombre completo, cédula, correo y estado de verificación de identidad|Alta|RF-048|
|RF-PERF-002|Accesos directos a: historial de compras, mis recetas, métodos de pago, configuración, cerrar sesión|Alta|RF-049|
|RF-PERF-003|Indicar verificación pendiente con CTA para completarla; sin verificación no se puede comprar Rx|Alta|RF-050, BR-M01|
|RF-PERF-004|Editar foto de perfil, correo y teléfono. La cédula NO es editable una vez verificada|Alta|RF-051|
|RF-PERF-005|Registrar y editar alergias y condiciones crónicas. Aviso explícito de uso solo local para alertas|Alta|RF-053, RF-055, Ley 81|
|RF-PERF-006|Listar métodos de pago vinculados (Yappy y tarjetas enmascaradas) con opción de desvincular|Alta|RF-056, RF-057|
|RF-PERF-007|Gestionar notificaciones push por tipo y activar/desactivar MFA para controlados|Alta|RF-058, RF-059|
|RF-PERF-008|Cambiar contraseña con validación de la actual y requisitos de complejidad|Alta|RF-060|
|RF-PERF-009|Eliminar cuenta con flujo de confirmación en 2 pasos, soft delete en PII y conservación de audit log anonimizado|Crítica|RF-061, Ley 81 Art.22, BR-M09|

---

## 5. Flujo Básico General del Sistema

Este flujo describe el ciclo completo desde que el médico emite una receta hasta que el paciente retira el medicamento.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       CICLO COMPLETO PHARMABOT                             │
└─────────────────────────────────────────────────────────────────────────────┘

[MÉDICO — PANEL WEB]
        │
        ▼
1.  Médico inicia sesión con RUC + contraseña + OTP
        │
        ▼
2.  Accede a la lista de pacientes del día
        │
        ▼
3.  Selecciona un paciente → visualiza expediente completo
        │
        ▼
4.  Registra nueva consulta (motivo, síntomas, diagnóstico, tratamiento)
        │
        ▼
5.  Presiona "Guardar y generar receta electrónica"
        │
        ▼
6.  Completa el editor de receta (medicamento, dosis, frecuencia, duración)
        │
        ▼
7.  Backend verifica idoneidad activa del médico
        │
        ▼
8.  Backend firma criptográficamente la receta (HMAC con credenciales del médico)
        │
        ▼
9.  Receta almacenada en BD → asociada al user_id del paciente → estado: ACTIVE
        │
        ▼
10. Notificación push genérica enviada al paciente ("Tienes una nueva receta disponible")

─────────────────────────────────────────────────────────────────────────────

[PACIENTE — APP MÓVIL]
        │
        ▼
11. Paciente recibe notificación → abre la app con JWT válido
        │
        ▼
12. En Home visualiza badge de receta activa nueva
        │
        ▼
13. Accede al módulo de Recetas → ve la receta en estado ACTIVE
        │
        ▼
14. Busca el medicamento en el mapa o en la barra de búsqueda
        │
        ▼
15. Selecciona una expendedora activa con stock > 0
        │
        ▼
        ─ ─ ─ ─ ─ ─ ─ ─ BIFURCACIÓN POR TIPO ─ ─ ─ ─ ─ ─ ─ ─
        │                    │                    │
   [OTC - Sin receta]  [Rx - Con receta]  [Controlado]
        │                    │                    │
   Continúa al           Verificación       Verificación
   paso de pago          de identidad       de identidad
                         (Tribunal          (Tribunal
                          Electoral)         Electoral)
                              │                    │
                              ▼                    ▼
                         Selecciona         MFA obligatorio
                         receta activa      (TOTP / SMS)
                              │                    │
                              └──────────┬──────────┘
                                         │
        ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
        │
        ▼
16. Pantalla de resumen: medicamento, máquina, cantidad, precio fijado
        │
        ▼
17. [Si medicamento de marca] Sistema muestra alternativa genérica (Ley 83)
        │
        ▼
18. Usuario confirma y elige método de pago (Yappy / Tarjeta tokenizada)
        │
        ▼
19. Backend crea orden en estado CREATED → reserva stock lógicamente
        │
        ▼
20. Gateway Yappy procesa el pago (timeout: 10s)
        │
        ▼
21. Pago confirmado → Orden pasa PAID → TOKEN_GENERATED
        │
        ▼
22. Backend genera QR con UUID v4 firmado HMAC-SHA256 (TTL: 15 min)
        │
        ▼
23. Usuario visualiza QR con temporizador de cuenta regresiva

─────────────────────────────────────────────────────────────────────────────

[EXPENDEDORA FÍSICA — IoT]
        │
        ▼
24. Usuario se acerca a la máquina y escanea el QR
        │
        ▼
25. Firmware lee el QR → extrae el UUID → envía al backend vía MQTT
        │
        ▼
26. Backend verifica: (a) firma HMAC válida, (b) estado PENDING, (c) no usado,
    (d) machine_id correcto, (e) slot asignado correcto
        │
        ▼
27. Backend envía comando MQTT firmado a la máquina (mTLS)
        │
        ▼
28. [Si controlado] Cámara captura foto del usuario al momento del retiro
        │
        ▼
29. Firmware ejecuta dispensación del slot → confirma ACK vía MQTT
        │
        ▼
30. Backend recibe ACK → Orden: DISPENSING → COMPLETED
    Stock: reservado → dispensado (decremento permanente)
    Receta: cantidad_dispensada += cantidad_retirada
    Estado receta: ACTIVE / PARTIAL / EXHAUSTED según saldo
        │
        ▼
31. Supabase Realtime actualiza stock en mapa y búsqueda en tiempo real
        │
        ▼
32. App del paciente muestra pantalla de confirmación de retiro exitoso
        │
        ▼
33. Audit log registra la dispensación completa (INSERT-only, retención 5 años)
```

---

## 6. Flujos Específicos con Flujos Alternos, Excepciones y Seguridad

---

### 6.1 Flujo: Registro de Paciente (App Móvil)

**Precondiciones:**

- El usuario tiene cédula panameña o pasaporte válido
- La API del Tribunal Electoral está disponible
- El usuario dispone de correo electrónico o teléfono activo

#### Flujo Básico

|Paso|Actor|Descripción|Control de Seguridad|
|---|---|---|---|
|1|Usuario|Descarga la app e inicia el proceso de registro|TLS 1.3 en todas las comunicaciones|
|2|Usuario|Ingresa cédula (o pasaporte), nombre, correo y contraseña|Validación Zod en cliente y servidor (BR-T05)|
|3|Backend|Verifica la cédula contra la API del Tribunal Electoral (≤ 5s)|Rate limiting: 5 intentos/IP/min (BR-T02)|
|4|Backend|Valida que la cédula no esté ya registrada (UNIQUE NOT NULL)|RLS Supabase activo|
|5|Sistema|Muestra pantalla de consentimiento explícito y granular de datos|Timestamp + versión del aviso registrados (Ley 81 Art.5)|
|6|Backend|Envía código OTP al correo o SMS del usuario|OTP de un solo uso, TTL 10 min|
|7|Usuario|Ingresa el código OTP recibido|Máximo 3 intentos antes de re-envío obligatorio|
|8|Backend|Valida OTP. Crea la cuenta con estado `pending_verification`|Contraseña almacenada con bcrypt cost 12|
|9|Sistema|Redirige a pantalla principal con estado de verificación visible|JWT generado con rol: patient, expiración 24h|

#### Flujos Alternos

**FA-REG-01: API del Tribunal Electoral no disponible**

1. Backend no puede verificar la cédula en tiempo real
2. Cuenta creada en estado `pending_verification` limitado (solo OTC, sin Rx)
3. Sistema programa reintento de verificación cuando el servicio se restablezca
4. Usuario informado: "Verificación en proceso, te notificaremos"

**FA-REG-02: Usuario ingresa pasaporte (extranjero)**

1. El sistema acepta pasaporte como documento alternativo
2. La verificación se hace contra el registro de extranjeros del Tribunal Electoral
3. El flujo continúa igual desde el paso 5

#### Flujos de Excepción

|Código|Condición|Acción|Mensaje|Severidad|
|---|---|---|---|---|
|FE-REG-01|Cédula no encontrada en Tribunal Electoral|Bloquea el registro. Instrucción de contactar soporte|"Cédula no encontrada en registros oficiales"|Crítica|
|FE-REG-02|Cédula ya registrada en el sistema|Rechaza. Ofrece recuperar contraseña|"Esta cédula ya tiene una cuenta"|Alta|
|FE-REG-03|OTP incorrecto (< 3 intentos)|Permite reintento. Muestra intentos restantes|"Código incorrecto. Intentos restantes: X"|Media|
|FE-REG-04|OTP expirado o 3 intentos fallidos|Invalida OTP. Habilita re-envío. Espera 60s|"Código expirado. Solicita uno nuevo"|Media|
|FE-REG-05|Contraseña no cumple requisitos|Bloquea avance. Muestra requisitos en tiempo real|"La contraseña no cumple los requisitos"|Media|

#### Controles de Seguridad

|Control|Descripción|Normativa|
|---|---|---|
|**JWT RS256**|Token firmado asimétricamente, expiración 24h|Ley 81 / RS-006|
|**bcrypt cost 12**|Hash lento para prevenir fuerza bruta offline|Best practice|
|**Rate limiting**|5 intentos/IP/10min. Bloqueo 15 min al exceder|RS-003, BR-T02|
|**OTP un solo uso**|Código 6 dígitos, TTL 10 min, invalidado tras uso|BR-M06|
|**Secure Storage**|JWT nunca en AsyncStorage plano. Expo SecureStore|RS-005|
|**Anti-enumeration**|Recovery no revela si el email/cédula existe|Ley 81|
|**Consentimiento ROPA**|Timestamp y versión del aviso aceptado registrados|Ley 81 Art.5|

---

### 6.2 Flujo: Emisión de Receta Electrónica (Panel Web)

**Precondiciones:**

- Médico autenticado con JWT válido y MFA completado
- Médico tiene idoneidad activa verificada en el sistema
- Paciente registrado y vinculado al médico
- Nueva consulta guardada previamente (o flujo de guardar y generar en un paso)

#### Flujo Básico

|Paso|Actor|Descripción|Control de Seguridad|
|---|---|---|---|
|1|Médico|Desde el panel del paciente presiona "Nueva consulta"|JWT verificado en cada request (BR-T07)|
|2|Médico|Completa el formulario de consulta (motivo, diagnóstico, tratamiento)|Validación de campos obligatorios en cliente y servidor|
|3|Médico|Presiona "Guardar y generar receta"|—|
|4|Sistema|Abre el editor de receta pre-vinculado a la consulta y al paciente|—|
|5|Médico|Ingresa: medicamento, dosis, frecuencia, duración, indicaciones adicionales|Validación Zod|
|6|Médico|Presiona "Vista previa" para revisar la receta antes de enviar|—|
|7|Médico|Confirma y presiona "Generar y enviar receta"|—|
|8|Backend|Verifica idoneidad activa del médico en este momento|Query a campo `status` del médico (BR-A01)|
|9|Backend|Firma criptográficamente el contenido completo de la receta|RS-002 / BR-A02|
|10|Backend|Almacena la receta en BD con estado ACTIVE y la asocia al `user_id` del paciente|RLS: solo el paciente puede leer sus recetas (BR-M08)|
|11|Backend|Envía notificación push genérica al dispositivo del paciente|Sin datos médicos en el push (BR-M10)|
|12|Sistema|Confirma al médico que la receta fue enviada exitosamente|—|
|13|Sistema|Marca la consulta como completada y actualiza el historial del paciente|Audit log registra el evento|

#### Flujos Alternos

**FA-RX-01: Medicamento controlado seleccionado**

1. Sistema detecta `es_controlado = TRUE`
2. Sistema verifica automáticamente que `requiere_receta = TRUE` (constraint PostgreSQL)
3. Muestra advertencia al médico indicando las restricciones adicionales para el paciente (MFA, foto, límite diario)
4. El flujo continúa normalmente; las restricciones se aplican en el lado del paciente

**FA-RX-02: Medicamento de marca con genérico disponible**

1. Sistema detecta `generico_equivalente_id` en la tabla de medicamentos
2. Muestra sugerencia al médico con el equivalente genérico y comparativo de precio
3. El médico puede optar por prescribir el genérico o mantener la marca innovadora
4. La decisión queda registrada

**FA-RX-03: Médico prescribe medicamento sin genérico registrado**

1. El sistema acepta la receta normalmente
2. Si `generico_equivalente_id` es NULL, el sistema lo registra justificado
3. No bloquea la emisión de la receta

#### Flujos de Excepción

|Código|Condición|Acción|Mensaje|Severidad|
|---|---|---|---|---|
|FE-RX-WEB-01|Campos obligatorios vacíos al generar|Resalta campos. No envía al backend|"Complete los campos obligatorios"|Media|
|FE-RX-WEB-02|Médico con idoneidad suspendida al momento de generar|Bloquea la emisión. Registra el intento|"No tienes permisos para emitir recetas"|Crítica|
|FE-RX-WEB-03|Error al enviar la receta al backend|Conserva los datos. Permite reintentar|"No se pudo enviar la receta"|Alta|
|FE-RX-WEB-04|Paciente no encontrado o cuenta eliminada|Bloquea la receta. Alerta al médico|"El paciente no está disponible"|Alta|
|FE-RX-WEB-05|Error del servidor al guardar|Muestra error. No guarda parcialmente|"Error interno, intente más tarde"|Alta|
|FE-RX-WEB-06|Sesión expirada durante la emisión|Intenta preservar datos. Redirige al login|"Sesión expirada"|Media|

#### Controles de Seguridad

|Control|Descripción|Normativa|
|---|---|---|
|**Firma digital de receta**|Backend firma al emitir usando credenciales del médico. El cliente nunca firma|BR-A02, RS-002|
|**Verificación de idoneidad en cada emisión**|El estado del médico se verifica en cada solicitud, no solo al login|BR-A01, Ley 1/2001|
|**RLS por médico**|El médico solo puede emitir recetas para sus propios pacientes|BR-A03, Ley 81|
|**Push sin PII**|La notificación al paciente no menciona el medicamento|BR-M10, Ley 81|
|**Audit log**|Toda emisión de receta queda en el audit log inmutable|CSS, BR-A07|
|**TLS 1.3**|Panel web ↔ backend siempre bajo TLS 1.3|BR-T04|

---

### 6.3 Flujo: Visualización de Recetas (App Móvil)

**Precondiciones:**

- Usuario autenticado con JWT válido
- El médico emisor tiene idoneidad activa
- La receta fue firmada criptográficamente al emitirse

#### Flujo Básico

|Paso|Actor|Descripción|Control de Seguridad|
|---|---|---|---|
|1|Sistema|Carga el listado de recetas del usuario clasificadas por estado|RLS: solo recetas del usuario autenticado (BR-M08)|
|2|Sistema|Muestra tabs: Activas, Parciales, Agotadas, Vencidas con contador en cada tab|—|
|3|Sistema|Muestra badge de alerta en recetas que vencen en menos de 48h|RF-041, BR-L10|
|4|Usuario|Toca una tarjeta para ver el detalle completo|—|
|5|Sistema|Muestra: principio activo, dosis, frecuencia, cantidades prescritas/dispensadas/restantes|RF-043|
|6|Sistema|Muestra información del médico: nombre, especialidad, número de idoneidad|RF-044|
|7|Sistema|Muestra historial de dispensaciones: fecha, máquina, cantidad por cada uso|RF-045|
|8|Sistema|Muestra botón "Usar esta receta" solo si: estado activa/parcial AND stock disponible en máquina cercana|RF-046, BR-M07|
|9|Usuario|Presiona "Usar esta receta" → inicia flujo de compra Rx|Flujo 6.4|
|10|Sistema|Opcionalmente, muestra QR de presentación para farmacia física (modo visual)|RF-047|

#### Flujos Alternos

**FA-REC-MOV-01: Búsqueda por nombre de medicamento**

1. Usuario escribe en la barra de búsqueda del módulo
2. Sistema filtra en tiempo real sobre los datos ya cargados (sin nueva llamada al backend)
3. Lista se actualiza instantáneamente

**FA-REC-MOV-02: Receta suspendida por médico desactivado**

1. Admin desactiva al médico en el panel
2. Backend cambia el estado de todas sus recetas activas/parciales a `SUSPENDED_BY_DOCTOR`
3. Paciente ve la receta en estado "Suspendida" con explicación genérica
4. Botón "Usar" queda deshabilitado
5. Paciente recibe notificación push genérica

#### Flujos de Excepción

|Código|Condición|Acción|Mensaje|Severidad|
|---|---|---|---|---|
|FE-REC-MOV-01|Receta expirada detectada en carga|Marca como EXPIRED. Deshabilita "Usar"|"Esta receta ha vencido"|Media|
|FE-REC-MOV-02|Receta agotada|Marca como EXHAUSTED. Deshabilita "Usar"|"Esta receta está agotada"|Media|
|FE-REC-MOV-03|Firma digital de receta inválida|Rechaza el uso. No permite dispensación|"Receta inválida o modificada"|Crítica|
|FE-REC-MOV-04|Médico con idoneidad suspendida al intentar usar|Bloquea el uso aunque la receta esté vigente|"Receta no disponible"|Crítica|
|FE-REC-MOV-05|Sin máquinas cercanas con stock|Deshabilita "Usar". Muestra mapa con opciones lejanas|"No hay máquinas con stock disponible"|Alta|
|FE-REC-MOV-06|Error al cargar historial de dispensaciones|Oculta esa sección. Muestra el resto normalmente|"No se pudo cargar el historial"|Baja|

#### Controles de Seguridad

|Control|Descripción|Normativa|
|---|---|---|
|**RLS Supabase**|Solo se cargan las recetas del `user_id` autenticado|BR-M08, Ley 81|
|**Verificación de firma en cada uso**|Firma digital verificada cada vez que se intenta usar la receta|BR-A02|
|**Verificación de idoneidad en cada solicitud**|El estado del médico se consulta en cada intento de compra, no solo al recibir la receta|BR-A01|
|**QR presentación ≠ QR dispensación**|El QR visual para farmacia física no tiene valor en la expendedora|RF-047|
|**Notificaciones sin PII**|Push de suspensión no menciona el medicamento|BR-M10, Ley 81|

---

### 6.4 Flujo: Compra con Receta — Rx (App Móvil + Expendedora)

**Precondiciones:**

- Usuario autenticado con JWT válido y método de pago vinculado
- Máquina seleccionada en estado Activa con stock > 0
- Usuario tiene receta activa o parcial vigente para el medicamento
- Fecha de vencimiento de la receta es posterior a NOW()

#### Flujo Básico

|Paso|Actor|Descripción|Control de Seguridad|
|---|---|---|---|
|1|Backend|Verifica identidad del usuario contra API Tribunal Electoral antes del pago|BR-M05 — obligatorio, sin excepción|
|2|Sistema|Muestra listado de recetas activas filtradas por el medicamento|RF-022|
|3|Usuario|Selecciona la receta y la cantidad (máx: saldo disponible)|—|
|4|Backend|Calcula saldo_disponible = cantidad_prescrita - cantidad_dispensada en transacción atómica|PostgreSQL transacción para evitar condición de carrera (BR-L04)|
|5|Sistema|Muestra pantalla de resumen con precio fijado en este momento|BR-L06|
|6|Sistema|Si hay genérico equivalente disponible, muestra popup comparativo|Ley 83/2012, BR-M04|
|7|Usuario|Confirma y presiona "Confirmar y Pagar"|—|
|8|Usuario|Selecciona método de pago|Datos de tarjeta enmascarados en pantalla|
|9|Backend|Crea orden CREATED → reserva stock lógicamente → inicia transacción Yappy|BR-L05: stock_reservado++|
|10|Yappy|Procesa el pago. Timeout: 10 segundos sin respuesta → PAYMENT_TIMEOUT|BR-M12|
|11|Backend|Recibe confirmación → Orden PAID → TOKEN_GENERATED → genera QR HMAC-SHA256|BR-I01: TTL 15 min, un solo uso|
|12|Sistema|Muestra QR con temporizador de cuenta regresiva|RF-017|
|13|Usuario|Se dirige a la máquina y escanea el QR|—|
|14|Máquina|Lee el QR → envía token al backend para verificación|BR-I02|
|15|Backend|Verifica: firma HMAC, estado PENDING, no usado, machine_id correcto|mTLS entre backend y broker (BR-I11)|
|16|Backend|Envía comando MQTT firmado a la máquina|—|
|17|Máquina|Recibe comando → dispensa medicamento → envía ACK vía MQTT|BR-I10: dispensación atómica|
|18|Backend|Recibe ACK → Orden COMPLETED → actualiza stock → actualiza cantidad_dispensada en receta|BR-I06, BR-L03|
|19|Sistema|Muestra pantalla de confirmación de retiro exitoso|RF-018|
|20|Backend|Registra dispensación en audit log inmutable|CSS, BR-A07|

#### Flujos Alternos

**FA-COMP-RX-01: QR escaneado en máquina incorrecta**

1. Máquina B escanea el QR generado para Máquina A
2. Backend verifica `machine_id` del token vs `machine_id` de la máquina que escanea
3. Backend rechaza el comando → token permanece en estado PENDING
4. Usuario puede ir a la máquina correcta dentro del TTL restante

**FA-COMP-RX-02: Pago con tarjeta guardada**

1. Usuario selecciona tarjeta guardada (mostrada enmascarada: `**** **** **** 1234`)
2. Backend usa el token PCI-DSS de la tarjeta para iniciar el cobro
3. El flujo continúa igual desde el paso 11

**FA-COMP-RX-03: Usuario sin MFA configurado intentando comprar controlado**

1. Sistema detecta `es_controlado = TRUE` y que el usuario no tiene MFA activo
2. Interrumpe el flujo de compra antes del pago
3. Guía al usuario para configurar MFA (TOTP o SMS)
4. Tras configurar exitosamente, regresa al punto de interrupción

#### Flujos de Excepción

|Código|Condición|Acción|Mensaje|Severidad|
|---|---|---|---|---|
|FE-COMP-01|Stock a 0 entre selección y confirmación|Bloquea compra. Libera reserva. Busca otra máquina|"Stock agotado, elige otra máquina"|Crítica|
|FE-COMP-02|Pago rechazado por Yappy|Muestra error. Opción de reintentar o cambiar método. No genera QR|"Pago rechazado, verifica tus datos"|Alta|
|FE-COMP-03|Timeout de pago > 10s|Orden → PAYMENT_TIMEOUT. Libera stock. Permite reintentar|"Tiempo de espera agotado"|Alta|
|FE-COMP-04|QR expirado (TTL 15 min sin escanear)|Token → EXPIRED. Reembolso automático iniciado con backoff|"QR expirado — reembolso iniciado"|Crítica|
|FE-COMP-05|Fallo mecánico al dispensar|Token → FAILED. Reembolso automático. Audit log del fallo|"Error en dispensación — reembolso iniciado"|Crítica|
|FE-COMP-06|Límite diario de controlados alcanzado|Bloquea completamente. Sin bypass posible|"Límite diario de controlados alcanzado"|Crítica|
|FE-COMP-07|Verificación Tribunal Electoral no disponible|Bloquea compras Rx y controlados. Solo permite OTC|"Verificación de identidad no disponible"|Crítica|
|FE-COMP-08|MFA incorrecto para controlado (< 3 intentos)|Permite reintento. Muestra intentos restantes|"Código incorrecto. Intentos restantes: X"|Alta|
|FE-COMP-09|Cámara de máquina no operativa (controlado)|Bloquea dispensación hasta restaurar cámara|"Dispensación no disponible en esta máquina"|Crítica|
|FE-COMP-10|Reembolso automático falla > 24h|Escala a cola manual. Alerta al admin. Nunca descartado|Admin recibe alerta crítica|Crítica|
|FE-COMP-11|Saldo de receta insuficiente para la cantidad solicitada|Rechaza. Muestra saldo disponible real|"La cantidad supera el saldo de tu receta"|Alta|

#### Controles de Seguridad

|Control|Descripción|Normativa|
|---|---|---|
|**HMAC-SHA256 en QR**|QR firmado con clave HMAC única por máquina. Verificación en backend|BR-I01, RS-001|
|**mTLS backend-broker**|Certificado TLS único por máquina. Rotación cada 90 días|BR-I11, RS-005|
|**Saga pattern**|Pago→token→dispensación→stock como unidad atómica con compensaciones|BR-I10, RNF-010|
|**Lock distribuido**|Un token solo puede estar en DISPENSING en una máquina a la vez|BR-L02, Redis|
|**Stock doble verificación**|Verificación en UI (selección) y en backend (creación de orden)|BR-I06, BR-L05|
|**Audit log inmutable**|Solo INSERT en schema audit. Ningún rol puede hacer UPDATE/DELETE|CSS, BR-A07|
|**Idempotency key**|Cada operación de pago tiene `idempotency_key` único del cliente|BR-L12|
|**Precio fijado al crear orden**|El precio no se recalcula al dispensar|BR-L06|
|**Verificación de identidad Tribunal Electoral**|Obligatoria para Rx, no bypasseable|BR-M05|
|**Foto del retiro (controlados)**|Cámara de la máquina captura foto cifrada en Storage|BR-I03, CSS|

---

### 6.5 Flujo: Compra OTC — Sin Receta (App Móvil + Expendedora)

**Precondiciones:**

- Usuario autenticado con JWT válido y método de pago vinculado
- Máquina seleccionada en estado Activa con stock > 0
- Medicamento clasificado como OTC (`requiere_receta = FALSE`)

#### Flujo Básico

|Paso|Actor|Descripción|Control de Seguridad|
|---|---|---|---|
|1|Usuario|Selecciona cantidad. Límite = min(stock disponible, límite_diario_OTC)|Stock reservado lógicamente al seleccionar (BR-L05)|
|2|Sistema|Si hay genérico equivalente en la misma máquina, muestra popup comparativo|Ley 83/2012, BR-M04 — registro de visualización|
|3|Sistema|Muestra resumen: medicamento, máquina, cantidad, precio total fijado|BR-L06|
|4|Usuario|Confirma y presiona "Confirmar y Pagar"|—|
|5|Usuario|Selecciona método de pago|Datos enmascarados en pantalla|
|6|Backend|Crea orden CREATED → reserva stock → inicia transacción Yappy|idempotency_key único (BR-L12)|
|7|Yappy|Procesa el pago. Timeout: 10 segundos|BR-M12|
|8|Backend|Confirmación de pago → Orden PAID → TOKEN_GENERATED → genera QR|TTL 15 min, HMAC-SHA256 (BR-I01)|
|9|Sistema|Muestra QR con temporizador|RF-017|
|10|Usuario|Escanea QR en la máquina|—|
|11–19|Sistema|Flujo idéntico a la compra Rx desde el paso 14 al 20|Mismos controles de seguridad IoT|

> Las excepciones y controles de seguridad son idénticos a los del flujo Rx (sección 6.4), con la excepción de que **no se requiere verificación de identidad con el Tribunal Electoral ni MFA**.

---

### 6.6 Flujo: Gestión de Perfil y Cuenta (App Móvil)

**Precondiciones:**

- Usuario autenticado con JWT válido

#### Flujo Básico

|Paso|Actor|Descripción|Control de Seguridad|
|---|---|---|---|
|1|Sistema|Carga info del usuario: foto, nombre, cédula (bloqueada si verificada), correo, estado de verificación|RLS: user_id = auth.uid()|
|2|Sistema|Muestra accesos directos: historial, recetas, métodos de pago, configuración, cerrar sesión|—|
|3|Usuario|Edita foto, correo o teléfono|La cédula es inmutable post-verificación (BR-M01)|
|4|Usuario|Accede a Perfil Médico. Registra alergias y condiciones crónicas|Aviso explícito: solo uso local para alertas (RF-055)|
|5|Usuario|Accede a Métodos de Pago. Ve tarjetas enmascaradas y Yappy vinculado|Tokenización PCI-DSS|
|6|Usuario|Desvincula un método de pago|Si es el único, confirmación explícita requerida|
|7|Usuario|Gestiona notificaciones por tipo y configura/desactiva MFA|MFA obligatorio si tiene historial de controlados|
|8|Usuario|Cambia contraseña con validación de la actual y requisitos de complejidad|Contraseña actual requerida, hash bcrypt|

#### Flujos Alternos

**FA-PERF-01: Eliminación de cuenta**

1. Usuario presiona "Eliminar mi cuenta" en Configuración
2. Sistema muestra pantalla con consecuencias claras (flujo de confirmación en 2 pasos)
3. Usuario confirma la eliminación
4. Backend aplica soft delete en datos personales (nombre, correo, foto, teléfono)
5. Backend conserva audit log con identidad anonimizada (SHA-256 con salt). Retención: 5 años
6. Todos los JWT activos son revocados. Sistema redirige al onboarding

**FA-PERF-02: Verificación de identidad pendiente**

1. Sistema detecta que el usuario no ha completado la verificación de cédula
2. Muestra banner prominente con CTA "Verificar identidad"
3. Usuario inicia verificación → ingresa cédula → validación Tribunal Electoral
4. Sin verificación: no puede comprar medicamentos Rx

#### Flujos de Excepción

|Código|Condición|Acción|Mensaje|Severidad|
|---|---|---|---|---|
|FE-PERF-01|Error al guardar cambios de perfil|Conserva datos previos. Permite reintentar|"No se pudieron guardar los cambios"|Media|
|FE-PERF-02|Intento de editar la cédula verificada|Campo bloqueado en UI y en backend|"La cédula no es editable una vez verificada"|Alta|
|FE-PERF-03|Error al desvincular método de pago|Conserva el método. Permite reintentar|"No se pudo desvincular el método"|Media|
|FE-PERF-04|Intentar eliminar el único método de pago|Confirmación especial. Advierte sobre impacto|"¿Seguro? No tendrás método de pago activo"|Media|
|FE-PERF-05|Contraseña actual incorrecta al cambiar|Rechaza. Registra intento fallido|"Contraseña actual incorrecta"|Alta|
|FE-PERF-06|Fallo en eliminación de cuenta (rollback)|Restaura estado anterior. No aplica soft delete parcial|"No se pudo eliminar la cuenta"|Crítica|

#### Controles de Seguridad

|Control|Descripción|Normativa|
|---|---|---|
|**Soft delete + anonimización**|Eliminar cuenta: PII removido, audit log con SHA-256 salt|Ley 81 Art.22, BR-M09|
|**Cédula inmutable post-verificación**|No editable ni en cliente ni en backend|BR-M01|
|**Admin sin acceso a perfil médico**|Admin solo ve estadísticas agregadas. No lee datos médicos individuales|Ley 81, BR-A10|
|**Datos salud solo locales**|Alergias y condiciones: alertas locales únicamente, no compartidas|RF-055, Ley 81|
|**JWT revocación al eliminar**|Todos los JWT activos son revocados al eliminar la cuenta|RS-006|

---

## 7. Resumen de Vacíos Identificados y Recomendaciones

|Vacío|Área|Recomendación|
|---|---|---|
|Mecanismo exacto de vinculación médico-paciente|Panel Web ↔ App|Definir si el médico busca al paciente por cédula o si el paciente solicita ser atendido por el médico|
|Flujo de exámenes (adjuntar archivos)|Panel Web|Definir formatos aceptados, tamaño máximo y si los archivos se almacenan en Supabase Storage o externos|
|Historial de compras del paciente|App Móvil|El módulo está mencionado en perfil pero no tiene flujo ni RF definidos|
|Panel del técnico (reabastecimiento)|Panel Web|BR-A05 lo menciona pero no hay pantallas definidas para el flujo del técnico|
|Flujo de supervisión farmacéutica remota|Legal|Ley 1/2001 requiere supervisión. Definir cómo el farmacéutico aprueba o supervisa las dispensaciones|
|Proceso de aprovisionamiento de nuevas máquinas|IoT|No hay documentación del proceso de instalación de certificados mTLS en nuevas expendedoras|
|Proceso de solicitud de consentimiento de médico al paciente|App ↔ Panel|RF-14 lo menciona pero no está desarrollado como flujo completo|

