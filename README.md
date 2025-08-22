# Podman v5 Traefik v3 Development Stack

## 🎯 Purpose

A complete **rootless Podman** development stack with **Traefik v3** reverse proxy, showcasing modern container orchestration patterns. This repository bridges the gap between Docker-focused tutorials and real-world Podman deployments.

> **⚠️ DEVELOPMENT ONLY**  
> This setup is **strictly for local development and learning**. All credentials are demo values. Never use this configuration for staging or production environments.

## 📋 Description

Full-featured localhost stack with automated TLS, HTTP→HTTPS redirection, and service discovery:

- **Frontend**: nginx:alpine serving static content
- **Backend**: Python FastAPI application with database integration
- **Database**: PostgreSQL with automated initialization
- **Reverse Proxy**: **Traefik v3** with SSL termination and auto-discovery
- **Admin Tools**: pgAdmin for database management
- **Automation**: Template-based configuration generation

## 🆕 What's New in This Version

### **Traefik v3 Upgrade**
- **File Provider + Docker Provider**: Clean separation of concerns
- **Improved SSL**: Automated certificate management with mkcert
- **Enhanced Routing**: Better host-based routing with middleware chains
- **Modern Configuration**: Updated for Traefik v3 syntax

### **Template System**
- **Automated Setup**: All configuration files generated from templates
- **Environment-Driven**: Single `.env.dev` file controls all services
- **Reproducible**: `setup_containers.sh` creates identical deployments

### **Enhanced Tooling**
- **Health Diagnostics**: `diagnose_miniapp.sh` for troubleshooting
- **Certificate Management**: `setup_certificates.sh` for SSL automation
- **Documentation**: Complete with working screenshots

## 🛠️ Prerequisites

### System Requirements
- **OS**: Debian 12+ / Ubuntu 22+ (tested on Debian 12 arm64)
- **User**: Non-root user with Podman access (example: `dockeruser`)
- **Resources**: 4GB RAM, 40GB storage minimum

### Required Software
for **podman v5.x** full build from sources (debian), please refer to : https://github.com/HornetGit/podman_v5.x_on_debian12
- **Podman**: v5.3.1+ (rootless mode)
- **podman-compose**: v1.4.0+
- **crun**: v1.22.0+
- **mkcert**: For local SSL self-signed certificates
- **Go**: 1.23+ (for building tools)

### Optional Tools
- **Visual Studio Code**: Recommended IDE
- **Tilix**: Terminal emulator for multi-pane log monitoring

## 🚀 Quick Start

### 1. System Preparation
```bash
# Create dedicated user for Podman (if needed)
sudo adduser dockeruser
sudo usermod -aG sudo dockeruser
su - dockeruser

# Create project directory
mkdir ~/podman-stack/version01 && cd ~/podman-stack/version01
```

### 2. Repository Setup
```bash
# Clone this repository
git clone https://github.com/HornetGit/podman-v5-traefik-v3-stack .

# Verify structure matches reference
cat tree.doc
```

### 3. Automated Setup
```bash
# Generate all configuration files from templates
./setup_containers.sh

# This automatically:
# - Creates SSL certificates (mkcert)
# - Generates all config files from templates
# - Sets up Podman socket for Traefik
# - Validates system requirements
```

### 4. Launch Stack
```bash
# Start all services
./start_containers.sh

# Or use restart for a full stack (re)build
./restart_application.sh
```

### 5. Verify Deployment
```bash
# Run comprehensive health checks
./diagnose_miniapp.sh
```

## 🌐 Access Points

| Service | HTTP | HTTPS | Purpose |
|---------|------|-------|---------|
| **Frontend** | http://frontend.localhost:8000/ | https://frontend.localhost:8443/ | Main web application |
| **Backend API** | http://backend.localhost:8000/ | https://backend.localhost:8443/ | REST API (try `/` endpoint) |
| **Traefik Dashboard** | http://traefik.localhost:8080/dashboard/ | https://traefik.localhost:8443/dashboard/ | Proxy management |
| **pgAdmin** | http://localhost:5050 | — | Database GUI |

> **Note**: HTTP URLs automatically redirect to HTTPS via Traefik middleware. Use HTTPS URLs for actual testing.

### Default Credentials (Demo Only)
- **pgAdmin**: admin@localtest.me / adminpass
- **Database**: miniapp_user / miniapp_password

## 📸 Screenshots

### Application Flow
![Frontend Interface](./docs/images/05_frontend.png)
*Main application at https://frontend.localhost:8443/ - submit messages to test the full stack*

![Backend API](./docs/images/06_backend.png)
*Backend API responding at https://backend.localhost:8443/ - JSON REST endpoint*

### Traefik v3 Management
![Traefik Dashboard](./docs/images/01_traefik_dashboard.png)
*Traefik v3 dashboard showing all discovered services and health status*

![HTTP Routers](./docs/images/02_traefik_http_routers.png)
*HTTP routers configuration - automatic service discovery in action*

![HTTP Services](docs/images/03_traefik_http_services.png)
*Backend services with load balancer configuration and health checks*

![Middlewares](./docs/images/04_traefik_http_middlewares.png)
*Middleware chain showing HTTP→HTTPS redirects and security headers*

### Database Management
![pgAdmin Interface](docs/images/07_pgadmin.png)
*pgAdmin showing stored messages from frontend form submissions*

## 🏗️ Architecture

```
Internet/Browser
       ↓ :8443 (HTTPS)
   ┌─────────────┐
   │   Traefik   │ ← SSL Termination, Auto-Discovery
   │     v3      │ ← File Provider (certs) + Docker Provider (services)
   └─────────────┘
       ↓ Container Network
┌──────────┬──────────────┐
│ Frontend │   Backend    │ ← Public Network (miniapp_public)
│ (nginx)  │  (FastAPI)   │
└──────────┴──────────────┘
               ↓ Database Connection
       ┌─────────────────┐
       │   PostgreSQL    │ ← Private Network (miniapp_private)  
       │   + pgAdmin     │
       └─────────────────┘
```

### Network Topology
- **Public Network** (`miniapp_public`): Frontend, Backend, Traefik
- **Private Network** (`miniapp_private`): Backend, Database, pgAdmin
- **Dual Network**: Backend bridges both networks for security

## 🔧 Configuration Details

### Template System
All configuration files are generated from templates in `*/templates/` directories:

```bash
backend/templates/main.py.template     → backend/app/main.py
frontend/templates/index.html.template → frontend/index.html
traefik/templates/traefik.yml.template → traefik/traefik.yml
# ... and many more
```

### Environment Variables
Single source of truth in `.env.dev`:
```bash
# Domain configuration
MINIAPP_TRAEFIK_BASE_DOMAIN=localhost
MINIAPP_FRONTEND_DOMAIN=frontend.localhost
MINIAPP_BACKEND_DOMAIN=backend.localhost

# Network configuration
NETWORK_PUBLIC_NAME=miniapp_public
NETWORK_PRIVATE_NAME=miniapp_private

# Service ports
MINIAPP_TRAEFIK_HTTPS_PORT_HOST=8443
MINIAPP_BACKEND_PORT_CONT=8001
# ... etc
```

### Traefik v3 Features
- **Hybrid Providers**: File provider for TLS/middleware, Docker provider for service discovery
- **Automatic HTTPS**: All services redirect HTTP→HTTPS automatically  
- **Host-Based Routing**: Clean domain separation (`frontend.localhost`, `backend.localhost`, etc.)
- **Self-Signed Certificates**: mkcert integration for trusted local SSL

## 🚧 Troubleshooting

### Quick Diagnostics
```bash
# Comprehensive system check
./diagnose_miniapp.sh

# Individual container logs
podman logs miniapp_traefik
podman logs miniapp_backend
podman logs miniapp_frontend

# Network inspection
podman network ls
podman network inspect miniapp_public

# Service started, and healthy if required
podman ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

### Common Issues

| Issue | Solution |
|-------|----------|
| **Certificate warnings** | Run `mkcert -install` and restart browser |
| **Port conflicts** | Check ports 8000, 8080, 8443, 5050 are free |
| **Service discovery fails** | Verify Podman socket: `systemctl --user status podman.socket` |
| **Database connection fails** | Wait 10s for PostgreSQL initialization |
| **404 errors** | Check Traefik dashboard for service registration |

### Log Monitoring
For development, monitor logs in separate terminals:
```bash
# Terminal 1: All services
podman-compose logs -f

# Terminal 2: Traefik only  
podman logs -f miniapp_traefik

# Terminal 3: Backend only
podman logs -f miniapp_backend
```

## 🧪 Testing the Stack

### End-to-End Test
1. **Frontend**: Visit https://frontend.localhost:8443/
2. **Submit Data**: Enter text in form and submit
3. **Backend Processing**: Watch backend logs for "✅ Inserted into DB"
4. **Database Verification**: Check pgAdmin for new record
5. **Traefik Routing**: Monitor dashboard for request flow

### API Testing
```bash
# Direct backend test
curl -k https://backend.localhost:8443/

# Form submission test
curl -k -X POST https://backend.localhost:8443/submit \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello from curl!"}'
```

## 📚 Learning Opportunities

### What You'll Learn
- **Podman vs Docker**: Rootless container orchestration
- **Traefik v3**: Modern reverse proxy patterns
- **Template Systems**: Configuration as code
- **Network Security**: Multi-tier application architecture
- **SSL/TLS**: Local certificate management
- **API Development**: FastAPI with database integration

### Next Steps
- **Monitoring**: Add Prometheus/Grafana stack
- **Security**: Implement authentication (OAuth2/JWT), add podman v5 healthchecks
- **Testing**: Add automated integration tests  
- **Scaling**: Add apps, Convert to Kubernetes/OpenShift
- **Production**: Implement Let's Encrypt automation, secure accesses etc (not comprehensive tasks)
- **Observability**: Add distributed tracing

## 🔄 Related Projects

This builds upon container orchestration patterns. Check out complementary stacks:
- **Kubernetes**: For production orchestration
- **Let's Encrypt**: For public SSL certificates
- **Monitoring Stack**: Prometheus + Grafana + AlertManager

## 🤝 Contributing

Contributions welcome! This project aims to fill the Podman and Traefik documentation gap for a up-to-date stack setup, especially on a debian distro.

### Ways to Help
- 🐛 **Bug Reports**: Found an issue? Open a GitHub issue
- 💡 **Feature Ideas**: Suggest improvements via discussions
- 📚 **Documentation**: Help improve setup instructions
- 🧪 **Testing**: Test on different Linux distributions
- 🔧 **Code**: Submit pull requests for enhancements

### Development Workflow
```bash
# Fork and clone
git clone https://github.com/HornetGit/podman-v5-traefik-v3-stack
cd podman-v5-traefik-v3-stack
mkdir version11 & cd version11

# Test your changes
./setup_containers.sh
./restart_application.sh
./diagnose_miniapp.sh

# Submit pull request
```

## 📝 License

MIT License - see [LICENSE](LICENSE) file for details.

## 👨‍💻 Author

**XCS HornetGit**
- GitHub: [@HornetGit](https://github.com/HornetGit)
- Email: hornet.foobar@gmail.com

---

## 🎉 Success Stories

> *"Finally, a working Podman example that isn't just 'replace docker with podman'!"*

> *"The template system makes it easy to understand how everything connects."*

> *"Great learning resource for a consistent Podman v5 / Traefik v3 base stack."*

---

**⭐ Star this repo if it helped you learn Podman v5 and Traefik v3!**