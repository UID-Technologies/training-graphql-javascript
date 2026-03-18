## Lab 01 — Introduction to GraphQL

### Overview

In this lab you’ll learn what GraphQL is, why it was created, how it compares to REST, and how schemas define the capabilities of a GraphQL API. You’ll also run queries against a real GraphQL endpoint and inspect a small schema to connect the concepts to practice.

### Duration

- **Estimated time**: 45–60 minutes

### Learning objectives

By the end of this lab, you will be able to:

- **Explain** what GraphQL is (and what it is not).
- **Describe** the history and motivation behind GraphQL.
- **Compare** REST and GraphQL trade-offs in practical terms.
- **Read** a GraphQL schema and identify types, fields, and arguments.
- **Run** a basic query and interpret the response shape.

### Prerequisites

- **Basic JavaScript knowledge** (objects, JSON).
- **Comfort reading APIs** (HTTP, endpoints, JSON payloads).
- **PowerShell** available (Windows default).

### Materials

- This lab document
- A browser
- Optional: an HTTP client (Postman / Insomnia). PowerShell commands are provided.

---

## What Is GraphQL?

### Introduction

GraphQL is a **query language for APIs** and a **runtime** for fulfilling those queries with your existing data. Instead of many endpoints that each return a fixed shape, GraphQL typically exposes a **single endpoint** that accepts queries describing exactly what data the client wants.

GraphQL is commonly used for:

- **Client-driven data fetching** (web + mobile)
- **Aggregating data** from multiple services behind one API
- **Reducing over-fetching/under-fetching**
- **Strongly typed contracts** between client and server

### History

- **2012**: GraphQL was developed internally at Facebook to improve mobile performance and developer experience.
- **2015**: GraphQL was open-sourced.
- **2018**: The GraphQL Foundation formed under the Linux Foundation to support community governance and evolution.

The key driver was practical: REST endpoints and payloads were hard to evolve for many clients, especially mobile, where bandwidth and latency matter and where clients need different data shapes for different screens.

### What Is GraphQL?

GraphQL has three core ideas:

- **A type system (schema)**: The API is described as types and fields.
- **Declarative queries**: Clients request a specific response shape.
- **Resolver-based execution**: Server code resolves fields and assembles the response.

Important clarifications:

- **GraphQL is not a database.** It can sit in front of any data source: SQL, NoSQL, REST services, gRPC, files, etc.
- **GraphQL is not tied to a language.** This course uses JavaScript, but GraphQL is language-agnostic.
- **GraphQL is not “always faster.”** It can reduce payload size and round-trips, but it can also create expensive queries if not designed and controlled properly.

### Who Is Using GraphQL?

GraphQL is widely adopted in industry. Examples include:

- **GitHub**: GraphQL API for flexible access to GitHub data.
- **Shopify**: Extensive GraphQL usage for commerce and admin APIs.
- **Meta (Facebook/Instagram)**: Original creators, heavy internal use.
- **Many SaaS and product teams**: Especially where multiple clients (web, iOS, Android) need different data shapes.

The adoption drivers are usually:

- **Multiple client types** with different needs
- **Rapid UI iteration**
- **Complex domain graphs** (users, orders, products, permissions)

---

## REST vs. GraphQL

### REST (typical characteristics)

- **Many endpoints** (e.g., `/users/:id`, `/users/:id/orders`, `/orders/:id/items`)
- Each endpoint returns a **fixed payload shape**
- Versioning often handled via **URL versions** (`/v1`, `/v2`) or custom headers

Common REST pain points in UI-heavy apps:

- **Over-fetching**: endpoint returns more data than the UI needs
- **Under-fetching**: UI needs data from multiple endpoints → multiple round-trips
- **Endpoint sprawl**: many endpoints for slightly different screens

### GraphQL (typical characteristics)

- Usually a **single endpoint** (e.g., `/graphql`)
- Client chooses the **exact shape** of returned data
- Schema defines the **capabilities**; clients query within those constraints

Key benefits:

- **Precise fetching**: clients request only needed fields
- **Fewer round-trips**: query can fetch related data in one request
- **Typed contract**: schema enables validation, tooling, and safer evolution

Key trade-offs and responsibilities:

- **Caching changes**: HTTP caching by URL is less straightforward; you typically rely on client caching (e.g., normalized caches) or persisted queries.
- **Complexity control**: servers often need depth/complexity limits, timeouts, and query whitelisting.
- **N+1 risks**: naive resolvers can cause many backend calls; solved with batching (e.g., DataLoader) and careful resolver design.

### Practical comparison (cheat sheet)

- **Best fit for REST**: simple resources, standard CRUD, strong HTTP caching needs, public APIs with predictable shapes.
- **Best fit for GraphQL**: complex UI needs, many clients, fast product iteration, domain modeled as relationships.

---

## GraphQL Schemas

### What is a schema?

A GraphQL schema is the **contract** of your API: it defines:

- **Types** (objects, scalars, enums, interfaces, unions)
- **Fields** on types (and their return types)
- **Arguments** accepted by fields
- **Entry points**: `Query`, `Mutation` (and optionally `Subscription`)

### Core schema elements (conceptual)

- **Scalar types**: `String`, `Int`, `Float`, `Boolean`, `ID` (plus custom scalars like `DateTime`)
- **Object types**: e.g., `User`, `Order`
- **Query type**: read operations (fetch data)
- **Mutation type**: write operations (create/update/delete)

### Example schema snippet (SDL)

This is the Schema Definition Language (SDL), a common way to describe schemas:

```graphql
type Query {
  user(id: ID!): User
}

type User {
  id: ID!
  name: String!
  email: String
}
```

How to read it:

- `user(id: ID!): User` means: “you can query `user` by a required `id` and get a `User` back.”
- `String!` means the field is **non-nullable**.
- `email: String` means it may be **null**.

---

## Exercise 1 — Run a GraphQL query (no local setup)

### Goal

Use a real GraphQL endpoint to:

- Explore schema documentation (introspection)
- Run a query and see the response shape

### Instructions (browser)

1. Open the GraphQL IDE for the SpaceX public GraphQL API:
  - Use this URL: `https://spacex-production.up.railway.app/`
2. Find the **Docs** panel (schema documentation).
3. Locate the `Query` type and identify a few available fields.

### Run this query

```graphql
query GetCompanyAndLaunches {
  company {
    name
    founder
    founded
    employees
  }
  launchesPast(limit: 3) {
    mission_name
    launch_date_utc
    rocket {
      rocket_name
    }
  }
}
```

### Questions

- **Q1**: Which parts of the response came from `company` vs `launchesPast`?
- **Q2**: If you remove `employees` from the query, what changes in the response?
- **Q3**: What type does the docs say `launchesPast` returns (list of what)?

---

## Exercise 2 — REST vs GraphQL (shape control)

### Goal

Experience the “response shape matches query shape” rule.

1. Modify the previous query to request:
  - Only `company { name }`
  - For launches, request only `mission_name`
2. Re-run the query.

### Deliverable

Paste (or describe) the **new response shape** and explain how it differs from a typical REST endpoint response.

---

## Exercise 3 — Read a schema like a contract

### Goal

Practice reading schema fields, arguments, and nullability.

Using the API docs from Exercise 1:

1. Find one field on `Query` that accepts arguments (for example, a `limit`).
2. Identify:
  - The **argument names**
  - Which arguments are **required** (non-null `!`)
  - The **return type** (object vs list vs scalar)

### Deliverable

Write a short summary:

- The field you chose
- Its arguments (with required/optional)
- Its return type

---

## Knowledge check (quick)

- **1)** In one sentence, what is GraphQL?
- **2)** Name two common REST pain points GraphQL aims to reduce.
- **3)** What is the purpose of the `Query` type in a schema?
- **4)** What does `String!` mean vs `String`?

---

