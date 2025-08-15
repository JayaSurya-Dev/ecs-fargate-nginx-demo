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

##  CI/CD 
<img width="1508" height="325" alt="{6CB3171C-24F7-4D31-A8A6-EEE6F32D48CD}" src="https://github.com/user-attachments/assets/fa222888-a769-46e0-af35-ce5820f8b234" />


## VPC
<img width="1483" height="643" alt="{5889A3FC-C4A3-4E5E-BC10-58373CCD1A83}" src="https://github.com/user-attachments/assets/9b273d0c-4239-49aa-b673-c35c043d7c63" />
flow
<img width="1790" height="685" alt="{608E0EEF-C486-4C8E-867C-F17B41729F87}" src="https://github.com/user-attachments/assets/b2ad92d7-fe44-40ed-935a-48a836dba107" />

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
- **Hostinger DNS**: CNAME `app → <ALB DNS name>`

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

### ECS Task Execution Role
- `AmazonECSTaskExecutionRolePolicy`

---



---

