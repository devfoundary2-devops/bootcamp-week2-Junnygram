# ShopMicro Service Architecture

## Overview
ShopMicro is a microservices-based e-commerce application with comprehensive observability. The system consists of application services, data stores, and observability stack components.

## Service Architecture Diagram

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│    Frontend     │    │    Backend      │    │   ML Service   │
│   (React/Nginx) │    │   (Node.js)     │    │   (Python)      │
│     Port: 80    │◄──►│   Port: 3001    │◄──►│   Port: 3002    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Data Layer                                   │
│  ┌─────────────┐              ┌─────────────┐                  │
│  │ PostgreSQL  │              │    Redis    │                  │
│  │ Port: 5432  │              │ Port: 6379  │                  │
│  └─────────────┘              └─────────────┘                  │
└─────────────────────────────────────────────────────────────────┘
         │                       │                       │
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                 Observability Stack                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │    Loki     │  │    Mimir    │  │   Grafana   │             │
│  │ Port: 3100  │  │ Port: 9009  │  │ Port: 3000  │             │
│  │   (Logs)    │  │ (Metrics)   │  │(Dashboards) │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
└─────────────────────────────────────────────────────────────────┘
```

## Service Details

### 1. Frontend Service
**Technology**: React.js with Nginx  
**Port**: 80  
**Image**: `shopmicro-frontend:latest`  
**Purpose**: User interface for the e-commerce application

#### Key Features:
- React-based single-page application
- Nginx reverse proxy for production
- Static asset serving with caching
- Health check endpoint at `/health`
- API proxy to backend at `/api/*`

#### Connections:
- **Backend**: HTTP requests to `http://backend:3001/api/*`
- **Users**: Serves web interface on port 80

#### Configuration:
- Nginx config includes security headers, gzip compression, and caching
- Proxies API requests to backend service
- Serves React build files from `/usr/share/nginx/html`

---

### 2. Backend Service
**Technology**: Node.js with Express  
**Port**: 3001  
**Image**: `shopmicro-backend:latest`  
**Purpose**: Main API server handling business logic

#### Key Features:
- RESTful API endpoints
- User authentication and authorization
- Product management
- Health monitoring endpoints
- OpenTelemetry tracing integration
- Prometheus metrics collection

#### API Endpoints:
- `/health` - Health check
- `/api/products` - Product management
- `/api/users` - User management
- `/api/frontend-metrics` - Frontend metrics collection
- `/metrics` - Prometheus metrics

#### Connections:
- **PostgreSQL**: Database connection at `postgresql://postgres:postgres@postgres:5432/shopmicro`
- **Redis**: Cache connection at `redis://redis:6379`
- **ML Service**: Recommendation requests (if needed)
- **Frontend**: Serves API requests
- **Observability**: Sends traces to Alloy at `http://alloy:4318/v1/traces`

#### Environment Variables:
- `NODE_ENV=production`
- `PORT=3001`
- `POSTGRES_URL=postgresql://postgres:postgres@postgres:5432/shopmicro`
- `REDIS_URL=redis://redis:6379`
- `OTEL_SERVICE_NAME=shopmicro-backend`

---

### 3. ML Service
**Technology**: Python with Flask  
**Port**: 3002  
**Image**: `shopmicro-ml-service:latest`  
**Purpose**: Machine learning recommendations and analytics

#### Key Features:
- Content-based recommendation engine
- TF-IDF vectorization for product similarity
- Redis caching for recommendations
- Prometheus metrics for ML operations
- OpenTelemetry tracing

#### API Endpoints:
- `/health` - Health check
- `/health/ready` - Kubernetes readiness probe
- `/health/live` - Kubernetes liveness probe
- `/api/recommendations/<user_id>` - Get user recommendations
- `/api/recommendations/popular` - Get popular products
- `/api/model/retrain` - Retrain the ML model
- `/metrics` - Prometheus metrics

#### Connections:
- **Redis**: Caching recommendations at `redis://redis:6379`
- **Backend**: Can fetch product data (configured via `BACKEND_URL`)
- **Observability**: Sends traces and metrics

#### Environment Variables:
- `FLASK_ENV=production`
- `PORT=3002`
- `REDIS_URL=redis://redis:6379`
- `BACKEND_URL=http://backend:3001`

---

### 4. PostgreSQL Database
**Technology**: PostgreSQL  
**Port**: 5432  
**Image**: `postgres:15-alpine`  
**Purpose**: Primary data store for application data

#### Key Features:
- Persistent storage for users, products, orders
- Initialization scripts for schema setup
- Health checks and monitoring

#### Connections:
- **Backend**: Primary database connection
- **Persistent Volume**: Data stored in Kubernetes persistent volume

#### Configuration:
- Database: `shopmicro`
- Username: `postgres`
- Password: `postgres`
- Initialization script: `/docker-entrypoint-initdb.d/init.sql`

---

### 5. Redis Cache
**Technology**: Redis  
**Port**: 6379  
**Image**: `redis:7-alpine`  
**Purpose**: Caching layer and session storage

#### Key Features:
- Session storage for user authentication
- Caching for ML recommendations
- Rate limiting data
- Performance optimization

#### Connections:
- **Backend**: Session and cache storage
- **ML Service**: Recommendation caching
- **Persistent Volume**: Optional data persistence

---

### 6. Loki (Log Aggregation)
**Technology**: Grafana Loki  
**Port**: 3100  
**Image**: `grafana/loki:latest`  
**Purpose**: Centralized log collection and storage

#### Key Features:
- Log aggregation from all services
- Label-based log indexing
- Integration with Grafana for log visualization
- Retention policies for log management

#### Connections:
- **All Services**: Receives logs via log drivers or agents
- **Grafana**: Data source for log visualization
- **Storage**: Filesystem storage for log data

#### Configuration:
- HTTP API on port 3100
- Filesystem storage in `/tmp/loki/`
- Schema version v13 with TSDB

---

### 7. Mimir (Metrics Storage)
**Technology**: Grafana Mimir  
**Port**: 9009 (HTTP), 9095 (gRPC)  
**Image**: `grafana/mimir:latest`  
**Purpose**: Prometheus-compatible metrics storage

#### Key Features:
- Long-term metrics storage
- High availability and scalability
- Prometheus-compatible API
- Multi-tenancy support (disabled in this setup)

#### Connections:
- **All Services**: Receives metrics via Prometheus scraping
- **Grafana**: Data source for metrics visualization
- **Storage**: Filesystem storage for metrics data

#### Configuration:
- HTTP API on port 9009
- gRPC API on port 9095
- Filesystem storage in `/data/mimir/`
- Single replica for development

---

### 8. Grafana (Visualization)
**Technology**: Grafana  
**Port**: 3000  
**Image**: `grafana/grafana:latest`  
**Purpose**: Observability dashboards and visualization

#### Key Features:
- Pre-configured dashboards for all services
- Multiple data source integration
- Alerting and notification capabilities
- User management and access control

#### Data Sources:
- **Mimir**: Metrics data source at `http://mimir:9009/prometheus`
- **Loki**: Logs data source at `http://loki:3100`
- **Tempo**: Traces data source at `http://tempo:3200` (if configured)

#### Connections:
- **Mimir**: Queries metrics data
- **Loki**: Queries log data
- **Users**: Web interface for dashboards

#### Configuration:
- Admin credentials: `admin/admin`
- Auto-provisioned data sources
- Dashboard provisioning from ConfigMaps

---

## Data Flow

### 1. User Request Flow
```
User → Frontend (Nginx) → Backend (Node.js) → PostgreSQL/Redis
                      ↓
                 ML Service (Python) → Redis (cache)
```

### 2. Observability Flow
```
All Services → Logs → Loki
            → Metrics → Mimir
            → Traces → Tempo (if configured)
                    ↓
                 Grafana (Visualization)
```

### 3. Recommendation Flow
```
Frontend → Backend → ML Service → Redis (cache check)
                              → Recommendation Engine
                              → Redis (cache store)
                              → Response
```

## Network Communication

### Internal Service Communication
- All services communicate within the `shopmicro` namespace
- Service discovery via Kubernetes DNS (e.g., `backend.shopmicro.svc.cluster.local`)
- ClusterIP services for internal communication

### External Access
- Frontend exposed via Ingress or NodePort
- Grafana accessible for monitoring dashboards
- All other services internal-only for security

## Health Checks and Monitoring

### Health Endpoints
- **Frontend**: `/health` (returns JSON status)
- **Backend**: `/health` (comprehensive health check)
- **ML Service**: `/health`, `/health/ready`, `/health/live`
- **PostgreSQL**: Built-in health checks
- **Redis**: Built-in health checks

### Monitoring Stack
- **Metrics**: Collected by Prometheus, stored in Mimir
- **Logs**: Aggregated by Loki
- **Traces**: OpenTelemetry integration (Tempo ready)
- **Dashboards**: Grafana with pre-built dashboards

## Security Considerations

### Network Security
- Services isolated within Kubernetes namespace
- No direct external access to databases
- API rate limiting implemented

### Application Security
- Helmet.js security headers in backend
- CORS configuration for cross-origin requests
- Input validation and sanitization
- Secure session management with Redis

## Scaling and Performance

### Horizontal Scaling
- **Frontend**: Can scale multiple replicas (currently 1)
- **Backend**: Configured for 2 replicas with load balancing
- **ML Service**: Single replica (can be scaled based on load)

### Performance Optimizations
- Redis caching for frequently accessed data
- Nginx static asset caching
- Database connection pooling
- Gzip compression for HTTP responses

## Troubleshooting Common Issues

### CrashLoopBackOff Issues
1. **Missing Images**: Ensure Docker images are built and available
2. **Configuration Errors**: Check environment variables and ConfigMaps
3. **Resource Limits**: Verify CPU/memory limits are appropriate
4. **Dependencies**: Ensure dependent services (DB, Redis) are running
5. **Health Checks**: Verify health check endpoints are responding

### Service Communication Issues
1. **DNS Resolution**: Check service names and namespace
2. **Port Configuration**: Verify service ports match container ports
3. **Network Policies**: Ensure no network policies block communication
4. **Firewall Rules**: Check for any blocking rules

This architecture provides a robust, observable, and scalable microservices platform suitable for development and production environments.