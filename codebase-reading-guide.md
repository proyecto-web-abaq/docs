# Guía Integral de Lectura del Código Base y Mapa Mental de Arquitectura (Abaq)

Bienvenido a la **Guía Integral de Lectura del Código Base y Mapa Mental de Arquitectura** de la plataforma **Abaq**. Este documento ha sido diseñado como una herramienta pedagógica de referencia técnica para ingenieros de software, estudiantes de servicio social, arquitectos de sistemas y evaluadores que deseen comprender, auditar, extender o mantener el sistema.

---

## Introducción y Propósito del Sistema Abaq

**Abaq** es una plataforma web integral (*full-stack*) diseñada para coordinar, registrar y supervisar las actividades de servicio social universitario orientadas al bienestar animal, rescate de fauna urbana, censos comunitarios en colonias vulnerables y campañas de esterilización y vacunación.

El sistema resuelve una problemática multidimensional:
1. **Para los estudiantes y voluntarios**: Provee un entorno digital transparente donde consultar convocatorias de actividades comunitarias, registrar su participación, dar seguimiento a sus horas acumuladas frente a los requisitos institucionales (habitualmente 480 horas), y gestionar su expediente oficial mediante la carga segura de cartas de presentación, aceptación y término en formato PDF.
2. **Para los coordinadores y administradores**: Ofrece un panel de control con métricas en tiempo real, administración de convocatorias comunitarias, pase de lista con acreditación y validación estricta de horas de servicio, visualización y descarga de expedientes estudiantiles en MongoDB GridFS, y supervisión de censos de bienestar animal levantados en campo.
3. **Para la comunidad y brigadas de rescate**: Facilita formularios públicos y dinámicos para el levantamiento de datos censales sobre mascotas, condiciones de hábitat y necesidades sanitarias en colonias de intervención prioritaria.

### Estructura de esta Guía

Para guiar el aprendizaje de manera estructurada, este documento se organiza en tres secciones fundamentales:
- **Sección 1: Fundamentos de la Pila Tecnológica**: Análisis profundo de los conceptos, paradigmas y patrones de diseño utilizados en Node.js, Express, Mongoose ODM y Angular 22 Standalone.
- **Sección 2: Flujo de Datos de Extremo a Extremo (End-to-End Data Flow)**: Diagramas y trazas secuenciales que detallan el viaje de la información a través de las capas del sistema, acompañados de tres casos de estudio de alta relevancia arquitectónica.
- **Sección 3: Lista de Verificación Interactiva Módulo por Módulo**: Un inventario exhaustivo y pedagógico de los **80 archivos no-boilerplate** del repositorio (36 en backend y 44 en frontend), organizados en 6 bloques temáticos con casillas interactivas `- [ ]`, enlaces relativos navegables y descripciones de su rol en el sistema.

---

## Sección 1: Fundamentos de la Pila Tecnológica

La plataforma Abaq implementa una arquitectura desacoplada moderna basada en el ecosistema JavaScript/TypeScript: una API RESTful construida con **Node.js**, **Express 4** y **Mongoose 8 / MongoDB**, consumida por una aplicación de página única (*Single Page Application* o SPA) desarrollada en **Angular 22** con componentes autónomos (*Standalone Components*).

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           CLIENTE (FRONTEND)                            │
│  Angular 22 (Standalone) │ RxJS 7.8 │ Bootstrap 5.3 │ Chart.js 4.4      │
│  • Inyección Funcional (inject)        • Formularios Reactivos (FormArray) │
│  • Interceptores HTTP Funcionales      • Autolimpieza (takeUntilDestroyed) │
│  • Mappers / Adaptadores DTO           • Guards de Enrutamiento (CanActivate)│
└────────────────────────────────────┬────────────────────────────────────┘
                                     │  Peticiones HTTP / REST JSON / Multipart
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           SERVIDOR (BACKEND)                            │
│  Node.js 24 │ Express 4.19 │ Mongoose 8.4 │ GridFS Bucket │ Nodemailer  │
│  • Pipeline de Middlewares en Cascada  • Factory de Controladores CRUD  │
│  • Async Wrapper (catchAsync)          • Setters Personalizados Mongoose│
│  • Errores Operacionales (AppError)    • Streaming Binario de Archivos  │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │  Driver BSON / TCP Socket
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       BASE DE DATOS (PERSISTENCIA)                      │
│  MongoDB Atlas Cluster                                                  │
│  • Colecciones Documentales: users, students, activities, surveys, ...  │
│  • Almacenamiento Binario GridFS: uploads.files y uploads.chunks        │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### 1.1 Node.js y el Ciclo de Vida de una Petición en Express

El backend de Abaq se ejecuta sobre el entorno de tiempo de ejecución Node.js, aprovechando su modelo de concurrencia basado en un **bucle de eventos no bloqueante** (*single-threaded Event Loop*). La recepción, procesamiento y despacho de peticiones HTTP es coordinado por **Express**.

#### A. Pipeline de Middlewares y Cadena de Responsabilidad
Express organiza el procesamiento de peticiones mediante el patrón de diseño **Cadena de Responsabilidad** (*Chain of Responsibility*). Cada petición entrante viaja secuencialmente a través de una serie de funciones intermediarias (*middlewares*) registradas en `backend/app.js`:

1. **CORS (`cors`)**: Valida el origen de la petición (`http://localhost:4200` en desarrollo local) y habilita la transmisión de credenciales y encabezados de autorización (`credentials: true`).
2. **Body Parsers (`express.json()`, `express.urlencoded()`)**: Inspeccionan la cabecera `Content-Type`. Si el cuerpo contiene JSON (`application/json`), parsea el flujo de bytes y adjunta el objeto resultante en `req.body`. Si proviene de un formulario codificado (`application/x-www-form-urlencoded`), lo decodifica con soporte de objetos anidados.
3. **Motor de Vistas Pug (`app.set('view engine', 'pug')`)**: Configura el renderizador en el servidor de plantillas HTML ubicadas en `backend/views/` (utilizado para el formulario alternativo de encuestas y vistas de bienvenida).
4. **Enrutadores Modulares (`app.use('/ruta', router)`)**: Despachan la petición hacia submódulos especializados según el prefijo de la URL:
   - `/auth`: Rutas públicas de autenticación y privadas de verificación de sesión.
   - `/student`: Gestión de perfiles y expedientes de estudiantes (protegida por JWT).
   - `/activity`: Convocatorias y acreditación de horas (protegida por JWT).
   - `/survey`: Ingesta pública de encuestas y consulta administrativa.
   - `/pet`: Censo de mascotas y pacientes veterinarios.
   - `/contact`: Seguimiento institucional a personas de contacto.
   - `/upload`: Carga multipart y descarga en streaming de archivos GridFS.
5. **Capturador 404 (`app.all('*')`)**: Si la URL solicitada no coincide con ninguna ruta previa, este middleware intercepta la petición y genera un error operacional: `next(new AppError(`${req.originalUrl} not found`, 404))`.
6. **Manejador Global de Errores (`(err, req, res, next)`)**: El punto final de contención que centraliza y formatea todas las respuestas de error del sistema.

#### B. La Función `next()` y el Desvío del Flujo
En Express, la función `next()` controla el avance en la tubería:
- **`next()` sin argumentos**: Indica a Express que el middleware actual concluyó exitosamente su labor y transfiere el control al siguiente middleware o controlador registrado en la pila.
- **`next(err)` con un argumento**: Indica que ha ocurrido una falla. Express suspende de inmediato la ejecución de cualquier middleware o controlador posterior en la cadena y salta directamente al **Manejador Global de Errores** (aquel middleware registrado con 4 parámetros: `err, req, res, next`).

#### C. El Patrón `catchAsync` y la Captura de Errores Asíncronos
En Express versión 4, las funciones asíncronas (`async (req, res, next) => { ... }`) que retornan una `Promise` rechazada (*rejected promise*) no delegan automáticamente dicho fallo a `next(err)`. Si una operación asíncrona (como una consulta a MongoDB) lanza una excepción y no cuenta con un bloque `try/catch`, la promesa queda como un *Unhandled Promise Rejection*, provocando fugas de recursos o la caída del proceso Node.js.

Para evitar escribir bloques `try/catch` idénticos en cada uno de los controladores, Abaq implementa el patrón de **función de orden superior** en `backend/utils/catchAsync.js`:

```javascript
// backend/utils/catchAsync.js
const catchAsync = (fn) => {
  return (req, res, next) => {
    fn(req, res, next).catch(next);
  };
};
```

**Mecanismo**: `catchAsync` recibe una función controladora asíncrona `fn` y devuelve una función de middleware estándar de Express. Al ejecutarse, invoca `fn(req, res, next)`. Dado que `fn` es asíncrona, retorna una promesa; encadenar `.catch(next)` garantiza que cualquier excepción o rechazo sea capturado y transferido automáticamente a `next(err)`, enrutándolo hacia el manejador global.

#### D. Errores Operacionales y la Clase `AppError`
En `backend/utils/appError.js`, Abaq define una clase que hereda de `Error` nativo para representar **errores operacionales** (errores predecibles derivados de la interacción del usuario o del entorno, tales como credenciales inválidas, identificadores inexistentes o validaciones fallidas):

```javascript
// backend/utils/appError.js
class AppError extends Error {
  constructor(msg, code) {
    super(msg);
    this.statusCode = code;
    this.status = `${code}`.startsWith("4") ? `Client/Server Error: ${code}` : `Error code: ${code}`;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}
```

- `isOperational = true`: Permite distinguir en producción entre fallas operacionales conocidas (que pueden comunicarse de manera segura al cliente con un código 4xx) y fallas de programación o bugs imprevistos (que deben responder con un código 500 genérico para no filtrar información del sistema).
- `Error.captureStackTrace`: Preserva el rastreo de pila omitiendo el constructor de `AppError`, facilitando la depuración en logs del servidor.

#### E. Manejador Global de Errores
Al final de `backend/app.js`, Express reconoce el manejador de errores global por su firma de cuatro parámetros `(err, req, res, next)`. Este componente formatea una respuesta HTTP uniforme en formato JSON:

```javascript
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  res.status(statusCode).json({
    status: "error",
    message: err.message,
  });
});
```

Cualquier excepción lanzada en controladores envueltos por `catchAsync` o generada por middlewares de seguridad converge en este punto, garantizando que el cliente reciba un formato consistente `{ status: "error", message: string }`.

---

### 1.2 Patrones de ODM Mongoose en la Persistencia

Mongoose actúa como una capa de modelado de objetos y mapeo relacional sobre MongoDB (*Object Document Mapper* o ODM), proporcionando tipado estricto, validación a nivel de aplicación y abstracción de consultas.

#### A. Esquemas (`Schema`) y Modelos (`Model`)
MongoDB es un motor de base de datos no relacional y flexible por naturaleza. Mongoose impone estructura y disciplina mediante esquemas definidos en la aplicación:
- **`User` (`backend/models/user.model.js`)**: Modela las credenciales de acceso. Implementa hashing criptográfico unidireccional con `bcryptjs` (factor de costo 12) mediante un método estático `hashPassword`, verificación en tiempo constante con `comparePassword`, y protección de privacidad forzando `select: false` en el campo `password` para evitar filtraciones en consultas regulares.
- **`Student` (`backend/models/student.model.js`)**: Modela el perfil universitario y de servicio social del estudiante, vinculado a `User` a través de la clave foránea lógica `authId`.
- **`Activity` (`backend/models/activity.model.js`)**: Modela las convocatorias de servicio comunitario, registrando fechas, horarios, límite de cupos y valor de acreditación en horas (`valueInHours`).
- **`Survey` (`backend/models/survey.model.js`)**: Modela el levantamiento censal de bienestar animal, registrando 12 indicadores clave sobre esterilización, vacunas, maltrato y condición de hábitat.
- **`Pet` (`backend/models/pet.model.js`)**: Modela el registro médico y seguimiento clínico de mascotas individuales atendidas en brigadas.
- **`Contact` (`backend/models/contact.model.js`)**: Modela registros de enlace y comunicación institucional con ciudadanos, vinculados a un estudiante asignado.

#### B. Subdocumentos y Documentos Embebidos
En bases de datos relacionales, una relación 1:N entre Alumnos y Actividades Asignadas requeriría una tabla intermedia (*join table*). En MongoDB y Mongoose, Abaq aprovecha el patrón de **documentos embebidos (subdocumentos)**:

```javascript
// Fragmento de backend/models/student.model.js
const assignedActivitySchema = new mongoose.Schema({
  activityId: { type: mongoose.Schema.Types.ObjectId, ref: "Activity", required: true },
  title: { type: String, required: true },
  type: { type: String, required: true },
  date: { type: Date, required: true },
  valueInHours: { type: Number, required: true },
  status: { type: String, enum: ["pending", "completed", "cancelled"], default: "pending" },
  validatedHours: { type: Number, default: 0 },
});

const studentSchema = new mongoose.Schema({
  // ... campos del estudiante
  assignedActivities: [assignedActivitySchema],
});
```

**Ventajas Arquitectónicas**:
- **Atomicidad por Documento**: Las actualizaciones de horas, cambios de estado de asistencia y desasignaciones se ejecutan de manera atómica sobre el documento del estudiante en una única operación de escritura en disco (`update`).
- **Eficiencia de Lectura**: Al consultar el expediente del estudiante (`GET /student/:id`), todo su historial de actividades y horas validadas se recupera sin necesidad de operaciones costosas `$lookup` (equivalentes a `JOIN`).

#### C. Setters Personalizados y Normalización de Datos
Un desafío común en aplicaciones web full-stack ocurre cuando un endpoint recibe datos provenientes de múltiples fuentes: formularios HTML tradicionales que serializan casillas como `"on"`, APIs REST que envían booleanos JSON nativos (`true`/`false`), o campos omitidos (`undefined`).

En `backend/models/survey.model.js`, Abaq resuelve este problema en la capa del ODM implementando un **setter personalizado** sobre los 12 campos booleanos:

```javascript
// backend/models/survey.model.js
const booleanSetter = (v) => v === "on" || v === true;

const surveySchema = new mongoose.Schema({
  // ...
  vacunas: { type: Boolean, default: false, set: booleanSetter },
  esterilizado: { type: Boolean, default: false, set: booleanSetter },
  // ...
});
```

**Comportamiento**: Antes de persistir el documento en MongoDB o aplicar validaciones, Mongoose ejecuta la función `booleanSetter(v)`. Si el valor entrante es el texto `"on"` (enviado por la plantilla Pug de navegador) o el booleano `true` (enviado por el cliente Angular), el valor almacenado en base de datos es estrictamente `true`. Cualquier otro valor (incluyendo cadenas vacías o `undefined`) resulta en `false`. Esto garantiza consistencia absoluta en las consultas analíticas posteriores.

#### D. Almacenamiento de Archivos Binarios con MongoDB GridFS
Los documentos BSON en MongoDB tienen un límite rígido de 16 megabytes por documento. Para almacenar documentos PDF institucionales (cartas de presentación, cartas de aceptación y constancias de acreditación de servicio social que pueden superar varios megabytes), Abaq utiliza **MongoDB GridFS** (`backend/utils/gridfs.js` y `backend/controllers/upload.controller.js`).

**Arquitectura de GridFS**:
GridFS es una especificación que divide cualquier archivo binario en bloques o fragmentos discretos (*chunks*) de 255 kilobytes cada uno, gestionados en dos colecciones internas de MongoDB:
1. `uploads.files`: Almacena los metadatos del archivo (nombre original, tipo MIME, longitud total en bytes, fecha de carga y un `_id` de tipo `ObjectId`).
2. `uploads.chunks`: Almacena los fragmentos binarios reales indexados por `files_id` y su número de secuencia `n`.

**Ciclo de Carga y Descarga en Streaming**:
- **Carga (`POST /upload`)**: Express utiliza `multer` configurado con almacenamiento en memoria (`multer.memoryStorage()`). El archivo se mantiene en un búfer de memoria volátil únicamente durante la petición. El controlador abre un flujo de escritura en GridFS (`bucket.openUploadStream(filename, { contentType })`), escribe los bytes del búfer y emite un identificador único `fileId`.
- **Descarga (`GET /upload/:fileId`)**: El controlador localiza los metadatos en `uploads.files` mediante `bucket.find({ _id })`, configura las cabeceras HTTP de respuesta (`Content-Type`, `Content-Disposition`) y abre un flujo de lectura (`bucket.openDownloadStream(fileId)`). Este flujo de lectura se conecta directamente a la respuesta HTTP mediante `.pipe(res)`, transmitiendo los bytes de manera continua sin sobrecargar la memoria RAM del servidor.

---

### 1.3 Arquitectura Standalone y Reactividad en Angular 22

El cliente de Abaq está construido con la versión más reciente de **Angular 22**, adoptando por completo los patrones modernos promovidos por el equipo de Angular que eliminan la complejidad de los módulos tradicionales (`NgModule`).

#### A. Filosofía Standalone (Sin NgModules)
Históricamente, Angular requería agrupar componentes, directivas y tuberías en clases anotadas con `@NgModule`. En Angular 22, cada componente, pipe o directiva es **Standalone** por defecto o explícitamente (`standalone: true`).
- Cada componente declara en su propiedad `imports: [...]` exactamente los componentes, directivas (`CommonModule`, `RouterLink`) o módulos de terceros (`ReactiveFormsModule`, `NgbHighlight`) que necesita para renderizar su plantilla HTML.
- **Beneficios**: Desacoplamiento total, compilación más rápida, mejor optimización de empaquetado (*tree-shaking*) y claridad absoluta sobre las dependencias de cada vista.

#### B. Inyección de Dependencias Funcional con `inject()`
En lugar de declarar dependencias en el constructor de las clases (`constructor(private authService: AuthService) { ... }`), Angular 22 permite utilizar la función `inject()` directamente en la inicialización de los campos de clase:

```typescript
export class LoginComponent {
  private fb = inject(FormBuilder);
  private authService = inject(AuthService);
  private router = inject(Router);
  private modalService = inject(ModalService);
}
```

`inject()` aprovecha el **contexto de inyección** (*injection context*) de Angular activo durante la instanciación. Esto simplifica la herencia de clases, reduce el código repetitivo y facilita la creación de funciones auxiliares reutilizables que requieren servicios de Angular.

#### C. Reactividad con RxJS y Estado de Sesión
Abaq utiliza la librería **RxJS** para coordinar flujos de datos asíncronos y eventos en el frontend:
- **`BehaviorSubject` en `AuthService`**: `AuthService` mantiene una instancia privada `userSubject = new BehaviorSubject<AuthUser | null>(null)` y expone un Observable público `user$ = this.userSubject.asObservable()`. Al ser un `BehaviorSubject`, almacena siempre el valor de sesión actual y lo emite inmediatamente a cualquier nuevo suscriptor en la interfaz (como la barra de navegación `NavbarComponent`), permitiendo que la UI reaccione instantáneamente al iniciar o cerrar sesión.

#### D. Prevención de Fugas de Memoria con `takeUntilDestroyed()`
Uno de los problemas más comunes en aplicaciones SPA son las **fugas de memoria** causadas por suscripciones a Observables que no se cancelan al desmontar un componente, manteniendo referencias vivas en memoria (*zombie subscriptions*).

Abaq soluciona esto de manera elegante en componentes como `StudentComponent`, `SurveyComponent`, `PetViewComponent` y `ActivityListComponent` mediante el operador `takeUntilDestroyed()`:

```typescript
export class StudentComponent implements OnInit {
  private destroyRef = inject(DestroyRef);
  searchFilter = new FormControl('');

  ngOnInit(): void {
    this.searchFilter.valueChanges
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe(term => this.filterStudents(term));
  }
}
```

`takeUntilDestroyed` se vincula automáticamente al `DestroyRef` del componente. En el instante exacto en que el usuario navega a otra pantalla y el componente es destruido, el operador completa internamente la tubería de RxJS y libera la suscripción, sin necesidad de implementar manualmente `ngOnDestroy` ni almacenar referencias a `Subscription`.

#### E. Formularios Reactivos Complejos y `FormArray` Dinámicos
Abaq maneja formularios extensos con validaciones en tiempo real mediante `ReactiveFormsModule`. Destacan dos casos de alta complejidad:
1. **`FormComponent` (`frontend/src/app/components/form/form.component.ts`)**: En el levantamiento de encuestas de bienestar animal, el usuario selecciona el tipo de mascota (perro, gato, ambos) y la cantidad. El componente reacciona dinámicamente instanciando un `FormArray` con tantos grupos de formulario anidados como mascotas existan, solicitando de forma dinámica los datos de vacunación y esterilización para cada animal individual.
2. **`SobreMiComponent` (`frontend/src/app/components/sobre-mi/sobre-mi.component.ts`)**: Permite a los estudiantes registrar sus talentos, pasatiempos y mascotas personales mediante `FormArray` dinámicos con botones de agregar y remover elementos en caliente.

#### F. Interceptores HTTP Funcionales
En `frontend/src/app/interceptors/auth.interceptor.ts`, Abaq implementa un interceptor HTTP funcional (`HttpInterceptorFn`):

```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const tokenService = inject(TokenService);
  const token = tokenService.getToken();

  if (token) {
    const clonedReq = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
    });
    return next(clonedReq);
  }

  return next(req);
};
```

**Mecanismo**: Cada vez que cualquier servicio de Angular (`HttpClient`) ejecuta una petición hacia el backend, el interceptor examina si `TokenService` tiene un JWT almacenado en `localStorage`. Si existe, **clona de forma inmutable** la petición agregando la cabecera `Authorization: Bearer <token>` y la entrega a la siguiente función en la cadena `next(clonedReq)`. Si no hay token, la petición continúa inalterada, permitiendo el consumo de rutas públicas (como login o envío de encuestas).

#### G. Guards de Enrutamiento (`CanActivate`)
En `frontend/src/app/guards/auth.guard.ts`, el sistema define dos guardias de navegación para proteger la seguridad de las rutas declaradas en `app.routes.ts`:
- **`AuthGuard`**: Intercepta intentos de navegación a rutas protegidas (`/home`, `/student`, `/activity`, `/survey`, etc.). Si el usuario no tiene una sesión activa válida, cancela la navegación y retorna un `UrlTree` que redirige inmediatamente a `/auth/login`.
- **`PublicGuard`**: Intercepta rutas de acceso público (`/auth/login`, `/auth/signup`). Si un usuario que ya inició sesión intenta entrar a estas páginas, lo redirige automáticamente hacia `/home` para evitar estados incoherentes.

#### H. Patrón Adaptador / Mappers (Separación DTO vs Dominio)
Existe una disparidad natural entre los esquemas de persistencia de MongoDB (cuyas propiedades están escritas tradicionalmente en español y reflejan la estructura de la base de datos) y las interfaces de TypeScript del frontend (diseñadas en inglés, fuertemente tipadas y optimizadas para el consumo visual).

Para mantener ambas capas desacopladas, Abaq utiliza el **patrón Adaptador** mediante mappers puros:
- **`student.mapper.ts`**: Traduce objetos `BackendStudent` (con campos como `nombreCompleto`, `correoInstitucional`, `horasCubrir`) a objetos de dominio `Student` (con campos como `fullName`, `schoolMail`, `requiredHours`, `periods`, `assignedActivities`), y realiza la conversión inversa al enviar actualizaciones al servidor (`mapFrontendStudentToBackend`), preservando identificadores y enlaces a archivos GridFS.
- **`survey.mapper.ts`**: Transforma las estructuras anidadas del formulario reactivo de encuestas (`FormGroup` y `FormArray` de mascotas) al formato plano de 12 booleanos y conteos agregados que espera el endpoint `POST /survey/form/submit`.

---

## Sección 2: Flujo de Datos de Extremo a Extremo (End-to-End Data Flow)

A continuación se detalla el ciclo de vida completo de la información en Abaq, desde la interacción física del usuario en el navegador hasta la persistencia física en el clúster de base de datos y su posterior confirmación visual.

### Diagrama Arquitectónico de Capas y Flujo Secuencial

```
  USUARIO           FRONTEND (ANGULAR 22)               RED                BACKEND (EXPRESS / NODE)             PERSISTENCIA
┌─────────┐   ┌───────────────────────────────┐     ┌──────────┐     ┌────────────────────────────────┐     ┌─────────────────┐
│ Evento  │──>│ 1. Componente Standalone      │     │          │     │                                │     │                 │
│ Click / │   │    (FormGroup / Event Handler)│     │          │     │                                │     │                 │
│ Teclado │   │               │               │     │          │     │                                │     │                 │
└─────────┘   │               ▼               │     │          │     │                                │     │                 │
              │ 2. Servicio Angular           │     │          │     │                                │     │                 │
              │    (Orquestador de Dominio)   │     │          │     │                                │     │                 │
              │               │               │     │          │     │                                │     │                 │
              │               ▼               │     │          │     │                                │     │                 │
              │ 3. HttpClient + Interceptores │     │          │     │                                │     │                 │
              │    (Inyecta Bearer JWT)       │     │          │     │                                │     │                 │
              └───────────────┬───────────────┘     │          │     │                                │     │                 │
                              │ Petición HTTP       │          │     │                                │     │                 │
                              └────────────────────>│ Tránsito │────>│ 4. Express Pipeline            │     │                 │
                                                    │  TCP /   │     │    • CORS & Body Parsers       │     │                 │
                                                    │  TLS     │     │    • auth.middleware (Valida)  │     │                 │
                                                    │          │     │    • express-validator         │     │                 │
                                                    │          │     │               │                │     │                 │
                                                    │          │     │               ▼                │     │                 │
                                                    │          │     │ 5. Controlador (catchAsync)    │     │                 │
                                                    │          │     │    • Lógica de Negocio         │     │                 │
                                                    │          │     │    • Orquestación de Modelos   │     │                 │
                                                    │          │     │               │                │     │                 │
                                                    │          │     │               ▼                │     │                 │
                                                    │          │     │ 6. Capa Mongoose ODM           │     │                 │
                                                    │          │     │    • Hooks & Setters           │     │                 │
                                                    │          │     │    • GridFS Bucket Stream      │     │                 │
                                                    │          │     └───────────────┬────────────────┘     │                 │
                                                    │          │                     │ Operación BSON       │                 │
                                                    │          │                     └─────────────────────>│ 7. MongoDB      │
                                                    │          │                                            │    • Colección  │
                                                    │          │                     ┌──────────────────────│    • Chunks     │
                                                    │          │                     │ Documento / ObjectId │                 │
                                                    │          │     ┌───────────────┴────────────────┐     └─────────────────┘
                                                    │          │     │ 8. Formateo de Respuesta       │
                                                    │          │     │    • JSON Envelope ({ status })│
                                                    │          │     │    • Manejador de Errores      │
                                                    │          │     └───────────────┬────────────────┘
                              ┌─────────────────────│          │<────────────────────┘
                              │ Respuesta HTTP      └──────────┘
                              ▼
              ┌───────────────────────────────┐
              │ 9. Mapper / DTO Adapter       │
              │    (Transforma modelo inglés) │
              │               │               │
              │               ▼               │
              │ 10. RxJS Stream a Componente  │
              │    (Actualiza estado UI)      │
              │               │               │
              │               ▼               │
              │ 11. Detección de Cambios      │
              │    (Renderiza vista en DOM)   │
              └───────────────────────────────┘
```

---

### Explicación Paso a Paso de las 11 Fases del Flujo

1. **Fase 1: Interacción en la Vista (Componente Standalone)**: El usuario interactúa con un formulario o botón (por ejemplo, inscribirse a una actividad comunitaria). El componente captura el evento, valida el estado del formulario reactivo (`form.valid`) y deshabilita temporalmente los controles para prevenir envíos duplicados.
2. **Fase 2: Invocación del Servicio Angular**: El componente delega la operación a un servicio especializado (`ActivityService`, `StudentService`, etc.) inyectado mediante `inject()`.
3. **Fase 3: Interceptor HTTP y Seguridad Saliente**: El servicio invoca un método de `HttpClient` (como `this.http.post(...)`). Antes de que la petición salga a la red, `authInterceptor` captura la instancia de `HttpRequest`, consulta el JWT en `TokenService` y clona la petición agregando el encabezado `Authorization: Bearer <token>`.
4. **Fase 4: Pipeline de Express y Filtrado de Seguridad**:
   - `cors` verifica que la petición provenga del origen permitido.
   - `express.json()` o `multer` extraen y deserializan el cuerpo de la petición.
   - Si la ruta es protegida, se ejecuta `auth.middleware.js`: extrae el Bearer token, verifica su firma criptográfica HMAC con `jwt.verify` y busca el usuario en MongoDB (`User.findById`). Si es válido, lo asocia a `req.user`.
   - Si la ruta cuenta con validaciones declarativas, `validator.js` ejecuta la cadena de comprobaciones de `express-validator`. Si alguna regla falla, responde inmediatamente con código `400 Bad Request` y un arreglo de errores.
5. **Fase 5: Ejecución del Controlador y Lógica de Negocio**: La petición entra al controlador de la ruta, el cual se encuentra protegido por la función de orden superior `catchAsync`. El controlador lee los parámetros validados (`req.params`, `req.body`, `req.user`).
6. **Fase 6: Capa ODM Mongoose y Validación de Dominio**: El controlador invoca métodos del modelo Mongoose (`create`, `findByIdAndUpdate`, `save`). Se disparan validadores de esquema, setters personalizados (como la normalización de booleanos en encuestas) y middleware pre-save si estuviesen configurados.
7. **Fase 7: Persistencia en MongoDB**: Mongoose emite comandos BSON a través del socket TCP hacia la base de datos MongoDB Atlas. En el caso de subida de archivos, se transmiten los fragmentos binarios hacia las colecciones de GridFS (`uploads.files` y `uploads.chunks`).
8. **Fase 8: Formateo de Respuesta o Enrutamiento de Errores**:
   - En caso de éxito, el controlador despacha una respuesta HTTP 200/201 con un envoltorio estandarizado: `{ status: "success", data: { ... } }`.
   - Si ocurre una excepción o validación fallida, `catchAsync` intercepta el rechazo y llama a `next(err)`. El manejador global captura el error, determina el código HTTP adecuado (4xx para `AppError` operacional, 500 para fallos inesperados) y emite un JSON `{ status: "error", message: string }`.
9. **Fase 9: Recepción en el Cliente y Mapeo de Datos (Mapper DTO)**: La respuesta arriba al `HttpClient` de Angular. El servicio recibe el JSON del backend y lo canaliza a través de un Mapper (`student.mapper.ts` o `survey.mapper.ts`). El mapper convierte los nombres de propiedades en español a la interfaz TypeScript en inglés, asegurando tipos de datos consistentes (como la conversión de fechas ISO a objetos `Date`).
10. **Fase 10: Propagación Reactiva al Componente**: El Observable resultante emite el objeto de dominio limpio hacia el componente suscriptor. El operador `takeUntilDestroyed` garantiza que la suscripción sea segura frente al ciclo de vida del componente.
11. **Fase 11: Detección de Cambios y Actualización del DOM**: Angular procesa la emisión, actualiza las variables de estado del componente y su motor de detección de cambios actualiza los elementos visuales del DOM (tablas, gráficos de Chart.js o modales de confirmación).

---

### Casos de Estudio de Extremo a Extremo

#### Caso de Estudio 1: Registro Transaccional de Estudiante y Rollback ante Fallos (`/auth/signup`)
El proceso de alta de un alumno en Abaq involucra la creación coordinada de dos entidades distintas en la base de datos:
1. Las credenciales de acceso en la colección `users` (`email`, `password`, `role = "student"`).
2. El expediente institucional en la colección `students` (`fullName`, `schoolId`, `horasCubrir`, etc.), vinculado mediante `authId: user._id`.

**Manejo de Fallas e Inconsistencias**:
Si la creación del usuario tiene éxito pero la creación del perfil de estudiante falla (por ejemplo, debido a una validación de duplicidad en la matrícula universitaria o fallo de red), el sistema quedaría en un estado inconsistente (una cuenta de acceso huérfana sin perfil de alumno asociado).

En `backend/controllers/auth.controller.js`, el método `signup` previene este problema mediante una **reversión transaccional manual (rollback)**:

```javascript
// Fragmento de backend/controllers/auth.controller.js
const user = await User.create({ email, password, role: "student" });

try {
  const student = await Student.create({
    authId: user._id,
    ...studentData
  });
  // Éxito: Emite token JWT y datos del alumno
} catch (error) {
  // ROLLBACK: Elimina el usuario recién creado para evitar credenciales huérfanas
  await User.findByIdAndDelete(user._id);
  return next(new AppError("Error registering student profile. Rolled back.", 400));
}
```

Este patrón garantiza la integridad referencial sin exigir que la base de datos soporte transacciones réplica complejas en configuraciones básicas de MongoDB.

---

#### Caso de Estudio 2: Ciclo de Carga de Documentos en GridFS y Prevención de Archivos Huérfanos
Cuando un alumno carga su Carta de Aceptación de Servicio Social desde el componente `StudentDetail` (`frontend/src/app/components/student-detail/student-detail.ts`):
1. **Selección del Archivo**: El componente valida que el archivo sea un PDF y no exceda los límites de tamaño.
2. **Subida a GridFS**: Invoca `FileUploadService.uploadFile(file)`. La petición viaja con `FormData` multipart hacia `POST /upload`. Multer recibe el búfer en memoria y `upload.controller.js` lo transmite a `GridFSBucket`. La API responde con la URL permanente: `http://localhost:3000/upload/<fileId>`.
3. **Actualización del Modelo de Estudiante**: El frontend invoca `studentService.updateStudentDocuments(studentId, { acceptanceLetter: fileUrl })`, enviando un `PATCH /student/:id` para guardar la URL en el expediente.
4. **Mecanismo de Limpieza de Archivos Huérfanos (*Orphan Cleanup*)**:
   Si el paso 3 falla (por ejemplo, si el servidor rechaza la actualización del estudiante o se pierde la conexión de red), el archivo recién subido a GridFS quedaría como un recurso huérfano consumiendo almacenamiento indefinidamente en MongoDB.
   `StudentDetail` implementa una salvaguarda en su bloque de captura: si la actualización del estudiante es rechazada, ejecuta de inmediato `fileUploadService.deleteFile(newFileUrl)`, eliminando el archivo binario de GridFS y preservando la higiene del almacenamiento.

---

#### Caso de Estudio 3: Levantamiento y Mapeo de Encuesta Comunitaria con `FormArray` Dinámicos
1. **Interacción Dinámica en la Vista**: En `FormComponent`, un encuestador en campo indica que una familia tiene 2 perros y 1 gato.
2. **Construcción Reactiva**: El formulario reacciona al cambio instanciando dinámicamente dos grupos en el `FormArray` de perros y un grupo en el `FormArray` de gatos, desplegando campos individuales sobre edad aproximada, vacunas antirrábicas y estatus de esterilización.
3. **Agregación en Mapper**: Al presionar enviar, el formulario entrega un árbol de datos anidado. `survey.mapper.ts` ejecuta `mapFormToBackendSurvey(formValue)`:
   - Recorre los arreglos dinámicos para calcular conteos consolidados (`dogCount: 2`, `catCount: 1`).
   - Mapea las respuestas de bienestar a los 12 indicadores booleanos requeridos por la API.
4. **Envío y Validación**: La petición viaja a la ruta pública `POST /survey/form/submit`.
5. **Validación Express-Validator**: `survey.validator.js` comprueba que los campos obligatorios cumplan con los formatos esperados.
6. **Persistencia y Normalización**: Mongoose aplica el `booleanSetter` en `survey.model.js`, persistiendo la encuesta de manera homogénea.
7. **Notificación Automatizada**: Tras guardar la encuesta, `survey.controller.js` invoca a `email.js`, despachando automáticamente un correo electrónico de confirmación mediante Nodemailer hacia la cuenta de la brigada comunitaria.

---

## Sección 3: Lista de Verificación Interactiva Módulo por Módulo

A continuación se presenta el catálogo pedagógico de los **80 archivos de código fuente no-boilerplate** que constituyen la plataforma Abaq. Cada bloque temático inicia con una descripción de su rol arquitectónico y sus responsabilidades técnicas, seguida de una lista de verificación interactiva con casillas `- [ ]` y enlaces relativos directamente navegables.

---

### Bloque 1: Autenticación, Seguridad y Sesión (Auth & Security)

#### Rol Arquitectónico y Responsabilidades
Este bloque constituye la primera línea de defensa y el fundamento de identidad del sistema Abaq. Su función es garantizar que únicamente usuarios legítimos accedan a los recursos protegidos y que cada operación se ejecute bajo el contexto de identidad apropiado.
- **En el Backend**: Centraliza la emisión y validación de JSON Web Tokens (JWT) firmados con algoritmos criptográficos simétricos (HMAC-SHA256). El middleware de autenticación extrae el encabezado `Authorization: Bearer <token>`, valida su caducidad y firma, y vincula el documento del usuario autenticado en `req.user`. Gestiona además el registro transaccional con reversión preventiva (*rollback*) para evitar perfiles huérfanos.
- **En el Frontend**: Mantiene el estado reactivo global de la sesión mediante `AuthService` y `TokenService`. Utiliza interceptores HTTP funcionales para inyectar automáticamente el token en cada petición saliente y `Guards` de enrutamiento para salvaguardar las rutas de administración y servicio social, redirigiendo a los usuarios no autorizados a la vista de inicio de sesión.

#### Lista de Verificación Interactiva (15 Archivos)

- [ ] [`backend/app.js`](../backend/app.js) — *Punto de entrada del backend; orquesta el pipeline global de Express, middlewares de CORS y parsers, montaje de enrutadores, capturador de errores 404 y manejador global de excepciones.*
- [ ] [`backend/models/user.model.js`](../backend/models/user.model.js) — *Esquema Mongoose para cuentas de usuario; encapsula el hashing unidireccional con bcryptjs (cost factor 12), comparación de contraseñas y exclusión de contraseña por defecto.*
- [ ] [`backend/controllers/auth.controller.js`](../backend/controllers/auth.controller.js) — *Controlador de autenticación; administra inicio de sesión, alta administrativa, consulta de perfil propio (`/me`) y registro transaccional coordinado de usuario y estudiante.*
- [ ] [`backend/routes/auth.route.js`](../backend/routes/auth.route.js) — *Definición de rutas de autenticación; expone endpoints públicos (`/login`, `/signup`, `/register`) y privados protegidos por middleware (`/me`).*
- [ ] [`backend/middlewares/auth.middleware.js`](../backend/middlewares/auth.middleware.js) — *Middleware de seguridad; intercepta la cabecera `Authorization: Bearer`, verifica la validez del token JWT y adjunta la entidad del usuario en `req.user`.*
- [ ] [`backend/utils/jwt.js`](../backend/utils/jwt.js) — *Módulo criptográfico auxiliar; abstrae la firma (`signToken`) y verificación (`verifyToken`) de JSON Web Tokens mediante variables de entorno.*
- [ ] [`backend/utils/appError.js`](../backend/utils/appError.js) — *Clase base de errores operacionales; extiende la clase nativa `Error` incorporando código de estado HTTP y bandera de error operacional para control del flujo.*
- [ ] [`backend/utils/catchAsync.js`](../backend/utils/catchAsync.js) — *Envoltorio de orden superior para controladores asíncronos; intercepta promesas rechazadas y las canaliza automáticamente hacia `next(err)`.*
- [ ] [`frontend/src/app/guards/auth.guard.ts`](../frontend/src/app/guards/auth.guard.ts) — *Guardias de enrutamiento Angular (`AuthGuard` y `PublicGuard`); restringen el acceso a vistas privadas y previenen reingreso a pantallas de autenticación.*
- [ ] [`frontend/src/app/interceptors/auth.interceptor.ts`](../frontend/src/app/interceptors/auth.interceptor.ts) — *Interceptor HTTP funcional; intercepta cada solicitud saliente para clonarla e incorporar de manera inmutable la cabecera `Authorization: Bearer <token>`.*
- [ ] [`frontend/src/app/services/token.service.ts`](../frontend/src/app/services/token.service.ts) — *Servicio de almacenamiento local; encapsula la persistencia, lectura y eliminación segura del token JWT en el `localStorage` del navegador.*
- [ ] [`frontend/src/app/services/auth.service.ts`](../frontend/src/app/services/auth.service.ts) — *Servicio central de autenticación en Angular; gestiona el estado reactivo del usuario con `BehaviorSubject`, orquesta login, logout y registro de estudiantes.*
- [ ] [`frontend/src/app/components/login/login.component.ts`](../frontend/src/app/components/login/login.component.ts) — *Componente de interfaz para inicio de sesión; implementa formularios reactivos con validaciones visuales, conmutador de visibilidad de contraseña y modales de error.*
- [ ] [`frontend/src/app/components/signup/signup.component.ts`](../frontend/src/app/components/signup/signup.component.ts) — *Asistente de registro de estudiantes; valida correo institucional, fortaleza de contraseña, enlaces a redes sociales y orquesta la carga previa de documentos.*
- [ ] [`frontend/src/app/components/navbar/navbar.component.ts`](../frontend/src/app/components/navbar/navbar.component.ts) — *Barra de navegación reactiva; reacciona a los cambios en el estado de autenticación (`user$`), desplegando opciones contextuales y disparando el cierre de sesión.*

---

### Bloque 2: Gestión de Estudiantes y Expediente Digital (Student Management)

#### Rol Arquitectónico y Responsabilidades
Este bloque modela el ciclo de vida del estudiante prestador de servicio social universitario, desde su incorporación hasta la culminación de su meta de horas (típicamente 480 horas).
- **En el Backend**: Almacena información académica, datos de contacto, metas de horas (`horasCubrir`), horas validadas y el subdocumento embebido `assignedActivities`. Provee endpoints REST para la consulta individual, actualización de datos de perfil y administración de expedientes.
- **En el Frontend**: Resuelve la discrepancia de nombres entre modelos de base de datos y la interfaz gráfica mediante `student.mapper.ts`. Ofrece una tabla administrativa con filtrado reactivo en tiempo real (`StudentComponent`), un panel personal para el estudiante con visualizaciones gráficas de avance mediante Chart.js (`HomeComponent`), y un expediente digital exhaustivo (`StudentDetail`) con capacidad de previsualización en visor de PDFs integrado y actualización segura de archivos en MongoDB GridFS.

#### Lista de Verificación Interactiva (10 Archivos)

- [ ] [`backend/models/student.model.js`](../backend/models/student.model.js) — *Esquema Mongoose de estudiantes; estructura el perfil académico, balance de horas, enlaces a documentos GridFS y el arreglo de subdocumentos de actividades asignadas.*
- [ ] [`backend/controllers/student.controller.js`](../backend/controllers/student.controller.js) — *Controlador de estudiantes; implementa operaciones CRUD reutilizando la factoría de controladores y proporciona búsqueda especializada por `authId`.*
- [ ] [`backend/routes/student.route.js`](../backend/routes/student.route.js) — *Enrutador REST protegido; define los puntos de acceso para listar alumnos, consultar expedientes específicos y actualizar información de servicio social.*
- [ ] [`backend/validators/student.validator.js`](../backend/validators/student.validator.js) — *Cadenas de validación con express-validator; verifica el formato de correo institucional, fechas de periodo y campos requeridos del estudiante.*
- [ ] [`frontend/src/app/interfaces/student.interfaces.ts`](../frontend/src/app/interfaces/student.interfaces.ts) — *Contratos de datos en TypeScript; define interfaces de dominio para estudiantes, periodos semestrales, redes sociales y actividades asignadas.*
- [ ] [`frontend/src/app/mappers/student.mapper.ts`](../frontend/src/app/mappers/student.mapper.ts) — *Capa adaptadora bidireccional; transforma el esquema en español del backend a la interfaz tipada en inglés del frontend, preservando referencias de documentos.*
- [ ] [`frontend/src/app/services/student.service.ts`](../frontend/src/app/services/student.service.ts) — *Servicio Angular de comunicación con `/student`; ejecuta consultas de alumnos, búsqueda por ID de autenticación y actualización de enlaces a documentos.*
- [ ] [`frontend/src/app/components/student/student.component.ts`](../frontend/src/app/components/student/student.component.ts) — *Vista tabular de administración de alumnos; incluye búsqueda reactiva en tiempo real sobre múltiples campos y cálculo dinámico de horas validadas.*
- [ ] [`frontend/src/app/components/student-detail/student-detail.ts`](../frontend/src/app/components/student-detail/student-detail.ts) — *Expediente digital detallado del estudiante; permite edición de datos, gestión de documentos GridFS con visor de PDF integrado y limpieza de huérfanos.*
- [ ] [`frontend/src/app/components/home/home.component.ts`](../frontend/src/app/components/home/home.component.ts) — *Tablero principal del estudiante voluntario; presenta gráficos interactivos de Chart.js (avance circular de horas e historial lineal) y listado de actividades.*

---

### Bloque 3: Encuestas Comunitarias y Formularios Dinámicos (Surveys & Forms)

#### Rol Arquitectónico y Responsabilidades
Este bloque implementa el instrumento de captura de información sociodemográfica y de bienestar animal durante intervenciones comunitarias en colonias populares.
- **En el Backend**: Expone un endpoint público de ingesta (`POST /survey/form/submit`) validado estrictamente por `survey.validator.js`, que guarda las encuestas censales y dispara notificaciones por correo electrónico a la coordinación comunitaria vía Nodemailer (`email.js`). Provee además rutas protegidas para que los administradores auditen los censos recopilados. Utiliza un setter personalizado en Mongoose para normalizar las respuestas booleanas.
- **En el Frontend**: Incluye un formulario reactivo altamente dinámico (`FormComponent`) que construye arrays de controles anidados (`FormArray`) en respuesta inmediata a la cantidad y especie de mascotas ingresadas por el encuestador. Dispone de un mapeador dedicado (`survey.mapper.ts`) que consolida y traduce los formularios al contrato esperado por el backend, y una tabla de consulta administrativa con acordeón expandible (`SurveyComponent`).

#### Lista de Verificación Interactiva (10 Archivos)

- [ ] [`backend/models/survey.model.js`](../backend/models/survey.model.js) — *Esquema Mongoose para encuestas comunitarias; incorpora setters de normalización booleana que unifican entradas desde formularios HTML ("on") y JSON (true).*
- [ ] [`backend/controllers/survey.controller.js`](../backend/controllers/survey.controller.js) — *Controlador de encuestas; procesa la recepción pública de censos, envía correos de notificación automática y atiende consultas administrativas.*
- [ ] [`backend/routes/survey.route.js`](../backend/routes/survey.route.js) — *Enrutador de encuestas; define la ruta pública `/form/submit` con validación estricta y rutas protegidas para inspección de encuestas levantadas.*
- [ ] [`backend/validators/survey.validator.js`](../backend/validators/survey.validator.js) — *Reglas de validación con express-validator; verifica exhaustivamente los 12 indicadores booleanos y metadatos de ubicación del censo.*
- [ ] [`backend/views/surveyForm.pug`](../backend/views/surveyForm.pug) — *Plantilla de vista Pug servida directamente por Express; provee una interfaz web alternativa renderizada en servidor para el levantamiento censal.*
- [ ] [`frontend/src/app/interfaces/survey.interfaces.ts`](../frontend/src/app/interfaces/survey.interfaces.ts) — *Interfaces TypeScript de encuestas; define la estructura de datos utilizada en tablas, resúmenes y filtros analíticos del frontend.*
- [ ] [`frontend/src/app/mappers/survey.mapper.ts`](../frontend/src/app/mappers/survey.mapper.ts) — *Mapeador de datos de encuestas; traduce formularios reactivos con arreglos dinámicos de mascotas al payload plano de 12 booleanos y conteos agregados.*
- [ ] [`frontend/src/app/services/survey.service.ts`](../frontend/src/app/services/survey.service.ts) — *Servicio Angular de encuestas; realiza el despacho HTTP de formularios a la API pública y gestiona la recuperación de registros para la administración.*
- [ ] [`frontend/src/app/components/survey/survey.component.ts`](../frontend/src/app/components/survey/survey.component.ts) — *Componente de supervisión de encuestas; ofrece tabla de resultados con filas colapsables tipo acordeón y filtrado en tiempo real con `takeUntilDestroyed`.*
- [ ] [`frontend/src/app/components/form/form.component.ts`](../frontend/src/app/components/form/form.component.ts) — *Formulario reactivo dinámico de captura de encuestas; genera controles dinámicos en tiempo real según el número y especie de mascotas registradas.*

---

### Bloque 4: Gestión de Actividades y Acreditación de Horas (Activities)

#### Rol Arquitectónico y Responsabilidades
Este bloque administra el catálogo de convocatorias comunitarias (brigadas de esterilización, vacunación masiva, pláticas de tenencia responsable) y la acreditación formal de horas de servicio social.
- **En el Backend**: Modela las actividades asignándoles un valor en horas (`valueInHours`), tipo (`Obligatorio`/`Opcional`), fechas y límites de inscripción. Provee lógica de negocio especializada para permitir que los alumnos se auto-inscriban (`POST /activity/:id/assign`), cancelen su participación, y que los administradores pasen asistencia y validen horas de manera controlada.
- **En el Frontend**: Ofrece vistas diferenciadas por rol: los alumnos pueden explorar convocatorias vigentes e inscribirse con un solo clic (`ActivityListComponent`), mientras que los administradores pueden convocar nuevos eventos (`ActivityComponent`), monitorear métricas clave en un panel ejecutivo (`AdminHome`), y acreditar horas estudiante por estudiante con modales de confirmación de seguridad en `ActivityDetail`.

#### Lista de Verificación Interactiva (10 Archivos)

- [ ] [`backend/models/activity.model.js`](../backend/models/activity.model.js) — *Esquema Mongoose para actividades comunitarias; almacena título, descripción, cupos, fechas, horarios y equivalencia en horas acreditables.*
- [ ] [`backend/controllers/activity.controller.js`](../backend/controllers/activity.controller.js) — *Controlador de actividades; gestiona el catálogo CRUD, auto-asignación de estudiantes, cancelación y validación administrativa de horas.*
- [ ] [`backend/routes/activity.route.js`](../backend/routes/activity.route.js) — *Enrutador protegido para actividades; define endpoints para registro de convocatorias, gestión de participantes y acreditación de servicio.*
- [ ] [`frontend/src/app/interfaces/activity.interfaces.ts`](../frontend/src/app/interfaces/activity.interfaces.ts) — *Interfaces TypeScript para actividades comunitarias; tipa propiedades de fechas, horarios, cupos y valores en horas de servicio social.*
- [ ] [`frontend/src/app/services/activity.service.ts`](../frontend/src/app/services/activity.service.ts) — *Servicio Angular de actividades; orquesta llamadas HTTP para alta de convocatorias, inscripción/cancelación de voluntarios y validación de horas.*
- [ ] [`frontend/src/app/components/activity/activity.component.ts`](../frontend/src/app/components/activity/activity.component.ts) — *Formulario reactivo de creación de actividades; valida fechas, horarios y horas acreditables con notificaciones temporizadas de retroalimentación.*
- [ ] [`frontend/src/app/components/activity-list/activity-list.component.ts`](../frontend/src/app/components/activity-list/activity-list.component.ts) — *Catálogo interactivo con soporte bimodal; permite auto-inscripción a estudiantes y gestión/eliminación de convocatorias a administradores.*
- [ ] [`frontend/src/app/components/activity-detail/activity-detail.ts`](../frontend/src/app/components/activity-detail/activity-detail.ts) — *Consola administrativa de evento; gestiona pase de lista de asistentes y ejecuta la acreditación controlada de horas con verificación de topes.*
- [ ] [`frontend/src/app/components/admin-home/admin-home.ts`](../frontend/src/app/components/admin-home/admin-home.ts) — *Panel de control para administradores; utiliza `forkJoin` para consolidar métricas de alumnos, convocatorias activas y horas por validar.*
- [ ] [`frontend/src/shared/constants.ts`](../frontend/src/shared/constants.ts) — *Constantes compartidas del cliente; centraliza identificadores de colecciones y configuraciones inmutables de la interfaz.*

---

### Bloque 5: Módulos Auxiliares y Contacto (Auxiliary & Contact)

#### Rol Arquitectónico y Responsabilidades
Este bloque agrupa funcionalidades especializadas de apoyo a la operación de campo y a la vinculación social de la asociación comunitaria.
- **Censo de Mascotas (`Pet`)**: Modela el historial de pacientes veterinarios individuales atendidos en brigadas (datos de esterilización, vacunación antirrábica y consultas médicas) para garantizar seguimiento en visitas subsecuentes.
- **Registro de Contactos (`Contact`)**: Almacena personas y líderes vecinales de enlace comunitario, vinculándolos a un estudiante responsable (`associatedStudentID`) para coordinar citas y seguimiento.
- **Perfil Personal Vocacional (`SobreMi`)**: Cuestionario aplicado a los alumnos prestadores de servicio para registrar sus talentos individuales, disponibilidad, pasatiempos e inventario de mascotas propias en casa.

#### Lista de Verificación Interactiva (15 Archivos)

- [ ] [`backend/models/pet.model.js`](../backend/models/pet.model.js) — *Esquema Mongoose para el censo de mascotas; registra especie, datos del propietario, estado de esterilización, vacunas y citas médicas.*
- [ ] [`backend/controllers/pet.controller.js`](../backend/controllers/pet.controller.js) — *Controlador de mascotas; implementa endpoints de alta, consulta y modificación delegando operaciones estándar en la factoría CRUD.*
- [ ] [`backend/routes/pet.route.js`](../backend/routes/pet.route.js) — *Enrutador protegido para el censo veterinario; expone endpoints REST para la administración de mascotas atendidas.*
- [ ] [`backend/models/contact.model.js`](../backend/models/contact.model.js) — *Esquema Mongoose para contactos de seguimiento; almacena información de líderes comunitarios y enlace con el estudiante asignado.*
- [ ] [`backend/controllers/contact.controller.js`](../backend/controllers/contact.controller.js) — *Controlador de contactos comunitarios; implementa registro y búsqueda con resolución automática de la relación `associatedStudentID`.*
- [ ] [`backend/routes/contact.route.js`](../backend/routes/contact.route.js) — *Enrutador protegido de contactos; define rutas para consulta, creación y actualización del estado de seguimiento vecinal.*
- [ ] [`backend/validators/contact.validator.js`](../backend/validators/contact.validator.js) — *Validaciones express-validator para contactos; garantiza formatos correctos de correo electrónico, teléfonos y marcas de tiempo ISO.*
- [ ] [`frontend/src/app/interfaces/pet.interfaces.ts`](../frontend/src/app/interfaces/pet.interfaces.ts) — *Interfaces TypeScript para mascotas; define contratos tipados para pacientes caninos/felinos, citas y estatus sanitario.*
- [ ] [`frontend/src/app/services/pet.service.ts`](../frontend/src/app/services/pet.service.ts) — *Servicio Angular de mascotas; comunica al frontend con los endpoints `/pet` para consulta y actualización del censo animal.*
- [ ] [`frontend/src/app/components/pet-view/pet-view.component.ts`](../frontend/src/app/components/pet-view/pet-view.component.ts) — *Directorio interactivo de mascotas; implementa búsqueda instantánea mediante RxJS y visualización tabular del estatus veterinario.*
- [ ] [`frontend/src/app/interfaces/contact.interfaces.ts`](../frontend/src/app/interfaces/contact.interfaces.ts) — *Interfaces TypeScript para contactos de seguimiento; tipa los datos personales y el historial de comunicaciones institucionales.*
- [ ] [`frontend/src/app/services/contact.service.ts`](../frontend/src/app/services/contact.service.ts) — *Servicio Angular de contactos; orquesta la obtención y persistencia de fichas de enlace comunitario.*
- [ ] [`frontend/src/app/components/contact/contact.component.ts`](../frontend/src/app/components/contact/contact.component.ts) — *Vista tabular de contactos vecinales; incorpora filtrado reactivo por nombre o teléfono y control de estatus de seguimiento.*
- [ ] [`frontend/src/app/interfaces/sobre-mi.ts`](../frontend/src/app/interfaces/sobre-mi.ts) — *Interfaces para perfil vocacional; tipa cuestionarios personales del voluntario sobre talentos, aficiones y mascotas del hogar.*
- [ ] [`frontend/src/app/components/sobre-mi/sobre-mi.component.ts`](../frontend/src/app/components/sobre-mi/sobre-mi.component.ts) — *Cuestionario vocacional reactivo; utiliza `FormArray` dinámicos para capturar habilidades personales e inventario de animales del estudiante.*

---

### Bloque 6: Infraestructura, Utilidades y Vistas (Infrastructure, Utilities & Views)

#### Rol Arquitectónico y Responsabilidades
Este bloque proporciona los cimientos técnicos, configuraciones de entorno y utilidades transversales que sostienen la ejecución de todos los demás módulos de la plataforma.
- **En el Backend**: Alberga la factoría genérica de controladores CRUD (`handler.controller.js`) que previene la duplicación de lógica de persistencia, el adaptador de validación con soporte para peticiones parciales `PATCH` (`validator.js`), el cliente singleton de streaming con MongoDB GridFS (`gridfs.js`), el servicio SMTP de correo (`email.js`), las plantillas Pug de vista en servidor y rutas stub para futuras expansiones.
- **En el Frontend**: Contiene el punto de arranque de Angular (`main.ts`), la configuración de proveedores e hidratación (`app.config.ts`), la tabla principal de enrutamiento con comodines 404 (`app.routes.ts`), el armazón raíz de la aplicación (`app.component.ts`), servicios y componentes modales reutilizables (`ModalService`, `ModalComponent`), el cliente de subida y streaming de archivos binarios (`FileUploadService`) y los archivos de variables de entorno para desarrollo y producción.

#### Lista de Verificación Interactiva (20 Archivos)

- [ ] [`backend/controllers/handler.controller.js`](../backend/controllers/handler.controller.js) — *Factoría genérica de controladores; genera funciones CRUD (`getAll`, `getOne`, `createOne`, `updateOne`, `deleteOne`) estandarizadas sobre cualquier modelo Mongoose.*
- [ ] [`backend/controllers/upload.controller.js`](../backend/controllers/upload.controller.js) — *Controlador de streaming GridFS; administra la subida de archivos binarios multipart a MongoDB y la descarga en streaming por `fileId`.*
- [ ] [`backend/routes/upload.route.js`](../backend/routes/upload.route.js) — *Enrutador para gestión de archivos; vincula el almacenamiento en memoria de Multer con los flujos de lectura y escritura de GridFS.*
- [ ] [`backend/routes/servicio.route.js`](../backend/routes/servicio.route.js) — *Enrutador stub complementario; reservado como punto de extensión arquitectónica para futuras reglas institucionales de servicio social.*
- [ ] [`backend/middlewares/validator.js`](../backend/middlewares/validator.js) — *Middleware puente de validación; ejecuta cadenas de comprobaciones de express-validator y adapta reglas a opcionales en peticiones `PATCH`.*
- [ ] [`backend/utils/gridfs.js`](../backend/utils/gridfs.js) — *Proveedor singleton de GridFS; inicializa y cachea la instancia `GridFSBucket` conectada a la base de datos de Mongoose bajo el bucket "uploads".*
- [ ] [`backend/utils/email.js`](../backend/utils/email.js) — *Cliente de correo saliente; encapsula el transporte SMTP de Nodemailer con cifrado SSL para notificaciones automáticas de encuestas.*
- [ ] [`backend/views/hello.pug`](../backend/views/hello.pug) — *Plantilla de vista Pug de bienvenida; servida por el endpoint raíz de Express como comprobación rápida de operatividad del servidor.*
- [ ] [`backend/views/student.pug`](../backend/views/student.pug) — *Plantilla de vista Pug de respaldo; interfaz servidor auxiliar para inspección rápida de información de alumnos.*
- [ ] [`frontend/src/main.ts`](../frontend/src/main.ts) — *Punto de arranque de la aplicación Angular; invoca `bootstrapApplication` cargando `AppComponent` y los proveedores de `appConfig`.*
- [ ] [`frontend/src/app/app.config.ts`](../frontend/src/app/app.config.ts) — *Configuración global de proveedores; establece la detección de cambios por zonas, enrutamiento, hidratación y `HttpClient` con interceptores.*
- [ ] [`frontend/src/app/app.routes.ts`](../frontend/src/app/app.routes.ts) — *Árbol central de enrutamiento; define rutas públicas, privadas protegidas con `AuthGuard`, y ruta comodín hacia página de error 404.*
- [ ] [`frontend/src/app/app.component.ts`](../frontend/src/app/app.component.ts) — *Armazón raíz de la interfaz; componente contenedor que aloja la barra de navegación global y el `<router-outlet>` principal.*
- [ ] [`frontend/src/app/services/fileupload.service.ts`](../frontend/src/app/services/fileupload.service.ts) — *Servicio Angular de gestión de archivos; ejecuta cargas multipart con `FormData`, descarga de archivos como `Blob` y borrado de recursos en GridFS.*
- [ ] [`frontend/src/app/services/modal.service.ts`](../frontend/src/app/services/modal.service.ts) — *Servicio para modales dinámicos; encapsula la apertura y cierre de diálogos emergentes construidos con ng-bootstrap.*
- [ ] [`frontend/src/app/components/modal/modal.component.ts`](../frontend/src/app/components/modal/modal.component.ts) — *Componente modal genérico reutilizable; provee diseño estándar para confirmaciones críticas (eliminaciones, acreditaciones) y alertas.*
- [ ] [`frontend/src/app/components/not-found/not-found.component.ts`](../frontend/src/app/components/not-found/not-found.component.ts) — *Página de error 404; desplegada de manera amigable cuando el usuario intenta acceder a rutas no registradas en la aplicación.*
- [ ] [`frontend/src/environments/environment.ts`](../frontend/src/environments/environment.ts) — *Configuración de entorno predeterminada; establece la URL base de la API backend (`http://localhost:3000`) para desarrollo local.*
- [ ] [`frontend/src/environments/environment.development.ts`](../frontend/src/environments/environment.development.ts) — *Configuración específica para el entorno de desarrollo activo en el cliente Angular.*
- [ ] [`frontend/src/environments/environment.production.ts`](../frontend/src/environments/environment.production.ts) — *Configuración optimizada para despliegue productivo; habilita optimizaciones de rendimiento y apunta a la API remota segura.*

---

## Sección 4: Rutas de Lectura Recomendadas por Rol

Para optimizar el tiempo de estudio según el perfil del ingeniero o estudiante que aborda el proyecto, se sugieren las siguientes rutas secuenciales:

### Ruta A: Inducción Rápida a la Arquitectura Global (*Onboarding Path*)
Ideal para desarrolladores recién incorporados que necesitan adquirir una visión panorámica en menos de dos horas:
1. `backend/app.js` ➔ Comprender el montaje general de Express y orden de middlewares.
2. `frontend/src/app/app.routes.ts` ➔ Visualizar el mapa completo de pantallas y seguridad de rutas.
3. `backend/models/user.model.js` y `backend/controllers/auth.controller.js` ➔ Entender la gestión de identidad y el registro transaccional.
4. `frontend/src/app/services/auth.service.ts` y `frontend/src/app/interceptors/auth.interceptor.ts` ➔ Conectar cómo el frontend almacena y transmite las credenciales.
5. `frontend/src/app/components/home/home.component.ts` ➔ Observar cómo se consumen y visualizan los datos en el panel principal.

### Ruta B: Especialización en Persistencia y Backend (*Backend Deep Dive*)
Para ingenieros enfocados en APIs, bases de datos no relacionales y procesamiento asíncrono:
1. `backend/utils/appError.js` y `backend/utils/catchAsync.js` ➔ Dominar el manejo centralizado de errores.
2. `backend/controllers/handler.controller.js` ➔ Estudiar el patrón factoría de controladores CRUD.
3. `backend/models/student.model.js` y `backend/controllers/student.controller.js` ➔ Analizar subdocumentos embebidos y transacciones lógicas.
4. `backend/models/survey.model.js` ➔ Comprender setters personalizados para normalización de tipos.
5. `backend/utils/gridfs.js` y `backend/controllers/upload.controller.js` ➔ Dominar el streaming binario de archivos en MongoDB.

### Ruta C: Especialización en Frontend y Experiencia de Usuario (*Frontend Deep Dive*)
Para ingenieros enfocados en interfaces de usuario reactivas, patrones de componentes y gestión de estado:
1. `frontend/src/app/app.config.ts` ➔ Entender la inicialización de Angular Standalone.
2. `frontend/src/app/mappers/student.mapper.ts` y `frontend/src/app/mappers/survey.mapper.ts` ➔ Estudiar el desacoplamiento de DTOs y modelos de dominio.
3. `frontend/src/app/components/form/form.component.ts` ➔ Dominar formularios reactivos dinámicos y manipulación de `FormArray`.
4. `frontend/src/app/components/student-detail/student-detail.ts` ➔ Analizar la integración de visores de documentos sanitizados y manejo de huérfanos.
5. `frontend/src/app/components/activity-detail/activity-detail.ts` ➔ Revisar flujos administrativos complejos de pase de lista y acreditación de horas.

---
*Documento elaborado como guía arquitectónica de referencia para la plataforma Abaq.*
