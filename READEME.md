# Scooter Rentals (Microservices/Nest.js/Kubernetes)

## Overview

The Scooter Rentals system is a microservices-based application developed using Node.js and Nest.js, specifically designed for deployment in a Kubernetes environment.

This system consists of two main microservices that facilitate scooter rentals, returns, and payment processing:

1. **ScooterRent Service** - This service manages scooter rental operations, ensuring that users can only rent one scooter at a time. It also rejects rental requests if users have any unpaid orders.

2. **Order Service** - This service handles rental payment transactions and order processing. Please note that this example demonstrates how microservices communicate, and there is no actual payment gateway included in this repository.

## How to Set Up Locally

### Prerequisites

- Node.js (v20.18)
- Docker
- Kubernetes (enable it on Docker desktop)

### Setup Steps

#### 1. Clone the repository
```bash
git clone https://github.com/IgnacioFan/scooter-rentals-kubernetes.git
cd scooter-rentals-kubernetes
```

#### 2. Create a `.env` file for each microservice
Each microservice requires a `.env` file with the following environment variables:

```env
DATABASE_URL=postgresql://wemo:wemo@postgres:5432/wemo-scooter-rent
RABBITMQ_URL=amqp://rabbitmq:5672
```

#### 3. Run with Docker Compose
By running through Docker Compose, each microservice will automatically build migrations, seed initial data, and acquire other available resources, such as Prisma Studio dashboards for each microservice and the RabbitMQ dashboard.

```bash
docker-compose up --build
```

#### 4. Run with Kubernetes
By running through Kubernetes, each microservice will automatically build migrations and seed initial data. You can use kubectl to review the current Kubernetes services, deployments, and pods' info.

```bash
kubectl apply -f kubernetes/
```

### Testing
Run unit tests with:
```bash
npm test
```

## System Architecture

The application consists of four components:

1. **ScooterRent Service** (Nest.js) - Handles external client requests related to scooter rentals.
2. **Order Service** (Nest.js) - Manages rental orders and payments internally.
3. **PostgreSQL** - The database for persisting rental and order data.
4. **RabbitMQ** - Message broker facilitating communication between services.

### Data Models
#### ScooterRent Service
- **User** (id, name, email, createdAt, updatedAt)
- **Scooter** (id, name, serialNumber, licensePlate, status, createdAt, updatedAt)
- **Rent** (id, userId, scooterId, startAt, endAt, status)

#### Order Service
- **Order** (id, rentId, userId, amount, currency, status, createdAt, updatedAt)
- **Payment** (id, orderId, status, createdAt, updatedAt)

## API Documentation

### `POST /v1/rents`
**Description:** Start renting a scooter.

#### Request
```bash
curl -X POST http://localhost:3000/v1/rents \
-H "Content-Type: application/json" \
-d '{"scooterId": 1, "userId": 1}'
```

#### Response
```json
{
  "id": 1,
  "startAt": "2024-02-10T12:00:00Z",
  "endAt": null,
  "status": "pending",
  "userId": 1,
  "scooterId": 1
}
```

### `PATCH /v1/rents/:id/end`
**Description:** Stop the rental.

#### Request
```bash
curl -X PATCH http://localhost:3000/v1/rents/1/end \
-H "Content-Type: application/json" \
-d '{"userId": 1}'
```

#### Response
```json
{
  "id": 1,
  "startAt": "2024-02-10T12:00:00Z",
  "endAt": "2024-02-10T12:30:00Z",
  "status": "pending",
  "userId": 1,
  "scooterId": 1
}
```

### `PATCH /v1/rents/:id/pay`
**Description:** Process rental payment.

#### Request
```bash
curl -X PATCH http://localhost:3000/v1/rents/1/pay \
-H "Content-Type: application/json" \
-d '{"userId": 1}'
```

#### Response
```json
{
  "id": 1,
  "startAt": "2024-02-10T12:00:00Z",
  "endAt": "2024-02-10T12:30:00Z",
  "status": "completed",
  "userId": 1,
  "scooterId": 1
}
```

## Future Enhancements

### Implement API Gateway, Authentication, and Authorization

Introducing an API Gateway to the system can provide several benefits. First, it can enhance security by centralizing authentication, load balancing, and rate limiting, thus protecting critical resources. Additionally, we can implement JWT combined with role-based authorization to ensure secure access control.

### Support Various Scooter Rental Rates

Currently, the system operates on a general pricing model of $2 per minute. To enhance profitability, we can introduce different pricing models based on factors such as scooter type, time of day, and other variables.

### Expand Order States

While the system currently simulates payment processing, the order status is relatively straightforward. In practice, we need to manage order status carefully in relation to payment transaction outcomes. Achieving this may require further discussions across different functional teams.

### API Improvements

- **Optimize Database Performance**: Implement indexing and caching (using Redis) to expedite query responses.
- **Integrate Observability Tools**: Use Sentry for error tracking, and New Relic or Datadog for performance monitoring. Additionally, incorporate Prometheus and Grafana for logging and metrics.
