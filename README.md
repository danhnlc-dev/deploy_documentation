# Deploy to DOCKER HUB

- Github cicd build docker image and push to docker hub repository

## Prerequisites

- Must to add `secret, var` in your remote repository

## Configuration

1. Create directory .github/workflows
2. Create file.yml
3. Add configure below to your file.yml

```bash
name: Deploy with Docker Hub

on:
  push:
    branches: ['master']
  pull_request:
    branches: ['master']


env:
  IMAGE_NAME: image-name
  TAG: latest

jobs:
  push_docker_hub:
    runs-on: ubuntu-latest
    steps:
      - name: Using github runner repo
        uses: actions/checkout@v3

      - name: Use packege extenal Docker - Build and push image
        uses: mr-smithers-excellent/docker-build-push@v6
        with:
          image: ${{ env.DOCKER_HUB_NAMESPACE }}/${{ env.IMAGE_NAME }}
          tags: ${{ env.TAG }}
          registry: ${{ env.REGISTRY }}
          username: ${{ vars.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}


      # Credentials
      - name: Login Docker Hub
        uses: docker/login-action@f4ef78c080cd8ba55a85445d5b36e214a81df20a
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ vars.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Set outputs
        id: vars
        run: echo "sha_short=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

      - name: Build and push Docker images
        uses: docker/build-push-action@3b5e8027fcad23fda98b2e3ac259d8d67585f671
        with:
          context: .
          push: true
          tags: |
            ${{ env.DOCKER_HUB_NAMESPACE }}/${{ env.IMAGE_NAME }}:${{ env.TAG }}
            ${{ env.DOCKER_HUB_NAMESPACE }}/${{ env.IMAGE_NAME }}:${{ steps.vars.outputs.sha_short }}
          tags: |
            ${{ env.DOCKER_HUB_NAMESPACE }}/${{ env.IMAGE_NAME }}:${{ env.TAG }}
            ${{ env.DOCKER_HUB_NAMESPACE }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
```
