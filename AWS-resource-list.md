#!/bin/bash

###############################################################
# Script Name  : aws_resource_list.sh
# Description  : List all active AWS resources across services
# Author        : Siddhgopal Soni
# GitHub        : https://github.com/siddhgopal
# LinkedIn      : https://linkedin.com/in/siddhgopal-soni-010846b8
# Version       : 1.0
# Usage         : ./aws_resource_list.sh <aws-region> <aws-profile>
# Example       : ./aws_resource_list.sh us-east-1 default
###############################################################

set -e

# ─── Colors ───────────────────────────────────────────────
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
NC='\033[0m' # No Color

# ─── Input Validation ─────────────────────────────────────
if [ $# -ne 2 ]; then
    echo -e "${RED}ERROR: Invalid number of arguments${NC}"
    echo -e "${YELLOW}Usage   : $0 <aws-region> <aws-profile>${NC}"
    echo -e "${YELLOW}Example : $0 us-east-1 default${NC}"
    exit 1
fi

AWS_REGION=$1
AWS_PROFILE=$2

# ─── Check AWS CLI installed ──────────────────────────────
if ! command -v aws &> /dev/null; then
    echo -e "${RED}ERROR: AWS CLI is not installed. Please install it first.${NC}"
    echo "Install: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html"
    exit 1
fi

# ─── Check AWS CLI configured ─────────────────────────────
if ! aws sts get-caller-identity --profile "$AWS_PROFILE" &> /dev/null; then
    echo -e "${RED}ERROR: AWS CLI not configured for profile '$AWS_PROFILE'${NC}"
    echo "Run: aws configure --profile $AWS_PROFILE"
    exit 1
fi

# ─── Header ───────────────────────────────────────────────
echo -e "${CYAN}"
echo "============================================================"
echo "         AWS RESOURCE LIST CHECKER"
echo "         Region  : $AWS_REGION"
echo "         Profile : $AWS_PROFILE"
echo "         Date    : $(date '+%Y-%m-%d %H:%M:%S')"
echo "============================================================"
echo -e "${NC}"

# ─── 1. EC2 Instances ─────────────────────────────────────
echo -e "${BLUE}[1] EC2 Instances${NC}"
aws ec2 describe-instances \
    --region "$AWS_REGION" \
    --profile "$AWS_PROFILE" \
    --query 'Reservations[*].Instances[*].[InstanceId,InstanceType,State.Name,PublicIpAddress,Tags[?Key==`Name`].Value|[0]]' \
    --output table
echo ""

# ─── 2. S3 Buckets ────────────────────────────────────────
echo -e "${BLUE}[2] S3 Buckets${NC}"
aws s3api list-buckets \
    --profile "$AWS_PROFILE" \
    --query 'Buckets[*].[Name,CreationDate]' \
    --output table
echo ""

# ─── 3. VPCs ──────────────────────────────────────────────
echo -e "${BLUE}[3] VPCs${NC}"
aws ec2 describe-vpcs \
    --region "$AWS_REGION" \
    --profile "$AWS_PROFILE" \
    --query 'Vpcs[*].[VpcId,CidrBlock,State,Tags[?Key==`Name`].Value|[0]]' \
    --output table
echo ""

# ─── 4. IAM Users ─────────────────────────────────────────
echo -e "${BLUE}[4] IAM Users${NC}"
aws iam list-users \
    --profile "$AWS_PROFILE" \
    --query 'Users[*].[UserName,UserId,CreateDate]' \
    --output table
echo ""

# ─── 5. RDS Instances ─────────────────────────────────────
echo -e "${BLUE}[5] RDS Instances${NC}"
aws rds describe-db-instances \
    --region "$AWS_REGION" \
    --profile "$AWS_PROFILE" \
    --query 'DBInstances[*].[DBInstanceIdentifier,DBInstanceClass,Engine,DBInstanceStatus,Endpoint.Address]' \
    --output table
echo ""

# ─── 6. Lambda Functions ──────────────────────────────────
echo -e "${BLUE}[6] Lambda Functions${NC}"
aws lambda list-functions \
    --region "$AWS_REGION" \
    --profile "$AWS_PROFILE" \
    --query 'Functions[*].[FunctionName,Runtime,LastModified,MemorySize]' \
    --output table
echo ""

# ─── 7. EKS Clusters ──────────────────────────────────────
echo -e "${BLUE}[7] EKS Clusters${NC}"
aws eks list-clusters \
    --region "$AWS_REGION" \
    --profile "$AWS_PROFILE" \
    --query 'clusters' \
    --output table
echo ""

# ─── 8. ECS Clusters ──────────────────────────────────────
echo -e "${BLUE}[8] ECS Clusters${NC}"
aws ecs list-clusters \
    --region "$AWS_REGION" \
    --profile "$AWS_PROFILE" \
    --query 'clusterArns' \
    --output table
echo ""

# ─── 9. ECR Repositories ──────────────────────────────────
echo -e "${BLUE}[9] ECR Repositories${NC}"
aws ecr describe-repositories \
    --region "$AWS_REGION" \
    --profile "$AWS_PROFILE" \
    --query 'repositories[*].[repositoryName,repositoryUri,createdAt]' \
    --output table
echo ""

# ─── 10. Load Balancers ───────────────────────────────────
echo -e "${BLUE}[10] Load Balancers (ELB/ALB/NLB)${NC}"
aws elbv2 describe-load-balancers \
    --region "$AWS_REGION" \
    --profile "$AWS_PROFILE" \
    --query 'LoadBalancers[*].[LoadBalancerName,Type,State.Code,DNSName]' \
    --output table
echo ""

# ─── 11. Auto Scaling Groups ──────────────────────────────
echo -e "${BLUE}[11] Auto Scaling Groups${NC}"
aws autoscaling describe-auto-scaling-groups \
    --region "$AWS_REGION" \
    --profile "$AWS_PROFILE" \
    --query 'AutoScalingGroups[*].[AutoScalingGroupName,MinSize,MaxSize,DesiredCapacity]' \
    --output table
echo ""

# ─── 12. CloudWatch Alarms ────────────────────────────────
echo -e "${BLUE}[12] CloudWatch Alarms${NC}"
aws cloudwatch describe-alarms \
    --region "$AWS_REGION" \
    --profile "$AWS_PROFILE" \
    --query 'MetricAlarms[*].[AlarmName,StateValue,MetricName]' \
    --output table
echo ""

# ─── 13. Route53 Hosted Zones ─────────────────────────────
echo -e "${BLUE}[13] Route53 Hosted Zones${NC}"
aws route53 list-hosted-zones \
    --profile "$AWS_PROFILE" \
    --query 'HostedZones[*].[Name,Id,Config.PrivateZone]' \
    --output table
echo ""

# ─── 14. SNS Topics ───────────────────────────────────────
echo -e "${BLUE}[14] SNS Topics${NC}"
aws sns list-topics \
    --region "$AWS_REGION" \
    --profile "$AWS_PROFILE" \
    --query 'Topics[*].TopicArn' \
    --output table
echo ""

# ─── 15. SQS Queues ───────────────────────────────────────
echo -e "${BLUE}[15] SQS Queues${NC}"
aws sqs list-queues \
    --region "$AWS_REGION" \
    --profile "$AWS_PROFILE" \
    --query 'QueueUrls' \
    --output table
echo ""

# ─── Footer ───────────────────────────────────────────────
echo -e "${GREEN}"
echo "============================================================"
echo "   AWS Resource List Check Completed Successfully!"
echo "   Region  : $AWS_REGION"
echo "   Profile : $AWS_PROFILE"
echo "============================================================"
echo -e "${NC}"
