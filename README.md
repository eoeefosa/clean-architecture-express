# clean-architecture-express

Implementation of a CRUD app using the **Clean Architecture** design pattern in **Express** and **TypeScript**. This solution ensures separation of concerns, allowing you to switch between different database implementations.

---

### Folder Structure
```
src/
├── domain/
│   ├── entities/
│   │   └── User.ts
│   ├── repositories/
│   │   └── IUserRepository.ts
├── infrastructure/
│   ├── database/
│   │   ├── MongoDBRepository.ts
│   │   ├── MySQLRepository.ts
│   │   ├── FileRepository.ts
│   │   └── JSONRepository.ts
├── application/
│   └── services/
│       └── UserService.ts
├── interfaces/
│   ├── controllers/
│   │   └── UserController.ts
│   └── routes/
│       └── UserRoutes.ts
├── app.ts
└── server.ts
```

---

### Implementation

#### 1. Domain Layer
**User Entity** (`src/domain/entities/User.ts`):
```typescript
export interface User {
  id: string;
  name: string;
  email: string;
}
```

**Repository Interface** (`src/domain/repositories/IUserRepository.ts`):
```typescript
import { User } from "../entities/User";

export interface IUserRepository {
  getAll(): Promise<User[]>;
  getById(id: string): Promise<User | null>;
  create(user: User): Promise<void>;
  update(id: string, user: Partial<User>): Promise<void>;
  delete(id: string): Promise<void>;
}
```

#### 2. Infrastructure Layer
**MongoDB Repository** (`src/infrastructure/database/MongoDBRepository.ts`):
```typescript
import { IUserRepository } from "../../domain/repositories/IUserRepository";
import { User } from "../../domain/entities/User";

export class MongoDBRepository implements IUserRepository {
  async getAll(): Promise<User[]> {
    // MongoDB implementation
    return [];
  }

  async getById(id: string): Promise<User | null> {
    return null;
  }

  async create(user: User): Promise<void> {}

  async update(id: string, user: Partial<User>): Promise<void> {}

  async delete(id: string): Promise<void> {}
}
```

Similarly, you can create `MySQLRepository.ts`, `JSONRepository.ts`, etc., with the same interface.

#### 3. Application Layer
**User Service** (`src/application/services/UserService.ts`):
```typescript
import { IUserRepository } from "../../domain/repositories/IUserRepository";
import { User } from "../../domain/entities/User";

export class UserService {
  constructor(private repository: IUserRepository) {}

  async getAllUsers(): Promise<User[]> {
    return this.repository.getAll();
  }

  async getUserById(id: string): Promise<User | null> {
    return this.repository.getById(id);
  }

  async createUser(user: User): Promise<void> {
    await this.repository.create(user);
  }

  async updateUser(id: string, user: Partial<User>): Promise<void> {
    await this.repository.update(id, user);
  }

  async deleteUser(id: string): Promise<void> {
    await this.repository.delete(id);
  }
}
```

#### 4. Interface Layer
**User Controller** (`src/interfaces/controllers/UserController.ts`):
```typescript
import { Request, Response } from "express";
import { UserService } from "../../application/services/UserService";

export class UserController {
  constructor(private userService: UserService) {}

  async getAll(req: Request, res: Response) {
    const users = await this.userService.getAllUsers();
    res.json(users);
  }

  async getById(req: Request, res: Response) {
    const user = await this.userService.getUserById(req.params.id);
    if (!user) return res.status(404).send("User not found");
    res.json(user);
  }

  async create(req: Request, res: Response) {
    await this.userService.createUser(req.body);
    res.status(201).send("User created");
  }

  async update(req: Request, res: Response) {
    await this.userService.updateUser(req.params.id, req.body);
    res.send("User updated");
  }

  async delete(req: Request, res: Response) {
    await this.userService.deleteUser(req.params.id);
    res.send("User deleted");
  }
}
```

**Routes** (`src/interfaces/routes/UserRoutes.ts`):
```typescript
import { Router } from "express";
import { UserController } from "../controllers/UserController";

const router = Router();

export const userRoutes = (userController: UserController) => {
  router.get("/", (req, res) => userController.getAll(req, res));
  router.get("/:id", (req, res) => userController.getById(req, res));
  router.post("/", (req, res) => userController.create(req, res));
  router.put("/:id", (req, res) => userController.update(req, res));
  router.delete("/:id", (req, res) => userController.delete(req, res));
  return router;
};
```

#### 5. Entry Point
**App Initialization** (`src/app.ts`):
```typescript
import express from "express";
import { MongoDBRepository } from "./infrastructure/database/MongoDBRepository";
import { UserService } from "./application/services/UserService";
import { UserController } from "./interfaces/controllers/UserController";
import { userRoutes } from "./interfaces/routes/UserRoutes";

const app = express();
app.use(express.json());

// Dependency Injection
const userRepository = new MongoDBRepository();
const userService = new UserService(userRepository);
const userController = new UserController(userService);

// Routes
app.use("/users", userRoutes(userController));

export default app;
```

**Server Start** (`src/server.ts`):
```typescript
import app from "./app";

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

---

### Key Points
- **Separation of Concerns**: Each layer is isolated, adhering to Single Responsibility Principle.
- **Interchangeable Databases**: Replace the `MongoDBRepository` with `MySQLRepository`, `FileRepository`, or `JSONRepository` without modifying other layers.
- **Scalable and Maintainable**: Clean Architecture ensures your app remains easy to maintain and extend.

