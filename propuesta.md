PROPUESTA TÉCNICA - TOURNEY MVP 2027

1. TECNOLOGÍA PROPUESTA

Frontend:
- Framework: Flutter (Dart)
  ✅ Una única base de código para Android e iOS
  ✅ Excelente rendimiento en ambas plataformas
  ✅ Hot reload para desarrollo ágil
  ✅ Comunidad madura y librerías de calidad
  ✅ Tiempos de compilación razonables

Backend:
- Lenguaje: Java 17+
- Framework: Spring Boot 3.x
- Build Tool: Maven o Gradle
- Base de datos: PostgreSQL (principal) + Redis (sesiones/caché)
- Autenticación: JWT + refresh tokens
- Traducción automática: Google Cloud Translation API
- Real-time Chat: Spring WebSocket + Stomp

Infraestructura:
- Plataforma de despliegue: Heroku o equivalente (Railway, Render, Fly.io)
- Base de datos: PostgreSQL (Heroku Postgres)
- Caché: Redis (Heroku Redis)
- CDN: CloudFlare
- Monitoring: Heroku Logs

Nota: Todos los costos de infraestructura (Heroku, Postgres, Redis, AWS S3) corren a cargo del cliente.

---

2. ARQUITECTURA PROPUESTA

[Diagrama ASCII de arquitectura]
┌─────────────────────────────────────────────────┐
│         MOBILE (Flutter)                        │
│  ┌──────────────────┐  ┌──────────────────┐    │
│  │   Android APK    │  │   iOS IPA        │    │
│  └──────────────────┘  └──────────────────┘    │
└──────────────────┬──────────────────────────────┘
                   │ HTTPS/REST + WebSocket
┌──────────────────▼──────────────────────────────┐
│    API GATEWAY (Spring Boot + Java)             │
│  • Rate limiting                                │
│  • CORS                                         │
│  • Autenticación JWT                            │
│  • Spring Security                              │
└──────────────────┬──────────────────────────────┘
                   │
        ┌──────────┼──────────┐
        │          │          │
   ┌────▼─────┐ ┌─▼──────┐ ┌─▼────────┐
   │Auth      │ │Chat    │ │Events    │
   │Service   │ │Service │ │Service   │
   └──────────┘ └────────┘ └──────────┘
        │          │          │
        └──────────┼──────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
   ┌────▼─────────┐  ┌───────▼──┐
   │PostgreSQL    │  │Redis     │
   │(persistencia)│  │(sesiones)│
   └──────────────┘  └──────────┘
        │
        ├─ Google Translate API
        └─ Email Service (SendGrid)

Patrones:
- Clean Architecture con separación clara de capas
- Repository Pattern para acceso a datos
- Inyección de dependencias (Spring DI)
- Middleware para validación y autenticación
- MVC Pattern con Spring MVC

---

3. BACKEND PROPUESTO (JAVA)

Stack Técnico:
- Java: OpenJDK 17+
- Framework: Spring Boot 3.1+
- ORM: Spring Data JPA + Hibernate
- Base de datos: PostgreSQL
- Cache: Spring Data Redis
- WebSocket: Spring WebSocket + STOMP
- API REST: Spring MVC + Spring HATEOAS
- Validación: Spring Validation + Bean Validation
- Seguridad: Spring Security + JWT (jjwt o auth0-java-jwt)
- Build: Maven o Gradle
- Despliegue: Docker containerizado para Heroku

Estructura de directorios:
backend/
├── src/main/java/com/tourney/
│   ├── auth/
│   │   ├── controller/
│   │   ├── service/
│   │   ├── repository/
│   │   ├── entity/
│   │   └── dto/
│   ├── users/
│   │   ├── controller/
│   │   ├── service/
│   │   ├── repository/
│   │   ├── entity/
│   │   └── dto/
│   ├── events/
│   ├── chat/
│   │   ├── controller/
│   │   ├── service/
│   │   ├── websocket/
│   │   ├── entity/
│   │   └── dto/
│   ├── ratings/
│   ├── common/
│   │   ├── config/
│   │   ├── exception/
│   │   ├── interceptor/
│   │   ├── filter/
│   │   └── utils/
│   └── TourneyApplication.java
├── src/main/resources/
│   ├── application.yml
│   ├── application-dev.yml
│   ├── application-qa.yml
│   ├── application-prod.yml
│   └── db/migration/ (Flyway)
├── src/test/java/
├── pom.xml (Maven)
├── Dockerfile
├── Procfile (Heroku)
├── docker-compose.yml
└── .env.example

---

4. FLUJO DE ARQUITECTURA DETALLADO

Autenticación:
1. Usuario se registra (email + contraseña)
2. Contraseña hasheada con bcrypt (Spring Security)
3. JWT generado (acceso 15min + refresh 7 días)
4. Recuperación de contraseña vía email (SendGrid)

Chat en Tiempo Real:
1. Conexión WebSocket via Spring WebSocket + STOMP
2. Mensajes almacenados en PostgreSQL
3. Traducción automática antes de enviar (Google Translate)

Gestión de Eventos y "Quiero ir":
1. Evento hardcodeado: Transgrancanaria 2027
2. Administración de entidades (como los eventos u otros) sin implementar o hardcodeada.
3. Asociación usuario-evento en tabla intermedia
4. Filtrado de anfitriones por evento

Valoraciones:
1. Cálculo de promedio de ratings
2. Visible en perfil de anfitrión

---

5. ENTIDADES PRINCIPALES (BD)

-- Users (viajeros y anfitriones)
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    avatar_url VARCHAR(500),
    city VARCHAR(255),
    country VARCHAR(255),
    bio TEXT,
    languages JSONB,
    role VARCHAR(50), -- 'TRAVELER', 'HOST', 'BOTH'
    created_at TIMESTAMP NOT NULL,
    updated_at TIMESTAMP NOT NULL
);

-- Events
CREATE TABLE events (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    location VARCHAR(255),
    created_at TIMESTAMP NOT NULL
);

-- User Events (Quiero ir)
CREATE TABLE user_events (
    user_id UUID REFERENCES users(id),
    event_id UUID REFERENCES events(id),
    interested_in_other_sports TEXT,
    created_at TIMESTAMP NOT NULL,
    PRIMARY KEY (user_id, event_id)
);

-- Messages (Chat)
CREATE TABLE messages (
    id UUID PRIMARY KEY,
    sender_id UUID REFERENCES users(id),
    receiver_id UUID REFERENCES users(id),
    content TEXT NOT NULL,
    translated_content JSONB, -- { "en": "...", "es": "..." }
    created_at TIMESTAMP NOT NULL,
    read_at TIMESTAMP
);

-- Ratings
CREATE TABLE ratings (
    id UUID PRIMARY KEY,
    from_user_id UUID REFERENCES users(id),
    to_user_id UUID REFERENCES users(id),
    event_id UUID REFERENCES events(id),
    stars INTEGER, -- 1-5
    comment TEXT,
    created_at TIMESTAMP NOT NULL
);

-- Conversations (para agrupar chats)
CREATE TABLE conversations (
    id UUID PRIMARY KEY,
    user1_id UUID REFERENCES users(id),
    user2_id UUID REFERENCES users(id),
    event_id UUID REFERENCES events(id),
    last_message_at TIMESTAMP,
    created_at TIMESTAMP NOT NULL,
    UNIQUE(user1_id, user2_id)
);

---

8. FASES PROPUESTAS

Fase 1: Planificación & Setup
- Definición detallada de requisitos
- Creación de repositorio Git
- Setup de infraestructura Heroku (QA + Producción)
- Base de datos PostgreSQL (QA + Producción)
- Redis (caché/sesiones)
- Dockerfile para Java/Spring Boot
- Configuración de CI/CD (GitHub Actions)
- Migraciones de BD con Flyway

Entregables:
- Repositorio funcional
- Infraestructura operativa en QA
- Pipeline de despliegue
- Dockerfile testeado

---

Fase 2: Backend Core
- Módulo de Autenticación (registro, login, recuperación)
- API REST de Usuarios (CRUD perfil)
- API REST de Eventos
- Spring Security + JWT
- Unit tests (JUnit 5 + Mockito)
- Endpoints documentados (Swagger/Springdoc)

Entregables:
- API REST funcional en QA
- Swagger documentation
- Tests con cobertura >80%
- Deployed en QA environment

---

Fase 3: Chat & Traducción
- WebSocket Spring para mensajería real-time
- STOMP protocol implementation
- Integración Google Translate
- Notificaciones push (Firebase Cloud Messaging)
- Rate limiting y validaciones
- Persistencia en PostgreSQL

Entregables:
- Chat funcional en QA
- Tests de WebSocket
- Documentación de API chat
- Performance tested

---

Fase 4: Frontend Flutter
- Semana 1: Onboarding (auth, login, registro)
- Semana 2: Perfil, eventos, lista de anfitriones
- Semana 3: Chat, valoraciones, UI polish

Entregables:
- App compilada para Android e iOS
- Pruebas en dispositivos reales
- Screenshots de flujo principal
- Integration con backend QA

---

Fase 5: Testing
- Pruebas manuales end-to-end
- Testing en múltiples dispositivos (Android/iOS)
- Ajustes de bugs críticos

---

Fase 6: Despliegue & Documentación
- Builds finales optimizadas
- Despliegue a Producción (Heroku)
- Documentación técnica completa
- Setup de monitoring y alertas

Entregables:
- Backend en Heroku Producción
- Documentación técnica completa
- Acceso a repositorio y servicios

---

9. ARQUITECTURA RECOMENDADA (Detalle)

Patrón: Clean Hexagonal Architecture con Spring Boot

Presentation Layer (Controllers - Spring MVC)
         ↓
Application/Service Layer (Business Logic)
         ↓
Domain Layer (Entities, Value Objects)
         ↓
Infrastructure Layer (Repositories, External Services)
         ↓
Data Layer (PostgreSQL + Redis)
         ↓
External Services (S3, Google Translate, SendGrid)

Ventajas de Java/Spring Boot:
- Excelente rendimiento en producción
- Escalabilidad horizontal fácil
- Seguridad robusta (Spring Security)
- Amplio ecosistema de librerías
- Excelente soporte para WebSocket
- Comunidad muy activa
- Ideal para aplicaciones empresariales

---

RIESGOS TÉCNICOS IDENTIFICADOS

Riesgo 1: Traducción automática en chat (ALTO)
Problema: Google Translate API puede ser lenta o causar latencia en mensajes.

Solución:
- Cache de traducciones frecuentes en Redis
- Traducir asincronamente después de enviar
- Fallback a chat sin traducción si falla API
- Rate limiting en Google Translate

---

Riesgo 2: WebSocket escalabilidad (MEDIO)
Problema: Spring WebSocket puede ser problemático con muchos usuarios simultáneos.

Solución:
- Usar Spring Data Redis como message broker
- Implementar load balancing en Heroku
- Monitoreo de conexiones activas
- Plan de escalado predefinido (upgrade de dyno)

---

Riesgo 3: Gestión de imágenes (BAJO)
Problema: Optimización de fotos, espacio de almacenamiento, velocidad.

Solución:
- Compresión automática en backend (ImageMagick/GraphicsMagick)
- CloudFlare CDN para servir imágenes
- Tamaño máximo por foto (5MB)

---

Riesgo 4: Timeline apretado (ALTO)
Problema: Pocas semanas para MVP completo es agresivo.

Solución:
- Equipo de 2-3 desarrolladores (no 1)
- Daily standups para identificar blockers
- Priorizar features core del briefing
- Reducir scope si es necesario
- Buffer de 1 semana antes de launch

---

ENTREGABLES FINALES

1. Repositorio Git completo
   ├── Backend (Java + Spring Boot)
   ├── Frontend (Flutter)
   ├── Infraestructura (Dockerfile + scripts)
   └── CI/CD (GitHub Actions)

2. Documentación
   ├── Guía de instalación (local + Heroku)
   ├── Arquitectura técnica (Spring Boot)
   ├── API Reference (Swagger/OpenAPI)

3. Aplicaciones compiladas
   ├── Android APK + signable build
   ├── iOS IPA + TestFlight build

4. Accesos entregados (Cliente)
   ├── GitHub repo privado
   ├── Heroku dashboard (QA + Prod)
   ├── PostgreSQL credentials
   ├── Redis credentials
   └── Terceros (Google Translate, SendGrid, FCM)

5. Monitoreo
   ├── Heroku logs