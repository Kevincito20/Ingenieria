


[[2. Interface]]LOGIN


| REGISTRAR USUARIO                                                       |
| ----------------------------------------------------------------------- |
| **Precondiciones:**                                                     |
| • El usuario tiene cédula panameña o pasaporte válido.                  |
| • La API del Tribunal Electoral está disponible.                        |
| • El usuario dispone de correo electrónico o número de teléfono activo. |

| **Paso**                                       | **Actor** | **Descripción**                                                                                  | **Seguridad**                                          |     |
| ---------------------------------------------- | --------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------ | --- |
| **1**                                          | Usuario   | Descarga la app e inicia el proceso de registro.                                                 | TLS 1.3 en todas las comunicaciones                    |     |
| **2**                                          | Usuario   | Ingresa cédula (o pasaporte), nombre completo, correo y contraseña.                              | Validación Zod en cliente y servidor                   |     |
| **3**                                          | Backend   | Verifica la cédula contra la API del Tribunal Electoral. Resultado esperado en ≤ 5 segundos.     | Rate limiting: 5 intentos/IP/min (BR-T02)              |     |
| **4**                                          | Backend   | Valida que la cédula no esté ya registrada en el sistema (campo UNIQUE NOT NULL).                | RLS Supabase activo                                    |     |
| **5**                                          | Sistema   | Presenta la pantalla de consentimiento explícito de datos. El usuario acepta de forma granular.  | Ley 81 Art.5 — timestamp + versión registrada (BR-M02) |     |
| **6**                                          | Backend   | Envía código OTP al correo o SMS del usuario.                                                    | OTP de un solo uso, TTL 10 minutos                     |     |
| **7**                                          | Usuario   | Ingresa el código OTP recibido.                                                                  | Máximo 3 intentos antes de re-envío obligatorio        |     |
| **8**                                          | Backend   | Valida OTP. Crea la cuenta con estado pending_verification.                                      | Contraseña almacenada con hash bcrypt (cost 12)        |     |
| **9**                                          | Sistema   | Redirige al usuario a la pantalla principal. Muestra estado de verificación pendiente si aplica. | JWT generado con rol: patient, expiración 24h          |     |
|                                                |           |                                                                                                  |                                                        |     |



| INICIAR SESION                                 |
| ---------------------------------------------- |
| **Precondiciones:**                            |
| • El usuario tiene cuenta activa y verificada. |
| • El dispositivo tiene conexión a internet.    |

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Usuario|Ingresa correo/cédula y contraseña en la pantalla de login.|Input sanitizado; no se loggea la contraseña (BR-T01)|
|**2**|Backend|Verifica credenciales contra BD. Compara hash bcrypt.|Tiempo de respuesta constante para prevenir timing attacks|
|**3**|Backend|Genera JWT con rol patient, user_id y expiración de 24h.|JWT firmado con RS256; secret en Railway (BR-T03)|
|**4**|Sistema|Redirige a la pantalla principal (Home).|Token almacenado en Secure Storage del dispositivo|

## 3.3 Flujos Alternos — Recuperación de Contraseña y MFA

### FA-AUTH-01: Recuperación de contraseña

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Usuario|Presiona 'Olvidé mi contraseña' en la pantalla de login.|—|
|**2**|Usuario|Ingresa correo o cédula registrada.|El sistema NO confirma si el correo existe (anti-enumeration)|
|**3**|Backend|Si el correo existe, envía enlace de reset con token de un solo uso (TTL 30 min).|Token HMAC-SHA256, almacenado hasheado en BD|
|**4**|Usuario|Accede al enlace, ingresa nueva contraseña y confirmación.|Requisitos: mínimo 8 chars, mayúscula, número, especial|
|**5**|Backend|Invalida el token y actualiza la contraseña. Cierra todas las sesiones activas.|Audit log registra el evento sin PII (BR-T01)|

### FA-AUTH-02: Activación de MFA para controlados

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Sistema|Detecta que el usuario intenta comprar un medicamento es_controlado = TRUE y no tiene MFA configurado.|Verificación en backend antes de renderizar pantalla de pago|
|**2**|Sistema|Solicita al usuario activar MFA (TOTP o SMS OTP) antes de continuar.|Obligatorio; sin bypass para ningún rol (BR-M06)|
|**3**|Usuario|Configura TOTP escaneando QR con app autenticadora, o registra número de teléfono para SMS.|Secreto TOTP cifrado AES-256 en reposo (Ley 81)|
|**4**|Backend|Verifica el primer código generado para confirmar configuración exitosa.|A partir de aquí, MFA es obligatorio para controlados|

## 3.4 Flujos de Excepción — AUTH

|**Código**|**Condición**|**Acción del Sistema**|**Mensaje al Usuario**|**Severidad**|
|---|---|---|---|---|
|FE-AUTH-01|Cédula no encontrada en Tribunal Electoral|Bloquea el registro. Muestra instrucción de contactar soporte.|"Cédula no encontrada en registros oficiales"|**Crítica**|
|FE-AUTH-02|API Tribunal Electoral no disponible|Registro queda en cola de verificación pendiente. Cuenta creada pero limitada.|"Verificación en proceso, te notificaremos"|**Alta**|
|FE-AUTH-03|Cédula ya registrada en el sistema|Rechaza el registro. Ofrece opción de recuperar contraseña.|"Esta cédula ya tiene una cuenta"|**Alta**|
|FE-AUTH-04|OTP incorrecto (< 3 intentos)|Permite reintento. Muestra intentos restantes.|"Código incorrecto. Intentos restantes: X"|**Media**|
|FE-AUTH-05|OTP expirado o 3 intentos fallidos|Invalida el OTP. Habilita botón de re-envío. Espera 60 seg.|"Código expirado. Solicita uno nuevo"|**Media**|
|FE-AUTH-06|Contraseña no cumple requisitos|Bloquea el avance. Muestra requisitos en tiempo real.|"La contraseña no cumple los requisitos"|**Media**|
|FE-AUTH-07|5 intentos fallidos de login|Bloquea la cuenta 15 minutos. Registra IP en audit log.|"Cuenta bloqueada temporalmente"|**Alta**|
|FE-AUTH-08|Sesión JWT expirada|Redirige al login. Intenta refresh token silencioso primero.|"Sesión expirada, ingresa nuevamente"|**Media**|

## 3.5 Controles de Seguridad — AUTH

|**Control**|**Categoría**|**Descripción**|**Normativa**|**Implementación**|
|---|---|---|---|---|
|**JWT RS256**|Autenticación|Token firmado asimétricamente. Expiración 24h.|Ley 81 / RS-006|Supabase Auth + middleware Fastify|
|**bcrypt cost 12**|Contraseñas|Hash lento que previene ataques de fuerza bruta offline.|Best practice|Supabase Auth|
|**Rate limiting login**|Brute force|Máximo 5 intentos/IP/10min. Bloqueo de 15 min al exceder.|RS-003|Middleware Fastify + Redis|
|**OTP un solo uso**|MFA|Código de 6 dígitos, TTL 10 min, invalidado tras uso.|BR-M06|Backend + SMS/Email|
|**Secure Storage**|Token storage|JWT nunca en AsyncStorage plano. Usa Expo SecureStore.|RS-005|expo-secure-store|
|**Anti-enumeration**|Privacidad|Recovery no revela si el email/cédula existe.|Ley 81|Respuesta genérica en backend|
|**Consentimiento ROPA**|Legal|Timestamp y versión del aviso aceptado registrados.|Ley 81 Art.5|Tabla consent_log|


[[2. Interface]]

# 4. Módulo INICIO — Pantalla Principal

Primera pantalla que ve el paciente al iniciar sesión. Concentra el acceso a las funciones más usadas: recetas activas, máquinas cercanas y búsqueda de medicamentos.

## 4.1 Flujo Básico — Carga del Home

|   |
|---|
|**Precondiciones:**|
|• Usuario autenticado con JWT válido.|
|• Dispositivo con GPS habilitado y conexión a internet.|

| **Paso** | **Actor** | **Descripción**                                                                             | **Seguridad**                                                 |
| -------- | --------- | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| **1**    | Sistema   | Valida JWT. Si expiró, intenta refresh token silencioso.                                    | JWT verificado en cada request (BR-T07)                       |
| **2**    | Sistema   | Solicita permiso de GPS si no ha sido otorgado previamente.                                 | El usuario puede negar; funciones de proximidad se desactivan |
| **3**    | Backend   | Recupera en paralelo: (a) recetas activas del usuario, (b) máquinas cercanas con stock > 0. | RLS: solo datos del usuario autenticado (BR-M08)              |
| **4**    | Sistema   | Renderiza el header con nombre del usuario y badge de notificaciones.                       | Notificaciones push no incluyen PII (BR-M10)                  |
| **5**    | Sistema   | Muestra tarjetas de recetas activas/parciales con badge de alerta si vencen en < 48h.       | RF-039, RF-041                                                |
| **6**    | Sistema   | Muestra módulo de dispensadores frecuentes ordenados por distancia GPS.                     | BR-M14: solo máquinas con stock > 0 del medicamento buscado   |
| **7**    | Sistema   | Muestra barra de búsqueda en el centro de la pantalla.                                      | Debounce 300ms para búsquedas en tiempo real (RF-006)         |

## 4.2 Flujos Alternos — Inicio

### FA-HOME-01: Usuario sin GPS habilitado

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Sistema|Detecta ausencia de permisos GPS.|No se bloquea la app|
|**2**|Sistema|Muestra el módulo de máquinas sin orden por distancia. Ofrece búsqueda por nombre/dirección.|Funciones de proximidad deshabilitadas|
|**3**|Usuario|Puede habilitar GPS desde configuración del dispositivo en cualquier momento.|La app detecta el cambio y re-renderiza|

### FA-HOME-02: Usuario con recetas próximas a vencer (POR AHORA NO IRA PERO POR SI ACASO LO PONGO)

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Sistema|Detecta recetas con fecha_vencimiento < NOW() + 48h.|Cron job (BR-L10) actualiza estados|
|**2**|Sistema|Muestra badge de alerta amarillo en la tarjeta de receta.|RF-041|
|**3**|Sistema|Envía notificación push genérica al dispositivo.|Push sin nombre de medicamento (BR-M10)|

## 4.3 Flujos de Excepción — Inicio

|**Código**|**Condición**|**Acción del Sistema**|**Mensaje al Usuario**|**Severidad**|
|---|---|---|---|---|
|FE-HOME-01|Error al cargar recetas activas|Muestra skeleton loader. Reintenta 1 vez automáticamente. Muestra empty state amigable.|"No se pudieron cargar tus recetas"|**Alta**|
|FE-HOME-02|Error al cargar máquinas cercanas|Muestra lista vacía con opción de reintentar. No bloquea la búsqueda.|"No se pudieron cargar las máquinas"|**Media**|
|FE-HOME-03|Sin conexión a internet|Muestra banner de modo offline. Deshabilita compra. Muestra datos del último caché local.|"Sin conexión — modo de solo lectura"|**Alta**|
|FE-HOME-04|JWT expirado en carga|Intenta refresh silencioso. Si falla, redirige a login preservando la ruta.|"Sesión expirada"|**Media**|
|FE-HOME-05|Notificaciones push fallidas|No bloquea la pantalla. Indicador discreto en el ícono de campana.|—|**Baja**|

## 4.4 Controles de Seguridad — Inicio

|**Control**|**Categoría**|**Descripción**|**Normativa**|**Implementación**|
|---|---|---|---|---|
|**RLS Supabase**|Datos propios|Recetas y compras solo del usuario autenticado. Imposible ver datos de otro paciente.|Ley 81 / BR-M08|RLS: user_id = auth.uid()|
|**Push sin PII**|Privacidad|Notificaciones no revelan medicamentos, diagnósticos ni cantidades.|Ley 81 / BR-M10|Template genérico en backend|
|**Caché local cifrado**|Offline|Datos cacheados en SQLite cifrado para modo offline.|RS-005|expo-sqlite + cifrado|
|**Refresh token silencioso**|Sesión|Renueva JWT sin interrumpir al usuario si el refresh token es válido.|UX + seguridad|Interceptor Axios|

[[Pantalla de búsqueda]]
# 5. Módulo BÚSQUEDA — Medicamentos

Permite al paciente buscar medicamentos por nombre comercial, principio activo o categoría terapéutica. Es el punto de entrada al flujo de compra.

## 5.1 Flujo Básico — Búsqueda OTC (Sin Receta)

|   |
|---|
|**Precondiciones:**|
|• Usuario autenticado.|
|• Máquina de destino seleccionada (o GPS activo para filtrar por cercanía).|

| **Paso** | **Actor** | **Descripción**                                                                                                          | **Seguridad**                                 |
| -------- | --------- | ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------- |
| **1**    | Usuario   | Escribe en la barra de búsqueda (debounce 300ms). Puede buscar por nombre, principio activo o categoría.                 | Input sanitizado; previene inyección (BR-T05) |
| **2**    | Backend   | Busca medicamentos que coincidan con el query. Retorna solo los disponibles (stock > 0) en máquinas activas.             | BR-M14: no muestra máquinas sin stock         |
| **3**    | Sistema   | Renderiza lista de resultados con: nombre, badge (Venta Libre / Requiere Receta), precio, cantidad de máquinas cercanas. | RF-007                                        |
| **4**    | Sistema   | Si el medicamento tiene genérico equivalente, lo muestra debajo con comparativo de precio (Ley 83).                      | RF-009 / BR-M04                               |
| **5**    | Usuario   | Selecciona un medicamento para ver su detalle.                                                                           | —                                             |
| **6**    | Sistema   | Abre pantalla de detalle del medicamento con información clínica completa.                                               | RF-010 / RF-013 (alerta alergias)             |
| **7**    | Sistema   | Si el usuario tiene alergia registrada relacionada con el principio activo, muestra alerta prominente.                   | BR-M11 / RF-013                               |

## 5.2 Flujo Básico — Búsqueda con Receta (Rx)(YA SE QUE ESTA EN CONVERSACION)

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Usuario|Busca el medicamento prescrito.|Igual al flujo OTC hasta el paso 3|
|**2**|Sistema|El resultado muestra badge 'Requiere Receta Médica' en color rojo/naranja.|RF-007|
|**3**|Usuario|Selecciona el medicamento.|—|
|**4**|Sistema|Detecta requiere_receta = TRUE. Verifica si el usuario tiene recetas activas/parciales para ese medicamento.|Backend — no solo cliente (BR-M07)|
|**5**|Sistema|Si hay recetas disponibles, las muestra filtradas con dosis restante.|RF-022|
|**6**|Sistema|Si la receta vence en < 48h, muestra badge de alerta.|RF-023|
|**7**|Usuario|Selecciona la receta a usar y continúa al flujo de compra.|Verificación de identidad obligatoria antes del pago (BR-M05)|

## 5.3 Flujos Alternos — Búsqueda

### FA-BUSQ-01: Sugerencia de genérico equivalente (Ley 83)

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Sistema|Detecta que el medicamento seleccionado es de marca innovadora y tiene generico_equivalente_id.|Ley 83/2012 / BR-M04|
|**2**|Sistema|Antes de la pantalla de confirmación de pago, muestra popup comparativo de precio.|El sistema registra que la sugerencia fue mostrada|
|**3**|Usuario|Puede ignorar la sugerencia o cambiar al genérico.|Decisión del usuario — ninguna opción es obligatoria|

### FA-BUSQ-02: Filtros aplicados

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Usuario|Aplica filtros: Venta Libre / Con Receta / Disponible ahora / Por distancia.|RF-008|
|**2**|Backend|Re-ejecuta la búsqueda con los parámetros de filtro.|Filtros aplicados en backend, no solo en frontend|
|**3**|Sistema|Actualiza la lista de resultados en tiempo real.|—|

## 5.4 Flujos de Excepción — Búsqueda

|**Código**|**Condición**|**Acción del Sistema**|**Mensaje al Usuario**|**Severidad**|
|---|---|---|---|---|
|FE-BUSQ-01|Sin resultados para el query|Muestra empty state con sugerencia de revisar ortografía o buscar por principio activo.|"No encontramos medicamentos para esa búsqueda"|**Baja**|
|FE-BUSQ-02|Medicamento disponible pero alerta de alergia|Muestra alerta prominente antes de mostrar el botón de compra. El usuario debe confirmar explícitamente.|"Atención: posible alergia registrada"|**Alta**|
|FE-BUSQ-03|Medicamento Rx sin recetas activas del usuario|Muestra el medicamento pero deshabilita el botón de compra. Informa que no hay recetas disponibles.|"No tienes recetas activas para este medicamento"|**Media**|
|FE-BUSQ-04|Error en la consulta al backend|Muestra resultados del caché si existen. Permite reintentar.|"Error al buscar, intenta nuevamente"|**Alta**|
|FE-BUSQ-05|Máquina sin stock durante la búsqueda|Oculta o atenúa la máquina. No guía al usuario a una máquina vacía.|"Agotado en esta máquina"|**Media**|

## 5.5 Controles de Seguridad — Búsqueda

|**Control**|**Categoría**|**Descripción**|**Normativa**|**Implementación**|
|---|---|---|---|---|
|**Input sanitizado**|Inyección|Zod valida y sanitiza todos los inputs de búsqueda en cliente y servidor.|BR-T05 / RS-007|Zod + Supabase prepared statements|
|**Alerta alergias**|Seguridad clínica|Perfil médico verificado por doctor. Alerta mostrada antes del botón de compra.|BR-M11|Tabla perfiles_medicos|
|**Genérico obligatorio**|Legal|La sugerencia de genérico debe mostrarse y registrarse antes de confirmar compra de marca.|Ley 83/2012 / BR-M04|Campo generico_equivalente_id NOT NULL justificado|
|**Stock verificado en backend**|Integridad|El stock mostrado en búsqueda se verifica nuevamente al crear la orden.|BR-I06 / BR-L05|Verificación doble: UI + orden|

[[2- Modulo MAQUINA EXPENDEDORA]]

# Módulo MAPA — Expendedoras

Permite al paciente localizar máquinas expendedoras en un mapa interactivo, ver su estado en tiempo real, consultar inventario y trazar rutas.

## 6.1 Flujo Básico — Localización y Selección de Máquina

|   |
|---|
|**Precondiciones:**|
|• Usuario autenticado.|
|• GPS habilitado en el dispositivo.|
|• Al menos una máquina activa registrada en el sistema.|

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Sistema|Carga el mapa interactivo centrado en la ubicación GPS del usuario.|RF-01 / RF-02|
|**2**|Backend|Recupera todas las máquinas con su telemetría en tiempo real (estado, stock, última señal).|Supabase Realtime — BR-A09|
|**3**|Sistema|Renderiza marcadores en el mapa con código de color: Verde (Activa), Rojo (Inactiva), Naranja (Mantenimiento).|RF-04|
|**4**|Usuario|Toca un marcador de máquina.|—|
|**5**|Sistema|Despliega tarjeta informativa con: nombre, dirección, estado (badge), distancia, última conexión e inventario.|RF-034 / RF-035|
|**6**|Sistema|Si la telemetría no se actualizó en > 15 min, muestra alerta de posible desfase en stock.|RF-034 — Alerta de desfase|
|**7**|Usuario|Presiona 'Cómo llegar'.|RF-036|
|**8**|Sistema|Abre la app de navegación nativa (Google Maps / Apple Maps) con las coordenadas de la máquina.|Deep link con coordenadas exactas|
|**9**|Usuario|Presiona 'Comprar aquí' (solo si la máquina está Activa y tiene stock).|RF-037|
|**10**|Sistema|Guarda el machine_id en el estado global. Redirige al catálogo filtrado por esa máquina.|BR-L08: solo productos del slot de esa máquina|

## 6.2 Flujos Alternos — Mapa

### FA-MAPA-01: Aplicación de filtros

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Usuario|Aplica filtros: Por provincia / Por estado (Activa, Inactiva, Mantenimiento) / Solo Favoritas.|RF-06|
|**2**|Backend|Filtra el listado de máquinas según los parámetros.|—|
|**3**|Sistema|Actualiza los marcadores del mapa en tiempo real.|Supabase Realtime|

### FA-MAPA-02: Máquina en modo offline degradado

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Sistema|Detecta que la máquina no tiene conexión MQTT por > 60 segundos.|BR-I04|
|**2**|Sistema|Muestra banner de advertencia en la tarjeta: 'Modo offline — solo OTC disponibles'.|RF-038|
|**3**|Sistema|Atenúa medicamentos Rx y controlados en el inventario de esa máquina.|BR-I04: controlados BLOQUEADOS en firmware|
|**4**|Usuario|Puede comprar OTC si tiene QR pre-autorizado. No puede comprar Rx ni controlados.|BR-I05: caché máximo 50 tokens OTC|

## 6.3 Flujos de Excepción — Mapa

|**Código**|**Condición**|**Acción del Sistema**|**Mensaje al Usuario**|**Severidad**|
|---|---|---|---|---|
|FE-MAPA-01|GPS no disponible o denegado|Centra el mapa en una vista general del país. Muestra todas las máquinas sin orden por distancia.|"Activa el GPS para ver máquinas cercanas"|**Media**|
|FE-MAPA-02|Error al cargar datos de máquinas|Muestra mapa vacío con opción de reintentar. No bloquea la navegación.|"No se pudieron cargar las máquinas"|**Alta**|
|FE-MAPA-03|Máquina seleccionada pasó a inactiva mientras se visualiza|Actualiza el badge en tiempo real. Deshabilita el botón 'Comprar aquí'.|"Esta máquina está fuera de servicio"|**Alta**|
|FE-MAPA-04|Stock de medicamento llegó a 0 mientras se visualiza|Atenúa el medicamento en el inventario. Deshabilita el botón 'Añadir al carrito'.|"Agotado en esta máquina"|**Media**|
|FE-MAPA-05|Error al abrir app de navegación nativa|Muestra las coordenadas en texto para que el usuario pueda copiarlas.|"No se pudo abrir la navegación"|**Baja**|
|FE-MAPA-06|Telemetría desactualizada (> 15 min)|Muestra advertencia de stock posiblemente desactualizado. No bloquea la compra.|"Stock podría no estar actualizado"|**Media**|

## 6.4 Controles de Seguridad — Mapa

|**Control**|**Categoría**|**Descripción**|**Normativa**|**Implementación**|
|---|---|---|---|---|
|**machine_id en state global**|Integridad de compra|Solo se pueden añadir al carrito productos del slot de la máquina seleccionada.|BR-L08|Context API / Zustand|
|**Supabase Realtime**|Datos en tiempo real|Estado de máquinas actualizado sin polling manual. Latencia < 5 seg.|BR-A09|Supabase Realtime subscriptions|
|**Botón deshabilitado offline**|Prevención errores|El botón 'Comprar aquí' está deshabilitado si la máquina no está activa.|BR-I04 / RF-037|Estado condicional en UI|
|**Alerta desfase stock**|Transparencia|Usuario informado si la telemetría lleva > 15 min sin actualizar.|RF-034|Campo last_telemetry en BD|

# COMPRAR RECETA

# 7. Módulo COMPRA — Checkout y Dispensación

Es el módulo más crítico del sistema desde el punto de vista de seguridad y cumplimiento legal. Gestiona el flujo completo desde la selección de cantidad hasta la dispensación física del medicamento.

|   |
|---|
|**Máquina de estados de la orden:**|
|CREATED → PAYMENT_PENDING → PAID → TOKEN_GENERATED → DISPENSING → COMPLETED|
|Compensaciones: PAYMENT_PENDING → PAYMENT_TIMEOUT \| PAID → REFUNDING → REFUNDED \| TOKEN_GENERATED → EXPIRED|
|Ninguna transición puede saltar un estado intermedio.|

## 7.1 Flujo Básico — Compra OTC (Sin Receta)

|   |
|---|
|**Precondiciones:**|
|• Usuario autenticado con método de pago vinculado.|
|• Máquina seleccionada en estado Activa con stock > 0.|
|• Medicamento es OTC (requiere_receta = FALSE).|

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Usuario|Selecciona cantidad (controles +/-). Límite = min(stock disponible, límite_diario_OTC).|BR-L05: stock reservado lógicamente al seleccionar|
|**2**|Sistema|Si hay genérico equivalente disponible en la misma máquina, muestra popup comparativo.|Ley 83/2012 / BR-M04 — registro de que fue mostrado|
|**3**|Sistema|Muestra pantalla de resumen: medicamento, máquina, cantidad, precio total.|RF-09 / BR-L06: precio fijado en este momento|
|**4**|Usuario|Confirma y presiona 'Confirmar y Pagar'.|—|
|**5**|Sistema|Selecciona método de pago (Yappy o tarjeta guardada).|Datos de tarjeta enmascarados en pantalla|
|**6**|Backend|Crea la orden en estado CREATED. Reserva el stock del slot. Inicia transacción Yappy.|BR-L05: stock_reservado++|
|**7**|Yappy|Procesa el pago. Timeout: 5 minutos. Si no responde en 10s → PAYMENT_TIMEOUT.|BR-M12: pago confirmado ANTES de generar QR|
|**8**|Backend|Recibe confirmación de pago. Orden pasa a PAID → TOKEN_GENERATED. Genera QR con UUID v4 firmado HMAC-SHA256.|BR-I01: TTL 15 min, un solo uso|
|**9**|Sistema|Muestra QR al usuario con temporizador de cuenta regresiva.|RF-017|
|**10**|Usuario|Se dirige a la máquina y escanea el QR.|—|
|**11**|Máquina|Lee el QR. Envía token al backend para verificación.|BR-I02: la máquina nunca actúa sola|
|**12**|Backend|Verifica: (a) firma HMAC, (b) estado PENDING, (c) no usado. Envía comando MQTT firmado a la máquina.|mTLS entre backend y broker HiveMQ (BR-I11)|
|**13**|Máquina|Recibe comando MQTT. Dispensa el medicamento. Confirma vía MQTT.|BR-I10: dispensación atómica|
|**14**|Backend|Recibe ACK. Orden → COMPLETED. Stock_reservado→dispensado. Decrementa stock total.|BR-I06: Supabase Realtime actualiza el mapa|
|**15**|Sistema|Muestra pantalla de confirmación de retiro exitoso con resumen.|RF-018|

## 7.2 Flujo Básico — Compra con Receta (EN DISCUSION)

|   |
|---|
|**Precondiciones adicionales:**|
|• El usuario tiene receta activa o parcial para el medicamento.|
|• La receta está vigente (fecha_vencimiento > NOW()).|
|• La cantidad solicitada no supera el saldo disponible de la receta.|

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Backend|Verifica identidad del usuario contra API Tribunal Electoral ANTES del pago.|BR-M05 / RF-020: obligatorio, sin excepción|
|**2**|Sistema|Muestra listado de recetas activas filtradas por el medicamento seleccionado.|RF-022|
|**3**|Usuario|Selecciona la receta a usar y la cantidad (máx: saldo_disponible).|BR-L04: verificación atómica en BD|
|**4**|Backend|Calcula: saldo_disponible = cantidad_prescrita - cantidad_dispensada. Verifica límite.|Transacción PostgreSQL para prevenir condición de carrera|
|**5**|Sistema|Continúa con el flujo de pago igual al OTC (pasos 3-15 del flujo 7.1).|—|
|**6**|Backend|Tras dispensación exitosa: actualiza cantidad_dispensada en la receta. Verifica si agotó la receta.|BR-L03: receta pasa a PARTIAL o EXHAUSTED|

## 7.3 Flujo Básico — Compra con Receta + Medicamento Controlado

|   |
|---|
|**Este flujo activa los controles de seguridad más estrictos del sistema.**|
|Es un flujo crítico bajo la regulación de la CSS y la Ley 1/2001.|
|No existe ningún bypass ni modo degradado para este flujo.|

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Backend|Detecta es_controlado = TRUE. Verifica que es_controlado implica requiere_receta = TRUE.|BR-L07: constraint PostgreSQL CHECK|
|**2**|Backend|Verifica identidad en Tribunal Electoral (igual que Rx).|BR-M05: obligatorio|
|**3**|Sistema|Solicita segundo factor de autenticación (TOTP o SMS OTP).|BR-M06: MFA obligatorio para controlados — sin bypass|
|**4**|Backend|Verifica el código MFA. Si falla, bloquea el flujo.|Máximo 3 intentos antes de bloqueo temporal|
|**5**|Backend|Verifica límite diario: máximo 1 receta de controlado por paciente por día.|BR-M13: SELECT COUNT(*) dispensaciones_dia_actual|
|**6**|Sistema|Si el límite diario ya se alcanzó, bloquea completamente el flujo.|"Límite diario de controlados alcanzado"|
|**7**|Backend / Máquina|Al momento de dispensar: la cámara de la máquina captura foto del usuario.|BR-I03: foto cifrada en Supabase Storage — obligatorio|
|**8**|Backend|Registra dispensación en audit log inmutable con: identidad, fecha, cantidad, médico prescriptor, foto.|CSS: trazabilidad mínima 5 años (BR-A07)|
|**9**|Sistema|Continúa con el flujo de pago y dispensación normal.|Todos los pasos de 7.1 aplican adicionalmente|

## 7.4 Flujos Alternos — Compra

### FA-COMPRA-01: QR escaneado en máquina diferente a la seleccionada

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Máquina B|Escanea QR generado para Máquina A.|—|
|**2**|Backend|Verifica machine_id del token vs machine_id de la máquina que escanea.|BR-L08|
|**3**|Backend|Rechaza el comando. Token permanece en estado PENDING.|El usuario puede ir a la máquina correcta dentro del TTL|

### FA-COMPRA-02: Pago con tarjeta guardada

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Usuario|Selecciona tarjeta guardada (mostrada enmascarada: **** **** **** 1234).|Tokenización PCI-DSS — el número real nunca en el sistema|
|**2**|Backend|Usa el token de la tarjeta para iniciar el cobro.|—|
|**3**|Sistema|Flujo continúa igual que el pago con Yappy.|—|

## 7.5 Flujos de Excepción — Compra

|**Código**|**Condición**|**Acción del Sistema**|**Mensaje al Usuario**|**Severidad**|
|---|---|---|---|---|
|FE-COMPRA-01|Stock llega a 0 entre la selección y la confirmación|Bloquea la compra. Libera la reserva. Ofrece buscar otra máquina.|"Stock agotado, elige otra máquina"|**Crítica**|
|FE-COMPRA-02|Pago rechazado por Yappy|Muestra error con opción de reintentar o cambiar método. No genera QR.|"Pago rechazado, verifica tus datos"|**Alta**|
|FE-COMPRA-03|Timeout de pago (> 10 segundos sin respuesta)|Orden → PAYMENT_TIMEOUT. Libera stock reservado. Permite reintentar.|"Tiempo de espera agotado"|**Alta**|
|FE-COMPRA-04|QR expirado (TTL 15 min sin escanear)|Token → EXPIRED. Reembolso automático iniciado. Reintento de reembolso con backoff exponencial.|"QR expirado — reembolso iniciado (BR-L09)"|**Crítica**|
|FE-COMPRA-05|Fallo mecánico de la máquina al dispensar|Token → FAILED. Reembolso automático. Audit log registra el fallo.|"Error en dispensación — reembolso iniciado"|**Crítica**|
|FE-COMPRA-06|Límite diario de controlados alcanzado|Bloquea completamente el flujo. No hay excepción ni bypass posible.|"Límite diario de controlados alcanzado"|**Crítica**|
|FE-COMPRA-07|Verificación Tribunal Electoral falla (servicio no disponible)|Bloquea compras Rx y controlados. Solo permite OTC.|"Verificación de identidad no disponible"|**Crítica**|
|FE-COMPRA-08|MFA incorrecto para controlado (< 3 intentos)|Permite reintento. Muestra intentos restantes.|"Código incorrecto. Intentos restantes: X"|**Alta**|
|FE-COMPRA-09|Cámara de máquina no operativa en compra de controlado|Bloquea la dispensación hasta restaurar la cámara.|"Dispensación no disponible en esta máquina"|**Crítica**|
|FE-COMPRA-10|Reembolso automático falla > 24h|Escala a cola de reembolsos manuales con alerta al admin. Nunca se descarta.|Admin recibe alerta crítica en dashboard|**Crítica**|

## 7.6 Controles de Seguridad — Compra

|**Control**|**Categoría**|**Descripción**|**Normativa**|**Implementación**|
|---|---|---|---|---|
|**HMAC-SHA256 en QR**|Anti-falsificación|QR firmado con clave HMAC única por máquina. Verificación en backend.|BR-I01 / RS-001|UUID v4 + HMAC + TTL 15min|
|**mTLS backend-broker**|Comunicación segura|Certificado TLS único por máquina. Rotación cada 90 días.|BR-I11 / RS-005|HiveMQ + OpenSSL|
|**Saga pattern**|Consistencia|Flujo atómico pago→token→dispensación→stock. Compensaciones automáticas.|BR-I10 / RNF-010|Máquina de estados en BD|
|**Lock distribuido**|Concurrencia|Un token solo puede estar en DISPENSING en una máquina a la vez.|BR-L02|Redis / Supabase advisory lock|
|**Stock doble verificación**|Sobreventa|Verificación en UI (selección) y en backend (creación de orden).|BR-I06 / BR-L05|Transacción PostgreSQL atómica|
|**Audit log inmutable**|Trazabilidad|Solo INSERT en schema audit. Ningún rol puede hacer UPDATE/DELETE.|CSS / BR-A07|RLS INSERT-only en Supabase|
|**Idempotency key**|Doble cobro|Cada operación de pago tiene idempotency_key único del cliente.|BR-L12|UUID v4 generado en cliente|
|**Precio fijado al crear orden**|Integridad comercial|El precio no se recalcula al dispensar. Fijado en CREATED.|BR-L06|Campo precio_pagado NOT NULL|


[[1. Descripción general de Recetas]]

# 8. Módulo RECETAS — Gestión

Permite al paciente gestionar todas sus recetas médicas digitales: verlas clasificadas por estado, consultar el detalle, ver el historial de dispensaciones y usarlas para iniciar una compra.

## 8.1 Flujo Básico — Visualización y Uso de Receta

|   |
|---|
|**Precondiciones:**|
|• Usuario autenticado.|
|• El médico emisor tiene idoneidad activa (BR-A01).|
|• La receta fue firmada criptográficamente al momento de la emisión (BR-A02).|

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Sistema|Carga el listado de recetas del usuario clasificadas por estado: Activas, Parciales, Agotadas, Vencidas.|RLS: solo recetas del usuario autenticado (BR-M08)|
|**2**|Sistema|Cada tarjeta muestra: medicamento, dosis, médico emisor, fecha de vencimiento, unidades disponibles.|RF-040|
|**3**|Sistema|Muestra badge de alerta en recetas que vencen en < 48h.|RF-041 / BR-L10|
|**4**|Usuario|Toca una tarjeta para ver el detalle completo.|—|
|**5**|Sistema|Muestra detalle: principio activo, dosis indicada, frecuencia, cantidad prescrita, dispensada y restante.|RF-043|
|**6**|Sistema|Muestra información del médico emisor: nombre, especialidad, número de idoneidad.|RF-044|
|**7**|Sistema|Muestra historial de dispensaciones: fecha, máquina, cantidad dispensada.|RF-045|
|**8**|Sistema|Muestra botón 'Usar esta receta' solo si: estado = activa o parcial AND hay stock en máquina cercana.|RF-046 / BR-M07|
|**9**|Usuario|Presiona 'Usar esta receta'. Inicia el flujo de compra Rx (Módulo 7.2).|—|
|**10**|Sistema|Muestra QR de presentación (solo visual para farmacia física, no válido para expendedora).|RF-047 — Modo presentación, sin valor de dispensación|

## 8.2 Flujos Alternos — Recetas

### FA-RECETA-01: Búsqueda de receta por nombre de medicamento

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Usuario|Escribe en la barra de búsqueda del módulo de recetas.|RF-042|
|**2**|Sistema|Filtra el listado en tiempo real por nombre de medicamento.|Búsqueda local en datos ya cargados|

### FA-RECETA-02: Receta suspendida por médico desactivado

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Admin|Desactiva un médico en el panel administrativo.|BR-A11|
|**2**|Backend|Todas las recetas activas/parciales del médico pasan a estado SUSPENDED_BY_DOCTOR.|Cron job o trigger inmediato|
|**3**|Sistema|El paciente ve la receta en estado 'Suspendida'. Se muestra explicación.|RF-046: botón 'Usar' deshabilitado|
|**4**|Sistema|El paciente recibe notificación push genérica informando que una receta fue suspendida.|BR-M10: push sin datos médicos sensibles|

## 8.3 Flujos de Excepción — Recetas

|**Código**|**Condición**|**Acción del Sistema**|**Mensaje al Usuario**|**Severidad**|
|---|---|---|---|---|
|FE-RECETA-01|Receta expirada detectada en carga|Marca la receta como EXPIRED. Deshabilita el botón 'Usar'. Muestra fecha de vencimiento.|"Esta receta ha vencido"|**Media**|
|FE-RECETA-02|Receta agotada (cantidad_dispensada = cantidad_prescrita)|Marca como EXHAUSTED. Deshabilita el botón 'Usar'.|"Esta receta está agotada"|**Media**|
|FE-RECETA-03|Firma digital de receta inválida|Rechaza el uso de la receta. No permite dispensación.|"Receta inválida o modificada"|**Crítica**|
|FE-RECETA-04|Médico con idoneidad suspendida al momento de usar la receta|Bloquea el uso aunque la receta esté vigente.|"Receta no disponible"|**Crítica**|
|FE-RECETA-05|Sin máquinas cercanas con stock del medicamento|Deshabilita el botón 'Usar esta receta'. Muestra mapa con máquinas más lejanas.|"No hay máquinas con stock disponible"|**Alta**|
|FE-RECETA-06|Error al cargar historial de dispensaciones|Oculta esa sección. Muestra el resto del detalle normalmente.|"No se pudo cargar el historial"|**Baja**|

## 8.4 Controles de Seguridad — Recetas

|**Control**|**Categoría**|**Descripción**|**Normativa**|**Implementación**|
|---|---|---|---|---|
|**Firma digital receta**|Anti-falsificación|Cada receta firmada con credenciales del médico al emitirla. Cualquier modificación invalida la firma.|BR-A02 / RS-002|Backend firma al emitir — cliente nunca firma|
|**Verificación de idoneidad en cada uso**|Legal|El estado activo del médico se verifica en cada solicitud de compra, no solo al emitir.|BR-A01 / Ley 1/2001|Query al campo status del médico|
|**Máquina de estados receta**|Integridad|ACTIVE → PARTIAL → EXHAUSTED/EXPIRED. Transiciones irreversibles.|BR-L03|Trigger PostgreSQL + cron|
|**QR presentación != QR dispensación**|Claridad|El QR de presentación para farmacia física no tiene valor para la expendedora.|RF-047|Tipo diferenciado en BD|
|**Notificaciones sin PII**|Privacidad|Push de suspensión de receta no menciona el medicamento.|Ley 81 / BR-M10|Template genérico|


[[2. Pantalla principal de perfil]]

# 9. Módulo PERFIL — Usuario

Permite al paciente gestionar su información personal, perfil médico, métodos de pago, seguridad y preferencias de privacidad.

## 9.1 Flujo Básico — Gestión del Perfil

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Sistema|Carga la información del usuario: foto, nombre, cédula (bloqueada si verificada), correo, estado de verificación.|RF-048 / RF-050|
|**2**|Sistema|Muestra accesos directos: Historial de compras, Mis recetas, Métodos de pago, Configuración, Cerrar sesión.|RF-049|
|**3**|Usuario|Edita foto de perfil, correo o teléfono. La cédula NO es editable una vez verificada.|RF-051: cédula UNIQUE NOT NULL verificada|
|**4**|Usuario|Accede a Perfil Médico. Registra alergias y condiciones crónicas.|RF-053 / RF-054: información verificable por el doctor|
|**5**|Sistema|Muestra aviso explícito: los datos de salud se usan solo localmente para alertas. No se comparten con terceros.|RF-055 / Ley 81 / BR-A10|
|**6**|Usuario|Accede a Métodos de Pago. Ve tarjetas enmascaradas y Yappy vinculado.|RF-056|
|**7**|Usuario|Desvincula un método de pago. Si es el único, el sistema pide confirmación explícita.|RF-057|
|**8**|Usuario|Accede a Configuración. Gestiona notificaciones push por tipo.|RF-058|
|**9**|Usuario|Activa/desactiva MFA para controlados (obligatorio si tiene historial de controlados).|RF-059 / BR-M06|
|**10**|Usuario|Cambia contraseña con validación de la actual y requisitos de complejidad.|RF-060: contraseña actual requerida|

## 9.2 Flujos Alternos — Perfil

### FA-PERFIL-01: Eliminación de cuenta

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Usuario|Presiona 'Eliminar mi cuenta' en Configuración.|RF-061|
|**2**|Sistema|Muestra pantalla de confirmación con consecuencias claras.|Flujo de confirmación en 2 pasos|
|**3**|Usuario|Confirma la eliminación.|—|
|**4**|Backend|Aplica soft delete en datos personales (nombre, correo, foto, teléfono).|Ley 81 Art.22: derecho al olvido (BR-M09)|
|**5**|Backend|Conserva el audit log con identidad anonimizada (SHA-256 con salt). Retención mínima 5 años.|CSS: trazabilidad inmutable (BR-M09)|
|**6**|Sistema|Cierra la sesión y redirige al onboarding.|Todos los JWT activos son revocados|

### FA-PERFIL-02: Verificación de identidad pendiente

|**Paso**|**Actor**|**Descripción**|**Seguridad**|
|---|---|---|---|
|**1**|Sistema|Detecta que el usuario no ha completado la verificación de cédula.|RF-050|
|**2**|Sistema|Muestra banner prominente en la pantalla de perfil con CTA 'Verificar identidad'.|Sin verificación: no puede comprar medicamentos Rx|
|**3**|Usuario|Inicia el proceso de verificación. Ingresa cédula para validar contra Tribunal Electoral.|BR-M01: verificación obligatoria para Rx|

## 9.3 Flujos de Excepción — Perfil

|**Código**|**Condición**|**Acción del Sistema**|**Mensaje al Usuario**|**Severidad**|
|---|---|---|---|---|
|FE-PERFIL-01|Error al guardar cambios de perfil|Conserva los datos previos. Muestra opción de reintentar.|"No se pudieron guardar los cambios"|**Media**|
|FE-PERFIL-02|Intento de editar la cédula verificada|Campo bloqueado en la interfaz. El backend también rechaza cambios.|"La cédula no es editable una vez verificada"|**Alta**|
|FE-PERFIL-03|Error al desvincular método de pago|Conserva el método. Muestra opción de reintentar.|"No se pudo desvincular el método"|**Media**|
|FE-PERFIL-04|Intentar eliminar el único método de pago|Muestra confirmación especial. Advierte que no podrá comprar sin método.|"¿Seguro? No tendrás método de pago activo"|**Media**|
|FE-PERFIL-05|Error al actualizar contraseña (contraseña actual incorrecta)|Rechaza el cambio. Registra el intento fallido.|"Contraseña actual incorrecta"|**Alta**|
|FE-PERFIL-06|Fallo en la eliminación de cuenta (rollback)|Restaura el estado anterior. No aplica soft delete parcial.|"No se pudo eliminar la cuenta"|**Crítica**|

## 9.4 Controles de Seguridad — Perfil

|**Control**|**Categoría**|**Descripción**|**Normativa**|**Implementación**|
|---|---|---|---|---|
|**Soft delete + anonimización**|Derecho al olvido|Eliminar cuenta: PII removido, audit log conservado con SHA-256 salt.|Ley 81 Art.22 / BR-M09|Función anonymize_patient_audit(user_id)|
|**Cédula inmutable post-verificación**|Integridad|La cédula no puede modificarse una vez verificada ni en cliente ni en backend.|BR-M01|Constraint NOT NULL + lógica backend|
|**Admin sin acceso a perfil médico**|Segregación|El admin operativo solo ve estadísticas agregadas. No puede leer datos médicos individuales.|Ley 81 / BR-A10|RLS por rol en Supabase|
|**Datos salud solo locales**|Privacidad|Alergias y condiciones crónicas: solo para alertas locales. No se comparten.|RF-055 / Ley 81|Aviso explícito en UI|
|**Versión app y políticas**|Transparencia|Siempre accesible la versión, términos de uso y políticas de privacidad.|RF-062 / Ley 81|Sección fija en Configuración|