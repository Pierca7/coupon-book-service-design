# Coupon Book Service 


## Assumptions

- For simplicity purposes, a coupon can only be assigned to one user.
- Locks need to expire after a certain time to prevent coupons to be perpetually locked but never redeemed.


## High Level System Architecture

## High Level Database Design

## API endpoints

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


# Coupon Book Service
Coupon Book Service API definition

## Version: 1.0.0

### /coupons

#### POST
##### Summary:

Create a new coupon book

##### Description:

Create a new coupon book

##### Responses

| Code | Description |
| ---- | ----------- |
| 201 | Coupon book created |
| 400 | Invalid request |

##### Security

| Security Schema | Scopes |
| --- | --- |
| bearerAuth | |

### /coupons/codes

#### POST
##### Summary:

Upload a list of codes to an existing coupon book

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Codes uploaded successfully |
| 400 | Invalid request |

##### Security

| Security Schema | Scopes |
| --- | --- |
| bearerAuth | |

### /coupons/assign

#### POST
##### Summary:

Assign a new random coupon code to a user

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Coupon assigned successfully |
| 400 | Invalid request |

##### Security

| Security Schema | Scopes |
| --- | --- |
| bearerAuth | |

### /coupons/assign/{code}

#### POST
##### Summary:

Assign a specific coupon code to a user

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| code | path |  | Yes | string |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Coupon assigned successfully |
| 400 | Invalid request |

##### Security

| Security Schema | Scopes |
| --- | --- |
| bearerAuth | |

### /coupons/lock/{code}

#### POST
##### Summary:

Temporarily lock a coupon for redemption

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| code | path |  | Yes | string |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Coupon locked successfully |
| 400 | Invalid request |
| 423 | Coupon is already locked |

##### Security

| Security Schema | Scopes |
| --- | --- |
| bearerAuth | |

### /coupons/redeem/{code}

#### POST
##### Summary:

Permanently redeem a coupon

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| code | path |  | Yes | string |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Coupon redeemed successfully |
| 400 | Invalid request |
| 423 | Coupon is already redeemed or locked |

##### Security

| Security Schema | Scopes |
| --- | --- |
| bearerAuth | |

### /users/{userId}/coupons

#### GET
##### Summary:

Get all assigned coupon codes for a user

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| userId | path |  | Yes | string |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | List of assigned coupons |
| 400 | Invalid request |

##### Security

| Security Schema | Scopes |
| --- | --- |
| bearerAuth | |
