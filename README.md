# Coupon Book Service 

## Overview

This is a design document describing the high-level architecture and API for a service that allows businesses to create, distribute, and manage coupons.

This service supports operations such as creating coupon books, assigning coupons to users, locking coupons temporarily during redemption attempts, and redeeming coupons

The original assumptions included:
- Coupon book codes can optionally be redeemed more than once per user (coupon book parameter level)
- Max number of codes per Coupon Book assigned per member can also be optionally specified (coupon book parameter level)

To those, the folowing assumptions were added:

- For simplicity purposes, a coupon can only be assigned to one user.
- Locks need to expire after a certain time to prevent coupons to be perpetually locked but never redeemed.

## High Level System Architecture

## High Level Database Design

![Database ER diagram](assets/images/database.png)

### Relationships
- CouponBooks (One) --> (Many) Coupons

### Contraint


#### CouponBooks
- `InternalId`: `NOT NULL AUTO_INCREMENT`.
- `Id`: `NOT NULL UNIQUE`.
- `Name`: `NOT NULL`.
- `AllowMultipleRedemptions`: `NOT NULL DEFAULT false`.
- `MaxPerUser`: `NOT NULL DEFAULT 1`.
- `CreatedAt`: `NOT NULL DEFAULT GETDATE()`.
- `UpdatedAt`: `NOT NULL DEFAULT GETDATE()`.

#### Coupons
- `InternalId`: `NOT NULL AUTO_INCREMENT`.
- `CouponBookId`: `NOT NULL`.
- `Code`: `NOT NULL UNIQUE`.
- `TimesRedeemed`: `NOT NULL DEFAULT 0 CHECK >= 0`.
- `CreatedAt`: `NOT NULL DEFAULT GETDATE()`.
- `UpdatedAt`: `NOT NULL DEFAULT GETDATE()`.

### Notes
- Users won't be stored in the database, but managed by an external service (i.e. AAD, Amazon Cognito, Firebase, etc) that would allow login with e-mail or social media accounts. The service will provide an unique Id per user, that would be used in the database in the `AssignedTo`, `LockedTo` fields to identify the user.
- Two Id related fields are used in tables
    - `InternalId` will be for internal use only and won't be exposed to the end clients.
    - `Id/Code` will be exposed to the end clients.

## API endpoints
The API endpoints are defined following the OpenAPI specification. The definition is located `assets/api-definition.yml` and can be seen in the online Swagger UI ([See API definition online](https://petstore.swagger.io/?url=https://raw.githubusercontent.com/Pierca7/coupon-book-service-design/refs/heads/main/assets/api-definition.yml))


### Highlights

#### /users/{userId}/coupons
This endpoint performs a read-only operation over the 

## Pseudocode for Key Operations

## High-Level Deployment Strategy

- GitOps
- CI/CD pipelines (Azure Pipelines, GitHub Actions)
- Infrastructure as code (Terraform, Pulumi) parametized to allow multiple environments and regions.
- Multiple environments
    - Development
    - QA
    - Staging
    - Production
- API is containerized and uploaded to container registry


