
**Distribución visual

**Barra superior: 
- Saludo personalizado: "Hola, [Nombre del usuario]"
- Barra de búsqueda (nombre completo, cédula).
- Configuración (por desarrollar)
- Información del Usuario (nombre, foto, especialidad)
- Notificaciones (no definido).

**Filtrado de información: 
- ==filtrado de paciente por estado (En espera, atendido, urgencias)==
- Área Central :
	- lista vertical con información del usuario.
	- cada card representa un paciente, conteniendo información como: 
		- Nombre. 
		- Edad. 
		- Hora de cita agendada. 
		- Estado.
		- Resumen del paciente (idk)


**Requisitos Funcionales:**

- **RF- 005 Carga de Pacientes:** 
	- El sistema debe recuperar y mostrar la lista de pacientes asignados al doctor activo para el día en curso.
    
- **RF-006 Búsqueda en Tiempo Real:**
	- El sistema debe filtrar la lista de pacientes instantáneamente a medida que el doctor escribe en el buscador, consultando la base de datos sin recargar la página.
    
- **RF-007 Actualización de Estado:** 
	- El sistema debe permitir cambiar el estado del paciente (ej. de "En Espera" a "En Consulta") con confirmación visual.
    
- **RF-007 Despliegue de Expediente:** 
	- Al hacer clic en la tarjeta de un paciente, el sistema debe comprimir la lista hacia la izquierda y desplegar el panel lateral derecho (Expediente) mediante una transición fluida. (Opino que debe redirigir al expediente directamente -> 004)



## **Flujos de Excepciones**



- **FE-001 Error en Carga de Pacientes:**
    - Si ocurre un fallo al recuperar la lista de pacientes desde la base de datos:
        - El sistema debe mostrar un mensaje: _"No se pudieron cargar los pacientes"_.
        - Debe ofrecer opción de reintentar.
        - Debe mostrar un estado vacío o skeleton loader mientras reintenta.
        - Debe registrar el error en logs del sistema.



- **FE-002 Lista de Pacientes Vacía:**
    - Si no existen pacientes asignados para el día:
        - El sistema debe mostrar un mensaje: _"No hay pacientes asignados para hoy"_.
        - Debe mostrar una vista vacía amigable (empty state).
        - No debe mostrar errores ni loaders innecesarios.



- **FE-003 Error en Búsqueda en Tiempo Real:**
    - Si falla la consulta durante la búsqueda:
        - El sistema debe mantener los resultados previos visibles.
        - Debe mostrar mensaje: _"Error en la búsqueda, intente nuevamente"_.
        - Debe permitir seguir escribiendo sin bloquear la interfaz.



- **FE-004 Sin Resultados en Búsqueda:**
    - Si la búsqueda no arroja coincidencias:
        - El sistema debe mostrar mensaje: _"No se encontraron resultados"_.
        - Debe mantener la barra de búsqueda activa.
        - Debe permitir limpiar filtros fácilmente.



- **FE-005 Error al Filtrar por Estado:**
    - Si ocurre un fallo al aplicar filtros:
        - El sistema debe restaurar la lista original.
        - Debe mostrar mensaje: _"No se pudo aplicar el filtro"_.
        - Debe permitir reintentar la acción.



- **FE-006 Error al Actualizar Estado del Paciente:**
    - Si falla el cambio de estado:
        - El sistema debe revertir el estado visual al anterior.
        - Debe mostrar mensaje: _"No se pudo actualizar el estado"_.
        - Debe permitir reintentar la acción.



- **FE-007 Conflicto de Actualización de Estado:**
    - Si otro usuario actualiza el estado simultáneamente:
        - El sistema debe mostrar mensaje: _"El estado ha sido actualizado por otro usuario"_.
        - Debe refrescar la información automáticamente.
        - Debe evitar sobrescribir datos sin confirmación.



- **FE-008 Error al Desplegar Expediente:**
    - Si falla la carga del expediente del paciente:
        - El sistema debe mostrar mensaje: _"No se pudo cargar el expediente"_.
        - Debe mantener la lista de pacientes visible.
        - Debe permitir reintentar o redirigir nuevamente.



- **FE-009 Paciente no Disponible:**
    - Si el paciente seleccionado ya no está disponible (eliminado o cambiado):
        - El sistema debe mostrar mensaje: _"El paciente ya no está disponible"_.
        - Debe actualizar la lista automáticamente.



- **FE-010 Error en Notificaciones:**
    - Si falla la carga de notificaciones:
        - El sistema debe mostrar un indicador discreto de error.
        - No debe bloquear el resto de la interfaz.
        - Debe permitir recargar manualmente.



- **FE-011 Error de Sesión (Expirada o Inválida):**
    - Si la sesión del usuario expira:
        - El sistema debe redirigir al login.
        - Debe mostrar mensaje: _"Sesión expirada"_.
        - Debe evitar pérdida de contexto crítico si es posible.