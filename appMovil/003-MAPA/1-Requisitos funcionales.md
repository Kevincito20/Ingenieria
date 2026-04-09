## Requisitos Funcionales  Mapa

_Estos requisitos describen las acciones específicas que el sistema y el usuario pueden realizar dentro del módulo._

- **RF-01: Visualización del Mapa Base**
    
    - El sistema debe mostrar un mapa interactivo centrado por defecto en la ubicación actual del usuario o en una vista general si no hay permisos de ubicación.

- **RF-02: Geolocalización del Usuario**
    
    - El sistema debe solicitar los permisos correspondientes y detectar la ubicación en tiempo real del dispositivo del usuario para utilizarla como punto de partida.

- **RF-03: Visualización de Máquinas Expendedoras (Marcadores)**
    
    - El sistema debe ubicar en el mapa iconos representativos para las máquinas expendedoras de medicamentos.
        
    - Al seleccionar una máquina (Ver máquinas expendedoras), el sistema debe desplegar una tarjeta informativa con detalles como la ubicación exacta, inventario disponible o identificador de la máquina.
        
- **RF-04: Indicadores de Estado Visual**
    
    - El sistema debe codificar los iconos de las máquinas mediante colores o símbolos para reflejar su estado operativo en tiempo real:
        
        - **Activa:** Operando con normalidad.
        
        - **Inactiva:** Fuera de servicio o sin inventario.
        
        - **Mantenimiento:** En proceso de revisión técnica.
        
- **RF-05: Barra de Búsqueda** (Fase 2 )
    
    - El sistema debe incluir un campo de texto que permita al usuario buscar máquinas específicas introduciendo una dirección, punto de referencia o el código de la máquina.
        
- **RF-06: Sistema de Filtros**
    
    - El sistema debe permitir al usuario refinar los resultados del mapa aplicando múltiples filtros de forma simultánea:
        
        - **Por Provincia:** Seleccionar una o varias provincias para limitar la vista.
            
        - **Por Estado:** Mostrar únicamente máquinas con un estado específico (ej. solo mostrar las "Activas").
            
        - **Por Favoritas:** Mostrar únicamente las máquinas que el usuario haya guardado previamente en su lista de favoritos.
            
- **RF-07: Cálculo y Visualización de Ruta**
    
    - El sistema debe calcular y trazar en el mapa la ruta más eficiente desde la ubicación actual del usuario hasta la máquina expendedora seleccionada.
        
- **RF-08: Distancia y Tiempo Estimado de Llegada (ETA)**
    
    - El sistema debe calcular y mostrar en pantalla la distancia exacta (en kilómetros o metros) y el tiempo estimado que le tomará al usuario llegar a la máquina seleccionada, basándose en la ruta trazada.

## Requisitos funcionales tarjeta de maquinas expendedoras
