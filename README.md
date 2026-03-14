
# GitHub Actions OIDC Authentication with AWS

This project demonstrates how to configure **GitHub Actions to securely access AWS using OpenID Connect (OIDC)** without storing long-term AWS credentials.

Using OIDC allows GitHub workflows to authenticate with AWS by assuming an IAM role and receiving **temporary security credentials** from AWS Security Token Service (STS).

---

# Prerequisites

Before starting the setup, make sure you have the following:

- AWS Account
- GitHub Repository
- Basic knowledge of IAM and GitHub Actions
- IAM permissions to create:
  - IAM Users
  - IAM Roles
  - Identity Providers

---

# Architecture Overview

GitHub Workflow
      │
      ▼
OIDC Token Request
      │
      ▼
GitHub OIDC Provider
      │
      ▼
AWS STS
      │
      ▼
IAM Role
      │
      ▼
Temporary AWS Credentials
      │
      ▼
AWS Services (EC2, EKS, etc.)

---

# Step 1 — Create IAM User

Go to AWS Console:

IAM → Users → Create User

Add a user name.

For demo purposes, attach the policy:

AdministratorAccess

⚠️ Note:
In real production environments, always follow the **principle of least privilege** and attach only the required permissions.

---

# Step 2 — Create OIDC Identity Provider

Navigate to:

IAM → Identity Providers → Add Provider

Select:

Provider Type: OpenID Connect

Add the following configuration:

Provider URL:
https://token.actions.githubusercontent.com

Audience:
sts.amazonaws.com

Then click **Add Provider**.

This step allows AWS to **trust GitHub as an authentication provider**.

---

# Step 3 — Create IAM Role for GitHub Actions

Go to:

IAM → Roles → Create Role

Select:

Trusted entity type: Web Identity

Configuration:

Identity Provider:
token.actions.githubusercontent.com

Audience:
sts.amazonaws.com

Repository configuration example:

repo:USERNAME/REPOSITORY:ref:refs/heads/main

Click **Next**.

---

# Step 4 — Attach Permissions to the Role

Attach the required policies.

For demo purposes you can attach:

AmazonEC2FullAccess

⚠️ In real projects, attach only the permissions required for your workflow.

Example policies:
- ECR permissions
- EKS permissions
- S3 permissions

---

# Step 5 — Create Role and Copy ARN

After creating the role, copy the **Role ARN**.

Example:

arn:aws:iam::123456789012:role/github-actions-role

This ARN will be used inside the GitHub workflow.

---

# Step 6 — Add Role ARN in GitHub Secrets

Navigate to your GitHub repository.

Repository → Settings → Secrets and Variables → Actions

Create a new secret.

Secret Name:
AWS_OIDC_ROLE

Value:
arn:aws:iam::123456789012:role/github-actions-role

Save the secret.

---

# Example GitHub Actions Workflow

Create a workflow file:

.github/workflows/aws-oidc.yml

```yaml
name: Test OIDC Authentication

on:
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  list-ec2:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Configure AWS credentials using OIDC
      uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: ${{ secrets.AWS_OIDC_ROLE }}
        aws-region: us-west-2

    - name: Verify AWS identity
      run: aws sts get-caller-identity

    - name: List EC2 Instances
      run: |
        aws ec2 describe-instances         --query "Reservations[*].Instances[*].InstanceId"         --output table
```

---

# Workflow Execution

When the workflow runs, the following process occurs:

GitHub Actions Workflow
        │
        ▼
Request OIDC Token
        │
        ▼
GitHub OIDC Provider
        │
        ▼
AWS STS Validate Token
        │
        ▼
Assume IAM Role
        │
        ▼
Temporary AWS Credentials Issued
        │
        ▼
Workflow Executes AWS Commands

---

# Benefits of Using OIDC

- No need to store AWS access keys in GitHub
- Improved security using temporary credentials
- Automatic credential rotation
- Reduced risk of credential leakage
- Fine-grained access control using IAM roles

---

# Conclusion

Using **OIDC authentication with GitHub Actions and AWS** allows secure and scalable CI/CD pipelines by removing the need for static AWS credentials and replacing them with **temporary tokens issued during workflow execution**.

This approach is considered a **best practice for modern DevOps workflows**.