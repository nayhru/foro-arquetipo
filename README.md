# foro-arquetipo

Arquetipo Maven para crear microservicios del **Sistema de Foros en Línea** con la misma estructura y configuración
base que `ms-foro`. Cada proyecto generado compila y arranca con un CRUD de ejemplo que sirve de modelo.

Coordenadas: `com.foro:foro-arquetipo:1.0.0`

## Qué incluye el proyecto generado

| Elemento | Detalle |
|---|---|
| Stack | Spring Boot 4.1.2, Java 21, Spring Data JPA, Bean Validation, Lombok y Actuator |
| Base de datos | Oracle Autonomous Database con wallet (`ojdbc17` y `oraclepki`), `ddl-auto=validate` y pool Hikari chico |
| Seguridad | Resource server que solo **valida** el JWT HS256 de ms-usuarios; el claim `rol` se convierte en `ROLE_xxx` |
| Errores | `GlobalExceptionHandler` con respuestas `ErrorResponse` para 400, 401, 403, 404, 405, 409 y 500 |
| CORS | Orígenes permitidos en `app.cors.origenes` |
| Configuración | `application.properties` con nombre y puerto parametrizados e import de `.env`; `.env.example` como guía |
| Ejemplo | Entidad `Ejemplo` con modelo, repositorio, servicio, controlador, DTOs (record con `@Builder`), script SQL y prueba unitaria |
| Extras | `Dockerfile` multi-stage, `.dockerignore`, `.gitignore` (excluye `.env` y wallet), `.gitattributes` y Maven Wrapper |

Estructura generada (con `-Dpackage=com.foro.moderacion`):

```
ms-moderacion
├── .env.example  .gitignore  .gitattributes  .dockerignore  Dockerfile  README.md
├── mvnw  mvnw.cmd  .mvn/wrapper/maven-wrapper.properties  pom.xml
├── database/script_ejemplo.sql
└── src
    ├── main/java/com/foro/moderacion
    │   ├── MsModeracionApplication.java
    │   ├── config       JwtConfig, SecurityConfig
    │   ├── controller   EjemploController
    │   ├── dto          EjemploRequest, EjemploResponse, ErrorResponse
    │   ├── exception    GlobalExceptionHandler y excepciones 403, 404 y 409
    │   ├── model        Ejemplo
    │   ├── repository   EjemploRepository
    │   └── service      EjemploService, EjemploServiceImpl, Permisos
    ├── main/resources/application.properties
    └── test/java/com/foro/moderacion/service/EjemploServiceImplTest.java
```

## Propiedades

| Propiedad | Obligatoria | Valor por defecto | Uso |
|---|---|---|---|
| `groupId` | Sí | | Grupo Maven |
| `artifactId` | Sí | | Nombre del proyecto y `spring.application.name` |
| `package` | Sí | `groupId` | Paquete raíz de las clases |
| `version` | No | `0.0.1-SNAPSHOT` | Versión del proyecto |
| `puerto` | No | `8080` | `server.port` y `EXPOSE` del Dockerfile |
| `descripcion` | No | `Microservicio del foro` | Descripción del pom y del README |
| `clasePrincipal` | No | `artifactId` en PascalCase | `ms-moderacion` genera `MsModeracionApplication` |

## Instalación

Requiere JDK 21. Desde esta carpeta:

```bash
./mvnw install
```

Esto deja el arquetipo en el repositorio local (`~/.m2/repository/com/foro/foro-arquetipo/1.0.0`) y lo registra en
`archetype-catalog.xml`. Durante el `install` se genera un proyecto de prueba (`src/test/resources/projects/basico`)
y se compila con sus pruebas, así que si el arquetipo se rompe, la instalación falla.

## Generar un microservicio

Desde la carpeta donde quieres crear el proyecto (en Windows usa `mvnw.cmd` y `^` en vez de `\`):

```bash
../foro-arquetipo/mvnw -B org.apache.maven.plugins:maven-archetype-plugin:3.4.0:generate \
  -DarchetypeGroupId=com.foro \
  -DarchetypeArtifactId=foro-arquetipo \
  -DarchetypeVersion=1.0.0 \
  -DgroupId=com.foro \
  -DartifactId=ms-moderacion \
  -Dpackage=com.foro.moderacion \
  -Dpuerto=8083 \
  "-Ddescripcion=Microservicio de moderacion de temas y comentarios del foro"
```

Con Maven instalado también sirve `mvn archetype:generate` con los mismos parámetros. Sin `-B` el plugin pide
confirmación y permite cambiar los valores por defecto.

Después de generar:

1. Copia `.env.example` como `.env` y completa la conexión a Oracle y el `JWT_SECRET` compartido.
2. Reemplaza `Ejemplo` y `database/script_ejemplo.sql` por las entidades y tablas del nuevo microservicio.
3. Agrega en `SecurityConfig` las reglas de acceso de tus rutas.
