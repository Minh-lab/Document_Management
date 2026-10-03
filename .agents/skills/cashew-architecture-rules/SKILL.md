---
name: cashew-architecture-rules
description: Defines the strict structural and data-flow constraints of the Cashew Architecture for Flutter applications. This skill forces agents to separate UI, Business Logic, and Database layers exactly as required by the project's setup. Use this skill whenever generating new features, pages, or data models to ensure architectural consistency.
---

# Cashew Architecture Rules

## Overview
This project strictly follows the **Cashew Architecture** for Flutter, which enforces a specific folder structure and unidirectional data flow. This skill ensures that code generation and refactoring respect these boundaries. 

Any violation of these rules (e.g., importing database tables directly into a UI page) is considered a critical failure.

## 1. Directory Structure Constraints
Every new file MUST be placed in its designated layer:

- **`lib/database/`**: 
  - ONLY for database setup and queries (using `drift` / `moor`).
  - Contains `tables.dart` (Schema) and `tables.g.dart` (Generated).
  - UI pages MUST NEVER import anything from this folder.

- **`lib/struct/`**: 
  - Contains pure Dart Data Models (Structs), global constants (like `defaultCategories.dart`), and settings.
  - Used to pass data between the `database` and `pages` layers.

- **`lib/pages/`**: 
  - Contains UI screens (`*_page.dart`). 
  - Must be as thin as possible. 
  - They ONLY import `widgets/`, `struct/`, and `functions.dart`.

- **`lib/widgets/`**: 
  - Reusable UI components (buttons, cards, forms).
  - Must not contain business logic.

- **`lib/functions.dart`**: 
  - The ONLY layer allowed to talk to `lib/database/`. 
  - Acts as the Business Logic / Service layer.
  - Exposes functions like `addDocument()`, `getDocuments()` that UI pages call.

- **`lib/colors.dart`**: 
  - Single source of truth for theming and colors.

## 2. Data Flow Rules (Strict)
1. **Pages -> Functions:** When a user interacts with the UI (e.g., clicks "Save"), the Page MUST call a global helper function inside `functions.dart`.
2. **Functions -> Database:** The helper function in `functions.dart` translates the request and executes the Drift query against `database/tables.dart`.
3. **Database -> Struct:** The database query results MUST be mapped to a pure model from `lib/struct/` before returning.
4. **Struct -> Pages:** The Page receives the Struct model (usually via `FutureBuilder` or state management) and renders it.

## 3. Mandatory Checklist for Agents
Before concluding any implementation task, verify:
- [ ] No `package:drift` imports exist inside `lib/pages/` or `lib/widgets/`.
- [ ] No SQL or Database logic exists inside `lib/pages/`.
- [ ] All database interactions happen via `lib/functions.dart`.
- [ ] UI components are maximally reused from `lib/widgets/`.

## 4. When to Use
Apply this skill constantly during implementation of:
- CRUD operations.
- Adding new UI screens.
- Modifying the database schema.
