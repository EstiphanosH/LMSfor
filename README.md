# Library Management System

A Library Management System API built with **NestJS, PostgreSQL, and Prisma ORM**.

This project provides the foundation for managing a library's book catalog, genres, users, and book reviews. It uses a relational database design to maintain data consistency and enforce relationships between entities.

## 1. Technology Stack

| Technology     | Purpose                              |
| -------------- | ------------------------------------ |
| NestJS         | Backend application framework        |
| TypeScript     | Application programming language     |
| PostgreSQL     | Relational database                  |
| Prisma ORM     | Database modeling and querying       |
| Prisma Migrate | Database schema migrations           |
| Prisma Client  | Type-safe database access            |
| Neon           | Hosted PostgreSQL database           |
| TSX            | Executing the TypeScript seed script |

## 2. Project Structure

text

```
library-management-system/
│
├── api/
│   ├── prisma/
│   │   ├── migrations/
│   │   ├── schema.prisma
│   │   └── seed.ts
│   │
│   ├── src/
│   │   ├── prisma/
│   │   │   ├── prisma.module.ts
│   │   │   └── prisma.service.ts
│   │   │
│   │   ├── app.controller.ts
│   │   ├── app.controller.spec.ts
│   │   ├── app.module.ts
│   │   ├── app.service.ts
│   │   └── main.ts
│   │
│   ├── .env
│   ├── .env.example
│   ├── package.json
│   ├── prisma.config.ts
│   └── tsconfig.json
│
└── README.md

```

## 3. Database Schema

The database currently contains four main models and one enum.

### Role Enum

The Role enum defines the supported user roles.

- `PATRON`: A regular library member who can access library services.
- `LIBRARIAN`: A library staff member responsible for managing library operations.

New users are assigned the `PATRON` role by default.

### User Model

Stores information about registered library users.

Fields include:

- `id`: Unique UUID identifier.
- `firstName`: User's first name.
- `lastName`: User's last name.
- `email`: Unique email address.
- `password`: Hashed password.
- `role`: User role.
- `createdAt`: Account creation timestamp.
- `updatedAt`: Last update timestamp.

A user can create multiple book reviews.

### Genre Model

Stores the categories used to organize books.

Fields include:

- `id`: Unique UUID identifier.
- `name`: Unique genre name.
- `description`: Optional genre description.
- `createdAt`: Creation timestamp.
- `updatedAt`: Last update timestamp.

A genre can contain multiple books.

### Book Model

Stores the library's book catalog.

Fields include:

- `id`: Unique UUID identifier.
- `isbn`: Unique International Standard Book Number.
- `title`: Book title.
- `author`: Book author.
- `description`: Optional book description.
- `publishedYear`: Publication year.
- `totalCopies`: Total copies owned by the library.
- `availableCopies`: Copies currently available.
- `genreId`: Foreign key referencing Genre.
- `createdAt`: Creation timestamp.
- `updatedAt`: Last update timestamp.

Each book belongs to exactly one genre.

### Review Model

Stores reviews submitted by library users.

Fields include:

- `id`: Unique UUID identifier.
- `bookId`: Foreign key referencing Book.
- `userId`: Foreign key referencing User.
- `rating`: Numeric book rating.
- `comment`: Optional review comment.
- `createdAt`: Creation timestamp.
- `updatedAt`: Last update timestamp.

A user can review multiple books, and a book can receive reviews from multiple users.

A composite unique constraint on `(bookId, userId)` prevents the same user from reviewing the same book more than once.

## 4. Schema Decisions

### Why does Book reference Genre instead of embedding it?

The system uses a relational database design, where Genre and Book are modeled as separate entities.

The Book model contains a `genreId` foreign key that references the primary key of the Genre model.

This decision was made for several reasons:

**1. Avoiding data duplication**

If genre information were embedded in every book record, the same genre name and description would be repeated across multiple books.

By storing genres separately, we maintain a single source of truth.

**2. Maintaining data consistency**

A foreign key ensures that every book references an existing genre.

This prevents books from being assigned to nonexistent genres.

**3. Easier updates**

If a genre description changes, we only need to update one Genre record rather than updating every associated book.

**4. Supporting efficient queries**

The relationship allows us to retrieve books by genre and retrieve a genre together with its associated books using Prisma queries.

**5. Maintaining referential integrity**

The relationship uses `onDelete: Restrict`, preventing a genre from being deleted while books are still associated with it.

This protects the consistency of the catalog.

### Prisma relational modeling vs. document database modeling

One thing that felt different when modeling in Prisma compared to a document database was how relationships are explicitly defined and enforced.

In Prisma with PostgreSQL, I needed to define separate models, foreign keys, and relation attributes. For example, the Book model references Genre through `genreId`, and PostgreSQL enforces that relationship.

In a document database such as MongoDB, I could instead embed genre information directly inside a book document or reference another document.

The relational approach required more upfront schema design, but it provides stronger referential integrity, reduces duplicated data, and makes managing shared genre information easier.

## 5. Database Relationships

The system uses the following relationships:

| Relationship  | Type        | Description                          |
| ------------- | ----------- | ------------------------------------ |
| Genre → Book  | One-to-many | A genre can contain multiple books.  |
| Book → Genre  | Many-to-one | Each book belongs to one genre.      |
| User → Review | One-to-many | A user can write multiple reviews.   |
| Book → Review | One-to-many | A book can receive multiple reviews. |

### Relationship overview

text

```
          GENRE
            |
            | 1
            |
            | N
           BOOK
            |
            | 1
            |
            | N
          REVIEW
            |
            | N
            |
            | 1
           USER

```

## 6. Database Configuration

The application uses a PostgreSQL database hosted on Neon.

The database connection is configured through an environment variable.

Create a `.env` file inside the `api` directory:

env

```
DATABASE_URL="postgresql://USERNAME:PASSWORD@HOST/DATABASE?sslmode=require"

```

## 7. Installation and Setup

### Prerequisites

Make sure you have installed:

- Node.js
- npm
- PostgreSQL database (local or hosted)
- Git

### Step 1: Clone the repository

bash

```
git clone https://github.com/EstiphanosH/library-management-system.git

```

Navigate into the project:

bash

```
cd library-management-system/api

```

### Step 2: Install dependencies

bash

```
npm install

```

### Step 3: Configure environment variables

Create your `.env` file and configure `DATABASE_URL`.

### Step 4: Validate the Prisma schema

bash

```
npx prisma validate

```

### Step 5: Generate Prisma Client

bash

```
npx prisma generate

```

### Step 6: Apply database migrations

For local development:

bash

```
npx prisma migrate dev

```

This applies the existing migrations and updates the database schema.

### Step 7: Seed the database

bash

```
npx prisma db seed

```

This executes the seed script and inserts the sample genres and books.

### Step 8: Start the application

bash

```
npm run start:dev

```

The NestJS application runs at:

text

```
http://localhost:3000

```

## 8. Database Seeding

The seed script is located at:

text

```
api/prisma/seed.ts

```

It inserts at least four genres and sixteen books distributed across those genres.

### Seeded Genres

| Genre           | Number of Books |
| --------------- | --------------- |
| Science Fiction | 4               |
| Fantasy         | 4               |
| Programming     | 4               |
| History         | 4               |
| **Total**       | **16**          |

The seed script uses Prisma `upsert` operations to avoid creating duplicate genres and books when executed repeatedly.

### Verify seeded records

Run:

bash

```
npx prisma studio

```

Prisma Studio opens at:

text

```
http://localhost:5555

```

You can inspect the Genre and Book records through its interface.

You can also verify record counts using SQL:

sql

```
SELECT COUNT(*) FROM genres;

SELECT COUNT(*) FROM books;

```

Expected results:

- Genres: 4
- Books: 16

## 9. Prisma Service Integration

The Prisma database integration is implemented in:

text

```
api/src/prisma/prisma.service.ts

```

`PrismaService` extends `PrismaClient` and implements NestJS lifecycle hooks.

- `OnModuleInit`: Establishes the database connection using `$connect()`.
- `OnModuleDestroy`: Closes the database connection using `$disconnect()`.

The service is registered through `PrismaModule`, which is imported into the root `AppModule`.

This provides centralized and reusable database access throughout the NestJS application.

## 10. Database Migrations

Prisma Migrate is used to manage database schema changes.

Create a new migration during development:

bash

```
npx prisma migrate dev --name migration_name

```

Check migration status:

bash

```
npx prisma migrate status

```

Generate Prisma Client:

bash

```
npx prisma generate

```

Migration files are stored in:

text

```
api/prisma/migrations/

```

These migration files should be committed to version control so that other developers can reproduce the database schema.

## 11. Data Integrity and Design Considerations

The following database constraints are implemented:

- UUID primary keys for all models.
- Unique email addresses for users.
- Unique ISBN values for books.
- Unique genre names.
- Foreign key relationship between Book and Genre.
- Foreign key relationships between Review, Book, and User.
- Composite unique constraint preventing duplicate reviews by the same user for the same book.
- Indexes on frequently queried fields, including book title, author, genre ID, and user role.
- Automatic creation and update timestamps.

The Book model also tracks total and available copies to support future borrowing and inventory functionality.

## 12. First Deliverable Summary

The first development milestone focuses on designing and implementing the library's book catalog data layer.

Completed functionality:

- Prisma database schema.
- Role enum with PATRON and LIBRARIAN.
- User model.
- Genre model.
- Book model with a Genre foreign key.
- Review model as an additional relationship.
- Hosted PostgreSQL database integration.
- Prisma database migrations.
- Seed script with four genres and sixteen books.
- NestJS PrismaService and PrismaModule.
- Environment configuration using DATABASE_URL.
- Schema documentation and modeling decisions.

## 13. Future Development

The next development milestones will extend the system with:

1. User authentication and registration.
2. JWT-based authentication.
3. Role-based access control.
4. Book and genre CRUD APIs.
5. Book search, filtering, and pagination.
6. Borrowing and returning books.
7. Loan tracking and due dates.
8. Review and rating APIs.
9. Automated testing.
10. API documentation using Swagger.

---

**Project:** Library Management System\
**Backend:** NestJS\
**Database:** PostgreSQL\
**ORM:** Prisma\
**Database Hosting:** Neon