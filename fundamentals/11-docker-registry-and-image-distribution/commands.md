# D11 — Docker Registry Commands

## Build

```bash
docker build -t d11-demo:1.0.0 .
```

## Login

```bash
docker login
```

## Tag

```bash
docker tag d11-demo:1.0.0 buildwithnarayan/d11-demo:1.0.0
```

## Push

```bash
docker push buildwithnarayan/d11-demo:1.0.0
```

## Pull

```bash
docker pull buildwithnarayan/d11-demo:1.0.0
```

## Run

```bash
docker run -d   --name d11-test   -p 8080:80   buildwithnarayan/d11-demo:1.0.0
```

## Inspect

```bash
docker image inspect buildwithnarayan/d11-demo:1.0.0
```

## List Images

```bash
docker images
```

## Remove

```bash
docker rm -f d11-test
docker rmi d11-demo:1.0.0
```

## Amazon ECR Login

```bash
aws ecr get-login-password --region <region>   | docker login     --username AWS     --password-stdin <account>.dkr.ecr.<region>.amazonaws.com
```

## ECR Push

```bash
docker push <account>.dkr.ecr.<region>.amazonaws.com/backend:1.0.0
```
