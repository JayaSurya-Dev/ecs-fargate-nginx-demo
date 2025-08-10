# ECS Fargate Nginx Reverse Proxy with CI/CD & Custom Domain

## Overview
This project demonstrates a **production-style AWS ECS Fargate deployment** using:
- **Two-container task** (Static HTML app + Nginx reverse proxy)
- **AWS CodePipeline + CodeBuild** for full CI/CD
- **Amazon ECR** for container image storage
- **Application Load Balancer** for routing & health checks
- **Custom subdomain** (`app.suryasblog.online`) via Hostinger DNS → ALB
- **Zero-manual deployment** — triggered automatically on push to `main` branch

**Goal:** Push code → pipeline builds & pushes Docker images → ECS service updates → changes live at your subdomain.

---

##  Architecture Diagram
  <img width="1536" height="1024" alt="aws ci cd pipeline architecute diagram" src="https://github.com/user-attachments/assets/993af5b5-6f11-4d28-9ce6-414a0ab8ae52" />


---

## 🛠 Setup Steps

### 1) Prerequisites
- AWS Account (with admin privileges)
- Domain: `suryasblog.online` (Hostinger) — subdomain `app.suryasblog.online`
- AWS CLI + Docker installed locally
- GitHub repository connected to AWS CodeStar Connections

### 2) AWS Resources
- **ECR**: `ecs-demo-app`, `ecs-demo-nginx`
- **ECS**: Cluster `ecs-demo-cluster`, Service `ecs-demo-service` (Fargate)
- **ALB**: Public, target group health check path `/health`
- **Route53 or Hostinger DNS**: CNAME `app → <ALB DNS name>`

### 3) Local Code Structure
```plaintext
/
  app/
    Dockerfile
    index.html
  nginx/
    Dockerfile
    nginx.conf
  buildspec.yml
```

---

##  CI/CD Flow

1. **Developer pushes to `main`**
   - CodeStar Connection webhook triggers **CodePipeline**.

2. **Source Stage**
   - Fetches the latest commit from GitHub.

3. **Build Stage** (CodeBuild)
   - Logs in to ECR
   - Builds app & nginx images
   - Tags with `latest` and commit short SHA
   - Pushes to ECR
   - Generates `imagedefinitions.json`

4. **Deploy Stage** (ECS)
   - Uses `imagedefinitions.json` to update ECS service
   - New tasks start → ALB health checks pass → old tasks drain

5. **User Access**
   - Traffic to `https://app.suryasblog.online` is routed via ALB → Nginx → App container

---

##  IAM Roles & Policies

### CodePipeline Service Role
- `codebuild:StartBuild`
- `ecs:UpdateService`, `ecs:DescribeServices`
- `iam:PassRole`
- `codestar-connections:UseConnection`

### CodeBuild Service Role
- ECR push/pull permissions
- CloudWatch Logs
- S3 (if used for artifacts)

### ECS Task Execution Role
- `AmazonECSTaskExecutionRolePolicy`

---

##  Challenges Faced & Solutions

| Challenge | Cause | Solution |
|-----------|-------|----------|
| `host not found in upstream` in Nginx | Containers in same Fargate task don’t use Docker bridge | Use `127.0.0.1:8080` in Nginx upstream |
| Deploy error: “container does not exist” | Wrong `imagedefinitions.json` keys | Match ECS container names exactly |
| `imagedefinitions.json` missing | Wrong artifact output in CodeBuild | Generate in `post_build` and match pipeline artifact name |
| ECR login denied | Missing IAM permissions | Add `ecr:GetAuthorizationToken` to CodeBuild role |
| ALB targets unhealthy | Wrong health check path or SG rules | Serve `/health` with HTTP 200 from Nginx |
| DNS not resolving | Missing/incorrect CNAME record | Create CNAME in Hostinger → ALB DNS name |

---

##  Verification Checklist
- [x] CodeStar Connection = Connected
- [x] Source branch = `main`
- [x] Build output artifact name matches Deploy input
- [x] Pipeline runs automatically on push
- [x] ECS service updates without manual steps
- [x] ALB target group healthy
- [x] `curl http://app.suryasblog.online` returns latest HTML

---
