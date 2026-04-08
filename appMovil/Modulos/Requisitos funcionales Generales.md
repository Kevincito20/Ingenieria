#Requisitos_funcionales 

| Código- | Funcionalidad                                                                                                                  | Prioridad |
| ------- | ------------------------------------------------------------------------------------------------------------------------------ | --------- |
| RFI-001 | Mostrar listado de máquinas expendedoras cercanas ordenadas por distancia al usuario, con indicador de estado activa/inactiva. | Alta      |

^c5a81e


| RFI-002 | Mostrar ~~banner~~ (barra??) de búsqueda rápida de medicamentos por nombre comercial o principio activo. | Alta |
| ------- | -------------------------------------------------------------------------------------------------------- | ---- |
|         |                                                                                                          |      |

^289c0a

| RFI-003 | Mostrar acceso rápido a "Última compra" con botón de repetir pedido si el medicamento sigue disponible | Media |
| ------- | ------------------------------------------------------------------------------------------------------ | ----- |

| RFI-004 | ~~Mostrar notificaciones pendientes (receta próxima a vencer, pedido listo para retirar) en banner visible.~~ Mostrar notificaciones de receta(Nueva receta, receta cercana a fecha de vencimiento, medicamentos faltantes) | Alta |
| ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---- |

^ea0f95

| RFI-005 | Mostrar indicador visual si el usuario tiene recetas activas disponibles para usar. | Alta |
| ------- | ----------------------------------------------------------------------------------- | ---- |

^6e489b

| RF-010 | Mostrar información completa: nombre, principio activo, dosis, presentación, precio, requiere receta, es controlado. |     |
| ------ | -------------------------------------------------------------------------------------------------------------------- | --- |

| RF-011 | Mostrar lista de máquinas que tienen el medicamento disponible con stock actual y distancia, ordenadas por cercanía. |     |
| ------ | -------------------------------------------------------------------------------------------------------------------- | --- |

| RF-012 | Botón "Comprar" que inicia el flujo OTC o con receta según el tipo de medicamento |     |
| ------ | --------------------------------------------------------------------------------- | --- |

| RF-013 | Mostrar alerta visible si el usuario tiene alergia registrada relacionada con el principio activo del medicamento. |     |
| ------ | ------------------------------------------------------------------------------------------------------------------ | --- |
|        |                                                                                                                    |     |


## RF-09: Generación del Resumen de Transacción**

- El sistema debe mostrar una pantalla de confirmación obligatoria previa al pago que presente de forma clara los siguientes datos:
	-  **Medicamento:** Nombre comercial y/o genérico y concentración.
   
    - **Máquina seleccionada:** Identificador único o nombre de la ubicación de la máquina donde se retirará el producto.
   
    - **Cantidad:** Número de unidades que el usuario pretende adquirir.

    - **Precio Total:** El costo final de la transacción, incluyendo impuestos o tasas aplicables.


## **RF-10: Validación de Límites de Dispensación**

-  El sistema debe verificar en tiempo real que la cantidad solicitada por el usuario no exceda el **límite diario permitido** para ese medicamento específico.

- Si la cantidad excede el límite, el sistema debe impedir el avance al pago y mostrar un mensaje de error explicando la restricción (por motivos de seguridad sanitaria).

## **RF-11: Sincronización de Disponibilidad en Máquina**

- Antes de permitir la confirmación, el sistema debe validar que la máquina seleccionada aún cuenta con el stock suficiente del medicamento para cubrir la cantidad solicitada.

## **RF-12: Confirmación Explícita de Compra**

- El sistema debe requerir una acción afirmativa del usuario (clic en un botón de "Confirmar y Pagar" o similar) para manifestar su conformidad con los datos del resumen y proceder a la pasarela de pagos.

## **RF-13: Edición de Selección**

- El sistema debe permitir al usuario regresar a la pantalla anterior desde el resumen de compra para modificar la cantidad o cambiar el medicamento si detecta un error en su selección.


----
## RF-14: Solicitud de Consentimiento para Acceso a Historial de Tratamientos


- El portal web debe proveer una funcionalidad que permita al doctor enviar una solicitud formal de consentimiento a un paciente específico. Esta solicitud es estrictamente necesaria para que el sistema le otorgue al doctor los permisos de lectura sobre los datos de tratamientos médicos e historial de medicamentos anteriores del paciente.

## RF-15: Sugerencia de Medicamento Genérico Equivalente

- El sistema debe identificar si el medicamento de "marca innovadora" seleccionado por el usuario tiene un equivalente genérico registrado. Antes de que el usuario proceda a la pantalla de confirmación de pago, la interfaz debe mostrar una notificación o ventana emergente sugiriendo la alternativa genérica disponible, comparando los precios para facilitar la decisión de compra.

## ~~RF-16: Registro Inmutable de Trazabilidad para Medicamentos Controlados~~
- El sistema debe generar y almacenar de forma automática un registro de auditoría (Audit Log) inmutable cada vez que la máquina expendedora dispense un medicamento clasificado como "controlado". Este registro tiene fines de cumplimiento legal y trazabilidad estricta, y debe operar de manera independiente a las acciones del usuario sobre su propio perfil.

## RF-17: Validación de identidad por medio de cédula
- El sistema de la maquina expendedora debe verificar que el usuario que este realizando la compra este verificado en el sistema y por medio de scaner de cedula validar si esta rejistrado y a el le pertenece la receta 

## RF-18: Registro con cedula 