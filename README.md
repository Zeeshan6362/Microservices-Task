# Node.js Microservices with Docker Compose

This project containerizes four Node.js services and runs them together with Docker Compose on a shared network.

## Services

| Service | Port | Main endpoints |
|---|---:|---|
| User Service | 3000 | `/health`, `/users` |
| Product Service | 3001 | `/health`, `/products` |
| Order Service | 3002 | `/health`, `/orders` |
| Gateway Service | 3003 | `/health`, `/api/users`, `/api/products`, `/api/orders` |

The gateway forwards requests to the user, product, and order services over the Compose network.

## Requirements

- Docker Desktop with the Docker Engine running
- Git, if you need to clone the repository

## Start the application

From the repository root, run:

```powershell
docker compose up --build -d
```

Check that all four services are running:

```powershell
docker compose ps
```

To follow the service logs:

```powershell
docker compose logs -f
```

Press `Ctrl+C` to stop following the logs. The containers will continue running.

## Test the services

Run these PowerShell commands to check each service's health endpoint:

```powershell
Invoke-RestMethod "http://localhost:3000/health"
Invoke-RestMethod "http://localhost:3001/health"
Invoke-RestMethod "http://localhost:3002/health"
Invoke-RestMethod "http://localhost:3003/health"
```

Check the gateway routes:

```powershell
Invoke-RestMethod "http://localhost:3003/api/users"
Invoke-RestMethod "http://localhost:3003/api/products"
Invoke-RestMethod "http://localhost:3003/api/orders"
```

Create an order through the gateway:

```powershell
Invoke-RestMethod `
    -Method Post `
    -Uri "http://localhost:3003/api/orders" `
    -ContentType "application/json" `
    -Body '{"userId":1,"productId":1}' |
    ConvertTo-Json -Depth 5
```

The order service stores orders in memory. Orders are cleared when its container restarts.

## Stop the application

Stop and remove the containers and Compose network:

```powershell
docker compose down
```

## Troubleshooting

- **Docker commands fail to connect:** Start Docker Desktop and wait until the engine is running.
- **A port is already in use:** Stop the other application using that port, then run `docker compose up --build -d` again.
- **A container is not running:** Check its status with `docker compose ps` and inspect its logs with `docker compose logs <service-name>`. Service names are `user-service`, `product-service`, `order-service`, and `gateway-service`.
- **Gateway routes return errors:** Confirm all four containers are running. The gateway calls the other services by their Compose service names on the shared network.

## Docker evidence

The screenshot below shows the four services running in Docker Desktop with their published ports.

![Docker Desktop showing the microservices running](screenshots/docker-desktop-containers.png)

## Project layout

```text
Microservices-Task/
├── Microservices/
│   ├── gateway-service/
│   │   ├── Dockerfile
│   │   ├── app.js
│   │   └── package.json
│   ├── order-service/
│   │   ├── Dockerfile
│   │   ├── app.js
│   │   └── package.json
│   ├── product-service/
│   │   ├── Dockerfile
│   │   ├── app.js
│   │   └── package.json
│   └── user-service/
│       ├── Dockerfile
│       ├── app.js
│       └── package.json
├── screenshots/
│   └── docker-desktop-containers.png
└── docker-compose.yml
```