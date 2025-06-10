# Prisma ORM: A Modern Database Toolkit

## What is an ORM?
ORMs are tools that let you manipulate data in your database and are widely used in the software industry. We'll dive deep into one ORM popular in the Node.js landscape: Prisma ORM.

## Introduction
Prisma ORM is a next-generation ORM that provides a type-safe database client for Node.js and TypeScript. It simplifies database operations and provides a modern developer experience.

## Challenges with Raw SQL
- **Repetitive Code**: Writing the same SQL statements repeatedly
- **Schema Visibility**: No clear view of database structure in codebase
- **Schema Changes**: Difficult to modify database structure (migrations)
- **Error-Prone**: Manual migrations are tedious and error-prone

## Advantages of ORMs
- **Productivity**: Reduces the amount of boilerplate code for database interactions.
- **Portability**: Makes it easier to switch databases without significant code changes.
- **Security**: Helps prevent SQL Injection by using parameterized queries.
- **Maintenance**: Simplifies the maintenance of database interactions and schema changes.
- **Performance**: Automatic query optimization
- **Validation**: Built-in schema validation and type checking
- **Flexibility**: Cross-database compatibility

## Disadvantages of ORMs
- **Learning Curve**: Steep learning curve for beginners
- **SQL Limitations**: May not support all SQL features
- **Performance Overhead**: Potential performance impact
- **Control Limitations**: Limited control over raw SQL
- **Migration Complexity**: Can be complex in large projects

## Prisma Core Components

### 1. Prisma Client
The client is a separate library that you will use to interact with your database.  It’s generated based on the models you define in your Prisma schema (schema.prisma) and provides a simple, intuitive, and type-safe API for CRUD operations, filtering, pagination, and more.

To generate a client, we use the command:
```bash
npx prisma generate
```

**Key Features:**
- Auto-generated based on schema
- Query building and execution
- Connection management
- Full TypeScript support

### 2. Prisma Migrate
Prisma migrate is a tool that helps you perform database migrations.
Migrations are versioned files that describe changes to your database schema. 

**Key Features:**
- Version control for database schema
- Safe schema changes
- Migration history tracking
- Rollback capabilities

### Migrations

In Prisma, migrations are a way to manage and apply changes to your database in a controlled and consistent way

Whenever you make modifications to your Prisma schema (schema.prisma), such as adding, removing, or altering fields or models, you create a new migration to reflect those changes in your actual database.

To run a migration, execute the following command

```npx prisma migrate dev --name MIGRATION-NAME```

### 3. Prisma Studio
A GUI for database management.

**Key Features:**
- Visual data browser
- CRUD operations interface
- Relationship visualization
- Data filtering and sorting

## Getting Started

### Installation
```bash
npm install prisma --save-dev
npm install @prisma/client
```

### Client Setup
```javascript
import { PrismaClient } from '@prisma/client'
const client = new PrismaClient()
```


### Models
Models represent the different tables in your database.
Models are made up of fields (fields correspond to the columns in our database).
Models represent database tables and their relationships in a way that feels intuitive and familiar to developers.

### Key concepts of Prisma Models﻿
**Data Models**: Define the tables and their columns (fields) in your database.
**Field Types**: Specify the data types for each field (e.g., String, Int, Boolean).
**Relations**: Describe how models relate to each other using relation fields.
**Attributes**: Add additional metadata to fields and models using attributes like @id, @default, @relation.

Fields are made up of four different parts:
- **Field name** (required) eg productTitle, name, etc.

- **Data type** of the field (required): e.g. String, Boolean, Integer etc

- **Field type** modifier (optional): for example, we can use the ? modifier to mean a field is optional.

- **Attributes** (optional): special annotations that you can apply to model fields to define various constraints, behaviors and relationships. They start with the @ symbol.


## Schema Definition

### Overview
The Prisma schema is a file where you will define your models. It consists of:
- Data sources (database connections)
- Generators (client generation settings)
- Data model definitions (application models and relations)

This schema file lives in your codebase and is tracked by version control and helps define database structure.
For more info, kindly refer to the [Prisma docs](https://www.prisma.io/docs/orm/prisma-schema/data-model/relations)
### Example Schema
The following is an example of a Prisma Schema that specifies:

1. A data source (PostgreSQL)
2. A generator (Prisma Client)
3. A data model definition with three models (User, Profile, Post) and their relations
4. Field mappings and attributes

```prisma
// schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id String @id @default(uuid())
  firstName String
  lastName String
  username String @unique
  emailAddress String @unique
  age Int? @map("user_age") @default(10)
  
  isDeleted Boolean @default(false)
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  profile Profile?
  posts Post[]

  @@map("users_table")
}

model Profile {
  profileId Int @default(autoincrement()) @id
  profileImageUrl String @map("profile_image_url")
  country String @map("country")
  userId String @unique
  user User @relation(fields: [userId], references: [id])
  @@map("profiles_tables")
}

model Post {
  post_id String @default(uuid()) @id
  title String @map("post_title")
  content String @map("post_content")
  isDelete Boolean @default(false)
  authorId String
  author User @relation(fields: [authorId], references: [id])
}
```

Key Schema Features:
- **Field Types**: Basic types (String, Int, Boolean, DateTime)
- **Field Attributes**: @id, @unique, @default, @map for database column mapping
- **Relations**: One-to-one (User ↔ Profile) and one-to-many (User → Posts)
- **Table Mapping**: @@map for custom table names
- **Timestamps**: Automatic createdAt and updatedAt fields
- **Default Values**: @default for field initialization

## CRUD Operations

Prisma Client provides a powerful and type-safe API for performing database operations. CRUD stands for:

- **C**reate: Insert new records into your database
- **R**ead: Query and retrieve records from your database
- **U**pdate: Modify existing records in your database
- **D**elete: Remove records from your database

Each operation is fully type-safe and provides autocompletion in your IDE. The examples below demonstrate basic CRUD operations, but Prisma Client offers many more features like:
- Filtering and sorting
- Pagination
- Relations and nested queries
- Transactions
- Raw queries

For a complete reference of all available methods and options, see the [Prisma Client API Reference](https://www.prisma.io/docs/orm/reference/prisma-client-reference).

### Create
```javascript
async function createUser() {
    const user = await client.user.create({
        data: {
            firstName: 'John',
            lastName: 'Doe',
            username: 'johndoe',
            emailAddress: 'johndoe@gmail.com'
        }
    });
    console.log(user);
}
```

### Read
```javascript
// Find all users
async function getUsers() {
    const users = await client.user.findMany();
    console.log(users);
    return users;
}

// Find with filters
const publishedPosts = await client.post.findMany({
    where: { published: true }
});

// Including relations
const userWithPosts = await client.user.findUnique({
    where: { id: 1 },
    include: { posts: true }
});
```

### Update
```javascript
// Single record
async function updateUser() {
    const updatedUser = await client.user.update({
        where: { id: 1 },
        data: {
            firstName: 'Regina',
            lastName: 'Makena'
        }
    });
    console.log(updatedUser);
}

// Update many
async function updateManyUsers() {
    const updatedUsers = await client.user.updateMany({
        where: { age: { gt: 18 } },
        data: { isLegal: true }
    });
    console.log(updatedUsers);
}
```

### Delete
```javascript
// Single record
async function deleteUser() {
    const deletedUser = await client.user.delete({
        where: { id: 1 }
    });
    console.log(deletedUser);
}

// Delete many
async function deleteManyUsers() {
    const deletedUsers = await client.user.deleteMany({
        where: { age: { lt: 18 } }
    });
    console.log(deletedUsers);
}
```

## Relationships

### Overview
Relations in Prisma connect models in your schema. For example, a User can have many Posts.

### Types of Relationships

1. **One-to-One (1-1)**
   - Use `@relation` with `@unique` constraint
   - Example: User ↔ Profile
   ```prisma
   model User {
     id      String   @id
     profile Profile?
   }
   
   model Profile {
     id     String @id
     user   User   @relation(fields: [userId], references: [id])
     userId String @unique
   }
   ```

2. **One-to-Many (1-n)**
   - Parent model with array field
   - Example: User → Posts
   ```prisma
   model User {
     id    String @id
     posts Post[]
   }
   
   model Post {
     id       String @id
     author   User   @relation(fields: [authorId], references: [id])
     authorId String
   }
   ```

3. **Many-to-Many (m-n)**
   - Junction table or direct relations
   - Example: Posts ↔ Tags
   ```prisma
   model Post {
     id    String @id
     tags  Tag[]
   }
   
   model Tag {
     id    String @id
     posts Post[]
   }
   ```

For futher info, refer to [Prisma Relations](https://www.prisma.io/docs/orm/prisma-schema/data-model/relations)