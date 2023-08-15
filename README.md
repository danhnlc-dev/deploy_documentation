# Deploy to AWS ECR

- Github cicd build docker image and push to aws repository

## Prerequisites

- Must to add `secret, var` in your remote repository

1. Create directory .github/workflow
2. Create file.yml
3. Add configure below to your file.yml

```bash
name: Deploy to AWS ECR

on:
push:
branches: ['master']
pull_request:
branches: ['master']

env:
IMAGE_NAME: ecocupid-api-prod-repo
TAG: latest

jobs:
deploy:
runs-on: ubuntu-latest
steps: - name: Using github runner repo
uses: actions/checkout@v3

      - name: Set short commit id for tag trace
        id: vars
        run: echo "sha_short=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

      # Connect AWS credentials
      - name: Configure/Connect AWS credentials
        uses: aws-actions/configure-aws-credentials@0e613a0980cbf65ed5b322eb7a1e075d28913a83
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ vars.AWS_REGION }}

      # Login to Amazon ECR
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@62f4f872db3836360b72999f4b87f1ff13310f3a

      # Build and push image to ECR
      - name: Deploy Amazon ECR | Build, tag commit id + tag latest, and push image to Amazon ECR private
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        run: |
          # Build a docker container and
          # push it to ECR so that it can
          #be deployed to ECS.
          docker build -t $ECR_REGISTRY/${{ env.IMAGE_NAME }}:${{ env.TAG }} .
          docker tag $ECR_REGISTRY/${{ env.IMAGE_NAME }}:${{ env.TAG }} $ECR_REGISTRY/${{ env.IMAGE_NAME }}:${{ steps.vars.outputs.sha_short }}
          docker push $ECR_REGISTRY/${{ env.IMAGE_NAME }}:${{ env.TAG }}
          docker push $ECR_REGISTRY/${{ env.IMAGE_NAME }}:${{ steps.vars.outputs.sha_short }}
          echo "image=$ECR_REGISTRY/${{ env.IMAGE_NAME }}:${{ env.TAG }}" >> $GITHUB_OUTPUT
```
