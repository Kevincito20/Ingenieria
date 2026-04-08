#Requisitos_funcionales 
## Requisitos funcionales


| Código- | Funcionalidad                                                                                                                                                                                                               | Prioridad |
| ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------- |
| RFI-001 | Mostrar listado de máquinas expendedoras cercanas ordenadas por distancia al usuario, con indicador de estado activa/inactiva.                                                                                              | Alta      |
| RFI-002 | Mostrar ~~banner~~ (barra??) de búsqueda rápida de medicamentos por nombre comercial o principio activo.                                                                                                                    | Alta      |
| RFI-003 | Mostrar acceso rápido a "Última compra" con botón de repetir pedido si el medicamento sigue disponible                                                                                                                      | Media     |
| RFI-004 | ~~Mostrar notificaciones pendientes (receta próxima a vencer, pedido listo para retirar) en banner visible.~~ Mostrar notificaciones de receta(Nueva receta, receta cercana a fecha de vencimiento, medicamentos faltantes) | Alta      |
| RFI-005 | Mostrar indicador visual si el usuario tiene recetas activas disponibles para usar.                                                                                                                                         | Alta      |
|         |                                                                                                                                                                                                                             |           |
^f62683


## Header


**Descripción del modulo**
En este modulo se encontraran el nombre de usuario, área de notificaciones y avatar del perfil de usuario.  

**Especificaciones de Interfaz (UI/UX):**
- Saludo personalizado: "Hola, [Nombre del Usuario]"

- Avatar/Perfil: Un ícono en la esquina superior derecha. Si el usuario aún no ha verificado su identidad (requisito para medicamentos con receta), este avatar podría tener un pequeño indicador visual (un punto rojo o un icono de alerta) para invitarlo a completar su perfil.

- Notificaciones: Un ícono de campana para avisos (ej. "Tu receta está por vencer" o "La máquina de tu zona ha sido reabastecida").[[Requisitos funcionales Generales#^ea0f95|Requisito funcional 004]]

![[Pasted image 20260404101118.png|440]]


## Modulo de recetas actuales
[[Requisitos funcionales Generales#^6e489b|Requisito funcional 005]]

**Descripción del modulo**
Modulo centrado en mostrar al usuario una previsualización de recetas vigentes. 


**Especificaciones de Interfaz (UI/UX):**
* **Título de la Sección:** "Tus máquinas frecuentes"
* **Layout:** Cuadrícula responsiva (Grid/Flexbox), optimizada para mostrar tarjetas medianas en una disposición de dos columnas (2 por fila en pantallas adecuadas).

**Datos Dinámicos por Tarjeta:**
- Titulo: Nombre genérico [Receta medica]
- Subtitulo: Nombre del hospital [Santo tomas
- Fecha: [12-12-2012]

![[Pasted image 20260405095858.png|320]]





## Módulo de Dispensadores Frecuentes
[[Requisitos funcionales Generales#^c5a81e|Requisito funcional 001]]

**Descripción del modulo:**
Interfaz de usuario orientada a la fidelización y acceso rápido, diseñada para mostrar al paciente los nodos de dispensación (máquinas) con los que interactúa habitualmente o que se encuentran en su proximidad inmediata.

**Especificaciones de Interfaz (UI/UX):**
* **Título de la Sección:** "Tus máquinas frecuentes"
* **Layout:** Cuadrícula responsiva (Grid/Flexbox), optimizada para mostrar tarjetas medianas en una disposición de dos columnas (2 por fila en pantallas adecuadas).

**Datos Dinámicos por Tarjeta:**
1. **Identificador del Nodo:** Nombre comercial o ubicación física (ej. *Expendedora Plaza Central*).
2. **Telemetría IoT (Tiempo Real):** Indicador visual de estado de red.
   * 🟢 **Operativo:** Hardware en línea y funcional.
   * 🔴 **Fuera de servicio:** Error de conexión o mantenimiento.
3. **Control de Stock:** Nivel de abastecimiento general del dispensador, expresado en porcentaje o métrica cualitativa (ej. *Nivel de inventario: 85%* o *Abastecimiento: Alto*). (*Opcional*)
4. **Geolocalización:** Distancia estimada desde la ubicación actual del usuario mediante GPS (ej. *a 1.2 km*).




 	![[Maquinas_cercanas.png]]


# Faltantes 
- Funcionalidad para elegir la maquina expendedora donde el usuario ara ara la búsqueda del medicamento 
- Diseño de las cards de medicamento, estas tarjetas se mostraran cuando el usuario haga la búsqueda del mentica mentó
- Consideración de mover la barra de búsqueda a una pantalla individual, con una previsualización de medicamentos. 