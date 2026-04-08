
### Módulos y Componentes de la Interfaz (Layout)

La interfaz se divide en los siguientes módulos estructurales, ordenados de la parte superior a la inferior de la pantalla:

**1. Módulo de Ubicación (Header / Selector de Máquina)**

- **Descripción:** Ubicado en la franja superior de la pantalla.
    
- **Elementos:** Una lista desplegable o un carrusel de etiquetas que muestra en qué máquinas expendedoras se encuentra disponible este medicamento específico. Si el usuario ya viene con una máquina seleccionada desde el mapa, esta se mostrará por defecto, pero podrá visualizar otras opciones cercanas.
    

**2. Módulo Visual y de Restricciones sanitarias**

- **Descripción:** El área gráfica principal del producto.
    
- **Elementos:**
    
    - **Imagen del Medicamento:** Fotografía clara del empaque del producto para fácil reconocimiento.
        
    - **Etiqueta de Restricción (_Badge_):** Una pequeña tarjeta superpuesta en la parte superior de la imagen, con colores contrastantes (ej. verde para venta libre, rojo/naranja para receta obligatoria), indicando claramente: _"Venta Libre"_ o _"Requiere Receta Médica"_.
        

**3. Módulo de Información Clínica**

- **Descripción:** Sección de texto descriptivo debajo de la imagen.
    
- **Elementos:**
    
    - **Título:** Nombre comercial, principio activo (genérico) y concentración del medicamento (ej. Paracetamol 500mg).
        
    - **Descripción:** Un bloque de texto que detalla las indicaciones, uso principal y posibles advertencias del medicamento.
        

**4. Módulo Comercial y de Disponibilidad**

- **Descripción:** Sección de datos numéricos clave, ubicada debajo de la descripción.
    
- **Elementos:**
    
    - **Precio Unitario:** El costo del medicamento de forma destacada.
        
    - **Indicador de Inventario:** Un texto o barra que muestra la cantidad exacta de unidades disponibles en la máquina expendedora actualmente seleccionada (ej. "Disponible: 15 unidades en esta máquina").
        

**5. Módulo de Interacción y Cantidad**

- **Descripción:** Área donde el usuario define su orden.
    
- **Elementos:**
    
    - **Selector de Cantidad:** Controles interactivos (botones de **[-]** y **[+]**) junto con un número central, que permiten al usuario indicar cuántas unidades desea comprar. _Nota funcional: Este selector no debe permitir elegir un número mayor al inventario de la máquina ni mayor al límite diario permitido por usuario._
        

**6. Módulo de Acción (_Call to Action_)**

- **Descripción:** Ubicado en el extremo inferior de la pantalla.
    
- **Elementos:**
    
    - **Botón Fijo (Sticky Button):** Un botón amplio y centrado en la parte inferior de la pantalla con el texto "Añadir al Carrito" o "Agregar a la Orden". Este botón puede mostrar dinámicamente el subtotal de la selección (ej. "Añadir al carrito - $5.50") y debe mantenerse visible aunque el usuario haga _scroll_ en la descripción.