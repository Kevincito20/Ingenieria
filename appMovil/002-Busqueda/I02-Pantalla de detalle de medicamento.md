#Requisitos_funcionales 

## Descripción de la pantalla
La pantalla de "Detalles de Medicamento" es el núcleo informativo del catálogo. Su objetivo principal es brindar al usuario toda la información clínica, de disponibilidad y comercial necesaria para tomar una decisión de compra segura. Esta vista combina detalles visuales del producto con datos en tiempo real sobre el inventario de la red de máquinas expendedoras, culminando en la acción de agregar el producto a la orden.

## Requisitos funcionales


| Id     | Funcionalidad                                                                                                        | Prioridad |
| ------ | -------------------------------------------------------------------------------------------------------------------- | --------- |
| RF-010 | Mostrar información completa: nombre, principio activo, dosis, presentación, precio, requiere receta, es controlado. |           |
| RF-011 | Mostrar lista de máquinas que tienen el medicamento disponible con stock actual y distancia, ordenadas por cercanía. |           |
| RF-012 | Botón "Comprar" que inicia el flujo OTC o con receta según el tipo de medicamento                                    |           |
| RF-013 | Mostrar alerta visible si el usuario tiene alergia registrada relacionada con el principio activo del medicamento.   |           |
|        |                                                                                                                      |           |


## Módulo de Ubicación (Header / Selector de Máquina)

- **Descripción:** Ubicado en la franja superior de la pantalla.
-
**Especificaciones de Interfaz (UI/UX):**


- **Elementos:** Una lista desplegable o un carrusel de etiquetas que muestra en qué máquinas expendedoras se encuentra disponible este medicamento específico. Si el usuario ya viene con una máquina seleccionada desde el mapa, esta se mostrará por defecto, pero podrá visualizar otras opciones cercanas.





	


## Flujo de Compra: Máquina Expendedora de Medicamentos(sin receta)

**Fase 1: Selección y Configuración**

1. **Selección del producto:** El usuario busca y selecciona el medicamento de su elección en el catálogo.
    
2. **Verificación de ubicación:** El usuario verifica y confirma que la máquina expendedora donde realizará la compra (y el retiro) sea la correcta.
    
3. **Selección de cantidad:** El usuario elige el número de unidades que desea adquirir de dicho medicamento.
    
4. **Agregar al carrito:** El usuario presiona el botón **"Agregar al carrito"**. _(Nota: El usuario puede repetir los pasos del 1 al 4 si desea comprar múltiples medicamentos diferentes)._
    

**Fase 2: Revisión y Checkout** 5. **Acceso al carrito:** Una vez elegidos todos los productos, el usuario se dirige a la pantalla del carrito de compras. 6. **Iniciar proceso de pago:** El usuario presiona el botón **"Confirmar orden y pagar"**. 7. **Selección de método de pago:** El sistema solicita al usuario que elija cómo desea pagar (tarjeta de crédito, billetera digital, etc.). 8. **Validación del resumen de compra:** Antes de cobrar, el sistema muestra una pantalla de revisión donde el usuario debe validar que la información sea correcta. Esto incluye:

- Medicamentos seleccionados.
    
- Cantidad de cada medicamento.
    
- Máquina expendedora donde lo va a retirar.
    
- Total a pagar de la compra.
    

9. **Confirmación final:** Al estar todo correcto, el usuario presiona el botón **"Pagar"**.
    

**Fase 3: Entrega** 10. **Generación de QR:** El sistema procesa el pago exitosamente y le proporciona al usuario un código QR en pantalla. 11. **Retiro físico:** El usuario se dirige a la máquina expendedora correspondiente, escanea el código QR proporcionado por el sistema y la máquina dispensa los medicamentos.




### Flujo de compra sin receta médica
RF-014Confirmar medicamento, máquina seleccionada, cantidad (dentro del límite diario) y precio total antes del pago.

Paso 2 — Pago

RF-015 Iniciar pago con Yappy mostrando el monto, nombre del medicamento y número de orden. Timeout de pago: 5 minutos.

RF-016 Mostrar pantalla de error con opción de reintentar si el pago es rechazado. No generar token si el pago falla.

Paso 3 — QR de retiro

RF-017 Mostrar QR de dispensación con temporizador de cuenta regresiva (TTL 15 minutos) y nombre de la máquina destino.
RF-018 Mostrar pantalla de confirmación de retiro exitoso con resumen de compra al recibir ACK del backend.

RF-019 Mostrar pantalla de error y opción de soporte si el QR expira sin ser escaneado. Iniciar reembolso automático.

### Flujo de compra con receta médica

Paso 1 — Verificación de identidad

RF-020 Solicitar número de cédula y verificar identidad contra API del Tribunal Electoral. Mostrar resultado en ≤ 5 segundos.

RF-021 Solicitar segundo factor (MFA por TOTP o SMS) para medicamentos controlados.

Paso 2 — Selección de receta

RF-022 Mostrar listado de recetas activas del usuario filtradas por el medicamento seleccionado, con dosis disponible restante.

RF-023 Mostrar alerta si la receta expira en menos de 48 horas.

Paso 3 — Pago y QR

RF-024 Confirmar cantidad a dispensar (no puede superar la cantidad prescrita restante). Mostrar desglose claro

RF-025 Generar QR de dispensación firmado tras confirmación de pago. Igual que OTC (RF-017, RF-018, RF-019).


## Resultado final

![[Pasted image 20260405195915.png]]
