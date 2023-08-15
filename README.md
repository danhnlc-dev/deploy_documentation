# Deploy with AWS ECR and AWS Codebuild

- Deploy cicd with aws codebuild

## Prerequisites

- Must to add secret, var in your remote repository

### Configuration

- Create file buildspec.yml in root project

```bash
version: 0.2

env:
  variables:
    AWS_REGION: ""
    ACCOUNT_ID: ""
    REPOSITORY_URI: ""

phases:
  pre_build:
    commands:
      - echo Log in to Amazon ECR...
      - aws --version
      - aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=${COMMIT_HASH:=latest}
  build:
    commands:
      - echo Building the Docker images...
      - docker build -t $REPOSITORY_URI:latest .
      - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker images...
      - docker push $REPOSITORY_URI:latest
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - echo Writing image definitions file...
      - printf '[{"name":"container-name","imageUri":"%s"}]' $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json
artifacts:
  files:
    - '**/*'
```
