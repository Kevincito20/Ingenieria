
==*Agregar columnas nuevas si falta algo*==

## **Tabla de cliente** 

|     id     |      bigint       |
| :--------: | :---------------: |
|   nombre   |    varchar(20)    |
|  apellido  |    varchar(20)    |
|   cedula   |    varchar(50)    |
|  password  | hash varchar(100) |
|   correo   |    varchar(50)    |
| asegurado  |      boolean      |
| verificado |      boolean      |
|    sexo    |    varchar(10)    |

## **Tabla de usuarios** "Administración" (Doctores)

|       id       |       bigint        |
| :------------: | :-----------------: |
|     nombre     |     varchar(20)     |
|    apellido    |     varchar(20)     |
|      rol       |     varchar(20)     |
|    password    |  hash varchar(20)   |
|    usuario     |     varchar(20)     |
|   ruc_doctor   | ==text (temporal)== |
| especialidades |     varchar(20)     |

## **Tabla de ficha medica**

|         id          |    bigint    |
| :-----------------: | :----------: |
|     id_cliente      |    bigint    |
|   tipo_sanguineo    | varchare(10) |
|      alergenos      | varcahr(40)  |
|  efermedad_cronica  | varcahr(40)  |
| id_historial_medico |              |
## **Tabla de historial medico**

|         id          |    bigint    |
| :-----------------: | :----------: |
|     id_cliente      |    bigint    |
|   fecha_consulta    |     date     |
|   motivo_consulta   | varchar(255) |
|     diagnostico     | varchar(500) |
|     tratamiento     | varchar(500) |
|    observaciones    | varchar(500) |
|  presion_arterial   | varchar(20)  |
|     temperatura     |   decimal    |
|        peso         |   decimal    |
|       altura        |   decimal    |
| frecuencia_cardiaca |     int      |
|       medico        | varchar(150) |
|   fecha_registro    |     date     |

## **Tabla de recetas**

|          id          |   bigint    |
| :------------------: | :---------: |
|      id_cliente      |   bigint    |
|   doctor_remitente   | varchar(20) |
| ruc_doctor_remitente | varchar(50) |
|  hospital_remitente  | varchar(20) |
|  telefono_hospital   | varchar(20) |
|        correo        | varchar(50) |
|        codigo        |     int     |
|        fecha         |    date     |


## **Tabla de dosis**

|       id       | bigint |
| :------------: | :----: |
| id_medicamento | bigint |
|   id_receta    | bigint |
|    cantidad    |  int   |
| instrucciones  |  text  |


## **Tabla de inventario**

|         id         |   bigint    |
| :----------------: | :---------: |
|     id_maquina     |   bigint    |
| nombre_medicamento | varchar(40) |
|       marca        | varchar(40) |
|       precio       |   decimal   |
|      cantidad      |     int     |
|      resetado      |   boolean   |


## **Tabla de maquina**

|    id     |   bigint    |
| :-------: | :---------: |
| ubicación | varchar(50) |
|  activo   |   boolean   |
|  latitud  |   number    |
| longitud  |   number    |




