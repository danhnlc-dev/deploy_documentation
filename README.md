# Deploy Static Website

- Deploy Static Website with AWS S3, CloudFront, SSL certificate

## Prerequisites

- Your domain has already registered.

## Step by Step

1. Build App
2. Create S3 bucket
3. Config permission for S3 bucket
4. Upload source build
5. Update config S3 bucket for enabling static website
6. Create certificate for your domain
7. Create cloudfront
8. Point your domain to cloudfront domain

### Build App

```bash
npm run build
```

#### Build App With environment

- Update script in package.json
- Add environment in .env, .env.staging, .env.production, ...

```bash
"scripts": {
    "start": "react-scripts start",
    "build": "sh -ac '. ./.env.$REACT_APP_ENV; react-scripts build'",
    "build:staging": "REACT_APP_ENV=staging npm run build",
    "build:production": "REACT_APP_ENV=production npm run build",
},
```

### Deploy App

#### Some Configures for S3

#### Implement Static Website with AWS S3, Cloudfront and HTTPS custom domain

- https://youtu.be/o2HTkVxzivA

#### React router issue on AWS Cloudfront and S3

- https://youtu.be/Qo9ORPiRVVs

#### Deploy React App to S3 with Custom Domain

- https://youtu.be/7djMZ5OTG_E
