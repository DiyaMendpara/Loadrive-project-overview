# Loadrive CRM Backend

Welcome to **Loadrive CRM**, a progressive, domain-driven CRM and logistics management system designed for freight, booking, and transport orchestration. This backend API serves as the backbone for managing transport pricing, quotation workflows, client and customer tracking, transporter dispatch, shift-based attendance, feedback collection, and overall system audits.

---

## Table of Contents
- [Project Overview](#project-overview)
- [Your Role (Backend Developer)](#your-role-backend-developer)
- [Tech Stack](#tech-stack)
- [System Architecture](#system-architecture)
- [Key Features](#key-features)
- [Challenges Solved](#challenges-solved)
- [API Design](#api-design)
- [Database Optimizations](#database-optimizations)
- [Learnings](#learnings)
- [Setup & Run Instructions](#setup--run-instructions)

---

## Project Overview

**Loadrive CRM Backend** is a highly structured RESTful API that handles complex shipping, quotation, and logistics workflows. The system manages the complete lifecycle of customer leads, drafts complex quotations based on transporter availability and distance-based pricing, auto-generates bookings upon quote acceptance, orchestrates transporter dispatches with auto-computed tracking numbers, tracks driver and staff shift attendance, logs analytical customer feedback, and resolves transporter and cargo disputes.

This is a production-grade system managing real freight logistics operations with thousands of active users across multiple regions.

---

## Your Role (Backend Developer)

As the **Lead Backend Developer** on this project, your responsibilities encompassed:

- **Architecting Domain Modules:** Designed and built 17 modular domains using clean separation of concerns with Controllers, Services, Models, and DTOs
- **Schema & Database Modeling:** Crafted scalable schemas with indexes, nested properties for cargo specifications, routes, and status history logs with soft-delete mechanics
- **Data Access Optimization:** Developed optimized database queries to support multi-faceted filtering, complex sorting on nested columns, and real-time dashboard statistics
- **Security Infrastructure:** Implemented JWT authentication and fine-grained Role-Based Access Control (RBAC) to restrict resources dynamically based on permissions
- **Business Logic & Automation:** Engineered complex workflows including auto-converting quotations into bookings, generating business-rule-compliant tracking numbers, managing Proof of Delivery states, and executing post-delivery feedback loops

---

## Tech Stack

The backend is built using a modern, scalable TypeScript stack:

- **Core Framework:** NestJS for dependency injection, controllers, guards, and module organization
- **Database Layer:** MongoDB with Mongoose for document-relational mapping, validation schemas, and aggregation pipelines
- **Authentication & Security:** Passport.js with JWT (JSON Web Tokens) for secure session management and endpoint authorization
- **Validation:** class-validator and class-transformer for strict runtime incoming DTO validation
- **Email Delivery:** SendGrid Mail API and Nodemailer for automated customer notifications, invitation URLs, and reset links
- **API Documentation:** Swagger/OpenAPI setup globally for live endpoint testing
- **Runtime Environment:** Node.js with TypeScript compiled to modern JavaScript standards

---

## System Architecture

The backend follows a layered, module-driven monolithic architecture:

**Core Layers:**
- **Entry Point:** Main application bootstrap with global middleware, pipes, and error handling
- **Security Layer:** JWT authentication guard validates incoming requests, loads user records, and retrieves associated role permissions
- **Authorization Layer:** Dynamic permissions guard validates if users hold permissions mandated by endpoint decorators
- **Controller Layer:** HTTP request handlers with input validation and response formatting
- **Service Layer:** Business logic implementation, workflow orchestration, and data transformations
- **Data Layer:** Mongoose models and schemas with MongoDB aggregation pipelines

**Key Components:**
- Global validation pipes enforce implicit type conversions and reject unauthorized parameters
- Custom JWT guard intercepts requests, validates tokens from authorization headers
- Database management module handles connection pooling and cloud cluster connectivity
- Module-based architecture enables scalability and maintainability

---

## Key Features

**1. Quotation Engine**
- Translates cargo parameters, pickup and delivery specifications into structured quotations
- Integrates transporter pricing and availability
- Supports multi-status tracking: follow-up, booked, and draft states
- Dynamic quote validity and pricing recalculation

**2. Automated Booking Workflow**
- Accepts finalized quotations and auto-drafts bookings per assigned transporter
- Dynamically generates tracking numbers with company-specific prefixes
- Enforces Proof of Delivery uploads before delivery confirmation
- Comprehensive booking status lifecycle management

**3. Attendance & Shift Management**
- Monitors working hours with precise check-in and check-out timestamps
- Links users to specific shift configurations
- Supports shift-based analytics and reporting
- Automated shift validation and conflict detection

**4. Analytical Customer Feedback**
- Post-delivery survey metrics and Net Promoter Score calculations
- Real-time aggregation of positive, neutral, and negative ratings
- Customer satisfaction trend analysis
- System-wide performance metrics

**5. Dynamic Role-Based Access Control**
- Granular user creation with role assignments
- Custom permissions management per role
- Dynamic permission loading during token validation
- Fine-grained endpoint access control

**6. Master Data System**
- Centralized registry for transit statuses and location types
- Team and shift schedule management
- Avoids hardcoded status values and ensures consistency
- Enables system-wide configuration changes

---

## Challenges Solved

### Challenge 1: Database Connectivity in Containerized Environments
**Problem:** Intermittent DNS failures during container initialization when resolving cloud database SRV records, preventing reliable connections.

**Solution:** Implemented explicit fallback nameserver configuration during application bootstrap to ensure consistent cloud cluster connectivity across all deployment environments.

### Challenge 2: Complex Sorting Across Related Entities with Pagination
**Problem:** Sorting parent documents (Bookings, Feedback) by populated child document fields (Customer name) while maintaining accurate pagination proved problematic with traditional query approaches.

**Solution:** Designed hybrid aggregation pipelines that perform early relationship lookups for sorting evaluation, apply pagination limits, and defer secondary entity lookups until after pagination to minimize memory usage.

### Challenge 3: Proof of Delivery Workflow Integrity
**Problem:** Ensuring strict status transitions, preventing bookings from being marked delivered without verified upload files, and maintaining accurate POD status across the system.

**Solution:** Programmed database validation rules within service layers checking status requirements, throwing appropriate errors when conditions are unfulfilled, and auto-calculating POD status with comprehensive status logs.

---

## API Design

The backend adopts REST API best practices ensuring high developer experience and integration reliability:

**Standardized Response Format:**
Every API response follows a consistent JSON structure including the action message, data payload, and pagination metadata (total count, current page, page limit, total pages).

**Request Validation:**
Strict payload filtering prevents unknown properties from being accepted, ensuring only whitelisted data reaches service layers. All incoming requests undergo type validation and automatic type coercion.

**API Documentation:**
Endpoints are self-documenting with OpenAPI attributes enabling developers to inspect, understand, and test endpoints directly through interactive Swagger documentation available when the server runs.

**Security by Default:**
All protected endpoints require valid JWT tokens in the authorization header. Permission validation occurs automatically based on endpoint decorators, preventing unauthorized access at the framework level.

---

## Database Optimizations

Production database performance is critical. The system implements several advanced optimization strategies:

**Late-Join Pattern:**
Lookups for unrelated relational data (transporters, quotations, users, locations) execute only after pagination has reduced the dataset, preventing unnecessary memory consumption loading thousands of related documents.

**Dual-Execution Pipeline for Pagination:**
Single network call retrieves both paginated data and total count by cloning the aggregation pipeline with a count stage, eliminating the overhead of separate count queries.

**Inline Accumulators for Analytics:**
Dashboard metrics (positive, neutral, negative ratings) calculate within single aggregation stages using conditional expressions, avoiding multiple query executions for dashboard assembly.

**Virtual Field Optimization:**
Case-insensitive sorting and numeric type conversions happen within aggregation stages using field transformations, ensuring consistent, database-level optimizations rather than in-application processing.

---

## Learnings

**1. Guard Execution Precedence in Modern Frameworks:**
Understanding how execution contexts resolve—authentication guards populate request user data, authorization guards evaluate permission arrays—enabled designing secure, zero-overhead routing blocks.

**2. Aggregations Over Traditional Queries:**
While simple query builders are readable, complex multi-entity sorting and dynamic analytics require native database aggregation pipelines to avoid server-side CPU blockages and memory exhaustion.

**3. Flexible Schemas with Hard Rules:**
Designing schemas with flexible properties (custom pricing structures, dynamic fields) requires strict validator constraints to ensure schema changes do not crash runtime calculations and maintain data integrity.

**4. Monitoring and Observability:**
Implementing comprehensive logging at service boundaries enables rapid debugging in production environments and reveals performance bottlenecks before they impact users.

---

## Setup & Run Instructions

### Prerequisites
- Node.js (version 18 or higher)
- npm or yarn package manager
- MongoDB instance (local or cloud)

### Installation

1. **Install dependencies:**
   Navigate to the project directory and run npm install

2. **Configure Environment Variables:**
   Create a `.env` file in the root directory with the following variables:
   - PORT: Application server port
   - MONGODB_URI: MongoDB connection string
   - JWT_SECRET: Secret key for JWT token generation
   - PAGINATION_LIMIT: Default items per page
   - BASE_URL: API base URL for generated links

### Execution

- **Development Mode:** Run with automatic file watching for development
- **Production Build:** Compile TypeScript to production-ready JavaScript
- **Production Mode:** Run optimized production build

### API Documentation

Once the server is running, access the interactive Swagger documentation at the `/api/docs` endpoint where you can inspect endpoint schemas and test REST requests directly.

---

## Project Status

This is an active, production-grade system currently managing real freight logistics operations. The platform handles thousands of daily transactions across multiple regions with enterprise-grade reliability and performance standards.
