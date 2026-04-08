#Marco egal panameñp aplicable

PharmaBot opera en el intersecto de tres marcos regulatorios panameños. Cada regla del sistema debe poder trazarse a al menos uno de estos fundamentos legales. La ausencia de regulación específica para vending machines de salud no exime al sistema de cumplir los marcos generales aplicables.

|                                                   |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Ley / Norma**                                   | **Aplicación en PharmaBot**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| **Ley 81 de 2019 Protección de Datos Personales** | Los datos médicos (historial de compras, recetas, alergias, condiciones crónicas) son datos sensibles. Requieren consentimiento explícito, finalidad específica, cifrado AES-256 en reposo, derecho al olvido (soft delete) y no pueden compartirse con terceros sin consentimiento. Sanción: hasta B/. 1,000,000. Ente fiscalizador: ANTAI.                                  [[Requisitos funcionales Generales#RF-14 Solicitud de Consentimiento para Acceso a Historial de Tratamientos\|RF-14 Solicitud de Consentimiento para Acceso a Historial de Tratamientos]] |
| **Ley 1 de 2001 Farmacia y Medicamentos**         | Solo farmacéuticos o regentes autorizados por MINSA pueden supervisar dispensación de controlados. El sistema actúa como auxiliar técnico bajo supervisión farmacéutica remota, no como reemplazo. OTC puede dispensarse sin profesional presencial; controlados requieren trazabilidad completa y receta válida.                                                                                                                                                                                                                                                       |
| **Ley 83 de 2012 Medicamentos Genéricos**         | El catálogo debe incluir el equivalente genérico de cada producto. El sistema debe mostrar la alternativa genérica disponible al paciente antes de confirmar compra de marca innovadora. Campo generico_equivalente obligatorio en la tabla medicamentos.                                                      [[Requisitos funcionales Generales#RF-15 Sugerencia de Medicamento Genérico Equivalente\|RF-15 Sugerencia de Medicamento Genérico Equivalente]]                                                                                                          |
| **Reglamentos CSS Trazabilidad de Controlados**   | La CSS exige registro inmutable de toda dispensación de controlado: identidad del receptor, fecha/hora, cantidad, médico prescriptor y número de receta. El audit log es obligación legal, no opción técnica. Retención mínima: 5 años. El sistema no puede eliminar estos registros ni con soft delete de perfil.  ~~[[Requisitos funcionales Generales#RF-16 Registro Inmutable de Trazabilidad para Medicamentos Controlados\|RF-16 Registro Inmutable de Trazabilidad para Medicamentos Controlados]]~~                                                             |


|                                                                                                                                                                                                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| VACÍO LEGAL IDENTIFICADO: Panamá no tiene regulación específica para vending machines de medicamentos a la fecha del MVP. Se recomienda operar bajo Ley 1/2001 y documentar supervisión farmacéutica remota. En v2 se debe buscar autorización formal de MINSA para el modelo de operación. |
## Reglas del negocio de la aplicacion móvil de parte del paciente
**IDENTIDAD Y ACCESO**

|            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |             |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| **BR-M01** | **Registro requiere cédula panameña válida**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | **CRÍTICA** |
|            | El sistema solo permite el registro de usuarios que posean cédula de identidad personal panameña (o pasaporte para extranjeros residentes). La cédula debe ser verificada contra la API del Tribunal Electoral antes de activar la cuenta. No se permiten cuentas anónimas.<br><br>**Fundamento técnico/legal:**<br><br>_Ley 81/2019 (identificación del titular del dato) + Ley 1/2001 (trazabilidad del receptor). El campo cedula en tabla profiles es UNIQUE NOT NULL._<br><br>Trazabilidad: RF-017  ·  Ley 81  ·  Tribunal Electoral API |             |

|            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |             |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| **BR-M02** | **Consentimiento explícito de datos médicos en onboarding**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | **CRÍTICA** |
|            | Antes de activar la cuenta, el usuario debe aceptar explícitamente el tratamiento de sus datos de salud. La aceptación es granular: el usuario puede aceptar datos de compra y rechazar perfil médico opcional. No se puede usar la app sin aceptar los datos mínimos. Se registra timestamp y versión del aviso de privacidad aceptado.<br><br>**Fundamento técnico/legal:**<br><br>_Ley 81/2019 Art. 5: consentimiento debe ser libre, específico, informado e inequívoco para datos sensibles._<br><br>Trazabilidad: Ley 81 Art.5  ·  ROPA  ·  ANTAI |             |
|            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |             |

**COMPRA DE MEDICAMENTOS OTC**

> [!WARNING]
> # PARTE A CORREGIR
> |            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |          |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| **BR-M03** | **OTC no requiere verificación de identidad pero sí cuenta activa**                                                                                                                                                                                                                                                                                                                                                                                                                            | **ALTA** |
|            | Los medicamentos OTC pueden comprarse sin validación adicional de cédula en el momento de compra, pero el usuario debe estar autenticado. El QR generado tiene TTL de 15 minutos. Si el QR expira, la orden queda en estado expired y el pago se reembolsa automáticamente dentro de 5 minutos. <br><br>**Fundamento técnico/legal:**<br><br>_Regla de compensación Saga: estado orden = expired → proceso reembolso Yappy automático._<br><br>Trazabilidad: RF-004  ·  Saga pattern  ·  Yappy |          |




|            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |          |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| **BR-M04** | **Mostrar equivalente genérico antes de confirmar compra**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | **ALTA** |
|            | Si el medicamento seleccionado tiene un genérico equivalente disponible en la misma máquina, el sistema debe mostrarlo con precio comparativo antes de confirmar el pago. El usuario puede ignorar la sugerencia, pero el sistema registra que fue mostrada.<br><br>**Fundamento técnico/legal:**<br><br>_Ley 83/2012 obliga a informar sobre genéricos. Campo generico_equivalente_id verificado en cada flujo de compra antes de renderizar la pantalla de pago._<br><br>Trazabilidad: Ley 83/2012  ·  [[Requisitos funcionales Generales#RF-15 Sugerencia de Medicamento Genérico Equivalente\|RF-15 Sugerencia de Medicamento Genérico Equivalente]] |          |

**COMPRA CON RECETA**

|            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |             |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| **BR-M05** | **Verificación de identidad obligatoria antes de pago con receta**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | **CRÍTICA** |
|            | Para toda compra de medicamento que requiera receta, el sistema verifica la identidad del paciente contra el Tribunal Electoral antes del pago. Si la verificación falla o el servicio no está disponible, la compra queda bloqueada. No existe excepción ni modo degradado para este control.<br><br>**Fundamento técnico/legal:**<br><br>_La verificación se ejecuta en backend (nunca solo cliente). El resultado se registra en la orden con timestamp._<br><br>Trazabilidad: [[Requisitos funcionales Generales#RF-17 Validación de identidad por medio de cédula\|RF-17 Validación de identidad por medio de cédula]] |             |

|   |   |   |
|---|---|---|
|**BR-M06**|**MFA obligatorio para medicamentos controlados**|**CRÍTICA**|
||Para la compra de medicamentos con es_controlado = TRUE, el paciente debe completar un segundo factor de autenticación (TOTP o SMS OTP) además de la verificación de cédula. Si el usuario no tiene MFA configurado, el sistema lo solicita antes de permitir la compra. No hay bypass por ningún rol.<br><br>**Fundamento técnico/legal:**<br><br>_Reglamento CSS + STRIDE RS-006. Protege contra cuentas comprometidas._<br><br>Trazabilidad: RS-006  ·  CSS  ·  Controlados||

|   |   |   |
|---|---|---|
|**BR-M07**|**La receta debe estar vigente y con dosis disponibles**|**CRÍTICA**|
||Tres condiciones verificadas: (1) estado activa o parcial, (2) fecha_vencimiento posterior a NOW(), (3) cantidad_dispensada < cantidad_prescrita. Si alguna condición falla, la receta se muestra como inválida con razón explícita. El paciente no puede forzar el uso de una receta inválida.<br><br>**Fundamento técnico/legal:**<br><br>_Verificación centralizada en backend obligatoria. El cliente puede mostrar el estado pero nunca es la fuente de verdad._<br><br>Trazabilidad: RF-018  ·  RF-019  ·  RF-020||

**HISTORIAL Y DATOS PERSONALES**

|   |   |   |
|---|---|---|
|**BR-M08**|**El paciente solo puede ver su propio historial**|**CRÍTICA**|
||RLS en Supabase garantiza que un paciente autenticado solo lee filas donde user_id = auth.uid(). Ningún endpoint del backend puede devolver datos de otro paciente, independientemente del rol del solicitante.<br><br>**Fundamento técnico/legal:**<br><br>_Ley 81/2019: datos médicos son sensibles. RLS es la segunda capa de defensa (después del JWT). Ambas son obligatorias y simultáneas._<br><br>Trazabilidad: RS-009  ·  Ley 81  ·  RLS||

|   |   |   |
|---|---|---|
|**BR-M09**|**Derecho al olvido: soft delete con retención de audit log**|**ALTA**|
||Eliminación de cuenta aplica soft delete en perfil y datos personales. Los registros de dispensación en audit log se conservan por obligación regulatoria (CSS: mínimo 5 años). Los registros retenidos se anonimizan: nombre y cédula se reemplazan por hash irreversible SHA-256 con salt.<br><br>**Fundamento técnico/legal:**<br><br>_Tensión legal: Ley 81 (derecho al olvido) vs. CSS (trazabilidad). Solución: anonimización, no eliminación. Función anonymize_patient_audit(user_id)._<br><br>Trazabilidad: Ley 81 Art.22  ·  CSS  ·  Audit log||

|   |   |   |
|---|---|---|
|**BR-M10**|**Notificaciones push no deben contener datos médicos sensibles**|**ALTA**|
||Las notificaciones push (Expo Push / FCM) no pueden incluir nombre de medicamentos con receta, cantidades, diagnósticos ni datos de identidad. El texto debe ser genérico: 'Tu pedido está listo para retiro' en lugar de revelar el medicamento específico.<br><br>**Fundamento técnico/legal:**<br><br>_Ley 81/2019: transferencia de datos a terceros. Los servidores de Apple/Google/Expo son terceros procesadores fuera del control del sistema._<br><br>Trazabilidad: RF-007  ·  Ley 81  ·  FCM/APNs||

|   |   |   |
|---|---|---|
|**BR-M11**|**Alerta de interacción medicamentosa antes de confirmar compra**|**MEDIA**|
||[RF-008 — Could Have / v2] Si el paciente tiene perfil médico con alergias o condiciones crónicas, el sistema verifica posibles interacciones con el medicamento seleccionado antes del pago. Si hay interacción, muestra advertencia que el usuario debe confirmar explícitamente. La tabla perfiles_medicos debe diseñarse desde MVP aunque la lógica sea stub.<br><br>**Fundamento técnico/legal:**<br><br>_Planificación anticipada: la arquitectura de datos debe soportar esta regla desde el inicio para no requerir migración en v2._<br><br>Trazabilidad: RF-008  ·  v2||

|   |   |   |
|---|---|---|
|**BR-M12**|**El pago debe confirmarse antes de generar el QR de dispensación**|**CRÍTICA**|
||El orden del Saga es invariable: (1) pago iniciado → (2) confirmación de pago recibida → (3) token generado → (4) QR mostrado al usuario. Si el gateway Yappy no responde en 10 segundos, la orden pasa a payment_timeout. No se genera QR bajo ninguna circunstancia sin confirmación de pago.<br><br>**Fundamento técnico/legal:**<br><br>_RNF-010: strong consistency. Invertir este orden constituye dispensación sin pago, violación regulatoria y pérdida financiera._<br><br>Trazabilidad: RF-004  ·  RF-005  ·  RNF-010  ·  Saga||

|   |   |   |
|---|---|---|
|**BR-M13**|**Límite diario de compra de controlados por paciente**|**CRÍTICA**|
||Un paciente no puede comprar más de una receta de medicamentos controlados por día, incluso con múltiples recetas activas. El backend verifica dispensaciones del día antes de autorizar. Límite se reinicia a las 00:00 UTC-5 (hora Panamá).<br><br>**Fundamento técnico/legal:**<br><br>_RS-008: mitigación de compra múltiple de controlados. Query: SELECT COUNT(*) FROM dispensaciones WHERE user_id=? AND es_controlado=true AND created_at>=TODAY()._<br><br>Trazabilidad: RS-008  ·  CSS  ·  Controlados||

|   |   |   |
|---|---|---|
|**BR-M14**|**El mapa solo muestra máquinas con stock del medicamento buscado**|**MEDIA**|
||Al buscar un medicamento específico, el mapa solo resalta máquinas con stock > 0 de ese medicamento. Las máquinas sin stock o desconectadas se muestran en gris con etiqueta correspondiente. No se guía al usuario a una máquina que no puede atenderlo.<br><br>**Fundamento técnico/legal:**<br><br>_UX crítico: una máquina sin stock que aparece en el mapa genera una experiencia frustrante y un viaje innecesario del paciente._<br><br>Trazabilidad: RF-001  ·  RF-003  ·  UX||

### Reglas del negocio de parte de app móvil junto a maquina

**TOKEN Y DISPENSACIÓN**

|   |   |   |
|---|---|---|
|**BR-I01**|**Token de dispensación: un solo uso, 15 minutos de vigencia**|**CRÍTICA**|
||Cada QR contiene un UUID v4 firmado con HMAC-SHA256. El token tiene TTL de 15 minutos y es de un solo uso. Una vez escaneado y procesado, el token se marca como consumido en BD. Cualquier reuso del mismo token (incluso dentro del TTL) es rechazado. Es la mitigación principal contra ataques de replay.<br><br>**Fundamento técnico/legal:**<br><br>_RS-001: columna token_id UUID UNIQUE NOT NULL en tabla dispensation_tokens. Verificación en backend ANTES de enviar comando MQTT. La máquina verifica la firma; el backend verifica el estado._<br><br>Trazabilidad: RS-001  ·  RF-011  ·  HMAC-SHA256||

|   |   |   |
|---|---|---|
|**BR-I02**|**La máquina actúa solo con señal MQTT firmada del backend**|**CRÍTICA**|
||El teléfono presenta el QR → la máquina lee el QR y envía el token al backend → el backend verifica y envía autorización vía MQTT firmado → la máquina actúa. Si el comando MQTT no llega en 5 segundos, la máquina aborta y notifica al usuario. El cliente NUNCA es fuente de confianza.<br><br>**Fundamento técnico/legal:**<br><br>_Cadena de confianza: backend → MQTT broker (mTLS) → firmware. La clave HMAC está en entorno seguro del firmware, no en código fuente._<br><br>Trazabilidad: RF-011  ·  RF-012  ·  MQTT mTLS||

|   |   |   |
|---|---|---|
|**BR-I03**|**Foto obligatoria al retirar medicamentos controlados**|**CRÍTICA**|
||Para dispensación de cualquier medicamento con es_controlado = TRUE, la cámara de la máquina captura una foto del usuario en el momento del retiro. La foto se almacena cifrada en el audit log con referencia a la dispensación. Si la cámara no está operativa, la dispensación de controlados queda bloqueada hasta restaurar.<br><br>**Fundamento técnico/legal:**<br><br>_Reglamento CSS: trazabilidad de controlados. La foto se almacena en Supabase Storage con acceso restringido a rol regulador. No se usa para identificación biométrica automática en MVP._<br><br>Trazabilidad: RF-013  ·  CSS  ·  RF-021||

**MODO OFFLINE DEGRADADO**

|   |   |   |
|---|---|---|
|**BR-I04**|**Controlados bloqueados sin conexión: regla no negociable**|**CRÍTICA**|
||Sin conexión MQTT por más de 60 segundos → modo offline degradado. En este modo: OTC con QR pre-cacheado pueden dispensarse. Medicamentos con receta o controlados quedan BLOQUEADOS automáticamente. Esta lógica vive en el firmware, sin depender de ninguna llamada al backend.<br><br>**Fundamento técnico/legal:**<br><br>_RNF-004 + RS-004: el firmware debe tener la lista de medicamentos controlados cacheada localmente (actualizada en cada reconexión). 'Fallar seguro' significa que ante la duda, no dispensa._<br><br>Trazabilidad: RF-015  ·  RNF-004  ·  RS-004||

|   |   |   |
|---|---|---|
|**BR-I05**|**QR offline solo válido si fue pre-autorizado en línea**|**ALTA**|
||En modo offline, la máquina solo valida localmente QRs generados y cacheados mientras tenía conexión. No puede generar nuevas autorizaciones. El caché local tiene máximo 50 tokens OTC. Los tokens en caché también tienen TTL de 15 minutos.<br><br>**Fundamento técnico/legal:**<br><br>_El usuario con QR nuevo (generado después del corte) no puede ser atendido para medicamentos con receta. SQLite local gestiona el caché de tokens._<br><br>Trazabilidad: RF-015  ·  SQLite local||

**INVENTARIO Y STOCK**

|   |   |   |
|---|---|---|
|**BR-I06**|**Stock se actualiza en tiempo real tras dispensación exitosa**|**CRÍTICA**|
||Inmediatamente tras dispensación exitosa (confirmada por firmware), el backend decrementa el stock del slot y publica la actualización vía Supabase Realtime. Si el decremento falla, se registra como incidencia y se alerta al admin. La verificación de stock es doble: en la selección (UI) y en el backend al crear la orden.<br><br>**Fundamento técnico/legal:**<br><br>_El usuario no puede comprar un medicamento con stock 0 incluso si la app aún lo muestra como disponible._<br><br>Trazabilidad: RF-002  ·  RF-010  ·  Supabase Realtime||

|   |   |   |
|---|---|---|
|**BR-I07**|**Alerta de stock bajo notifica antes de agotarse**|**ALTA**|
||Cada slot tiene un umbral mínimo configurable por el admin (ej: 5 unidades). Al llegar al umbral, se genera alerta al admin y técnico. El medicamento sigue disponible para compra hasta stock = 0; la alerta no bloquea ventas. Con stock = 0 el medicamento se marca como no disponible en mapa y búsqueda.<br><br>**Fundamento técnico/legal:**<br><br>_La telemetría MQTT reporta el estado de stock cada 30 segundos para mantener el umbral actualizado._<br><br>Trazabilidad: RF-014  ·  RF-010||

**SEGURIDAD FÍSICA**

|   |   |   |
|---|---|---|
|**BR-I08**|**Apertura no autorizada del gabinete genera alerta inmediata**|**CRÍTICA**|
||Si el gabinete se abre fuera de un proceso de reabastecimiento autorizado (sin sesión de técnico activa), el firmware envía alerta inmediata al backend vía MQTT → notificación al admin en tiempo real → registro en audit log con timestamp y foto de cámara.<br><br>**Fundamento técnico/legal:**<br><br>_RS-004: el proceso de reabastecimiento debe iniciarse desde panel admin ANTES de abrir la máquina. El técnico se autentica en el sistema antes de la apertura física._<br><br>Trazabilidad: RS-004  ·  RF-013  ·  RF-024||

|   |   |   |
|---|---|---|
|**BR-I09**|**Telemetría cada 30 segundos; ausencia de señal = alerta**|**ALTA**|
||Cada máquina publica telemetría (estado, temperatura, stock por slot, uptime) al broker MQTT cada 30 segundos. Sin telemetría por más de 90 segundos → alerta de 'máquina sin señal' al admin → máquina aparece como 'fuera de línea' en dashboard y mapa automáticamente.<br><br>**Fundamento técnico/legal:**<br><br>_RNF-001: el monitoreo de telemetría es la base del SLA de 99.5%. Sin este mecanismo no hay forma de medir el uptime real de la flota._<br><br>Trazabilidad: RF-010  ·  RNF-001  ·  MQTT||

|   |   |   |
|---|---|---|
|**BR-I10**|**La dispensación es atómica: todo o nada**|**CRÍTICA**|
||La secuencia pago → token → dispensación → confirmación → stock update debe completarse de forma atómica. Si la dispensación falla después del pago (fallo mecánico, timeout), se lanza compensación automática: token marcado como failed, stock revertido, reembolso Yappy iniciado. El usuario nunca pierde dinero sin recibir el medicamento.<br><br>**Fundamento técnico/legal:**<br><br>_RNF-010 + Saga pattern. Máquina de estados del token: pending → dispensing → consumed \| expired \| failed. Cada transición se registra en audit log._<br><br>Trazabilidad: RNF-010  ·  Saga  ·  Yappy reembolso||

|   |   |   |
|---|---|---|
|**BR-I11**|**Comunicación MQTT cifrada con certificado único por máquina**|**CRÍTICA**|
||Cada máquina tiene un certificado TLS único (mTLS) para conexión MQTT al broker HiveMQ. Un certificado comprometido no compromete el resto de la flota. Los certificados se rotan cada 90 días. No se permite conexión sin certificado válido al broker.<br><br>**Fundamento técnico/legal:**<br><br>_RS-005: certificados generados con OpenSSL en el proceso de aprovisionamiento de cada dispositivo. La clave privada nunca sale del dispositivo._<br><br>Trazabilidad: RS-005  ·  mTLS  ·  HiveMQ||

|   |   |   |
|---|---|---|
|**BR-I12**|**Temperatura fuera de rango bloquea dispensación del slot afectado**|**ALTA**|
||[RF-016 — Could Have / v2] Para slots con medicamentos de cadena de frío, si el sensor detecta valores fuera del rango configurado (ej: 2-8°C), ese slot específico se bloquea para dispensación y se alerta al admin. Los demás slots no se ven afectados. Campos temperatura_minima y temperatura_maxima deben existir en esquema desde MVP.<br><br>**Fundamento técnico/legal:**<br><br>_Planificación anticipada: aunque es v2, la arquitectura de slots debe soportar los campos de temperatura desde el inicio._<br><br>Trazabilidad: RF-016  ·  v2  ·  Cadena de frío||
#### Regla del negocio de App móvil con Admin /Médico

**GESTIÓN DE MÉDICOS Y RECETAS**

|   |   |   |
|---|---|---|
|**BR-A01**|**Solo médicos con idoneidad verificada pueden emitir recetas**|**CRÍTICA**|
||El admin registra manualmente al médico con su número de idoneidad (Consejo Técnico de Salud de Panamá) y verifica su estatus activo. Los médicos suspendidos o retirados deben desactivarse inmediatamente. Una receta emitida por un médico con idoneidad suspendida es inválida aunque esté dentro del período de vigencia.<br><br>**Fundamento técnico/legal:**<br><br>_Ley 1/2001. El sistema verifica el estado activo del médico en CADA validación de receta, no solo al momento de la emisión. Esto permite bloquear recetas futuras de médicos sancionados._<br><br>Trazabilidad: RF-026  ·  RF-018  ·  Ley 1/2001||

|   |   |   |
|---|---|---|
|**BR-A02**|**La receta digital es firmada criptográficamente por el médico**|**CRÍTICA**|
||Al emitir una receta, el backend genera una firma digital del contenido (medicamento, dosis, cantidad, fecha, paciente_id, medico_id) usando las credenciales del médico. Cualquier modificación posterior invalida la firma. El sistema rechaza recetas cuya firma no pueda verificarse.<br><br>**Fundamento técnico/legal:**<br><br>_RS-002: Spoofing de receta. La firma previene falsificación por terceros, aunque no previene que el médico emita recetas ilegítimas (eso lo controla el registro de idoneidad)._<br><br>Trazabilidad: RS-002  ·  RF-018  ·  Firma digital||

|   |   |   |
|---|---|---|
|**BR-A03**|**El médico solo puede ver recetas y compras de sus propios pacientes**|**CRÍTICA**|
||Un médico autenticado puede consultar el historial de dispensaciones de los pacientes para quienes ha emitido recetas, para verificar cumplimiento del tratamiento. No puede acceder a datos de pacientes de otros médicos. RLS enforza esta restricción a nivel de base de datos.<br><br>**Fundamento técnico/legal:**<br><br>_Ley 81/2019: principio de minimización. El acceso del médico tiene finalidad específica (seguimiento del tratamiento) y el paciente acepta este acceso en el consentimiento de onboarding._<br><br>Trazabilidad: RS-009  ·  Ley 81  ·  RLS||

**GESTIÓN DE INVENTARIO Y CATÁLOGO**

|   |   |   |
|---|---|---|
|**BR-A04**|**Catálogo de medicamentos: campos obligatorios por ley**|**ALTA**|
||Campos obligatorios en cada medicamento: nombre_comercial, principio_activo, requiere_receta (boolean), es_controlado (boolean), generico_equivalente_id (nullable), temperatura_almacenamiento, dosis_maxima_diaria. Ninguno es opcional para el admin al registrar un producto.<br><br>**Fundamento técnico/legal:**<br><br>_Ley 1/2001 (clasificación de medicamentos) + Ley 83/2012 (genéricos obligatorios). El campo generico_equivalente_id solo puede ser NULL para genéricos mismos. El admin debe justificar si no hay equivalente disponible._<br><br>Trazabilidad: RF-025  ·  Ley 1/2001  ·  Ley 83/2012||

|   |   |   |
|---|---|---|
|**BR-A05**|**Reabastecimiento requiere sesión de técnico autenticada**|**ALTA**|
||El técnico debe iniciar el proceso de reabastecimiento desde el panel admin antes de abrir el gabinete. Registra qué medicamentos agrega a qué slots y en qué cantidades. El sistema registra el reabastecimiento en stock_movimientos con el técnico responsable, fecha y cantidades.<br><br>**Fundamento técnico/legal:**<br><br>_Sin este registro es imposible distinguir reabastecimiento legítimo de manipulación no autorizada. El sensor de apertura del gabinete compara si hay una sesión de reabastecimiento activa._<br><br>Trazabilidad: RF-024  ·  RS-004||

**REPORTES REGULATORIOS**

|   |   |   |
|---|---|---|
|**BR-A06**|**Reportes de controlados no deben incluir PII en texto plano**|**ALTA**|
||Los reportes exportables para reguladores (MINSA/CSS) incluyen los datos de dispensación requeridos por ley, pero con identidad del paciente pseudonimizada (hash de cédula). El estándar no incluye nombres completos. El regulador puede solicitar datos completos mediante proceso legal, lo que se registra como evento de acceso extraordinario.<br><br>**Fundamento técnico/legal:**<br><br>_Ley 81/2019: minimización de datos en transferencias. El regulador tiene acceso de solo lectura y no puede modificar nada en el sistema._<br><br>Trazabilidad: RF-027  ·  Ley 81  ·  RS-009||

|   |   |   |
|---|---|---|
|**BR-A07**|**Admin no puede modificar ni eliminar registros del audit log**|**CRÍTICA**|
||El schema audit en Supabase tiene políticas RLS que solo permiten INSERT. Ningún usuario, incluyendo el DBA, puede hacer UPDATE o DELETE sobre registros de dispensación. Si un registro es erróneo, se agrega uno de corrección referenciando el original. El historial es siempre acumulativo.<br><br>**Fundamento técnico/legal:**<br><br>_Reglamento CSS: trazabilidad inmutable. Implementar trigger en PostgreSQL que rechace UPDATE/DELETE en schema audit. Esta es la diferencia entre un audit log real y una tabla de logs normal._<br><br>Trazabilidad: RF-021  ·  CSS  ·  RLS INSERT-only||

|   |   |   |
|---|---|---|
|**BR-A08**|**El rol regulador tiene acceso de solo lectura al audit log**|**ALTA**|
||El rol regulador_minsa tiene acceso de solo lectura al audit log y reportes de dispensación. No puede ver datos de perfiles de pacientes más allá del hash de identidad, ni acceder a módulos operativos. Cada acceso del regulador queda registrado en un log de accesos separado.<br><br>**Fundamento técnico/legal:**<br><br>_RBAC obligatorio: Paciente / Médico / Técnico / Admin Farmacia / Regulador MINSA. El acceso regulatorio es de auditoría, no de operación._<br><br>Trazabilidad: RF-023  ·  RS-006  ·  RBAC||

|   |   |   |
|---|---|---|
|**BR-A09**|**Alertas del dashboard en tiempo real, no bajo demanda**|**ALTA**|
||El dashboard del admin refleja el estado de la flota en tiempo real vía Supabase Realtime. Las alertas (stock bajo, máquina offline, apertura no autorizada) aparecen automáticamente. La latencia de una alerta desde el evento hasta su visualización no puede superar 5 segundos.<br><br>**Fundamento técnico/legal:**<br><br>_RNF-002 + RNF-011. Una alerta tardía puede significar medicamentos controlados robados. El tiempo de respuesta de alertas es un requisito de seguridad, no solo de UX._<br><br>Trazabilidad: RF-023  ·  Supabase Realtime  ·  RNF-002||

|   |   |   |
|---|---|---|
|**BR-A10**|**El admin no puede ver datos médicos privados de pacientes**|**CRÍTICA**|
||El administrador de farmacia tiene acceso al dashboard operativo (máquinas, inventario, reportes agregados) pero NO tiene acceso al perfil médico privado de los pacientes (alergias, condiciones, historial individual). Solo puede ver estadísticas agregadas por medicamento y máquina, nunca datos por paciente individual.<br><br>**Fundamento técnico/legal:**<br><br>_Ley 81/2019: principio de finalidad. El admin opera la infraestructura; no tiene necesidad legítima de acceder a datos médicos individuales. RLS enforza esto incluso con acceso directo a Supabase dashboard._<br><br>Trazabilidad: RS-009  ·  Ley 81  ·  RBAC||

|   |   |   |
|---|---|---|
|**BR-A11**|**Desactivar médico invalida sus recetas pendientes**|**ALTA**|
||Al desactivar un médico (estado inactivo o suspendido), todas sus recetas en estado activa o parcial pasan a suspendida_por_medico. Los pacientes afectados reciben notificación. La desactivación no puede revertirse sin revalidación manual del admin.<br><br>**Fundamento técnico/legal:**<br><br>_Escenario regulatorio real: un médico puede ser sancionado por el Consejo Técnico de Salud. Sus recetas en curso deben invalidarse inmediatamente para evitar dispensaciones ilegítimas bajo su nombre._<br><br>Trazabilidad: RF-026  ·  RF-018  ·  Ley 1/2001||

Reglas de seguridad

|   |   |   |
|---|---|---|
|**BR-T01**|**Logs nunca contienen datos médicos identificables**|**CRÍTICA**|
||Los logs estructurados (Pino/JSON) no deben registrar: cédulas completas, nombres de pacientes, nombres de medicamentos con receta, tokens de pago ni resultados de verificación de identidad. Los UUIDs internos sí pueden loggearse para correlación. Esta regla aplica a logs de aplicación, no al audit log médico.<br><br>**Fundamento técnico/legal:**<br><br>_RS-010: Information Disclosure. Definir lista explícita de campos prohibidos en logs y validarla en CI. Un log de error con datos médicos se convierte en filtración si los logs son accedidos por personal de operaciones._<br><br>Trazabilidad: RS-010  ·  Ley 81  ·  Pino||

|   |   |   |
|---|---|---|
|**BR-T02**|**Rate limiting en endpoints de verificación de identidad**|**ALTA**|
||Endpoint de verificación de cédula: máximo 5 intentos por IP por minuto. Al tercer intento fallido consecutivo se activa CAPTCHA. Al quinto intento fallido, la IP queda bloqueada 15 minutos. Los intentos fallidos se registran con IP hasheada en el audit log.<br><br>**Fundamento técnico/legal:**<br><br>_RS-003: fuerza bruta de cédulas. El Tribunal Electoral puede suspender el acceso al sistema si detecta uso abusivo. El rate limiting también protege la disponibilidad del servicio externo._<br><br>Trazabilidad: RS-003  ·  RF-017||

|   |   |   |
|---|---|---|
|**BR-T03**|**Secretos nunca en código fuente ni en variables no cifradas**|**CRÍTICA**|
||Las claves HMAC de máquinas, claves de API de Yappy, JWT secrets y credenciales de Supabase se gestionan exclusivamente como variables de entorno en Railway. GitLeaks corre en cada PR para detectar secretos accidentalmente comiteados. Un commit con secreto debe ser revocado inmediatamente y la clave rotada.<br><br>**Fundamento técnico/legal:**<br><br>_DevSecOps pipeline: GitLeaks + Semgrep + Snyk en GitHub Actions en cada PR. Un secreto en código fuente público es una brecha inmediata, no un riesgo futuro._<br><br>Trazabilidad: DevSecOps  ·  GitLeaks  ·  Railway||

|   |   |   |
|---|---|---|
|**BR-T04**|**Toda comunicación usa TLS 1.3; no se acepta versión inferior**|**CRÍTICA**|
||Toda comunicación entre app móvil ↔ backend, backend ↔ Supabase, y firmware ↔ broker MQTT debe usar TLS 1.3. No se acepta TLS 1.2 ni inferior en ningún componente. Certificate pinning en la app móvil para el dominio del backend para prevenir ataques de hombre en el medio.<br><br>**Fundamento técnico/legal:**<br><br>_RS-005. TLS 1.3 elimina cipher suites débiles heredados. En el contexto de una app de salud, comunicación débilmente cifrada es inadmisible bajo Ley 81/2019._<br><br>Trazabilidad: RS-005  ·  Ley 81  ·  TLS 1.3||

|   |   |   |
|---|---|---|
|**BR-T05**|**Validación de inputs en cliente y en servidor; servidor es fuente de verdad**|**ALTA**|
||Toda entrada de usuario se valida con Zod tanto en el cliente (UX) como en el backend (seguridad). La validación del cliente es conveniente; la del servidor es obligatoria. El backend rechaza cualquier request con datos inválidos independientemente de si el cliente los validó.<br><br>**Fundamento técnico/legal:**<br><br>_RS-007: SQL injection y input tampering. Supabase usa prepared statements; el backend nunca concatena strings en queries. Zod schemas son el contrato entre cliente y servidor._<br><br>Trazabilidad: RS-007  ·  Zod||

|   |   |   |
|---|---|---|
|**BR-T06**|**Sistema no puede dispensar controlados sin backend activo, bajo ninguna circunstancia**|**CRÍTICA**|
||Si el backend no está disponible, el MQTT broker no responde, la verificación de identidad falla, o el token es inválido — en cualquiera de estos casos, la dispensación de medicamentos controlados queda bloqueada. La máquina debe fallar de forma segura, no de forma abierta.<br><br>**Fundamento técnico/legal:**<br><br>_Principio de 'fail secure over fail open'. Esta regla tiene prioridad sobre cualquier consideración de UX o disponibilidad. Es la restricción no negociable más importante del sistema._<br><br>Trazabilidad: RNF-004  ·  RS-001  ·  Fail Secure  ·  CSS||

|   |   |   |
|---|---|---|
|**BR-T07**|**RBAC enforzado en backend y en base de datos simultáneamente**|**CRÍTICA**|
||El control de acceso por roles se enforza en dos capas independientes: (1) middleware Fastify valida el claim de rol en el JWT antes de procesar cualquier request, (2) políticas RLS en PostgreSQL aplican restricciones a nivel de fila. Comprometer una capa no compromete la otra.<br><br>**Fundamento técnico/legal:**<br><br>_RS-006: escalada de privilegios. Defensa en profundidad: cada capa de seguridad debe ser capaz de defender el sistema sola. Un error en el middleware de Fastify no debe poder ser explotado si RLS está bien configurado._<br><br>Trazabilidad: RS-006  ·  RBAC  ·  RLS  ·  JWT||

|   |   |   |
|---|---|---|
|**BR-T08**|**El sistema mantiene un registro de consentimientos actualizado (ROPA)**|**ALTA**|
||Se mantiene un Registro de Actividades de Tratamiento (ROPA) que documenta: qué datos se procesan, con qué finalidad, cuánto tiempo se retienen, quién tiene acceso y bajo qué base legal. Al actualizar el aviso de privacidad, todos los usuarios activos deben aceptar la nueva versión antes de continuar usando el sistema.<br><br>**Fundamento técnico/legal:**<br><br>_Ley 81/2019: obligación del responsable del tratamiento. La ANTAI es el ente fiscalizador. El ROPA es la evidencia de cumplimiento ante una inspección._<br><br>Trazabilidad: Ley 81/2019  ·  ROPA  ·  ANTAI||
Reglas de lógica de negocio

permitidas, invariantes del dominio y comportamientos de compensación. Son las reglas que el código debe implementar como lógica pura, independientemente de la interfaz o el canal.

**MÁQUINA DE ESTADOS — ÓRDENES Y TOKENS**

|   |   |   |
|---|---|---|
|**BR-L01**|**Máquina de estados de la orden: transiciones permitidas**|**CRÍTICA**|
||Una orden sigue la máquina de estados: CREATED → PAYMENT_PENDING → PAID → TOKEN_GENERATED → DISPENSING → COMPLETED. Las transiciones de compensación son: PAYMENT_PENDING → PAYMENT_TIMEOUT (si Yappy no responde en 10s), PAID → REFUNDING → REFUNDED (si dispensación falla), TOKEN_GENERATED → EXPIRED (si TTL de 15min se agota sin escaneo). Ninguna transición puede saltarse un estado intermedio.<br><br>**Fundamento técnico/legal:**<br><br>_Un salto de estado (ej: CREATED → DISPENSING sin PAID) es una vulnerabilidad que permite dispensación sin pago. El backend debe validar el estado actual antes de cada transición._<br><br>Trazabilidad: RF-004  ·  RF-005  ·  RNF-010  ·  Saga||

|   |   |   |
|---|---|---|
|**BR-L02**|**Máquina de estados del token de dispensación**|**CRÍTICA**|
||El token sigue la máquina de estados: PENDING → SCANNING → DISPENSING → CONSUMED. Compensaciones: PENDING → EXPIRED (TTL 15 min), DISPENSING → FAILED (fallo mecánico o timeout 5s). Un token en estado CONSUMED, EXPIRED o FAILED no puede reactivarse. Un token solo puede estar en estado DISPENSING en una máquina a la vez (lock distribuido).<br><br>**Fundamento técnico/legal:**<br><br>_El lock distribuido previene condición de carrera donde dos máquinas intentan procesar el mismo token simultáneamente. Usar Redis o Supabase advisory lock durante la fase DISPENSING._<br><br>Trazabilidad: RS-001  ·  BR-I01  ·  RF-011||

|   |   |   |
|---|---|---|
|**BR-L03**|**Máquina de estados de la receta digital**|**CRÍTICA**|
||Una receta sigue los estados: ACTIVE → PARTIAL (cuando cantidad_dispensada > 0 pero < cantidad_prescrita) → EXHAUSTED (cuando cantidad_dispensada = cantidad_prescrita). Además: ACTIVE/PARTIAL → EXPIRED (fecha_vencimiento < NOW()), ACTIVE/PARTIAL → SUSPENDED_BY_DOCTOR (médico desactivado). EXPIRED y EXHAUSTED son estados terminales no reversibles. SUSPENDED_BY_DOCTOR puede reactivarse si el médico es reactivado.<br><br>**Fundamento técnico/legal:**<br><br>_Todos los estados se evalúan en el momento de la solicitud de compra, no solo al momento de la emisión. Un cron job debe marcar EXPIRED las recetas vencidas diariamente a las 00:00 UTC-5._<br><br>Trazabilidad: RF-018  ·  RF-019  ·  RF-020  ·  BR-A11||

**CÁLCULOS Y CANTIDADES**

|   |   |   |
|---|---|---|
|**BR-L04**|**La cantidad a dispensar nunca puede superar el saldo de la receta**|**CRÍTICA**|
||Al procesar una compra con receta, el sistema calcula: saldo_disponible = cantidad_prescrita - cantidad_dispensada. Si la cantidad solicitada en la compra > saldo_disponible, la compra se rechaza con error explicativo. Si la cantidad solicitada <= saldo_disponible, se dispensa la cantidad solicitada y se actualiza cantidad_dispensada. Esta verificación es atómica con la creación de la orden (transacción de BD).<br><br>**Fundamento técnico/legal:**<br><br>_La verificación de saldo y la creación de la orden deben ocurrir en la misma transacción de PostgreSQL para evitar condición de carrera con compras concurrentes del mismo paciente._<br><br>Trazabilidad: RF-019  ·  RS-008||

|   |   |   |
|---|---|---|
|**BR-L05**|**Stock de slot: la reserva precede a la dispensación física**|**ALTA**|
||Cuando se genera un token de dispensación (orden en estado TOKEN_GENERATED), el stock del slot se reserva lógicamente (stock_reservado + 1). Solo cuando la dispensación se confirma, el stock_reservado se convierte en stock_dispensado (stock_total - 1). Si el token expira o falla, la reserva se libera (stock_reservado - 1). Esto previene sobreventa de un slot con una sola unidad.<br><br>**Fundamento técnico/legal:**<br><br>_Sin reserva lógica, dos usuarios podrían pagar por el último medicamento de un slot y solo uno puede recibirlo. El stock visible en la app = stock_total - stock_reservado - stock_dispensado._<br><br>Trazabilidad: RF-002  ·  BR-I06  ·  RNF-010||

|   |   |   |
|---|---|---|
|**BR-L06**|**Precio de la orden se fija al momento de la compra, no al de la dispensación**|**ALTA**|
||El precio que el usuario paga se registra en la orden en el momento de iniciar el pago (precio_unitario * cantidad). Si el precio del medicamento cambia después de que el usuario inició el flujo de pago, la orden se procesa con el precio original. El usuario no puede ser cobrado un precio mayor al que vio en pantalla.<br><br>**Fundamento técnico/legal:**<br><br>_Principio de inmutabilidad del precio comprometido. El campo precio_pagado en la tabla ordenes es NOT NULL y se establece en la creación de la orden, no se recalcula._<br><br>Trazabilidad: RF-004  ·  RF-005  ·  UX||

**REGLAS DE VALIDACIÓN CRUZADA**

|   |   |   |
|---|---|---|
|**BR-L07**|**Un medicamento controlado siempre requiere receta, sin excepciones**|**CRÍTICA**|
||Si es_controlado = TRUE en la tabla medicamentos, automáticamente se implica requiere_receta = TRUE. El sistema debe rechazar cualquier intento de configurar un medicamento como controlado = TRUE y receta = FALSE. Esta es una invariante del dominio que debe validarse tanto en la capa de aplicación (Zod) como en la base de datos (constraint CHECK).<br><br>**Fundamento técnico/legal:**<br><br>_Constraint PostgreSQL: CHECK (NOT es_controlado OR requiere_receta). Un medicamento controlado que no requiere receta es una violación regulatoria grave bajo Ley 1/2001 y reglamentos CSS._<br><br>Trazabilidad: RF-025  ·  Ley 1/2001  ·  CSS||

|   |   |   |
|---|---|---|
|**BR-L08**|**La máquina no puede dispensar medicamentos no asignados a sus slots**|**ALTA**|
||Cada slot de cada máquina tiene asignado un medicamento_id específico. El sistema debe verificar que el medicamento solicitado en la orden esté asignado al slot correcto de la máquina correcta. Un comando de dispensación para un medicamento_id que no corresponde al slot no debe ejecutarse, aunque el token sea válido.<br><br>**Fundamento técnico/legal:**<br><br>_Esta verificación ocurre en el backend al generar el token (token incluye slot_id y medicamento_id) y en el firmware al ejecutar el comando (el firmware verifica que el slot físico corresponda al slot_id del token)._<br><br>Trazabilidad: RF-011  ·  RF-012  ·  RS-001||

|   |   |   |
|---|---|---|
|**BR-L09**|**Reembolso automático tiene ventana máxima de 24 horas**|**ALTA**|
||Si el sistema no puede completar un reembolso automático inmediatamente (ej: falla de la API de Yappy), el proceso de reembolso se reintenta con backoff exponencial (1min, 5min, 15min, 1h) hasta completarse. Si después de 24 horas el reembolso no se ha completado, el caso se escala a una cola de reembolsos manuales con alerta al admin. El reembolso nunca se descarta silenciosamente.<br><br>**Fundamento técnico/legal:**<br><br>_RNF-010: el usuario nunca pierde su dinero. Una orden en estado REFUNDING por más de 24 horas es una incidencia que requiere intervención manual. Registrar cada intento de reembolso en el audit log._<br><br>Trazabilidad: RNF-010  ·  Saga  ·  Yappy||

###**REGLAS DE EXPIRACIÓN Y LIMPIEZA**

|   |   |   |
|---|---|---|
|**BR-L10**|**Cron job diario verifica y expira recetas vencidas**|**MEDIA**|
||Un proceso automático programado para las 00:05 UTC-5 cada día actualiza el estado de todas las recetas donde fecha_vencimiento < NOW() y estado IN ('active', 'partial') → estado = 'expired'. La operación se registra en el audit log con el número de recetas expiradas. Los pacientes con recetas expiradas reciben notificación push.<br><br>**Fundamento técnico/legal:**<br><br>_Sin este proceso, una receta que naturalmente vence solo se detectaría en el próximo intento de compra del paciente. La expiración proactiva mejora la UX y asegura integridad del estado._<br><br>Trazabilidad: RF-020  ·  RF-007||

|   |   |   |
|---|---|---|
|**BR-L11**|**Los logs de aplicación se retienen 90 días; el audit log médico, 5 años**|**ALTA**|
||Dos políticas de retención diferenciadas: (1) Logs de aplicación (Pino/Grafana): retención de 90 días, luego se eliminan. Contienen información operativa sin PII médico. (2) Audit log médico (schema audit en Supabase): retención mínima de 5 años por obligación CSS. Incluso con cuenta eliminada, los registros anonimizados se conservan el período completo.<br><br>**Fundamento técnico/legal:**<br><br>_La retención diferenciada reduce el costo de almacenamiento de logs operativos mientras cumple la obligación regulatoria de trazabilidad médica. Implementar política de retención automática en Grafana Cloud._<br><br>Trazabilidad: Ley 81  ·  CSS  ·  RF-021  ·  BR-M09||

|   |   |   |
|---|---|---|
|**BR-L12**|**Idempotencia en operaciones críticas de dispensación y pago**|**CRÍTICA**|
||Los endpoints de creación de orden, confirmación de pago y comando de dispensación deben ser idempotentes. Si el cliente reintenta una operación (por timeout de red), el sistema debe retornar el mismo resultado que la primera ejecución exitosa, sin ejecutar la operación dos veces. Cada operación tiene un idempotency_key único del lado del cliente.<br><br>**Fundamento técnico/legal:**<br><br>_Sin idempotencia, un timeout de red en el momento del pago puede resultar en doble cobro o doble dispensación. El idempotency_key debe almacenarse en BD y verificarse antes de procesar cualquier operación de pago._<br><br>Trazabilidad: RNF-010  ·  RF-004  ·  RF-005  ·  Yappy||