# Coupon Book Service 

## Overview

This is a design document describing the high-level architecture and API for a service that allows businesses to create, distribute, and manage coupons.

This service supports operations such as creating coupon books, assigning coupons to users, locking coupons temporarily during redemption attempts, and redeeming coupons

The original assumptions included:
- Coupon book codes can optionally be redeemed more than once per user (coupon book parameter level)
- Max number of codes per Coupon Book assigned per member can also be optionally specified (coupon book parameter level)

To those, the folowing assumptions were added:

- Coupon book names aren't unique.
- For simplicity purposes, a coupon can only be assigned to one user.
- Coupon codes are unique even across coupon books.
- Locks need to expire after a certain time to prevent coupons to be perpetually locked but never redeemed.

### Possible improvements
In the future, the following features could be added for a more advanced and feature rich system.

- Allow multiple users to redeem the same code.
    - Useful for global campaings where a code can be used up to a certain amount of times, or until a specific date.
    - Would require not locking the code for a user and adding a mechanism to disable the code after amount of usages/expirtation time is reached.

## High Level System Architecture
Azure is used as cloud platform, but most services have an equivalent one in AWS/GCP - See [Azure for AWS professionals](https://learn.microsoft.com/en-us/azure/architecture/aws-professional/) or [Azure for GCP professionals](https://learn.microsoft.com/en-us/azure/architecture/gcp-professional/).

![Architecture diagram](assets/images/architecture.png)

1. Front Door receives public requests and redirects it to the closest available region. Provides load balancing, health checking, DDOS protection, application firewall, among other things.
2. Requests are then forwarded to API Management. Provides endpoint consolidation on a single URL in case of multiple services, user level caching and rate limiting, cache invalidation capabilities, authentication, versioning, among other things.
3. Requests are then forwarded the API, which privately comunicates with the SQL database.

### Notes
- No budget contraints were considered for the architecture.
- APIs are containerized and each revision is stored in a container registry. The app service uses these containers to run the API.
- The cronjob will periodically update the SQL database to remove locks on coupons that weren't redeem after a certain amount of times.
    - To avoid heavy load on the database, a messaging queue could be implemented in the middle to update a one or a set of coupons atr a time.
- For bigger projects App Service and Azure Functions could be replaced by AKS using deployments and cronjobs. This will be more flexible and will remove the dependency on cloud specific services, but at the cost of more management overhead.
- Caching can be done at the API gateway level. Since we only have one read related endpoint, coupons for a user, the data returned can be cached per user for that endpoint and the cache invalidated on API calls that perform write operations related to that user.
    - If more fine-grained cache control is needed an external cache like Redis can be added.
- An external service (i.e. AAD, Amazon Cognito, Firebase, etc) will manage the user registration and access control. Authentication would be OAuth2 based using Bearer tokens. The token validation can be performed at the API gateway level through policies, or at the API itself though a middleware service.
- A SQL database was chosen due to it's ACID properties. We'll make use of transactions and optimictic locking to make sure the application is highly available and strongly consistent.
    - An active-passive configuration for SQL database failover groups will be used - writes will be done to one region only, and reads can be performed on the closest replica. If the primary becomes unavailable a replica will take its place. This will ensure consistency both consistency and high availability at the cost of slower writes.
- Private networking should be implemented - Only Front Door should be accesible to the end client. Front Door should comunicate thrught private link to API Management.
- The codebase and each managed service will log information, warnings and errors to Application Insights for monitoring and debugging purposes. Alerts can be configured on specific situations, like an API endpoint exceeding a certain threshold of errors.
    - If we want to avoid cloud specific services, a solution based on Open Telemetry and tools like Prometheus and Grafana can be used.

## High Level Database Design

![Database ER diagram](assets/images/database.png)

### Relationships
- CouponBooks (One) --> (Many) Coupons

### Contraints

NOTE: Constraints are written in pseudocode. they aren't meant to be valid SQL.

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
- `AssignedTo`: `DEFAULT NULL`.
- `LockedBy`: `DEFAULT NULL`.
- `LockExpiresAt`: `DEFAULT NULL`.
- `CreatedAt`: `NOT NULL DEFAULT GETDATE()`.
- `UpdatedAt`: `NOT NULL DEFAULT GETDATE()`.

### Indexes
- `Coupons.CouponBookId` to allow for faster lookup of cupons by coupon book.
- `Coupons.Code` to allow for faster single coupon lookup.
- `Coupons.AssignedTo` and `Coupons.LockedBy` to allow for faster lookup of cupons by user.

### Notes
- Users won't be stored in the database, but managed by an external service (i.e. AAD, Amazon Cognito, Firebase, etc) that would allow login with e-mail or social media accounts. The service will provide an unique Id per user, that would be used in the database in the `AssignedTo`, `LockedTo` fields to identify the user.
- Two Id related fields are used in tables
    - `InternalId` will be for internal use only and won't be exposed to the end clients.
    - `Id/Code` will be exposed to the end clients.

## API endpoints
The API endpoints are defined following the OpenAPI specification. The definition is located `assets/api-definition.yml` and can be seen in the online Swagger UI ([See API definition online](https://petstore.swagger.io/?url=https://raw.githubusercontent.com/Pierca7/coupon-book-service-design/refs/heads/main/assets/api-definition.yml))


### Highlights

#### /users/{userId}/coupons
This endpoint performs a read-only operation over the coupons of a user.

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


