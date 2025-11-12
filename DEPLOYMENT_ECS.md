# ECS Fargate Deployment Guide (WebSocket Support)

App Runner has limitations with WebSocket connections that Streamlit requires. ECS Fargate with an Application Load Balancer provides full WebSocket support.

## Quick Deploy to ECS Fargate

### Prerequisites
- AWS CLI configured
- Docker image already in ECR (from App Runner setup)

### Steps

1. **Create ECS Cluster:**
```bash
aws ecs create-cluster --cluster-name ukp-cluster --region us-east-1
```

2. **Create Task Definition:**
```bash
aws ecs register-task-definition \
  --family ukp-app \
  --network-mode awsvpc \
  --requires-compatibilities FARGATE \
  --cpu 256 \
  --memory 512 \
  --execution-role-arn arn:aws:iam::381492054783:role/ecsTaskExecutionRole \
  --task-role-arn arn:aws:iam::381492054783:role/ecsTaskRole \
  --container-definitions '[
    {
      "name": "ukp-app",
      "image": "381492054783.dkr.ecr.us-east-1.amazonaws.com/ukp-app:latest",
      "portMappings": [{"containerPort": 8501, "protocol": "tcp"}],
      "environment": [
        {"name": "STREAMLIT_SERVER_PORT", "value": "8501"},
        {"name": "STREAMLIT_SERVER_ADDRESS", "value": "0.0.0.0"},
        {"name": "STREAMLIT_SERVER_HEADLESS", "value": "true"}
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/ukp-app",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]' \
  --region us-east-1
```

3. **Create Application Load Balancer** (supports WebSockets):
```bash
# Create ALB with WebSocket support
# This requires VPC, subnets, and security groups
# See AWS Console for easier setup
```

4. **Create ECS Service:**
```bash
aws ecs create-service \
  --cluster ukp-cluster \
  --service-name ukp-service \
  --task-definition ukp-app \
  --desired-count 1 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxx],securityGroups=[sg-xxx],assignPublicIp=ENABLED}" \
  --load-balancers "targetGroupArn=arn:aws:elasticloadbalancing:us-east-1:381492054783:targetgroup/ukp-tg/xxx,containerName=ukp-app,containerPort=8501" \
  --region us-east-1
```

## Alternative: Use EC2 (Simpler, WebSocket Support)

EC2 deployment is simpler and has full WebSocket support. See `DEPLOYMENT.md` for EC2 setup instructions.

