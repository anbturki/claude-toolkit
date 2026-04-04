---
name: drizzle-schema
description: Scaffold a Drizzle ORM database schema with tables, relations, indexes, and types. Use when adding new database tables to a Drizzle project.
argument-hint: [entity-name]
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(ls *), Bash(npx *), Bash(bunx *), Bash(pnpm *)
---

# Create Drizzle Schema

Scaffold a new Drizzle schema for `$ARGUMENTS`.

## Step 1: Learn Project Patterns

1. **Read CLAUDE.md** for database conventions
2. **Detect database dialect**: PostgreSQL, MySQL, or SQLite
3. **Find existing schemas**:
   ```
   Glob("**/schema/**/*.ts")
   Glob("**/schemas/**/*.ts")
   ```
4. **Read 2-3 existing schemas** — learn:
   - ID generation (cuid2, uuid, nanoid, auto-increment)
   - Timestamp patterns
   - Naming convention (snake_case columns)
   - Index patterns
   - How relations are defined
   - Multi-tenant patterns (orgId column)

## Step 2: Scaffold Schema

**File location:** Match existing schema file locations.

### PostgreSQL
```typescript
import { relations } from "drizzle-orm";
import {
  index,
  pgTable,
  text,
  timestamp,
  boolean,
  integer,
  jsonb,
} from "drizzle-orm/pg-core";
// Match project's ID generation
import { createId } from "@paralleldrive/cuid2"; // or nanoid, uuid, etc.

export const ${entities} = pgTable(
  "${entities}",
  {
    id: text("id")
      .primaryKey()
      .$defaultFn(() => createId()),
    name: text("name").notNull(),
    // Add fields matching project conventions
    createdAt: timestamp("created_at").defaultNow().notNull(),
    updatedAt: timestamp("updated_at")
      .defaultNow()
      .$onUpdate(() => new Date())
      .notNull(),
  },
  (table) => [
    index("${entities}_created_at_idx").on(table.createdAt),
  ],
);

// Relations
export const ${entities}Relations = relations(${entities}, ({ one, many }) => ({
  // Define relations matching project patterns
}));
```

### MySQL
```typescript
import { mysqlTable, varchar, timestamp, int } from "drizzle-orm/mysql-core";

export const ${entities} = mysqlTable("${entities}", {
  id: varchar("id", { length: 36 }).primaryKey().$defaultFn(() => createId()),
  name: varchar("name", { length: 255 }).notNull(),
  createdAt: timestamp("created_at").defaultNow().notNull(),
  updatedAt: timestamp("updated_at").defaultNow().onUpdateNow().notNull(),
});
```

### SQLite
```typescript
import { sqliteTable, text, integer } from "drizzle-orm/sqlite-core";

export const ${entities} = sqliteTable("${entities}", {
  id: text("id").primaryKey().$defaultFn(() => createId()),
  name: text("name").notNull(),
  createdAt: integer("created_at", { mode: "timestamp" }).$defaultFn(() => new Date()),
  updatedAt: integer("updated_at", { mode: "timestamp" }).$defaultFn(() => new Date()),
});
```

## Column Type Reference

| Type | PostgreSQL | MySQL | SQLite |
|------|-----------|-------|--------|
| String | `text("name")` | `varchar("name", { length: 255 })` | `text("name")` |
| Integer | `integer("count")` | `int("count")` | `integer("count")` |
| Boolean | `boolean("active")` | `boolean("active")` | `integer("active", { mode: "boolean" })` |
| Timestamp | `timestamp("at")` | `timestamp("at")` | `integer("at", { mode: "timestamp" })` |
| JSON | `jsonb("data")` | `json("data")` | `text("data", { mode: "json" })` |
| Decimal | `numeric("amount", { precision: 10, scale: 2 })` | `decimal("amount", { precision: 10, scale: 2 })` | `real("amount")` |

## Foreign Key Patterns

```typescript
// Required (cascade delete)
orgId: text("org_id")
  .notNull()
  .references(() => organizations.id, { onDelete: "cascade" }),

// Optional (set null on delete)
createdBy: text("created_by")
  .references(() => users.id, { onDelete: "set null" }),

// Required (restrict delete)
parentId: text("parent_id")
  .notNull()
  .references(() => parents.id, { onDelete: "restrict" }),
```

## Index Patterns

```typescript
(table) => [
  index("${entities}_org_id_idx").on(table.orgId),
  index("${entities}_org_name_idx").on(table.orgId, table.name),  // composite
  unique("${entities}_org_name_unique").on(table.orgId, table.name),  // unique
]
```

## Type Inference

```typescript
// Infer types from schema — never define manually
export type ${Entity} = typeof ${entities}.$inferSelect;
export type New${Entity} = typeof ${entities}.$inferInsert;
```

## Step 3: After Creation

1. **Export** from schemas index: `export * from "./${entity}";`
2. **Generate migration**:
   ```bash
   npx drizzle-kit generate  # or bunx drizzle-kit generate
   ```
3. **Apply migration**:
   ```bash
   npx drizzle-kit migrate  # or project's migrate command
   ```

## Rules

1. **Match existing ID generation** — use the same library (cuid2, nanoid, uuid, etc.)
2. **Match existing timestamp pattern** — same column names and defaults
3. **Add indexes on foreign keys** — always index columns used in joins/filters
4. **Use type inference** — `$inferSelect` and `$inferInsert`, never manual types
5. **snake_case columns** — convention for database column names
6. **Generate migrations** — never manually create migration files
