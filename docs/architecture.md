# LogikBetPrediction — System Architecture

## Overview

LogikBetPrediction is a full-stack football prediction platform built with Next.js, React, TypeScript, Prisma, and PostgreSQL.

The application is designed around four primary areas:

1. Football data ingestion and prediction management
2. User authentication and authorization
3. VIP subscriptions and payments
4. Administrative management and platform operations

The production source code is maintained in a private repository. This document provides a high-level technical overview without exposing proprietary implementation details, credentials, or internal source code.

---

# Technology Stack

## Frontend

* Next.js
* React
* TypeScript
* Tailwind CSS
* Responsive component-based UI
* Dark/light theme support

## Backend

* Next.js server-side functionality
* API routes
* Server-side authentication
* Prisma ORM

## Database

* PostgreSQL
* Neon

## Authentication

* Auth.js / NextAuth
* Credentials authentication
* Prisma Adapter
* bcrypt password hashing
* Role-based authorization

## Football Data

The platform supports multiple football-data providers.

Current integrations include:

* API-Football
* PlayerElo

The architecture allows external fixture and prediction data to be normalized before being stored in the platform database.

---

# High-Level Architecture

```text
                         USERS
                           │
                           ▼
                ┌─────────────────────┐
                │     Next.js App     │
                │                     │
                │  React + TypeScript │
                │    Tailwind CSS     │
                └──────────┬──────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
   Authentication      Application       Admin
      System             APIs           System
          │                │                │
          └────────────────┼────────────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │   Prisma    │
                    │     ORM     │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │ PostgreSQL  │
                    │    Neon     │
                    └─────────────┘


                 EXTERNAL SERVICES
                        │
            ┌───────────┼───────────┐
            ▼           ▼           ▼
       API-Football  PlayerElo   Email/Payments
```

---

# Application Layers

## 1. Presentation Layer

The presentation layer is implemented with Next.js, React, TypeScript, and Tailwind CSS.

It provides interfaces for:

* Visitors
* Registered users
* VIP subscribers
* Administrators

The interface is responsive across desktop and mobile devices and supports light and dark themes.

---

## 2. Authentication Layer

Authentication is handled using Auth.js / NextAuth.

The authentication system supports:

* User registration
* Credentials login
* Password hashing
* Session management
* Email verification
* Password recovery
* Protected routes
* Role-based authorization

The platform distinguishes between standard users and administrators.

Subscription information provides an additional authorization layer for premium content.

---

# Football Data Pipeline

One of the main architectural components of LogikBetPrediction is the football data ingestion pipeline.

External football providers supply fixture and prediction information.

```text
Football Data Provider
          │
          ▼
   Provider API
          │
          ▼
    Sync Service
          │
          ▼
  Data Validation
          │
          ▼
 Competition Filtering
          │
          ▼
 Fixture Normalization
          │
          ▼
     Prisma ORM
          │
          ▼
     PostgreSQL
          │
          ▼
 Prediction Platform
```

Before external data reaches the user interface, it passes through validation and normalization.

This allows the application to handle inconsistent or incomplete responses from third-party providers.

---

# API-Football Integration

API-Football is used as one of the platform's football data providers.

The synchronization process retrieves fixture information and applies platform-specific rules before storing it.

The pipeline performs tasks such as:

* Fetching daily fixtures
* Filtering supported competitions
* Validating fixture information
* Preventing duplicate fixtures
* Creating new database records
* Updating existing fixtures
* Limiting imports where appropriate
* Tracking provider usage

This prevents the frontend from depending directly on the structure of an external provider.

---

# PlayerElo Integration

PlayerElo provides another football-data source used by the platform.

Incoming records are validated before being accepted.

Validation checks include:

* Fixture identifiers
* Kickoff timestamps
* Home probabilities
* Draw probabilities
* Away probabilities
* Historical fixture detection

Incomplete provider records can therefore be skipped rather than creating unreliable predictions.

---

# Database Architecture

Prisma ORM provides the application data layer.

Major entities include:

```text
User
 │
 ├── Account
 ├── Session
 ├── Subscription
 ├── Payment
 ├── PredictionView
 ├── NotificationPreference
 └── Notification


Fixture
 │
 └── Prediction


Administrator
 │
 └── AdminAction
```

Core application models include:

### User

Stores platform user information and authorization roles.

### Fixture

Represents football matches imported from external data providers.

### Prediction

Stores prediction information associated with fixtures.

### Subscription

Tracks user access to premium platform features.

### Payment

Stores subscription-related payment records.

### Notification

Represents messages delivered or scheduled for users.

### NotificationPreference

Stores individual user communication preferences.

### AdminAction

Provides records of important administrative actions.

---

# Prediction Architecture

Predictions are associated with fixtures rather than being stored as isolated content.

Conceptually:

```text
Competition
      │
      ▼
   Fixture
      │
      ▼
 Prediction
      │
 ┌────┴─────┐
 ▼          ▼
FREE       VIP
```

This makes it possible to control visibility independently from the underlying match information.

---

# Subscription Architecture

The platform supports premium access through subscriptions.

```text
User
 │
 ▼
Subscription
 │
 ├── Plan
 ├── Status
 ├── Start Date
 └── Expiration
 │
 ▼
VIP Authorization
 │
 ▼
Premium Predictions
```

Payment infrastructure is kept separate from prediction logic.

This allows payment providers to change without requiring major changes to the prediction system.

---

# Admin Architecture

Administrative functionality is separated from the normal user experience.

The admin system provides interfaces for:

* Platform overview
* Prediction management
* User management
* Payment management
* Site configuration
* Integration management
* Football-data synchronization

Administrative API operations require authorization before protected operations can execute.

---

# Notifications

The notification architecture supports multiple communication channels.

```text
Platform Event
      │
      ▼
Notification Service
      │
 ┌────┴─────┐
 ▼          ▼
Email    WhatsApp
```

Users can maintain notification preferences independently of their authentication information.

---

# Reliability & Data Validation

External APIs cannot always be assumed to return complete or valid information.

The platform therefore validates external records before storing them.

Examples of handled conditions include:

* Missing probabilities
* Missing fixture IDs
* Invalid kickoff timestamps
* Historical fixtures
* Duplicate fixtures
* Provider rate limits
* Temporary API failures
* Empty provider responses

Synchronization diagnostics also provide visibility into how many records were received, created, updated, or skipped.

---

# Security Considerations

The architecture incorporates several security practices:

* Password hashing
* Server-side authentication
* Role-based authorization
* Protected administrative routes
* Environment-based secret management
* API credential isolation
* Server-side database access
* Input validation
* Restricted administrative operations

Sensitive credentials and production configuration are never included in this public showcase repository.

---

# Deployment Architecture

```text
                    GitHub
                      │
                      ▼
                   Vercel
                      │
              ┌───────┴────────┐
              ▼                ▼
         Next.js App       Server APIs
              │                │
              └───────┬────────┘
                      ▼
                  Prisma ORM
                      │
                      ▼
               Neon PostgreSQL

External Providers
       │
       ├── API-Football
       ├── PlayerElo
       ├── Email Service
       └── Payment Provider
```

The application is deployed using Vercel while PostgreSQL infrastructure is provided by Neon.

---

# Architectural Principles

The project follows several core principles:

**Separation of concerns** — football data, authentication, payments, subscriptions, notifications, and administration are handled as distinct application domains.

**Provider independence** — external football providers are normalized before their data reaches core application functionality.

**Server-side security** — sensitive database and authentication operations remain on the server.

**Failure tolerance** — invalid external records can be rejected without interrupting the entire synchronization process.

**Scalability** — the architecture allows additional football providers, notification channels, payment providers, and prediction strategies to be introduced without redesigning the entire application.

---

## Source Code

This repository contains project documentation and visual demonstrations only.

The production implementation of LogikBetPrediction is maintained separately in a private repository.

Source code may be made available privately for technical review when appropriate.
