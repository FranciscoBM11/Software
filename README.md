# Sistema de Gestión de Proyectos de Graduación UNI

Plataforma web para registrar, revisar, aprobar y dar seguimiento a proyectos de graduación. Caso de estudio de Ingeniería de Software.

## Arquitectura

### C4 Nivel 1 - Contexto

```mermaid
---
config:
  c4:
    c4ShapeMargin: 100
---
C4Context
  title Sistema de Gestión de Proyectos de Graduación UNI - Contexto
  Person(estudiante, "Estudiante", "Registra y da seguimiento a su proyecto de graduación")
  Person(tutor, "Tutor", "Revisa proyectos, registra observaciones y valida avances")
  Person(coordinador, "Coordinador", "Administra asignaciones, estados y seguimiento académico")
  Person(admin, "Administrador", "Administra usuarios, roles y catálogos")
  System(uni, "Sistema de Gestión de Proyectos de Graduación UNI", "Plataforma web para registrar, revisar, aprobar y dar seguimiento a proyectos de graduación")
  System_Ext(email, "Servicio de correo institucional", "Envía notificaciones académicas")
  System_Ext(storage, "Almacenamiento de documentos", "Conserva documentos entregados por los estudiantes")
  Rel(estudiante, uni, "Registra propuestas")
  Rel(tutor, uni, "Revisa proyectos")
  Rel(coordinador, uni, "Gestiona estados")
  Rel(admin, uni, "Administra usuarios")
  Rel(uni, email, "Envía avisos")
  Rel(uni, storage, "Guarda documentos")
  UpdateRelStyle(admin, uni, $offsetX="-70", $offsetY="-18")
  UpdateRelStyle(uni, email, $offsetX="-5", $offsetY="-18")
  UpdateRelStyle(estudiante, uni, $offsetX="-110", $offsetY="12")
  UpdateRelStyle(coordinador, uni, $offsetX="110", $offsetY="12")
  UpdateRelStyle(tutor, uni, $offsetX="35", $offsetY="0")
  UpdateRelStyle(uni, storage, $offsetX="-110", $offsetY="0")
  UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

### C4 Nivel 2 - Contenedores

```mermaid
---
config:
  c4:
    c4ShapeMargin: 100
---
C4Container
  title Sistema de Gestión de Proyectos de Graduación UNI - Contenedores
  Person(estudiante, "Estudiante", "Usuario académico")
  Person(tutor, "Tutor", "Docente tutor")
  Person(coordinador, "Coordinador", "Responsable académico")
  System_Boundary(uni, "Sistema de Proyectos UNI") {
    ContainerDb(db, "Base de Datos", "SQL Server", "Usuarios, proyectos, revisiones, estados y catálogos")
    Container(api, "API REST", "ASP.NET Core Web API / C#", "Autenticación, reglas de negocio y servicios HTTP")
    Container(web, "Aplicación Web", "React / Angular / JavaScript", "Interfaz para estudiantes, tutores y coordinadores")
    Container(files, "Gestor de Documentos", "Almacenamiento de objetos", "Documentos de propuestas, avances y defensa")
  }
  System_Ext(email, "Correo institucional", "SMTP / proveedor de correo")
  Rel(estudiante, web, "Usa", "HTTPS")
  Rel(tutor, web, "Usa", "HTTPS")
  Rel(coordinador, web, "Usa", "HTTPS")
  Rel(web, api, "Consume API", "HTTPS/JSON")
  Rel(api, db, "Lee y escribe", "EF Core")
  Rel(api, files, "Guarda documentos", "HTTPS")
  Rel(api, email, "Envía avisos", "SMTP")
  UpdateRelStyle(estudiante, web, $offsetX="-60", $offsetY="25")
  UpdateRelStyle(tutor, web, $offsetX="25", $offsetY="25")
  UpdateRelStyle(coordinador, web, $offsetX="45", $offsetY="10")
  UpdateRelStyle(web, api, $offsetX="0", $offsetY="-28")
  UpdateRelStyle(api, db, $offsetX="0", $offsetY="-28")
  UpdateRelStyle(api, files, $offsetX="75", $offsetY="25")
  UpdateRelStyle(api, email, $offsetX="5", $offsetY="60")
  UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

### Secuencia

```mermaid
sequenceDiagram
  autonumber
  actor E as Estudiante
  participant W as Aplicación Web
  participant A as API REST
  participant DB as SQL Server
  E->>W: Ingresa correo y contraseña
  W->>A: POST /api/auth/login
  activate A
  A->>DB: Buscar usuario por correo
  DB-->>A: Usuario + hash + rol
  A->>A: Verificar contraseña
  alt Credenciales válidas
    A-->>W: 200 OK + token + datos del usuario
    W-->>E: Mostrar panel principal
    E->>W: Solicitar mis proyectos
    W->>A: GET /api/projects/mine
    A->>DB: Consultar proyectos del estudiante
    DB-->>A: Lista de proyectos
    A-->>W: 200 OK + JSON
    W-->>E: Mostrar proyectos
  else Credenciales inválidas
    A-->>W: 401 Unauthorized
    W-->>E: Mostrar mensaje de autenticación
  end
  deactivate A
```

## API

El contrato completo está en `openapi.yaml` (OpenAPI 3.0.3).
