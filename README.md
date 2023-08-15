# Deploy with AWS ECR and AWS Fargate

- `AWS ECR`: Using for storing docker images
- `AWS Fargate`: Using for running docker container

## Configuration

- Must to add `secret, var` in your remote repository

1. Create directory .github/workflows
2. Create file.yml
3. Add configure below to your file.yml

```bash
name: Deploy CICD with AWS ECR and AWS Fargate

on:
push:
branches: ['master']
pull_request:
branches: ['master']

env:
ECS_CLUSTER: cluster-name
ECS_SERVICE: service-name
ECS_TASK_DEFINITION: task-definition-name
IMAGE_NAME: image-name
CONTAINER_NAME: container-name
TAG: latest

jobs:
deploy:
runs-on: ubuntu-latest
steps: - name: Using github runner repo
uses: actions/checkout@v3

      - name: Set short commit id for tag trace
        id: vars
        run: echo "sha_short=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

      # Using tutorial
      - name: Configure/Connect AWS credentials
        uses: aws-actions/configure-aws-credentials@0e613a0980cbf65ed5b322eb7a1e075d28913a83
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ vars.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@62f4f872db3836360b72999f4b87f1ff13310f3a

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

      # Dowload file for internal repo
      - name: Download file task definition from task created in aws
        run: |
          aws ecs describe-task-definition --task-definition ${{ env.ECS_TASK_DEFINITION }} \
          --query taskDefinition > task-definition.json

      - name: Fill in the new image ID in the Amazon ECS task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@c804dfbdd57f713b6c079302a4c01db7017a36fc
        with:
          task-definition: task-definition.json
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ steps.build-image.outputs.image }}

      # Add environment variables to ECS task definition
      - name: Add environment variables to ECS task definition
        id: add-env-var
        uses: cvmaker-bv/amazon-ecs-task-environment@v1
        env:
          APP_ENVIRONMENT: ${{ vars.BE_APP_ENVIRONMENT }}
          TIMEZONE: ${{ vars.BE_TIMEZONE }}
          #.... add more envs
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          container-name: ${{ env.CONTAINER_NAME }}
          env-variables: '${{ toJson(env) }}'

      # Deploy task definition to Amazon ECS
      - name: Deploy Amazon ECS | Task definition
        uses: aws-actions/amazon-ecs-deploy-task-definition@df9643053eda01f169e64a0e60233aacca83799a
        with:
          task-definition: ${{ steps.add-env-var.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true

```
