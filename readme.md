# Rincón Sur - Sistema de Conteo y Control de Aforo

Aplicación móvil Android diseñada para el control de aforo, análisis de tasa de conversión de compra y trazabilidad de flujo de visitantes en accesos de tiendas retail.

---

## Información del Proyecto

* **Asignatura:** DSY1105 – Desarrollo de Aplicaciones Móviles
* **Evaluación:** Evaluación Parcial 2
* **Docente:** Victor Rosendo
* **Integrantes:**
  * Matias Olavarria
  * Renata Valdés
* **Gestión del Proyecto:** [Tablero en Jira](https://duocuc-team-f0sc9kf8.atlassian.net/jira/software/projects/LT/boards/3?filter=&groupBy=none&atlOrigin=eyJpIjoiMDY5NGU2ZjhmMjM3NGJlZWI3YTc5ZGRhMmZiZDY2OWQiLCJwIjoiaiJ9)

---

## Definición del Problema y Solución

### Problema
La tienda cuenta con cuatro accesos independientes:
* **A1:** Principal
* **A2:** Lateral
* **A3:** Estacionamiento
* **A4:** Conexión

Se requiere registrar el flujo total de personas e identificar con precisión cuántas de ellas realizan una compra efectiva en comparación con aquellas que utilizan la tienda únicamente como vía de tránsito.

### Solución Propuesta
El sistema implementa un modelo de conteo manual combinado con verificación de compras mediante escaneo de comprobantes:

1. **Ingreso:** El supervisor registra el ingreso presionando `+ Visitante` (estado asignado: `En_Tienda`).
2. **Salida sin compra:** Al presionar `- Visitante`, el registro activo más antiguo cambia su estado a `No_Compro`.
3. **Conversión de compra:** Al procesar un pago, se escanea el código de barras de la boleta mediante la cámara, cambiando el estado del visitante a `Compro`.
4. **Cierre diario:** Al finalizar la jornada, todos los registros que permanezcan en estado `En_Tienda` pasan automáticamente a `No_Compro`.

---

## Reglas de Negocio y Validaciones

### Reglas de Negocio
* **Unicidad de comprobante:** Cada boleta puede procesarse una sola vez. Si se reintenta, el sistema despliega la advertencia: *"Esta boleta ya fue procesada"*.
* **Formato de boleta:** Solo se aceptan folios numéricos con una extensión de 6 a 10 dígitos.
* **Persistencia lógica:** Ningún registro se elimina físicamente de la base de datos. Se utiliza una columna lógica `activo` para habilitar o deshabilitar registros.
* **Restricciones de jerarquía:** 
  * Un administrador no puede crear otros administradores desde la aplicación.
  * Un administrador no puede deshabilitar su propia cuenta.
  * El sistema exige que permanezca al menos un usuario administrador activo en todo momento.

### Validaciones en Capa de Lógica (ViewModel)
* **Correo electrónico:** Debe cumplir con formato válido y pertenecer exclusivamente al dominio `@rinconsur.cl`.
* **RUT Chileno:** Validación mediante algoritmo Módulo 11 y comprobación de unicidad.
* **Nombre completo:** Requiere ingresar al menos un nombre y un apellido.
* **Seguridad de credenciales:** Las contraseñas se encriptan mediante el algoritmo BCrypt. Las cuentas nuevas reciben una clave temporal generada con `SecureRandom`.
* **Programación de sesiones:** Limitada estrictamente a los accesos A1, A2, A3 y A4, prohibiendo el agendamiento en fechas pasadas.
* **Seguridad de acceso:** Las cuentas deshabilitadas tienen bloqueado el inicio de sesión. Cada acción administrativa valida nuevamente el perfil activo en la base de datos.

---

## Módulos y Funcionalidades

* **Inicio:** Panel general con resumen diario, sesiones programadas para la jornada, total de visitantes en tienda, acceso rápido a escaneo de boletas y ejecución del cierre diario.
* **Sesiones:** Módulo de agenda basado en calendario para programar turnos de conteo por acceso y horario.
* **Analítica:** Métricas e informes de flujo por acceso, rangos etarios y periodos seleccionables (personalizado, semanal o mensual).
* **Equipo:** Gestión de personal con filtros de búsqueda por perfil. Permite a administradores la creación de cuentas y modificación de estados o roles.
* **Perfil:** Configuración de usuario, gestión de foto de perfil (cámara o galería), edición de descripción, consulta de permisos asignados y cierre de sesión.
* **Autenticación:** Inicio de sesión, recuperación de contraseña y soporte nativo para modo claro y oscuro.

---

## Matriz de Roles y Permisos

| Rol | Permisos y Alcance |
| :--- | :--- |
| **Administrador** | Acceso total: creación y edición de sesiones, modificación de analítica, registro de personal y gestión de perfiles. |
| **Supervisor** | Acceso global con restricciones: permisos de lectura en Sesiones, Analítica y Equipo. |
| **Observador** | Acceso global con restricciones: permisos de lectura en Sesiones, Analítica y Equipo. |

---

## Arquitectura y Tecnologías

* **Lenguaje:** Kotlin
* **Interfaz de Usuario:** Jetpack Compose con Material Design 3
* **Arquitectura:** Patrón MVVM (Model-View-ViewModel) con StateFlow
* **Navegación:** Navigation Compose
* **Base de Datos Local:** SQLite (`SQLiteOpenHelper`), archivo único `rinconsur.db`
* **Almacenamiento Local:** `SharedPreferences` para la sesión activa
* **Librerías y SDKs Nativos:**
  * CameraX + ML Kit Barcode Scanning
  * BCrypt (`at.favre.lib:bcrypt`)
  * Permisos nativos (`POST_NOTIFICATIONS`, `CAMERA`, `READ_MEDIA_IMAGES`)

### Estructura del Proyecto

```text
app/src/main/java/com/example/rinconsur/
├── data/         # Base de datos, preferencias, manejadores de fechas y conversión de imágenes
├── model/        # Entidades (Usuario, Colaborador, Sesión, Flujo, Visitante) y Enums
├── navigation/   # Definición de rutas y grafo de navegación (AppNavigation)
├── permisos/     # Control de permisos nativos del sistema
├── ui/           # Componentes reutilizables, pantallas, estilos y temas
└── viewmodel/    # Lógica de negocio y validaciones por pantalla
```

---

## Usuarios de Prueba

| Perfil | Correo Electrónico | Contraseña |
| :--- | :--- | :--- |
| Administrador | `daniela@rinconsur.cl` | `Daniela123` |
| Supervisor | `nicole@rinconsur.cl` | `Nicole123` |
| Supervisor | `renata@rinconsur.cl` | `Renata123` |
| Observador | `matias@rinconsur.cl` | `Matias123` |

---

## Instrucciones de Ejecución e Instalación

1. Clonar el repositorio localmente.
2. Abrir el proyecto en Android Studio (versión Dolphin o superior recomendada).
3. Sincronizar el proyecto con los archivos de Gradle.
4. Desplegar la aplicación en un dispositivo físico o emulador con Android 8.0 (API nivel 26) o superior.
5. Otorgar los permisos de Cámara y Notificaciones al iniciar la aplicación.

> **Nota sobre Base de Datos:** Si se incrementa la versión de la base de datos en el código, esta se recreará automáticamente reiniciando los datos. Para realizar una limpieza completa de la persistencia local, desinstale la aplicación del dispositivo.

### Pruebas de Escaneo de Boletas
Para validar la funcionalidad del escáner:
1. Generar un código de barras formato **Code 128** que contenga una secuencia numérica de 6 a 10 dígitos (por ejemplo, utilizando el servicio en línea *TEC-IT Barcode Generator*).
2. Escanear el código generado directamente desde la pantalla utilizando la cámara de la aplicación.