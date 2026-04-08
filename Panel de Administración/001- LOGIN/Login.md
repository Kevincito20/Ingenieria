
***Descripción del módulo:***

Este módulo, es importante para mantener la aplicación segura, permitiendo una distribución organizada y minimalista.

**Distribución visual 

Para la distribución general, se propone un diseño centrado, donde toda la información este almacenada en una card sobresaliente del fondo.  destacando la información para el acceso a la aplicación.


**Utilizando campos de acceso como: 
 - RUC doctor (identificador único).
 - Contraseña.

**Acciones: 
- Botón principal de "Ingresar"
- Enlace de "Olvide mi contraseña"
- Footer "Políticas de privacidad"

## Requisitos Funcionales (RF)

- **RF-001 :Autenticación 

	- El sistema debe permitir el ingreso al personal médico verificando las credenciales contra la base de datos.
	
- **RF-002: MFA (Multi-Factor Authentication): 

	- Tras validar la contraseña, el sistema debe solicitar un segundo factor utilizando OTP mediante correo o SMS para cumplir normativas de seguridad de acceso a datos sensibles.
- **RF-003: Bloqueo por fuerza bruta: 

	- El sistema debe bloquear temporalmente la cuenta después de 5 intentos fallidos de inicio de sesión.
	
- **RF-004: Gestión de Sesión:

	- El sistema debe generar un token JWT como un tiempo de expiración definido y rol especifico.


## Flujo de Excepciones

** **FE-001: Credenciales inválidas**

**Condición:**  
El RUC o la contraseña no coinciden con los registros.

**Flujo:**

1. El usuario ingresa RUC y contraseña.
2. El sistema valida contra la base de datos.
3. La validación falla.
4. El sistema:
    - Muestra mensaje: _"Credenciales incorrectas"_
    - Incrementa contador de intentos fallidos.
5. Permanece en la misma pantalla.


**FE-002: Campo vacío

**Condición:**  
Uno o ambos campos (RUC / contraseña) están vacíos.

**Flujo:**

1. Usuario presiona "Ingresar".
2. El sistema valida campos.
3. Detecta campos vacíos.
4. El sistema:
    - Resalta campos incompletos.
    - Muestra mensaje: _"Todos los campos son obligatorios"_
5. No realiza petición al backend.


**FE-003: Cuenta bloqueada por intentos fallidos**

(Relacionado con RF-003)

**Condición:**  
Se superan los 5 intentos fallidos.

**Flujo:**

1. Usuario intenta iniciar sesión repetidamente.
2. El sistema detecta 5 intentos fallidos.
3. El sistema bloquea la cuenta temporalmente.
4. Muestra mensaje:
    - _"Cuenta bloqueada temporalmente. Intente más tarde"_
5. Deshabilita el botón "Ingresar".


 **FE-004: Usuario no registrado**

**Condición:**  
El RUC no existe en la base de datos.

**Flujo:**

1. Usuario ingresa RUC válido en formato.
2. El sistema no encuentra coincidencias.
3. El sistema:
    - Muestra mensaje: _"Usuario no registrado"_
    - Sugiere contactar soporte.


**FE-005: OTP incorrecto**

**Condición:**  
El código OTP ingresado no coincide.

**Flujo:**

1. Usuario ingresa OTP.
2. El sistema valida código.
3. Código incorrecto.
4. El sistema:
    - Muestra mensaje: _"Código incorrecto"_
    - Permite reintento limitado.


**FE-006: OTP expirado**

**Condición:**  
El tiempo del código OTP ha expirado.

**Flujo:**

1. Usuario ingresa OTP después del tiempo límite.
2. El sistema valida y detecta expiración.
3. El sistema:
    - Muestra mensaje: _"Código expirado"_
    - Habilita botón "Reenviar código".