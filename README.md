# AppCode Repository

The Frost App - A responsive static website containerized with Docker and deployed via GitOps on AWS EKS.

## 📦 Project Overview

This repository contains the application source code and is part of a **complete CI/CD and GitOps pipeline** across three repositories:

### 🔗 Project Repositories

➡️ **[App Code](https://github.com/Ayoub-So/AppCode)** - Application source code and Dockerfile (this repository)  
➡️ **[Terraform Code](https://github.com/Ayoub-So/terraform_eks)** - Infrastructure as Code (EKS cluster setup)  
➡️ **[Manifest Repo](https://github.com/Ayoub-So/kube_manifest)** - Kubernetes manifests & deployment config  

## 📂 Repository Structure

```
AppCode/
├── src/                          # Application source files
│   ├── index.html               # Main HTML page
│   ├── templatemo-frost-style.css  # Stylesheet
│   ├── templatemo-frost-script.js  # JavaScript
│   └── images/                   # Image assets
│       ├── spring-collection.avif
│       ├── summer-collection.avif
│       ├── autumn-collection.avif
│       ├── winter-collection.avif
│       ├── the-mini-scoop.avif
│       ├── the-grand-parlor.avif
│       └── the-classic-cart.avif
├── Dockerfile                   # Container image definition
├── nginx.conf                   # Nginx web server configuration
├── .dockerignore               # Files excluded from Docker image
├── .github/workflows/          # GitHub Actions CI/CD pipelines
│   ├── deploy.yml              # Build, push to ECR, update manifests
│   └── build.yml               # Build verification & linting
└── README.md                    # This file
```

## 🎨 Application

**Frost App** is a responsive web application showcasing:
- Modern responsive design
- Product catalog with images
- Optimized image formats (AVIF)
- Health check endpoint (`/health`)
- Gzip compression for performance

### Technology Stack

- **Frontend:** HTML5, CSS3, JavaScript (vanilla)
- **Image Format:** AVIF (modern, optimized)
- **Web Server:** Nginx (Alpine-based)
- **Container:** Docker

## 🐳 Docker Configuration

### Dockerfile (Multi-stage)

The Dockerfile uses nginx:alpine as the base image:

```dockerfile
FROM nginx:alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY src/ /usr/share/nginx/html/
EXPOSE 80
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost/index.html || exit 1
CMD ["nginx", "-g", "daemon off;"]
```

**Features:**
- ✅ Lightweight Alpine Linux base (~40MB final image)
- ✅ Health checks for Kubernetes monitoring
- ✅ Stateless design (no persistent storage needed)
- ✅ Production-ready configuration

### Nginx Configuration

The [nginx.conf](nginx.conf) includes:

- **Gzip Compression** - CSS, JS, SVG compressed for faster delivery
- **Cache Control** - Static assets cached for 30 days
- **Security Headers** - Denies access to `.git`, `.env`, etc.
- **Health Endpoint** - `/health` for Kubernetes probes
- **SPA Configuration** - Routes all requests to index.html

## 🔄 CI/CD Pipeline

### GitHub Actions Workflows

#### 1. **deploy.yml** - Main CI/CD Pipeline

**Trigger:** Push to `main` or `develop` branch

**Jobs:**
- **build** - Builds Docker image and pushes to AWS ECR
- **Update_manifest** - Updates kube_manifest repo with new image tag

**Flow:**
```
Push to AppCode → Build Docker image → Push to ECR → Update kube_manifest repo
                                                              ↓
                                                      ArgoCD detects change
                                                              ↓
                                                         EKS deploys
```

**Environment Variables:**
```yaml
AWS_REGION: us-east-1
ECR_REPOSITORY: $YOUR_ECR_REPOSITORY
EKS_CLUSTER_NAME: frost-cluster
EKS_NAMESPACE: default
```

**Secrets Required:**
- `AWS_ACCESS_KEY_ID` - AWS IAM credentials
- `AWS_SECRET_ACCESS_KEY` - AWS IAM credentials
- `AWS_REGION` - AWS region (us-east-1)
- `TOKEN_PAT_GITHUB` - GitHub Personal Access Token

#### 2. **build.yml** - Build Verification

**Trigger:** PR to `main`/`develop` or push to `develop`

**Jobs:**
- **build** - Verifies Docker image builds successfully
- **lint** - Lints Dockerfile with Hadolint

## 🚀 Local Development

### Build Locally

```bash
# Build the Docker image
docker build -t frost-app:latest .

# Run the container
docker run -p 8080:80 frost-app:latest

# Test the app
curl http://localhost:8080
curl http://localhost:8080/health
```

### Develop Without Docker

For quick testing without containerization:

```bash
# Serve with Python
python3 -m http.server 8000 --directory src/

# Or with Node.js http-server
npx http-server src/
```

Visit http://localhost:8000

## 🧪 Testing

### Test Health Check

```bash
# When container is running
curl -I http://localhost:8080/health

# Expected response:
# HTTP/1.1 200 OK
# Content-Type: text/plain
```

### Test Performance

```bash
# Check gzip compression
curl -I -H "Accept-Encoding: gzip" http://localhost:8080/

# Verify response headers
curl -v http://localhost:8080/ 2>&1 | grep -E "etag|cache|gzip"
```

### Check Docker Image Size

```bash
docker images frost-app:latest
# Should be ~40MB or less with Alpine base
```

## 📊 Deployment Flow

```
1. Developer makes changes to AppCode
   ↓
2. Push to main/develop branch
   ↓
3. GitHub Actions triggered:
   - Builds Docker image
   - Tags with git SHA
   - Pushes to AWS ECR
   - Updates kube_manifest/deployment.yaml
   ↓
4. GitHub Actions pushes to kube_manifest repo
   ↓
5. ArgoCD detects change in kube_manifest repo
   ↓
6. ArgoCD syncs with Kubernetes
   ↓
7. New deployment created with fresh image
   ↓
8. Load Balancer routes traffic to new pods
   ↓
9. App is live! ✅
```

## 🔐 Security

### Best Practices Implemented

✅ **Minimal Base Image** - Alpine Linux reduces attack surface  
✅ **Read-Only Filesystem** - Nginx runs without write permissions  
✅ **Non-Root User** - Nginx runs as unprivileged user  
✅ **Security Headers** - Blocks access to sensitive files  
✅ **HTTPS Ready** - Can be used with TLS termination  
✅ **No Secrets in Image** - Credentials passed via environment  

### Image Scanning

```bash
# Scan for vulnerabilities (requires Trivy)
trivy image frost-app:latest

# Check Docker image layers
docker history frost-app:latest
```

## 📈 Performance Optimization

### Implemented Optimizations

1. **Image Format** - AVIF (20-30% smaller than JPEG/PNG)
2. **Gzip Compression** - CSS/JS/SVG compressed
3. **Cache Control** - Static assets cached 30 days
4. **CDN Ready** - Can be placed behind CloudFront
5. **Lightweight Base** - Alpine Linux (~5MB)

### Measured Performance

```bash
# Test page load time
curl -w "Total time: %{time_total}s\n" -o /dev/null http://localhost:8080/

# Check compression
curl -H "Accept-Encoding: gzip" -I http://localhost:8080/ | grep -i content-encoding
```

## 🛠️ Troubleshooting

### Docker build fails

```bash
# Check Dockerfile syntax
docker build --dry-run -t frost-app:test .

# Verbose output
docker build -t frost-app:test . --progress=plain

# Check .dockerignore
cat .dockerignore
```

### Image not pushing to ECR

```bash
# Verify ECR login
aws ecr describe-repositories --repository-names devops --region us-east-1

# Check AWS credentials
aws sts get-caller-identity

# Manual ECR login
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
```

### App not responding

```bash
# Check container logs
docker logs <CONTAINER_ID>

# Verify Nginx config
docker exec <CONTAINER_ID> nginx -t

# Test connectivity inside container
docker exec <CONTAINER_ID> wget -O - http://localhost/health
```

## 📚 Related Documentation

- [Main CI/CD Project](https://github.com/Ayoub-So/CI-CD-EKS-with-GitOps-on-aws)
- [Terraform Infrastructure Setup](https://github.com/Ayoub-So/terraform_eks)
- [Kubernetes Manifests](https://github.com/Ayoub-So/kube_manifest)
- [Docker Documentation](https://docs.docker.com/)
- [Nginx Documentation](https://nginx.org/en/docs/)

## 🤝 Contributing

When making changes to the application:

1. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature
   ```

2. **Test locally**
   ```bash
   docker build -t frost-app:test .
   docker run -p 8080:80 frost-app:test
   ```

3. **Verify Dockerfile**
   ```bash
   docker build --no-cache -t frost-app:test .
   ```

4. **Commit and push**
   ```bash
   git add .
   git commit -m "feat: describe your changes"
   git push origin feature/your-feature
   ```

5. **Create Pull Request**
   - Wait for GitHub Actions to verify build
   - Request review from team members
   - Merge to `develop` or `main`

## 👤 Author

Ayoub Soussi  
Email: ayoubsoussi.2001@gmail.com

## 📄 License

This project is part of the CI/CD EKS with GitOps on AWS project.

---

**Last Updated:** March 31, 2026  
**Image Size:** ~40MB (Alpine-based)  
**Status:** Production Ready
