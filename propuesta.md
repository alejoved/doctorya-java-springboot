# Propuesta Técnica - Tourney MVP 2027

## 1. Tecnología Propuesta

### Frontend
- **Framework:** React o Flutter
- Una única base de código para Android e iOS
- Excelente rendimiento en ambas plataformas
- Hot reload para desarrollo ágil
- Comunidad madura y librerías de calidad
- Tiempos de compilación razonables

### Backend
- **Lenguaje:** Java 17+
- **Framework:** Spring Boot 3.x
- **Build Tool:** Maven o Gradle
- **Base de datos:** PostgreSQL (principal) + Redis (sesiones/caché)
- **Autenticación:** JWT + refresh tokens
- **Traducción automática:** Google Cloud Translation API
- **Real-time Chat:** Spring WebSocket + STOMP

### Infraestructura
- **Plataforma de despliegue:** Heroku o equivalente (Railway, Render, Fly.io)
- **Base de datos:** PostgreSQL (Heroku Postgres)
- **Caché:** Redis (Heroku Redis)
- **CDN:** CloudFlare
- **Monitoring:** Heroku Logs

> **Nota:** Todos los costos de infraestructura (Heroku, Postgres, Redis, AWS S3) corren a cargo del cliente.

---

## 2. Arquitectura Propuesta

### Diagrama ASCII de arquitectura
```text
┌─────────────────────────────────────────────────┐
│         MOBILE (React o Flutter)                        │
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
```

### Patrones
- Clean Architecture con separación clara de capas
- Repository Pattern para acceso a datos
- Inyección de dependencias (Spring DI)
- Middleware para validación y autenticación
- MVC Pattern con Spring MVC

---

## 3. Backend Propuesto (Java)

### Stack Técnico
- **Java:** OpenJDK 17+
- **Framework:** Spring Boot 3.1+
- **ORM:** Spring Data JPA + Hibernate
- **Base de datos:** PostgreSQL
- **Cache:** Spring Data Redis
- **WebSocket:** Spring WebSocket + STOMP
- **API REST:** Spring MVC + Spring HATEOAS
- **Validación:** Spring Validation + Bean Validation
- **Seguridad:** Spring Security + JWT (jjwt o auth0-java-jwt)
- **Build:** Maven o Gradle
- **Despliegue:** Docker containerizado para Heroku

### Estructura de directorios
```text
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
```

---

## 4. Flujo de Arquitectura Detallado

### Autenticación
1. Usuario se registra (email + contraseña)
2. Contraseña hasheada con bcrypt (Spring Security)
3. JWT generado (acceso 15min + refresh 7 días)
4. Recuperación de contraseña vía email (SendGrid)

### Perfil del Viajero
1. Informacion basica.
2. Resumen de deportes seleccionados
3. Valoracion del viajero

### Perfil del Anfitrion
1. Solicitudes pendientes para chatear
2. Informacion basica
3. Resumen de deportes seleccionados
4. Valoracion del viajero
5. Reviews de viajeros

### Flujo seleccion de deportes
1. Van a estar hardcodeado los deportes Running y MTB
2. Se podra seleccionar alguno de los dos deportes anteriores

### Flujo de la red social
1. Eventos, mensajes, amigos, post, imagenes y videos
2. Formularios para posts, imagenes y videos
3. Feed por cada deporte

### Chat en Tiempo Real
1. Conexión WebSocket via Spring WebSocket + STOMP
2. Mensajes almacenados en PostgreSQL
3. Alternativa de Chat Por WhatsApp

### Gestión de Eventos
1. Evento hardcodeado: Transgrancanaria 2027
2. Administración de entidades (como los eventos u otros) sin implementar o hardcodeada
3. Asociación usuario-evento en tabla intermedia
4. Filtrado de anfitriones por evento

### Valoraciones
1. Cálculo de promedio de ratings
2. Visible en perfil de anfitrión
3. Enviat Valoracion

---

## 5. Fases Propuestas

### Fase 1: Planificación & Setup
- Definición detallada de requisitos
- Creación de repositorio Git
- Setup de infraestructura Heroku (QA + Producción)
- Base de datos PostgreSQL (QA + Producción)
- Redis (caché/sesiones)
- Dockerfile para Java/Spring Boot
- Configuración de CI/CD (GitHub Actions)
- Migraciones de BD con Flyway

**Entregables:**
- Repositorio funcional
- Infraestructura operativa en QA
- Pipeline de despliegue
- Dockerfile testeado

---

### Fase 2: Backend Core
- Módulo de Autenticación (registro, login, recuperación)
- API REST de Usuarios (CRUD perfil)
- API REST de Eventos
- Spring Security + JWT
- Unit tests (JUnit 5 + Mockito)
- Endpoints documentados (Swagger/Springdoc)

**Entregables:**
- API REST funcional en QA
- Swagger documentation
- Tests con cobertura >80%
- Deployed en QA environment

---

### Fase 3: Chat & Traducción
- WebSocket Spring para mensajería real-time
- STOMP protocol implementation
- Integración Google Translate
- Notificaciones push (Firebase Cloud Messaging)
- Rate limiting y validaciones
- Persistencia en PostgreSQL

**Entregables:**
- Chat funcional en QA
- Tests de WebSocket
- Documentación de API chat
- Performance tested

---

### Fase 4: Frontend React o Flutter
- Semana 1: Onboarding (auth, login, registro)
- Semana 2: Perfil, eventos, lista de anfitriones
- Semana 3: Chat, valoraciones, UI polish

**Entregables:**
- App compilada para Android e iOS
- Pruebas en dispositivos reales
- Screenshots de flujo principal
- Integration con backend QA

---

### Fase 5: Testing
- Pruebas manuales end-to-end
- Testing en múltiples dispositivos (Android/iOS)
- Ajustes de bugs críticos

---

### Fase 6: Despliegue & Documentación
- Builds finales optimizadas
- Despliegue a Producción (Heroku)
- Documentación técnica completa
- Setup de monitoring y alertas

**Entregables:**
- Backend en Heroku Producción
- Documentación técnica completa
- Acceso a repositorio y servicios

---

## 6. Arquitectura Recomendada (Detalle)

**Patrón:** Clean Hexagonal Architecture con Spring Boot

```text
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
```

### Ventajas de Java/Spring Boot
- Excelente rendimiento en producción
- Escalabilidad horizontal fácil
- Seguridad robusta (Spring Security)
- Amplio ecosistema de librerías
- Excelente soporte para WebSocket
- Comunidad muy activa
- Ideal para aplicaciones empresariales

---

## Riesgos Técnicos Identificados

### Riesgo 1: Traducción automática en chat (ALTO)
**Problema:** Google Translate API puede ser lenta o causar latencia en mensajes.

**Solución:**
- Cache de traducciones frecuentes en Redis
- Traducir asincronamente después de enviar
- Fallback a chat sin traducción si falla API
- Rate limiting en Google Translate

---

### Riesgo 2: WebSocket escalabilidad (MEDIO)
**Problema:** Spring WebSocket puede ser problemático con muchos usuarios simultáneos.

**Solución:**
- Usar Spring Data Redis como message broker
- Implementar load balancing en Heroku
- Monitoreo de conexiones activas
- Plan de escalado predefinido (upgrade de dyno)

---

### Riesgo 3: Gestión de imágenes (BAJO)
**Problema:** Optimización de fotos, espacio de almacenamiento, velocidad.

**Solución:**
- Compresión automática en backend (ImageMagick/GraphicsMagick)
- CloudFlare CDN para servir imágenes
- Tamaño máximo por foto (5MB)

---

### Riesgo 4: Timeline apretado (ALTO)
**Problema:** Pocas semanas para MVP completo es agresivo.

**Solución:**
- Equipo de 2-3 desarrolladores (no 1)
- Daily standups para identificar blockers
- Priorizar features core del briefing
- Reducir scope si es necesario
- Buffer de 1 semana antes de launch

---

## 7. Entregables Finales

### 1. Repositorio Git completo
```text
├── Backend (Java + Spring Boot)
├── Frontend (Flutter)
├── Infraestructura (Dockerfile + scripts)
└── CI/CD (GitHub Actions)
```

### 2. Documentación
```text
├── Guía de instalación (local + Heroku)
├── Arquitectura técnica (Spring Boot)
└── API Reference (Swagger/OpenAPI)
```

### 3. Aplicaciones compiladas
```text
├── Android APK + signable build
└── iOS IPA + TestFlight build
└── Publicacion Android APK en Play Store
└── Publicacion iOS IPA en App Store
```

### 4. Accesos entregados (Cliente)
```text
├── GitHub repo privado
├── Heroku dashboard (QA + Prod)
├── PostgreSQL credentials
├── Redis credentials
└── Terceros (Google Translate, SendGrid, FCM)
```

### 5. Monitoreo
```text
└── Heroku logs
```

## 8. Presupuesto

### Construccion de marca
Identidad de Marca: **1.500 €**

### Aplicativo Quemado — **7.400 €**

| Área | Precio |
|---|---|
| BackEnd | 3.700 € |
| Diseño y FrontEnd | 3.700 € |

- Autenticación completa
- Selección de deportes básica
- Perfil del viajero completo
- Perfil del anfitrión completo
- Red social **basico**
- Gestión de eventos **basico**
- Valoraciones completo
- Chat en tiempo real completo

---

### Aplicativo Medio — **10.400 €**

| Área | Precio |
|---|---|
| BackEnd | 5.700 € |
| Diseño y FrontEnd | 4.700 € |

- Autenticación completa
- Selección de deportes básica
- Perfil del viajero completo
- Perfil del anfitrión completo
- Red social **completo**
- Gestión de eventos **basico**
- Valoraciones completo
- Chat en tiempo real completo

---

### Aplicativo Completo — **12.400 €**

| Área | Precio |
|---|---|
| BackEnd | 6.700 € |
| Diseño y FrontEnd | 5.700 € |

- Autenticación completa
- Selección de deportes **completo**
- Perfil del viajero completo
- Perfil del anfitrión completo
- Red social **completo**
- Gestión de eventos **completo**
- Valoraciones completo
- Chat en tiempo real completo

---

### Propuesta de Trabajo Mensual

| Plan | BackEnd | Diseño y FrontEnd |
|---|---|---|
| Aplicativo quemado | 750 € | 750 € |
| Aplicativo medio | 1.200 € | 950 € |
| Aplicativo completo | 1.400 € | 1.200 € |

---

## 9. Costos de Infraestructura

> **Nota:** Todos los costos listados a continuación corren a cargo del cliente y son independientes del presupuesto de desarrollo. Los precios están en USD y son los vigentes a julio de 2026.

### Heroku — Contenedores (Dynos)

| Servicio | Plan recomendado | Costo/mes |
|---|---|---|
| Backend (Spring Boot) | Standard-1X (0.5 GB RAM) | $25 |
| Backend (Standard-2X si se necesita más memoria) | Standard-2X (1 GB RAM) | $50 |
| Frontend / Servidor estático | Basic (0.5 GB RAM) | $7 |

> Para QA/staging se puede usar el plan **Basic ($7/mes)** para reducir costos.

---

### Heroku — Base de Datos PostgreSQL

| Plan | Almacenamiento | Conexiones | Costo/mes | Uso recomendado |
|---|---|---|---|---|
| Essential-0 | 1 GB | 20 | ~$5 | Desarrollo / QA |
| Essential-1 | 10 GB | 20 | ~$9 | Producción MVP inicial |
| Essential-2 | 32 GB | 40 | ~$18 | Producción con mayor crecimiento |
| Standard-0 | 64 GB | 200 | ~$50 | Producción con alta disponibilidad |

> **Configuración:** Essential-0 para QA (~$5/mes) y Standard-0 para producción (~$50/mes).

---

### Heroku — Caché Redis (Key-Value Store)

| Plan | Memoria | Costo/mes | Uso recomendado |
|---|---|---|---|
| Mini | 25 MB | ~$3 | Desarrollo / QA |
| Premium-0 | 100 MB | ~$15 | Producción MVP |
| Premium-1 | 250 MB | ~$30 | Producción con mayor carga |

> **Configuración:** Mini para QA (~$3/mes) y Premium-0 para producción (~$15/mes).

---

### Servicios de Terceros

| Servicio | Plan | Costo/mes | Notas |
|---|---|---|---|
| Google Cloud Translation API | Pay per use | ~$20 | $20 por 1M caracteres; primeros 500K/mes gratis |
| SendGrid (email transaccional) | Free / Essentials | $0–$20 | Free: 100 emails/día; Essentials: 50K emails/mes |
| Firebase Cloud Messaging (push) | Free | $0 | Notificaciones push sin costo |
| CloudFlare CDN | Free | $0 | CDN, DNS y protección DDoS |

---

### Almacenamiento de Medios — Red Social (Imágenes y Videos)

La red social requiere almacenamiento de objetos y procesamiento de medios. Se recomiendan dos servicios complementarios:

#### Cloudflare R2 — Almacenamiento de objetos

> **Ventaja clave:** egress (descarga) completamente **gratuito**, a diferencia de AWS S3 que cobra ~$0.09/GB de salida.

| Concepto | Tier gratuito | Precio si supera el free tier |
|---|---|---|
| Almacenamiento | 10 GB-mes/mes gratis | $0.015 / GB-mes |
| Operaciones Class A (subida) | 1M requests/mes gratis | $4.50 / millón |
| Operaciones Class B (lectura) | 10M requests/mes gratis | $0.36 / millón |
| Egress (descarga a internet) | **Gratis siempre** | **Gratis siempre** |

**Estimación MVP (primeros meses):** con menos de 500 usuarios activos el consumo cabe en el free tier o cuesta ~$5–15/mes de almacenamiento.

#### Cloudinary — Procesamiento y entrega de imágenes/videos

Permite redimensionar, comprimir, convertir formatos y transcodificar videos automáticamente antes de guardar en R2 o servir desde su CDN.

| Plan | Créditos/mes | Costo/mes | Uso recomendado |
|---|---|---|---|
| Free | 25 créditos | $0 | MVP / primeros usuarios |
| Plus | 225 créditos | $89 | Crecimiento moderado |
| Advanced | 600 créditos | $224 | Alta demanda de media |

> 1 crédito ≈ 1 GB transformado/entregado o ~1 min de video transcodificado.
> El plan **Free es suficiente para el MVP** con pocos cientos de usuarios.

---

### Publicación en Tiendas de Aplicaciones

| Tienda | Tipo de pago | Costo |
|---|---|---|
| Google Play Store | Registro único | **$25** (pago único, de por vida) |
| Apple App Store | Membresía anual | **$99/año** (~$8.25/mes) |

---

### Resumen de Costos Mensuales Estimados — Producción

#### Escenario A: Cloudinary Free (MVP inicial)

| Concepto | Plan | Costo/mes (USD) |
|---|---|---|
| Heroku — Dyno Backend | Standard-1X (0.5 GB RAM) | $25 |
| Heroku — Dyno Frontend | Standard-1X (0.5 GB RAM) | $25 |
| Heroku — PostgreSQL | Standard-0 (64 GB, 200 conn.) | ~$50 |
| Heroku — Redis | Premium-0 (100 MB) | ~$15 |
| Google Cloud Translation API | Pay per use | ~$20 |
| SendGrid | Essentials (50K emails/mes) | ~$20 |
| Cloudflare R2 | ~50 GB almacenamiento + ops | ~$10 |
| Cloudinary | Free (25 créditos/mes) | $0 |
| Firebase Cloud Messaging | Free | $0 |
| CloudFlare CDN | Free | $0 |
| **Total mensual producción (Escenario A)** | | **~$165/mes** |

#### Escenario B: Cloudinary Plus (crecimiento moderado)

| Concepto | Plan | Costo/mes (USD) |
|---|---|---|
| Heroku — Dyno Backend | Standard-1X (0.5 GB RAM) | $25 |
| Heroku — Dyno Frontend | Standard-1X (0.5 GB RAM) | $25 |
| Heroku — PostgreSQL | Standard-0 (64 GB, 200 conn.) | ~$50 |
| Heroku — Redis | Premium-0 (100 MB) | ~$15 |
| Google Cloud Translation API | Pay per use | ~$20 |
| SendGrid | Essentials (50K emails/mes) | ~$20 |
| Cloudflare R2 | ~100 GB almacenamiento + ops | ~$15 |
| Cloudinary | Plus (225 créditos/mes) | $89 |
| Firebase Cloud Messaging | Free | $0 |
| CloudFlare CDN | Free | $0 |
| **Total mensual producción (Escenario B)** | | **~$259/mes** |

---

### Resumen de Costos Mensuales Estimados — QA / Staging

| Concepto | Plan | Costo/mes (USD) |
|---|---|---|
| Heroku — Dyno Backend | Basic (0.5 GB RAM) | $7 |
| Heroku — Dyno Frontend | Basic (0.5 GB RAM) | $7 |
| Heroku — PostgreSQL | Essential-0 (1 GB, 20 conn.) | ~$5 |
| Heroku — Redis | Mini (25 MB) | ~$3 |
| Google Cloud Translation API | Pay per use (uso mínimo) | ~$2 |
| SendGrid | Free (100 emails/día) | $0 |
| Firebase Cloud Messaging | Free | $0 |
| CloudFlare CDN | Free | $0 |
| **Total mensual QA** | | **~$24/mes** |

---

### Costos Únicos / Anuales

| Concepto | Costo (USD) |
|---|---|
| Google Play Store — Registro | $25 (único) |
| Apple Developer Program — Membresía | $99/año |
| **Total primer año** | **~$124** |
