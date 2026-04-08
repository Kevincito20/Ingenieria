 En este apartado los usuarios podrán ver un mapa de las maquinas expendedoras, y a su vez estado de estas (Activa, Inactiva, Mantenimiento). 



## Ver mapa 


## DETALLE DE MAQUINA EXPENDEDORA

### **RF-034: Detalle de Ubicación y Telemetría**

El sistema debe desplegar una tarjeta informativa al seleccionar una máquina en el mapa, garantizando la precisión de los siguientes datos:

- **Identificación del Sitio:** * **Nombre:** Título descriptivo del lugar (ej. "Mall El Dorado - Nivel 1").
    
    - **Dirección:** Ubicación física completa para orientación del usuario.
        
- **Estado Operacional (Badge):**
    
    - Etiqueta visual codificada por colores: **Verde (Activa)**, **Rojo (Inactiva)** o **Naranja (Mantenimiento)**, incluyendo texto descriptivo para accesibilidad.
        
- **Métricas de Proximidad:**
    
    - **Distancia Real:** Cálculo dinámico entre el usuario y la máquina, mostrado en metros (m) o kilómetros (km) con un decimal.
        
- **Sincronización de Datos:**
    
    - **Última Conexión:** Tiempo relativo de la última señal recibida (ej. "Hace 5 min").
        
    - **Alerta de Desfase:** Aviso automático si la telemetría no se ha actualizado en más de 15 minutos, indicando posible variación en el stock.



### **RF-035: Gestión y Visualización de Inventario**

El sistema debe listar el contenido detallado de la máquina seleccionada, permitiendo al usuario conocer la disponibilidad y condiciones de compra antes de interactuar físicamente con ella.

- **Catálogo de Productos:**
    
    - **Identificación:** Nombre comercial del medicamento y presentación (ej. "Ibuprofeno 400mg - 10 tabletas").
        
    - **Precio:** Costo unitario claramente visible con el símbolo de moneda local (USD/PAB).
        
- **Control de Disponibilidad:**
    
    - **Stock por Slot:** Cantidad exacta disponible en la máquina. Si el stock es 0, el producto debe aparecer sombreado o con la etiqueta "Agotado".
        
    - **Indicador de Receta:** Icono distintivo (ej. una hoja de prescripción) para medicamentos que requieren validación médica obligatoria para desbloquear la compra.
        
- **Filtros de Búsqueda Interna:**
    
    - Campo de texto dentro de la tarjeta para buscar medicamentos específicos por nombre o categoría dentro del inventario de esa máquina.
        
- **Reglas de Negocio:**
    
    - **Actualización en Tiempo Real:** Los niveles de stock deben sincronizarse con cada transacción realizada en la máquina física.
        
    - **Validación de Compra:** El botón de "Añadir al carrito" o "Comprar" se deshabilitará automáticamente si el stock es igual a cero.
![[Pasted image 20260406220654.png|240]]

### **RF-036: Integración de Rutas y Navegación ("Cómo llegar")**

El sistema debe proveer un botón de "Cómo llegar" en la tarjeta de detalle para trazar la ruta desde la ubicación actual del usuario hasta la máquina expendedora.

- **Navegación Externa (Comportamiento Base):** * El botón generará un enlace profundo (_deep link_ / URI scheme) que abrirá directamente la aplicación de mapas nativa del dispositivo (Google Maps en Android, Apple Maps en iOS) con las coordenadas de destino pre-cargadas.
    
- **Navegación In-App y Personalización (React Native):**
    
    - Para la aplicación móvil desarrollada en **React Native**, se evaluará la integración de rutas nativas dentro de la misma app para evitar que el usuario salga del ecosistema.
        
- **Librerías a Evaluar para el Entorno Móvil:**
    
    - **`react-native-maps`:** Para una implementación estándar y confiable que utiliza directamente los SDKs de Google Maps y Apple Maps, permitiendo trazado de polilíneas (rutas) sobre el mapa.
        
    - **Mapbox (`@rnmapbox/maps`):** Como alternativa principal para lograr una mayor personalización visual (colores de la marca, estilos de mapa vectoriales fluidos) y un control avanzado sobre la experiencia de navegación paso a paso.


### **RF-037: Botón de Acción Principal y Flujo de Compra**

El sistema debe incluir un botón prominente de "Comprar aquí" en la tarjeta de detalle para iniciar el proceso de adquisición sin fricciones.

- **Lógica de Estado y Visibilidad:**
    
    - **Condición Estricta:** El botón debe estar habilitado y visible **únicamente** si el estado de la máquina es "Activa" (RF-034) y el inventario global reporta al menos un producto disponible.
        
    - **Prevención de Errores:** Si la máquina está "Inactiva", en "Mantenimiento" o el stock total es cero, el botón debe deshabilitarse (estado _disabled_ o atenuado visualmente) o ser reemplazado por un mensaje de "Temporalmente no disponible para compras".
        
- **Transición y Manejo de Contexto:**
    
    - **Pre-selección de Inventario:** Al presionar el botón, el sistema redirigirá al usuario a la pantalla de catálogo/carrito.
        
    - **Gestión de Estado (State Management):** El sistema (usando Context API, Zustand, Redux, etc. en React Native) debe guardar el `machine_id` de forma global para la sesión actual, garantizando que el usuario solo pueda agregar al carrito productos que existan físicamente en esa máquina específica.
        
- **Jerarquía Visual (UX/UI):**
    
    - **Call to Action (CTA):** Este botón debe ser el elemento visual principal de la interfaz, utilizando el color primario del diseño y destacando claramente sobre el botón secundario de "Cómo llegar".

### **RF-038: Gestión de Estado Degradado (Modo Offline)**

El sistema debe adaptar dinámicamente la interfaz y las opciones de compra cuando la máquina reporte pérdida de conexión con el servidor central o fallos en sus módulos de validación, garantizando una experiencia de usuario segura.

- **Activación del Estado (Trigger):**
    
    - El sistema activará este modo si la API reporta el estado de la máquina como `degradado` o si el tiempo de la última telemetría (RF-034) excede el umbral crítico (ej. $> 30\text{ minutos}$ sin conexión).
        
- **Notificación Visual (UI):**
    
    - **Banner de Alerta:** Desplegar un _banner_ prominente (color de advertencia, ej. amarillo o naranja) en la parte superior de la tarjeta de detalle con el texto: _"Modo offline — solo medicamentos de venta libre (OTC) disponibles"_.
        
- **Restricción Dinámica de Inventario:**
    
    - **Filtro Automático:** El sistema debe evaluar la propiedad `requires_prescription` de cada producto en el catálogo.
        
    - **Bloqueo de Medicamentos Rx:** Los medicamentos que requieran receta médica se mostrarán atenuados (opacidad reducida o en escala de grises) y su botón de compra estará bloqueado o reemplazado por la etiqueta "No disponible en modo offline".
        
    - **Vía Libre para OTC:** Los medicamentos de venta libre (OTC) mantendrán su flujo de compra normal, asumiendo que la máquina aún puede procesar pagos físicos locales o despachar sin validación remota.

![[Pasted image 20260406222525.png|182]]