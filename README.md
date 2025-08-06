# 🏢 NeighborMarket

[![README Español](https://img.shields.io/badge/README-Español-red)](docs/README.es.md)

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/postgresql-%23336791.svg?style=for-the-badge&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/redis-%23DD0031.svg?style=for-the-badge&logo=redis&logoColor=white)
![RabbitMQ](https://img.shields.io/badge/rabbitmq-%23FF6600.svg?style=for-the-badge&logo=rabbitmq&logoColor=white)

> **Community marketplace platform for apartment buildings and residential communities. Built with scalable microservices architecture on Kubernetes, featuring membership tiers, image optimization, and real-time notifications.**

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Microservices](#microservices)
- [Communication](#communication)
- [Infrastructure](#infrastructure)
- [Technologies](#technologies)
- [Deployment](#deployment)
- [API Endpoints](#api-endpoints)

## 🏛️ Overview

NeighborMarket is a complete microservices architecture designed for residential communities, allowing neighbors to buy and sell products within their building or community. The system features a membership tier system, image processing, and real-time notifications, all deployed on a cost-effective VPS with Kubernetes.

The system is composed of several independent microservices that communicate through HTTP (synchronous) and a Message Broker (asynchronous). At the center is the **API Gateway**, acting as the "gatekeeper" of the application, routing all traffic to the appropriate services.

```
Frontend Client → API Gateway → Microservices → Database
                      ↓
                Message Broker ← → Event Consumers
```

## 🏗️ Architecture

### System Components

- **Frontend**: React/Vue/Angular application served as static files
- **API Gateway**: Central routing and authentication hub
- **Auth Service**: User authentication and membership management
- **Products Service**: Product CRUD operations and business logic
- **Files Service**: Image processing and optimization
- **Admin Service**: User and membership administration
- **Notifications Service**: Email and event-driven notifications

## 🔧 Microservices

### 1. 📱 **Frontend Client**
- **Technology**: React, Vue.js, Angular, or any frontend framework
- **Communication**: Exclusively communicates with API Gateway via HTTP REST
- **Deployment**: Static files served from Nginx

### 2. 🚪 **API Gateway**
- **Technology**: Express.js, FastAPI, NestJS, or similar
- **Port**: 4000
- **Functions**:
    - Receives all requests from frontend
    - Routes requests to correct microservice
    - Handles basic validation and initial authentication
    - Rate limiting and throttling

**Routing Examples:**
- `GET /productos` → Routes to Products Service
- `POST /auth/login` → Routes to Auth Service
- `POST /files/upload` → Routes to Files Service

### 3. 🔐 **Auth Service**
- **Port**: 3001
- **Technology**: Backend framework + Passport.js/OAuth libraries
- **Database**: PostgreSQL
- **Features**:
    - Authentication with Google, Instagram, and other providers
    - JWT token management
    - User registration and login
    - Permission and role validation
    - Membership management (Free, Bronze, Silver, Gold)

**Communication:**
- Receives HTTP requests from API Gateway
- Sends asynchronous events to Message Broker on user registration
- Consulted by other services for permission verification

### 4. 📦 **Products Service**
- **Port**: 3002
- **Technology**: Backend framework + ORM
- **Database**: PostgreSQL
- **Features**:
    - Complete CRUD operations for products
    - Membership tier limit validation
    - Category and metadata management
    - Integration with Files Service for images

**Communication:**
- Receives HTTP requests from API Gateway
- Synchronously communicates with Auth Service for permission verification
- Communicates with Files Service for image management
- Sends events to Message Broker about product creation/updates

### 5. 👨‍💼 **Admin Service**
- **Port**: 3003
- **Technology**: Backend framework
- **Database**: PostgreSQL (shared or separate)
- **Features**:
    - User and role management
    - Membership request approval/rejection
    - Administrative panel
    - Statistics and reports

**Communication:**
- Receives HTTP requests from API Gateway
- Synchronously communicates with Auth Service to update roles
- Listens to Message Broker events about membership requests
- Sends events about membership changes

### 6. 🖼️ **Files Service**
- **Port**: 3005
- **Technology**: Backend framework + image processing libraries
- **Storage**: Kubernetes Persistent Volumes
- **Features**:
    - Image reception and validation
    - Image processing and optimization:
        - WebP conversion
        - Thumbnail generation
        - Web resizing
        - Intelligent compression
    - Public URL generation
    - Static file management

**Processing Flow:**
1. Receives original image
2. Validates type, size, and security
3. Saves original to `/uploads/original/`
4. Processes and optimizes image
5. Saves optimized versions to `/uploads/optimized/`
6. Returns public URL: `https://neighbormarket.com/uploads/optimized/products/user123/image456.webp`
7. Saves URL to database

### 7. 📧 **Notifications Service**
- **Port**: 3004
- **Technology**: Backend framework + email library
- **Features**:
    - Message Broker event consumption
    - Transactional email sending
    - Welcome notifications
    - Membership confirmations
    - Administrative alerts

**Communication:**
- Primarily consumes asynchronous events from Message Broker
- Can be called synchronously for urgent notifications
- Integrates with external email services (SendGrid, Mailgun)

## 📡 Communication

### Synchronous Communication (HTTP)
- **Client ↔ API Gateway**: REST API
- **API Gateway ↔ Services**: Internal HTTP
- **Services ↔ Services**: HTTP for operations requiring immediate response

### Asynchronous Communication (Message Broker)
- **Technology**: RabbitMQ, Apache Kafka, or Redis
- **Usage**: Events and notifications that don't require immediate response

**Event Examples:**
- `UserRegistered`: Auth Service → Notifications Service
- `MembershipRequested`: Auth Service → Admin Service
- `MembershipApproved`: Admin Service → Notifications Service
- `ProductCreated`: Products Service → Notifications Service
- `ImageProcessed`: Files Service → Products Service

## 🏗️ Infrastructure

### VPS + Kubernetes
- **Provider**: DigitalOcean, Hetzner, Contabo
- **Kubernetes**: k3s (lightweight)
- **Resources**: 4GB RAM, 80GB SSD
- **Load Balancer**: Nginx Ingress
- **SSL**: Cloudflare (free)

### Kubernetes Namespaces
- **frontend**: React/Vue/Angular application
- **gateway**: API Gateway
- **services**: All microservices
- **data**: Databases and message broker

### Persistent Storage
- **Databases**: PostgreSQL with Persistent Volumes
- **Cache**: Redis
- **Files**: Volumes for original and optimized images
- **Message Broker**: RabbitMQ with persistence

## 🛠️ Technologies

### Backend (Language Agnostic)
- **Options**: Node.js, Python, Java, Go, .NET
- **Frameworks**: Express, FastAPI, Spring Boot, Gin, ASP.NET
- **Database**: PostgreSQL
- **Cache**: Redis
- **Message Broker**: RabbitMQ

### Frontend
- **Frameworks**: React, Vue.js, Angular
- **Build Tools**: Vite, Webpack, Create React App
- **Deployment**: Static files in Nginx

### DevOps
- **Containerization**: Docker
- **Orchestration**: Kubernetes (k3s)
- **Reverse Proxy**: Nginx
- **CDN**: Cloudflare
- **Monitoring**: Prometheus + Grafana
- **CI/CD**: GitHub Actions, GitLab CI

### Image Processing
- **Node.js**: Sharp
- **Python**: Pillow (PIL)
- **Java**: ImageIO + BufferedImage
- **Go**: imaging package

## 🚀 Deployment

### Directory Structure
```
neighbormarket/
├── frontend/                 # Frontend application
├── api-gateway/             # API Gateway
├── services/
│   ├── auth-service/        # Authentication service
│   ├── products-service/    # Products service  
│   ├── admin-service/       # Administrative service
│   ├── files-service/       # Files service
│   └── notifications-service/ # Notifications service
├── k8s/                     # Kubernetes manifests
│   ├── namespaces/
│   ├── deployments/
│   ├── services/
│   ├── ingress/
│   └── persistent-volumes/
├── docker-compose.yml       # For local development
└── README.md
```

### Deployment Commands
```bash
# Build Docker images
docker build -t auth-service ./services/auth-service
docker build -t products-service ./services/products-service
# ... other services

# Deploy to Kubernetes
kubectl apply -f k8s/namespaces/
kubectl apply -f k8s/persistent-volumes/
kubectl apply -f k8s/deployments/
kubectl apply -f k8s/services/
kubectl apply -f k8s/ingress/
```

## 🌐 API Endpoints

### API Gateway Routes
```
Frontend: https://neighbormarket.com/
API Base: https://neighbormarket.com/api/

Auth:
POST /api/auth/login
POST /api/auth/register  
GET  /api/auth/profile
PUT  /api/auth/membership

Products:
GET    /api/products
POST   /api/products
GET    /api/products/:id
PUT    /api/products/:id
DELETE /api/products/:id

Files:
POST /api/files/upload
GET  /uploads/optimized/:path

Admin:
GET  /api/admin/users
PUT  /api/admin/users/:id/role
GET  /api/admin/membership-requests
PUT  /api/admin/membership-requests/:id/approve
```

### File URL Structure
```
Optimized images:
https://neighbormarket.com/uploads/optimized/products/user123/image456.webp
https://neighbormarket.com/uploads/optimized/products/user123/thumb_image456.webp
https://neighbormarket.com/uploads/optimized/avatars/user789/profile.webp
```

## 🔒 Security Considerations

- **Authentication**: JWT tokens with refresh tokens
- **Authorization**: Permission middleware per service
- **Rate Limiting**: At API Gateway level
- **Validation**: Input sanitization across all services
- **Files**: File type validation, size limits
- **Network**: Internal pod-to-pod communication
- **SSL**: SSL termination at Nginx/Cloudflare

## 📊 Monitoring and Logging

- **Metrics**: Prometheus + Grafana
- **Logs**: Centralized by namespace
- **Health Checks**: `/health` endpoints on each service
- **Alerts**: Configured in Grafana

---

This architecture provides a solid and scalable foundation for a community marketplace system, optimized for costs and maintenance while maintaining professional monitoring and deployment capabilities.

## 🤝 Contributing

1. Fork the project
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.