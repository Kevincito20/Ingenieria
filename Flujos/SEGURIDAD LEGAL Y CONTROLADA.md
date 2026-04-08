
# APP MÓVIL

## Autenticación y Control de Acceso

|**Control**|**Categoría**|**Descripción**|**Normativa**|**Implementación**|
|---|---|---|---|---|
|**JWT RS256**|Autenticación|Token firmado asimétricamente para cada sesión de paciente.|Ley 81 / RS-006|Supabase Auth + middleware Fastify|
|**bcrypt cost 12**|Contraseñas|Hash lento que previene ataques de fuerza bruta offline.|Best practice|Supabase Auth|
|**MFA TOTP/SMS**|Multi-factor|Obligatorio para controlados. Configurable para todo uso.|BR-M06 / CSS|expo-totp / SMS gateway|
|**Secure Storage**|Token storage|JWT en Expo SecureStore, nunca en AsyncStorage.|RS-005|expo-secure-store|
|**RLS Supabase**|Autorización BD|user_id = auth.uid() en todas las tablas de paciente.|Ley 81 / BR-M08|Políticas RLS en Supabase|
|**RBAC doble capa**|Autorización|Middleware Fastify + RLS PostgreSQL. Ambas capas independientes.|BR-T07 / RS-006|JWT claim rol + RLS|

# Protección de Datos

|**Control**|**Categoría**|**Descripción**|**Normativa**|**Implementación**|
|---|---|---|---|---|
|**TLS 1.3 obligatorio**|Tránsito|Toda comunicación app ↔ backend bajo TLS 1.3. Sin versiones inferiores.|BR-T04 / Ley 81|Certificate pinning en React Native|
|**AES-256 en reposo**|Almacenamiento|Datos médicos cifrados en reposo en Supabase.|Ley 81 Art.5|Supabase encryption at rest|
|**PII mínimo en logs**|Privacidad logs|Logs de app nunca contienen cédulas, nombres ni medicamentos Rx.|BR-T01 / Ley 81|Lista de campos prohibidos en logger|
|**Push sin PII**|Notificaciones|Notificaciones push con texto genérico. Sin datos médicos.|BR-M10 / Ley 81|Template server-side|
|**Soft delete + hash**|Derecho al olvido|Al eliminar cuenta: PII removido, audit log con SHA-256 salt conservado.|Ley 81 Art.22 / BR-M09|anonymize_patient_audit()|
|**Consentimiento granular**|Legal|Consentimiento explícito en onboarding. Timestamp y versión registrados.|Ley 81 Art.5 / BR-M02|Tabla consent_log|

##  Integridad de Transacciones

|**Control**|**Categoría**|**Descripción**|**Normativa**|**Implementación**|
|---|---|---|---|---|
|**Saga pattern**|Atomicidad|Pago → token → dispensación → stock como unidad atómica con compensaciones.|BR-I10 / RNF-010|Máquina de estados en BD|
|**Idempotency key**|Anti duplicados|Cada operación de pago tiene clave única del cliente para prevenir doble cobro.|BR-L12|UUID v4 en header X-Idempotency-Key|
|**Stock doble verificación**|Sobreventa|UI y backend verifican stock antes de crear la orden.|BR-I06 / BR-L05|SELECT FOR UPDATE en PostgreSQL|
|**Precio inmutable**|Integridad precio|precio_pagado fijado al crear la orden. No se recalcula.|BR-L06|Campo NOT NULL en tabla ordenes|
|**Audit log insert-only**|Trazabilidad|Schema audit solo permite INSERT. Ningún rol puede modificar o eliminar.|CSS / BR-A07|RLS INSERT-only en Supabase|

## Seguridad del Canal App-Expendedora

|**Control**|**Categoría**|**Descripción**|**Normativa**|**Implementación**|
|---|---|---|---|---|
|**QR HMAC-SHA256**|Anti-falsificación|QR firmado con clave HMAC única por máquina. TTL 15 min. Un solo uso.|BR-I01|UUID v4 + HMAC + estado PENDING→CONSUMED|
|**mTLS MQTT**|Canal IoT|Certificado TLS único por máquina para la conexión MQTT.|BR-I11 / RS-005|HiveMQ + OpenSSL. Rotación 90 días|
|**Verificación en backend**|Confianza cero|La máquina nunca actúa solo con el QR. Siempre verifica en backend antes.|BR-I02|Arquitectura: QR → máquina → backend → MQTT → máquina|
|**Fail secure**|Modo seguro|Sin backend activo: controlados BLOQUEADOS en firmware. Nunca fail open.|BR-T06 / BR-I04|Lógica local en firmware|
|**Lock distribuido**|Concurrencia|Token solo puede estar en DISPENSING en una máquina a la vez.|BR-L02|Redis o Supabase advisory lock|

## Cumplimiento Legal Panameño

|**Control**|**Categoría**|**Descripción**|**Normativa**|**Implementación**|
|---|---|---|---|---|
|**Ley 81/2019**|Datos personales|Consentimiento, cifrado, derecho al olvido, ROPA, ANTAI.|Ley 81/2019|consent_log + anonymize + ROPA documentado|
|**Ley 1/2001**|Farmacia|Trazabilidad Rx, supervisión farmacéutica remota, idoneidad activa.|Ley 1/2001|audit_log + verificación idoneidad en cada uso|
|**Ley 83/2012**|Genéricos|Mostrar equivalente genérico antes de confirmar compra de marca.|Ley 83/2012|generico_equivalente_id + registro de visualización|
|**Reglamento CSS**|Controlados|Audit log inmutable 5 años, foto al dispensar, límite diario.|CSS|Schema audit INSERT-only + cámara + BR-M13|
|**Tribunal Electoral**|Identidad|Verificación de cédula obligatoria para Rx y controlados.|BR-M01 / BR-M05|API TE + rate limiting BR-T02|

# EXPENDEDORA FÍSICA
