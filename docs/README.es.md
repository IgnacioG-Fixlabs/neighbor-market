# 🏢 NeighborMarket

[![README English](https://img.shields.io/badge/README-English-blue)](../README.md)

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/postgresql-%23336791.svg?style=for-the-badge&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/redis-%23DD0031.svg?style=for-the-badge&logo=redis&logoColor=white)
![RabbitMQ](https://img.shields.io/badge/rabbitmq-%23FF6600.svg?style=for-the-badge&logo=rabbitmq&logoColor=white)

> **Plataforma de marketplace comunitario para edificios de apartamentos y comunidades residenciales. Construida con arquitectura de microservicios escalable en Kubernetes, con niveles de membresía, optimización de imágenes y notificaciones en tiempo real.**

## 📋 Tabla de Contenidos

- [Descripción General](#descripción-general)
- [Arquitectura](#arquitectura)
- [Microservicios](#microservicios)
- [Comunicación](#comunicación)
- [Infraestructura](#infraestructura)
- [Tecnologías](#tecnologías)
- [Despliegue](#despliegue)
- [Endpoints de la API](#endpoints-de-la-api)

## 🏛️ Descripción General

NeighborMarket es una arquitectura completa de microservicios diseñada para comunidades residenciales, permitiendo que los vecinos compren y vendan productos dentro de su edificio o comunidad. El sistema cuenta con un sistema de niveles de membresía, procesamiento de imágenes y notificaciones en tiempo real, todo desplegado en un VPS económico con Kubernetes.

El sistema está compuesto por varios microservicios independientes que se comunican a través de HTTP (síncrono) y un Message Broker (asíncrono). En el centro se encuentra el **API Gateway**, que actúa como el "portero" de la aplicación, dirigiendo todo el tráfico hacia los servicios apropiados.

```
Cliente Frontend → API Gateway → Microservicios → Base de Datos
                      ↓
                Message Broker ← → Consumidores de Eventos
```

## 🏗️ Arquitectura

### Componentes del Sistema

- **Frontend**: Aplicación React/Vue/Angular servida como archivos estáticos
- **API Gateway**: Centro de enrutamiento y autenticación
- **Auth Service**: Autenticación de usuarios y gestión de membresías
- **Products Service**: Operaciones CRUD de productos y lógica de negocio
- **Files Service**: Procesamiento y optimización de imágenes
- **Admin Service**: Administración de usuarios y membresías
- **Notifications Service**: Emails y notificaciones basadas en eventos

## 🔧 Microservicios

### 1. 📱 **Cliente Frontend**
- **Tecnología**: React, Vue.js, Angular, o cualquier framework frontend
- **Comunicación**: Se comunica exclusivamente con el API Gateway vía HTTP REST
- **Despliegue**: Archivos estáticos servidos desde Nginx

### 2. 🚪 **API Gateway**
- **Tecnología**: Express.js, FastAPI, NestJS, o similar
- **Puerto**: 4000
- **Funciones**:
    - Recibe todas las peticiones del frontend
    - Enruta peticiones al microservicio correcto
    - Maneja validaciones básicas y autenticación inicial
    - Rate limiting y throttling

**Ejemplos de Enrutamiento:**
- `GET /productos` → Enruta al Products Service
- `POST /auth/login` → Enruta al Auth Service
- `POST /files/upload` → Enruta al Files Service

### 3. 🔐 **Auth Service (Servicio de Autenticación)**
- **Puerto**: 3001
- **Tecnología**: Framework backend + librerías Passport.js/OAuth
- **Base de Datos**: PostgreSQL
- **Características**:
    - Autenticación con Google, Instagram y otros proveedores
    - Gestión de tokens JWT
    - Registro y login de usuarios
    - Validación de permisos y roles
    - Gestión de membresías (Gratuito, Bronce, Plata, Oro)

**Comunicación:**
- Recibe peticiones HTTP del API Gateway
- Envía eventos asíncronos al Message Broker cuando se registra un usuario
- Es consultado por otros servicios para verificación de permisos

### 4. 📦 **Products Service (Servicio de Productos)**
- **Puerto**: 3002
- **Tecnología**: Framework backend + ORM
- **Base de Datos**: PostgreSQL
- **Características**:
    - Operaciones CRUD completas para productos
    - Validación de límites por nivel de membresía
    - Gestión de categorías y metadatos
    - Integración con Files Service para imágenes

**Comunicación:**
- Recibe peticiones HTTP del API Gateway
- Se comunica síncronamente con Auth Service para verificación de permisos
- Se comunica con Files Service para gestión de imágenes
- Envía eventos al Message Broker sobre creación/actualizaciones de productos

### 5. 👨‍💼 **Admin Service (Servicio Administrativo)**
- **Puerto**: 3003
- **Tecnología**: Framework backend
- **Base de Datos**: PostgreSQL (compartida o separada)
- **Características**:
    - Gestión de usuarios y roles
    - Aprobación/rechazo de solicitudes de membresía
    - Panel administrativo
    - Estadísticas e informes

**Comunicación:**
- Recibe peticiones HTTP del API Gateway
- Se comunica síncronamente con Auth Service para actualizar roles
- Escucha eventos del Message Broker sobre solicitudes de membresía
- Envía eventos sobre cambios de membresía

### 6. 🖼️ **Files Service (Servicio de Archivos)**
- **Puerto**: 3005
- **Tecnología**: Framework backend + librerías de procesamiento de imágenes
- **Almacenamiento**: Volúmenes Persistentes de Kubernetes
- **Características**:
    - Recepción y validación de imágenes
    - Procesamiento y optimización de imágenes:
        - Conversión a WebP
        - Generación de thumbnails
        - Redimensionado para web
        - Compresión inteligente
    - Generación de URLs públicas
    - Gestión de archivos estáticos

**Flujo de Procesamiento:**
1. Recibe imagen original
2. Valida tipo, tamaño y seguridad
3. Guarda original en `/uploads/original/`
4. Procesa y optimiza imagen
5. Guarda versiones optimizadas en `/uploads/optimized/`
6. Retorna URL pública: `https://neighbormarket.com/uploads/optimized/products/user123/image456.webp`
7. Guarda URL en base de datos

### 7. 📧 **Notifications Service (Servicio de Notificaciones)**
- **Puerto**: 3004
- **Tecnología**: Framework backend + librería de email
- **Características**:
    - Consumo de eventos del Message Broker
    - Envío de emails transaccionales
    - Notificaciones de bienvenida
    - Confirmaciones de membresía
    - Alertas administrativas

**Comunicación:**
- Principalmente consume eventos asíncronos del Message Broker
- Puede ser llamado síncronamente para notificaciones urgentes
- Se integra con servicios de email externos (SendGrid, Mailgun)

## 📡 Comunicación

### Comunicación Síncrona (HTTP)
- **Cliente ↔ API Gateway**: API REST
- **API Gateway ↔ Servicios**: HTTP interno
- **Servicios ↔ Servicios**: HTTP para operaciones que requieren respuesta inmediata

### Comunicación Asíncrona (Message Broker)
- **Tecnología**: RabbitMQ, Apache Kafka, o Redis
- **Uso**: Eventos y notificaciones que no requieren respuesta inmediata

**Ejemplos de Eventos:**
- `UserRegistered`: Auth Service → Notifications Service
- `MembershipRequested`: Auth Service → Admin Service
- `MembershipApproved`: Admin Service → Notifications Service
- `ProductCreated`: Products Service → Notifications Service
- `ImageProcessed`: Files Service → Products Service

## 🏗️ Infraestructura

### VPS + Kubernetes
- **Proveedor**: DigitalOcean, Hetzner, Contabo
- **Kubernetes**: k3s (ligero)
- **Recursos**: 4GB RAM, 80GB SSD
- **Load Balancer**: Nginx Ingress
- **SSL**: Cloudflare (gratuito)

### Namespaces de Kubernetes
- **frontend**: Aplicación React/Vue/Angular
- **gateway**: API Gateway
- **services**: Todos los microservicios
- **data**: Bases de datos y message broker

### Almacenamiento Persistente
- **Bases de Datos**: PostgreSQL con Volúmenes Persistentes
- **Cache**: Redis
- **Archivos**: Volúmenes para imágenes originales y optimizadas
- **Message Broker**: RabbitMQ con persistencia

## 🛠️ Tecnologías

### Backend (Agnóstico al Lenguaje)
- **Opciones**: Node.js, Python, Java, Go, .NET
- **Frameworks**: Express, FastAPI, Spring Boot, Gin, ASP.NET
- **Base de Datos**: PostgreSQL
- **Cache**: Redis
- **Message Broker**: RabbitMQ

### Frontend
- **Frameworks**: React, Vue.js, Angular
- **Herramientas de Build**: Vite, Webpack, Create React App
- **Despliegue**: Archivos estáticos en Nginx

### DevOps
- **Contenerización**: Docker
- **Orquestación**: Kubernetes (k3s)
- **Reverse Proxy**: Nginx
- **CDN**: Cloudflare
- **Monitoreo**: Prometheus + Grafana
- **CI/CD**: GitHub Actions, GitLab CI

### Procesamiento de Imágenes
- **Node.js**: Sharp
- **Python**: Pillow (PIL)
- **Java**: ImageIO + BufferedImage
- **Go**: paquete imaging

## 🚀 Despliegue

### Estructura de Directorios
```
neighbormarket/
├── frontend/                 # Aplicación frontend
├── api-gateway/             # API Gateway
├── services/
│   ├── auth-service/        # Servicio de autenticación
│   ├── products-service/    # Servicio de productos  
│   ├── admin-service/       # Servicio administrativo
│   ├── files-service/       # Servicio de archivos
│   └── notifications-service/ # Servicio de notificaciones
├── k8s/                     # Manifiestos de Kubernetes
│   ├── namespaces/
│   ├── deployments/
│   ├── services/
│   ├── ingress/
│   └── persistent-volumes/
├── docker-compose.yml       # Para desarrollo local
└── README.md
```

### Comandos de Despliegue
```bash
# Construir imágenes Docker
docker build -t auth-service ./services/auth-service
docker build -t products-service ./services/products-service
# ... otros servicios

# Desplegar a Kubernetes
kubectl apply -f k8s/namespaces/
kubectl apply -f k8s/persistent-volumes/
kubectl apply -f k8s/deployments/
kubectl apply -f k8s/services/
kubectl apply -f k8s/ingress/
```

## 🌐 Endpoints de la API

### Rutas del API Gateway
```
Frontend: https://neighbormarket.com/
Base API: https://neighbormarket.com/api/

Autenticación:
POST /api/auth/login
POST /api/auth/register  
GET  /api/auth/profile
PUT  /api/auth/membership

Productos:
GET    /api/products
POST   /api/products
GET    /api/products/:id
PUT    /api/products/:id
DELETE /api/products/:id

Archivos:
POST /api/files/upload
GET  /uploads/optimized/:path

Administración:
GET  /api/admin/users
PUT  /api/admin/users/:id/role
GET  /api/admin/membership-requests
PUT  /api/admin/membership-requests/:id/approve
```

### Estructura de URLs de Archivos
```
Imágenes optimizadas:
https://neighbormarket.com/uploads/optimized/products/user123/image456.webp
https://neighbormarket.com/uploads/optimized/products/user123/thumb_image456.webp
https://neighbormarket.com/uploads/optimized/avatars/user789/profile.webp
```

## 🔒 Consideraciones de Seguridad

- **Autenticación**: Tokens JWT con refresh tokens
- **Autorización**: Middleware de permisos por servicio
- **Rate Limiting**: A nivel del API Gateway
- **Validación**: Sanitización de inputs en todos los servicios
- **Archivos**: Validación de tipos de archivo, límites de tamaño
- **Red**: Comunicación interna entre pods
- **SSL**: Terminación SSL en Nginx/Cloudflare

## 📊 Monitoreo y Logs

- **Métricas**: Prometheus + Grafana
- **Logs**: Centralizados por namespace
- **Health Checks**: Endpoints `/health` en cada servicio
- **Alertas**: Configuradas en Grafana

---

Esta arquitectura proporciona una base sólida y escalable para un sistema de marketplace comunitario, optimizada para costos y mantenimiento, manteniendo capacidades profesionales de monitoreo y despliegue.

## 🤝 Contribuyendo

1. Haz fork del proyecto
2. Crea tu rama de feature (`git checkout -b feature/CaracteristicaIncreible`)
3. Haz commit de tus cambios (`git commit -m 'Agrega una CaracteristicaIncreible'`)
4. Haz push a la rama (`git push origin feature/CaracteristicaIncreible`)
5. Abre un Pull Request

## 📝 Licencia

Este proyecto está licenciado bajo la Licencia MIT - mira el archivo [LICENSE](LICENSE) para más detalles.

## 🏢 Contexto del Proyecto

Este sistema fue diseñado específicamente para comunidades residenciales, permitiendo que los vecinos de un edificio o conjunto residencial puedan comercializar productos entre ellos de manera segura y organizada. El sistema de membresías permite diferentes niveles de acceso y límites de productos, creando una experiencia personalizada para cada tipo de usuario de la comunidad.