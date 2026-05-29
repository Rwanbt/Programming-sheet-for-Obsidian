# TypeScript Node.js et Backend

> [!tip] Analogie
> Imagine que tu construis une maison. Node.js avec JavaScript, c'est construire sans plan : tu poses les briques en esperant que ca tient. TypeScript cote serveur, c'est le meme chantier mais avec des plans d'architecte, des tolerances definies, et un chef de projet qui crie si tu essaies de poser une fenetre la ou doit aller une porte. Le batiment final est identique — mais tu n'as pas eu de surprise a la livraison.

Tu connais deja Node.js et Express. Tu as probablement ecrit des routes qui recevaient `req.body` sans savoir exactement ce qu'il y avait dedans. Tu as eu des `undefined` mysterieux a l'execution. Tu as ecrit des fonctions qui acceptaient `any` parce que tu n'avais pas le temps de typer.

Cette note t'explique comment TypeScript transforme le developpement backend : depuis la configuration initiale jusqu'a une API REST complete, en passant par Prisma, la gestion des erreurs et les tests. Le tout avec des exemples concrets, pas des abstractions.

Voir aussi : [[TypeScript/01 - Introduction a TypeScript]] et [[TypeScript/02 - Types Avances]] pour les fondamentaux.

---

## Pourquoi TypeScript cote serveur ?

### Le probleme concret du backend JavaScript

Cote frontend, une erreur de type provoque un bouton qui ne repond pas — l'utilisateur voit le bug. Cote backend, une erreur de type peut corrompre des donnees en base, retourner un 500 sans log exploitable, ou pire : retourner silencieusement des donnees incorrectes que personne ne remarque pendant des semaines.

```
FRONTEND              BACKEND
---------             --------
Erreur type  →  UI    Erreur type  →  Donnees corrompues
cassee             →  Perte de donnees
Visible tout        →  Silencieuse
de suite            →  Difficile a debugger
```

### Ce que TypeScript apporte au serveur

| Probleme JS pur | Solution TypeScript |
|---|---|
| `req.body` est `any`, tu peux lire n'importe quoi | Types sur les DTOs — l'IDE te dit ce qui existe |
| Refactoring casse silencieusement des routes | Le compilateur detecte tous les call sites |
| Les fonctions async retournent `Promise<any>` | `Promise<User>` — tu sais ce que tu attends |
| Les variables d'env peuvent etre `undefined` | Validation au demarrage avec types derives |
| Les erreurs sont des `any` dans les catch | Classes d'erreur typees avec codes structurees |
| L'ORM retourne des objets opaques | Prisma genere les types automatiquement |

> [!info] TypeScript ne ralentit pas ton serveur
> Le code TypeScript est compile en JavaScript pur avant execution. A l'execution, ton serveur Node.js tourne du JS standard — TypeScript n'existe plus. Le gain est entierement au moment du developpement et du build.

---

## 1. Setup Node + TypeScript

### Installation de base

```bash
mkdir mon-api
cd mon-api
npm init -y
npm install --save-dev typescript ts-node @types/node
```

`ts-node` permet d'executer directement des fichiers `.ts` sans compiler manuellement — indispensable en developpement.

### tsconfig.json pour Node.js

La configuration TypeScript doit etre adaptee a Node, pas au navigateur. Les cibles, modules et options sont differents.

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "CommonJS",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

> [!warning] `module: "CommonJS"` vs `"ESM"`
> Node.js supporte les modules ES (`.mjs`, `"type": "module"` dans package.json), mais CommonJS reste le standard dans la majorite des projets backend existants. Si tu pars de zero en 2024+, tu peux utiliser `"module": "NodeNext"` avec `"type": "module"` dans package.json. Mais CommonJS est plus compatible avec l'ecosysteme npm actuel.

**Options importantes expliquees :**

| Option | Valeur | Pourquoi |
|---|---|---|
| `target` | `ES2022` | Node 18+ supporte ES2022 nativement |
| `module` | `CommonJS` | Format de modules Node.js standard |
| `strict` | `true` | Active tous les checks stricts — ne jamais mettre false |
| `esModuleInterop` | `true` | Permet `import express from 'express'` au lieu de `import * as express` |
| `resolveJsonModule` | `true` | Permet `import config from './config.json'` |
| `sourceMap` | `true` | Debugger pointe sur le .ts, pas le .js compile |

### Scripts package.json

```json
{
  "scripts": {
    "dev": "ts-node --watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "jest"
  }
}
```

Avec `tsx` (alternative plus rapide a `ts-node`) :

```bash
npm install --save-dev tsx
```

```json
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc --noEmit && esbuild src/index.ts --bundle --platform=node --outfile=dist/index.js"
  }
}
```

### tsx vs ts-node : quelle difference ?

```
ts-node          tsx
--------         --------
Utilise tsc      Utilise esbuild en interne
Plus lent        Tres rapide (10-50x)
Respecte         Ne verifie pas les types
tsconfig strict  (verification separee)
Debug maps       Debug maps
Mature           Moderne, recommande
```

> [!tip] Workflow recommande en 2024
> Utilise `tsx watch` pour le hot-reload en developpement (rapide), et `tsc --noEmit` dans un script separe ou en pre-commit pour la verification des types. Ca te donne le meilleur des deux mondes.

---

## 2. @types/node et les modules Node

### Pourquoi @types/node existe

Node.js est ecrit en JavaScript. Ses API (`fs`, `path`, `http`, `crypto`...) n'ont pas de types TypeScript natifs. Le package `@types/node` est un fichier de declaration (`.d.ts`) qui decrit toutes ces API a TypeScript.

```bash
npm install --save-dev @types/node
```

Sans ca, TypeScript ne connait pas `process`, `__dirname`, `Buffer`, ni aucun module Node.

### Utilisation des modules Node types

```typescript
import * as fs from 'fs/promises';
import * as path from 'path';
import { EventEmitter } from 'events';
import { IncomingMessage, ServerResponse } from 'http';

// fs.readFile retourne Promise<Buffer> — TypeScript le sait
async function lireConfig(chemin: string): Promise<string> {
  const contenu = await fs.readFile(chemin, 'utf-8');
  // contenu est string ici, pas Buffer, grace au second argument
  return contenu;
}

// path.join retourne string — pas de surprise
const cheminAbsolu = path.join(__dirname, 'config', 'default.json');

// process.env est Record<string, string | undefined>
// TypeScript t'oblige a gerer le cas undefined
const port = process.env.PORT; // type: string | undefined
const portNumber = parseInt(process.env.PORT ?? '3000', 10);
```

### Buffer et streams types

```typescript
import { Readable, Writable, Transform } from 'stream';
import { pipeline } from 'stream/promises';

// Transform stream type — entree string, sortie Buffer
class UpperCaseTransform extends Transform {
  _transform(
    chunk: Buffer | string,
    encoding: BufferEncoding,
    callback: (error?: Error | null) => void
  ): void {
    const texte = chunk.toString().toUpperCase();
    this.push(texte);
    callback();
  }
}

// EventEmitter avec types custom
interface MesEvenements {
  data: [payload: string];
  error: [err: Error];
  close: [];
}

class MonEmitter extends EventEmitter {
  on<K extends keyof MesEvenements>(
    event: K,
    listener: (...args: MesEvenements[K]) => void
  ): this {
    return super.on(event, listener);
  }
}
```

---

## 3. Express + TypeScript

### Installation

```bash
npm install express
npm install --save-dev @types/express
```

### Les types fondamentaux d'Express

```typescript
import express, { Request, Response, NextFunction, Application } from 'express';

const app: Application = express();
app.use(express.json());

// Route basique typee
app.get('/health', (req: Request, res: Response): void => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});
```

> [!info] Tu n'as pas toujours besoin d'annoter req et res
> TypeScript infere les types depuis `app.get()`, `app.post()` etc. L'annotation explicite `(req: Request, res: Response)` est optionnelle mais utile pour la lisibilite dans des fonctions extraites.

### Generiques sur Request

C'est la partie la plus puissante. `Request` est un type generique avec quatre parametres :

```
Request<Params, ResBody, ReqBody, Query>
         |       |        |        |
         |       |        |        └─ req.query
         |       |        └─ req.body
         |       └─ type du corps de reponse (rarement utilise)
         └─ req.params
```

```typescript
import { Request, Response } from 'express';

// Types pour une route GET /users/:id
interface UserParams {
  id: string;
}

// Types pour une route POST /users
interface CreateUserBody {
  name: string;
  email: string;
  age?: number;
}

// Types pour les query params GET /users?page=1&limit=10
interface UserQuery {
  page?: string;
  limit?: string;
  search?: string;
}

// Route avec params types
app.get('/users/:id', (
  req: Request<UserParams>,
  res: Response
): void => {
  const { id } = req.params; // id: string — TypeScript le sait
  res.json({ id });
});

// Route avec body type
app.post('/users', (
  req: Request<{}, {}, CreateUserBody>,
  res: Response
): void => {
  const { name, email, age } = req.body;
  // name: string, email: string, age: number | undefined
  res.status(201).json({ name, email, age });
});

// Route avec query params types
app.get('/users', (
  req: Request<{}, {}, {}, UserQuery>,
  res: Response
): void => {
  const page = parseInt(req.query.page ?? '1', 10);
  const limit = parseInt(req.query.limit ?? '20', 10);
  res.json({ page, limit });
});
```

> [!warning] req.body n'est pas valide a l'execution
> Le type `Request<{}, {}, CreateUserBody>` dit a TypeScript "fais confiance a ce type", mais a l'execution il n'y a aucune garantie. Un client malveillant peut envoyer n'importe quoi. Toujours valider avec un schema (Zod, Joi) avant d'utiliser `req.body`. La section 6 couvre ca.

---

## 4. Middlewares types

### Middleware basique

```typescript
import { Request, Response, NextFunction } from 'express';

// Middleware de logging
function loggerMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
): void {
  const debut = Date.now();
  console.log(`→ ${req.method} ${req.path}`);

  res.on('finish', () => {
    const duree = Date.now() - debut;
    console.log(`← ${req.method} ${req.path} ${res.statusCode} (${duree}ms)`);
  });

  next();
}

app.use(loggerMiddleware);
```

### Etendre le type Request

Un pattern tres courant : ajouter des proprietes sur `req` dans un middleware (utilisateur authentifie, permissions, etc.).

```typescript
// src/types/express.d.ts
import { User } from '../models/User';

// Augmentation du module express
declare global {
  namespace Express {
    interface Request {
      user?: User;
      requestId?: string;
    }
  }
}
```

```typescript
// src/middlewares/auth.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import { User } from '../models/User';

export function authMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
): void {
  const token = req.headers.authorization?.replace('Bearer ', '');

  if (!token) {
    res.status(401).json({ error: 'Token manquant' });
    return;
  }

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as User;
    req.user = payload; // Possible grace a l'augmentation de type
    next();
  } catch {
    res.status(401).json({ error: 'Token invalide' });
  }
}
```

> [!tip] Le fichier `.d.ts` doit etre dans `include`
> Pour que l'augmentation soit prise en compte, le fichier `src/types/express.d.ts` doit etre dans le scope de compilation (couvert par le pattern `src/**/*` du tsconfig).

### Middleware avec un type de retour explicite

```typescript
// Fabrique de middleware — retourne un RequestHandler
import { RequestHandler } from 'express';

function requireRole(role: string): RequestHandler {
  return (req, res, next) => {
    if (!req.user) {
      res.status(401).json({ error: 'Non authentifie' });
      return;
    }

    if (req.user.role !== role) {
      res.status(403).json({ error: 'Permission insuffisante' });
      return;
    }

    next();
  };
}

// Utilisation
app.delete('/admin/users/:id', authMiddleware, requireRole('admin'), deleteUser);
```

---

## 5. Gestion des erreurs typee

### Classe d'erreur custom

```typescript
// src/errors/AppError.ts

export type ErrorCode =
  | 'NOT_FOUND'
  | 'UNAUTHORIZED'
  | 'FORBIDDEN'
  | 'VALIDATION_ERROR'
  | 'CONFLICT'
  | 'INTERNAL_ERROR';

export class AppError extends Error {
  readonly statusCode: number;
  readonly code: ErrorCode;
  readonly isOperational: boolean;

  constructor(
    message: string,
    code: ErrorCode,
    statusCode: number,
    isOperational = true
  ) {
    super(message);
    this.name = 'AppError';
    this.code = code;
    this.statusCode = statusCode;
    this.isOperational = isOperational;

    // Necessaire pour que instanceof fonctionne avec les classes qui etendent Error
    Object.setPrototypeOf(this, AppError.prototype);
  }

  static notFound(resource: string): AppError {
    return new AppError(`${resource} introuvable`, 'NOT_FOUND', 404);
  }

  static unauthorized(message = 'Non authentifie'): AppError {
    return new AppError(message, 'UNAUTHORIZED', 401);
  }

  static forbidden(message = 'Acces refuse'): AppError {
    return new AppError(message, 'FORBIDDEN', 403);
  }

  static conflict(message: string): AppError {
    return new AppError(message, 'CONFLICT', 409);
  }

  static validation(message: string): AppError {
    return new AppError(message, 'VALIDATION_ERROR', 400);
  }

  static internal(message = 'Erreur interne'): AppError {
    return new AppError(message, 'INTERNAL_ERROR', 500, false);
  }
}
```

### Middleware de gestion d'erreurs global

```typescript
// src/middlewares/errorHandler.ts
import { Request, Response, NextFunction } from 'express';
import { AppError } from '../errors/AppError';

interface ErrorResponse {
  error: {
    code: string;
    message: string;
    details?: unknown;
  };
}

// Express reconnait un middleware d'erreur a ses 4 arguments
export function errorHandler(
  err: Error,
  req: Request,
  res: Response<ErrorResponse>,
  next: NextFunction  // Obligatoire meme si inutilise — Express verifie l'arite
): void {
  // Erreur applicative attendue
  if (err instanceof AppError) {
    res.status(err.statusCode).json({
      error: {
        code: err.code,
        message: err.message,
      },
    });
    return;
  }

  // Erreur inattendue — ne pas exposer les details en production
  console.error('Erreur non geree:', err);

  res.status(500).json({
    error: {
      code: 'INTERNAL_ERROR',
      message:
        process.env.NODE_ENV === 'development'
          ? err.message
          : 'Une erreur interne est survenue',
    },
  });
}
```

### Wrapper async pour capturer les erreurs

Sans wrapper, une erreur dans un handler async n'est pas automatiquement passee a `next()` :

```typescript
// src/utils/asyncHandler.ts
import { Request, Response, NextFunction, RequestHandler } from 'express';

type AsyncRequestHandler = (
  req: Request,
  res: Response,
  next: NextFunction
) => Promise<void>;

export function asyncHandler(fn: AsyncRequestHandler): RequestHandler {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}
```

```typescript
// Utilisation — plus de try/catch dans chaque route
app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await userService.findById(req.params.id);

  if (!user) {
    throw AppError.notFound('Utilisateur');
  }

  res.json(user);
}));
```

```
Requete
  |
  v
Handler async
  |
  |-- succes --> res.json(data)
  |
  |-- erreur --> catch(next) --> errorHandler middleware
                                    |
                                    |-- AppError --> 4xx avec code
                                    |-- Error    --> 500 (log + message generique)
```

---

## 6. Variables d'environnement typees

### Le probleme avec process.env

```typescript
// Sans typage
const port = process.env.PORT;         // string | undefined
const secret = process.env.JWT_SECRET; // string | undefined

// Tu dois gerer undefined a chaque acces
app.listen(parseInt(port!, 10)); // L'assertion ! est dangereuse
```

### Pattern avec validation manuelle

```typescript
// src/config/env.ts

interface Config {
  port: number;
  nodeEnv: 'development' | 'production' | 'test';
  database: {
    url: string;
  };
  jwt: {
    secret: string;
    expiresIn: string;
  };
}

function requireEnv(key: string): string {
  const value = process.env[key];
  if (!value) {
    throw new Error(`Variable d'environnement manquante : ${key}`);
  }
  return value;
}

function parsePort(key: string, defaultValue: number): number {
  const value = process.env[key];
  if (!value) return defaultValue;
  const parsed = parseInt(value, 10);
  if (isNaN(parsed)) {
    throw new Error(`${key} doit etre un nombre, recu: ${value}`);
  }
  return parsed;
}

export const config: Config = {
  port: parsePort('PORT', 3000),
  nodeEnv: (process.env.NODE_ENV as Config['nodeEnv']) ?? 'development',
  database: {
    url: requireEnv('DATABASE_URL'),
  },
  jwt: {
    secret: requireEnv('JWT_SECRET'),
    expiresIn: process.env.JWT_EXPIRES_IN ?? '7d',
  },
};
```

### Pattern avec Zod (recommande)

```bash
npm install zod
```

```typescript
// src/config/env.ts
import { z } from 'zod';

const envSchema = z.object({
  PORT: z.string().regex(/^\d+$/).transform(Number).default('3000'),
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32, 'JWT_SECRET doit faire au moins 32 caracteres'),
  JWT_EXPIRES_IN: z.string().default('7d'),
});

// parse() lance une exception descriptive si une variable est invalide
const env = envSchema.parse(process.env);

// Le type est infere automatiquement depuis le schema
export const config = {
  port: env.PORT,
  nodeEnv: env.NODE_ENV,
  database: { url: env.DATABASE_URL },
  jwt: {
    secret: env.JWT_SECRET,
    expiresIn: env.JWT_EXPIRES_IN,
  },
} as const;

// Type derive automatiquement
export type Config = typeof config;
```

> [!tip] Parser les variables au demarrage
> Appeler `envSchema.parse(process.env)` le plus tot possible dans `src/index.ts`, avant toute autre initialisation. Si une variable manque, le processus s'arrete immediatement avec un message clair — pas un crash mysterieux 5 minutes plus tard.

---

## 7. Prisma ORM + TypeScript

Prisma est l'ORM TypeScript le plus populaire pour Node.js. Il genere automatiquement des types depuis le schema de base de donnees.

### Installation et setup

```bash
npm install prisma @prisma/client
npx prisma init
```

### Schema Prisma

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  todos     Todo[]
}

model Todo {
  id          Int      @id @default(autoincrement())
  title       String
  description String?
  completed   Boolean  @default(false)
  userId      Int
  user        User     @relation(fields: [userId], references: [id])
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

```bash
npx prisma generate  # Genere les types dans node_modules/@prisma/client
npx prisma migrate dev --name init  # Cree la migration et la base
```

### Types auto-generes par Prisma

Apres `prisma generate`, tu as acces a des types complets :

```typescript
import { PrismaClient, User, Todo, Prisma } from '@prisma/client';

const prisma = new PrismaClient();

// User est le type exact de la table — mis a jour automatiquement
async function getUserById(id: number): Promise<User | null> {
  return prisma.user.findUnique({ where: { id } });
}

// Prisma.UserCreateInput — type pour creer un user
async function createUser(data: Prisma.UserCreateInput): Promise<User> {
  return prisma.user.create({ data });
}

// Prisma.TodoUpdateInput — type pour mettre a jour
async function updateTodo(
  id: number,
  data: Prisma.TodoUpdateInput
): Promise<Todo> {
  return prisma.todo.update({ where: { id }, data });
}
```

### Inclure les relations

```typescript
// Type pour un Todo avec son User inclus
type TodoWithUser = Prisma.TodoGetPayload<{
  include: { user: true };
}>;

async function getTodosWithUsers(): Promise<TodoWithUser[]> {
  return prisma.todo.findMany({
    include: { user: true },
  });
}

// Type pour un User avec ses Todos — sans email
type UserWithTodos = Prisma.UserGetPayload<{
  select: {
    id: true;
    name: true;
    todos: {
      select: {
        id: true;
        title: true;
        completed: true;
      };
    };
  };
}>;
```

> [!info] Prisma.XGetPayload
> Ce type utilitaire derive le type exact retourne par une requete Prisma avec `include` ou `select`. C'est une des killer features de Prisma — le type correspond precisement aux donnees recues, sans aucun champ en plus.

### Singleton PrismaClient

```typescript
// src/lib/prisma.ts
import { PrismaClient } from '@prisma/client';

// Evite de creer une nouvelle connexion a chaque hot-reload en dev
declare global {
  var prisma: PrismaClient | undefined;
}

export const prisma: PrismaClient =
  global.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query'] : [],
  });

if (process.env.NODE_ENV !== 'production') {
  global.prisma = prisma;
}
```

---

## 8. API REST complete : exemple Todo

Voici une API Todo complete, typee de A a Z, qui illustre tous les concepts vus.

### Structure du projet

```
src/
  config/
    env.ts
  errors/
    AppError.ts
  lib/
    prisma.ts
  middlewares/
    auth.ts
    errorHandler.ts
    asyncHandler.ts
  routes/
    todos.ts
    users.ts
  services/
    todoService.ts
    userService.ts
  types/
    express.d.ts
  index.ts
```

### DTOs (Data Transfer Objects)

```typescript
// src/types/todo.ts

export interface CreateTodoDto {
  title: string;
  description?: string;
}

export interface UpdateTodoDto {
  title?: string;
  description?: string;
  completed?: boolean;
}

export interface TodoParams {
  id: string;
}

export interface TodoQuery {
  completed?: string;
  page?: string;
  limit?: string;
}

export interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}
```

### Service Todo

```typescript
// src/services/todoService.ts
import { Todo, Prisma } from '@prisma/client';
import { prisma } from '../lib/prisma';
import { AppError } from '../errors/AppError';
import { CreateTodoDto, UpdateTodoDto, PaginatedResponse } from '../types/todo';

export class TodoService {
  async findAll(
    userId: number,
    page = 1,
    limit = 20,
    completed?: boolean
  ): Promise<PaginatedResponse<Todo>> {
    const where: Prisma.TodoWhereInput = {
      userId,
      ...(completed !== undefined && { completed }),
    };

    const [data, total] = await prisma.$transaction([
      prisma.todo.findMany({
        where,
        skip: (page - 1) * limit,
        take: limit,
        orderBy: { createdAt: 'desc' },
      }),
      prisma.todo.count({ where }),
    ]);

    return {
      data,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
      },
    };
  }

  async findById(id: number, userId: number): Promise<Todo> {
    const todo = await prisma.todo.findFirst({
      where: { id, userId },
    });

    if (!todo) {
      throw AppError.notFound('Todo');
    }

    return todo;
  }

  async create(data: CreateTodoDto, userId: number): Promise<Todo> {
    return prisma.todo.create({
      data: {
        ...data,
        userId,
      },
    });
  }

  async update(id: number, data: UpdateTodoDto, userId: number): Promise<Todo> {
    await this.findById(id, userId); // Verifie l'existence et l'ownership

    return prisma.todo.update({
      where: { id },
      data,
    });
  }

  async delete(id: number, userId: number): Promise<void> {
    await this.findById(id, userId);
    await prisma.todo.delete({ where: { id } });
  }

  async toggleComplete(id: number, userId: number): Promise<Todo> {
    const todo = await this.findById(id, userId);

    return prisma.todo.update({
      where: { id },
      data: { completed: !todo.completed },
    });
  }
}

export const todoService = new TodoService();
```

### Router Todo

```typescript
// src/routes/todos.ts
import { Router, Request, Response } from 'express';
import { asyncHandler } from '../middlewares/asyncHandler';
import { authMiddleware } from '../middlewares/auth';
import { todoService } from '../services/todoService';
import { AppError } from '../errors/AppError';
import {
  CreateTodoDto,
  UpdateTodoDto,
  TodoParams,
  TodoQuery,
} from '../types/todo';

const router = Router();

// Toutes les routes Todo necessitent l'authentification
router.use(authMiddleware);

// GET /todos
router.get(
  '/',
  asyncHandler(async (
    req: Request<{}, {}, {}, TodoQuery>,
    res: Response
  ) => {
    const page = parseInt(req.query.page ?? '1', 10);
    const limit = Math.min(parseInt(req.query.limit ?? '20', 10), 100);
    const completed =
      req.query.completed !== undefined
        ? req.query.completed === 'true'
        : undefined;

    const result = await todoService.findAll(req.user!.id, page, limit, completed);
    res.json(result);
  })
);

// GET /todos/:id
router.get(
  '/:id',
  asyncHandler(async (req: Request<TodoParams>, res: Response) => {
    const id = parseInt(req.params.id, 10);

    if (isNaN(id)) {
      throw AppError.validation('ID invalide');
    }

    const todo = await todoService.findById(id, req.user!.id);
    res.json(todo);
  })
);

// POST /todos
router.post(
  '/',
  asyncHandler(async (
    req: Request<{}, {}, CreateTodoDto>,
    res: Response
  ) => {
    const { title, description } = req.body;

    if (!title || typeof title !== 'string' || title.trim() === '') {
      throw AppError.validation('Le titre est requis');
    }

    const todo = await todoService.create(
      { title: title.trim(), description },
      req.user!.id
    );

    res.status(201).json(todo);
  })
);

// PATCH /todos/:id
router.patch(
  '/:id',
  asyncHandler(async (
    req: Request<TodoParams, {}, UpdateTodoDto>,
    res: Response
  ) => {
    const id = parseInt(req.params.id, 10);

    if (isNaN(id)) {
      throw AppError.validation('ID invalide');
    }

    const todo = await todoService.update(id, req.body, req.user!.id);
    res.json(todo);
  })
);

// PATCH /todos/:id/toggle
router.patch(
  '/:id/toggle',
  asyncHandler(async (req: Request<TodoParams>, res: Response) => {
    const id = parseInt(req.params.id, 10);
    const todo = await todoService.toggleComplete(id, req.user!.id);
    res.json(todo);
  })
);

// DELETE /todos/:id
router.delete(
  '/:id',
  asyncHandler(async (req: Request<TodoParams>, res: Response) => {
    const id = parseInt(req.params.id, 10);
    await todoService.delete(id, req.user!.id);
    res.status(204).send();
  })
);

export default router;
```

### Point d'entree index.ts

```typescript
// src/index.ts
import express from 'express';
import { config } from './config/env';
import todoRouter from './routes/todos';
import { errorHandler } from './middlewares/errorHandler';

const app = express();

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.get('/health', (req, res) => {
  res.json({ status: 'ok', env: config.nodeEnv });
});
app.use('/api/todos', todoRouter);

// Middleware d'erreur — doit etre le dernier use()
app.use(errorHandler);

async function bootstrap(): Promise<void> {
  app.listen(config.port, () => {
    console.log(`Serveur demarre sur le port ${config.port}`);
    console.log(`Environnement : ${config.nodeEnv}`);
  });
}

bootstrap().catch((err) => {
  console.error('Echec au demarrage:', err);
  process.exit(1);
});
```

---

## 9. Tests avec Jest + TypeScript

### Installation

```bash
npm install --save-dev jest ts-jest @types/jest
```

### Configuration

```javascript
// jest.config.js
/** @type {import('ts-jest').JestConfigWithTsJest} */
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/*.test.ts', '**/*.spec.ts'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  coverageDirectory: 'coverage',
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/index.ts',
  ],
};
```

### Tester un service

```typescript
// src/services/todoService.test.ts
import { TodoService } from './todoService';
import { prisma } from '../lib/prisma';
import { AppError } from '../errors/AppError';

// Mock du client Prisma
jest.mock('../lib/prisma', () => ({
  prisma: {
    todo: {
      findFirst: jest.fn(),
      findMany: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
      count: jest.fn(),
    },
    $transaction: jest.fn(),
  },
}));

const mockPrisma = prisma as jest.Mocked<typeof prisma>;

describe('TodoService', () => {
  let service: TodoService;

  beforeEach(() => {
    service = new TodoService();
    jest.clearAllMocks();
  });

  describe('findById', () => {
    it('retourne le todo si trouve', async () => {
      const mockTodo = {
        id: 1,
        title: 'Test',
        description: null,
        completed: false,
        userId: 1,
        createdAt: new Date(),
        updatedAt: new Date(),
      };

      (mockPrisma.todo.findFirst as jest.Mock).mockResolvedValue(mockTodo);

      const result = await service.findById(1, 1);
      expect(result).toEqual(mockTodo);
      expect(mockPrisma.todo.findFirst).toHaveBeenCalledWith({
        where: { id: 1, userId: 1 },
      });
    });

    it('lance AppError.NOT_FOUND si le todo n\'existe pas', async () => {
      (mockPrisma.todo.findFirst as jest.Mock).mockResolvedValue(null);

      await expect(service.findById(99, 1)).rejects.toMatchObject({
        code: 'NOT_FOUND',
        statusCode: 404,
      });
    });
  });

  describe('create', () => {
    it('cree un todo et le retourne', async () => {
      const input = { title: 'Mon todo', description: 'Details' };
      const created = { id: 1, ...input, completed: false, userId: 1,
        createdAt: new Date(), updatedAt: new Date() };

      (mockPrisma.todo.create as jest.Mock).mockResolvedValue(created);

      const result = await service.create(input, 1);
      expect(result).toEqual(created);
      expect(mockPrisma.todo.create).toHaveBeenCalledWith({
        data: { ...input, userId: 1 },
      });
    });
  });
});
```

### Tester les routes avec supertest

```bash
npm install --save-dev supertest @types/supertest
```

```typescript
// src/routes/todos.test.ts
import request from 'supertest';
import express from 'express';
import todoRouter from './todos';
import { errorHandler } from '../middlewares/errorHandler';
import { todoService } from '../services/todoService';

// Mock du service et du middleware auth
jest.mock('../services/todoService');
jest.mock('../middlewares/auth', () => ({
  authMiddleware: (req: any, res: any, next: any) => {
    req.user = { id: 1, email: 'test@example.com', role: 'user' };
    next();
  },
}));

const mockTodoService = todoService as jest.Mocked<typeof todoService>;

const app = express();
app.use(express.json());
app.use('/todos', todoRouter);
app.use(errorHandler);

describe('GET /todos', () => {
  it('retourne une liste paginee', async () => {
    const mockResult = {
      data: [{ id: 1, title: 'Test', completed: false, userId: 1,
        description: null, createdAt: new Date(), updatedAt: new Date() }],
      pagination: { page: 1, limit: 20, total: 1, totalPages: 1 },
    };

    mockTodoService.findAll.mockResolvedValue(mockResult);

    const response = await request(app)
      .get('/todos')
      .expect(200);

    expect(response.body.pagination.total).toBe(1);
    expect(response.body.data).toHaveLength(1);
  });
});

describe('POST /todos', () => {
  it('retourne 400 si le titre est vide', async () => {
    const response = await request(app)
      .post('/todos')
      .send({ title: '' })
      .expect(400);

    expect(response.body.error.code).toBe('VALIDATION_ERROR');
  });
});
```

> [!example] Organisation des tests
> Mettre les tests a cote des fichiers sources (`service.test.ts` pres de `service.ts`) plutot que dans un dossier `__tests__` a la racine. Ca facilite la navigation et le refactoring — si tu deplacez un module, son test suit automatiquement.

---

## 10. Build pour la production

### Option 1 : tsc (natif)

La methode standard — utilise directement le compilateur TypeScript.

```bash
# Build
npx tsc

# Resultat dans dist/
node dist/index.js
```

**Avantages** : officiel, genere les `.d.ts`, source maps natifs.
**Inconvenients** : lent sur les gros projets, ne bundle pas.

### Option 2 : esbuild (recommande)

```bash
npm install --save-dev esbuild
```

```json
{
  "scripts": {
    "build": "esbuild src/index.ts --bundle --platform=node --target=node18 --outfile=dist/index.js --external:@prisma/client",
    "build:check": "tsc --noEmit",
    "build:prod": "npm run build:check && npm run build"
  }
}
```

> [!warning] esbuild ne verifie pas les types
> `esbuild` transforme le TypeScript en JS ultra-rapidement mais ne verifie jamais les types. Toujours coupler avec `tsc --noEmit` dans le pipeline de build. Voir l'option `build:prod` ci-dessus.

**Avantages** : 10-100x plus rapide que tsc, bundle tout en un seul fichier.
**Inconvenients** : ne genere pas les types, external dependencies a gerer manuellement.

### Option 3 : SWC

```bash
npm install --save-dev @swc/core @swc/cli
```

```json
{
  ".swcrc": {
    "jsc": {
      "parser": {
        "syntax": "typescript",
        "decorators": true
      },
      "target": "es2022"
    },
    "module": {
      "type": "commonjs"
    }
  }
}
```

SWC est utilise par ts-node en mode `transpileOnly` et par Next.js. Aussi rapide qu'esbuild, sans bundling.

### Comparatif des options de build

```
                 tsc       esbuild     swc
---------       ------     -------    ------
Verification     OUI        NON        NON
des types

Vitesse         Lente      Tres       Tres
                           rapide     rapide

Bundle          NON        OUI        NON
(single file)

Decorators      OUI        Partiel    OUI
(NestJS)

.d.ts           OUI        NON        NON
generes

Source maps     OUI        OUI        OUI

Recommande      Prod        Prod       ts-node
pour            (types)     (apps)     alternatif
```

### Dockerfile pour production

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build:prod

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

---

## Patterns avances

### Repository pattern avec generiques

```typescript
// src/repositories/BaseRepository.ts
import { PrismaClient } from '@prisma/client';

// T = modele Prisma, CreateInput = type de creation
export abstract class BaseRepository<T, CreateInput, UpdateInput> {
  constructor(
    protected prisma: PrismaClient,
    // Nom de la "table" Prisma (ex: 'todo', 'user')
    protected model: keyof PrismaClient
  ) {}

  // Implementation specifique dans les sous-classes
  abstract findById(id: number): Promise<T | null>;
  abstract create(data: CreateInput): Promise<T>;
  abstract update(id: number, data: UpdateInput): Promise<T>;
  abstract delete(id: number): Promise<void>;
}
```

### Type guards pour les erreurs

```typescript
// Verifier qu'une erreur est une AppError
function isAppError(error: unknown): error is AppError {
  return error instanceof AppError;
}

// Verifier qu'une erreur est une erreur Prisma de contrainte unique
import { PrismaClientKnownRequestError } from '@prisma/client/runtime/library';

function isPrismaUniqueError(error: unknown): error is PrismaClientKnownRequestError {
  return (
    error instanceof PrismaClientKnownRequestError &&
    error.code === 'P2002'
  );
}

// Utilisation dans le service
async function createUser(data: CreateUserDto): Promise<User> {
  try {
    return await prisma.user.create({ data });
  } catch (error) {
    if (isPrismaUniqueError(error)) {
      throw AppError.conflict('Un utilisateur avec cet email existe deja');
    }
    throw error;
  }
}
```

### Validation avec Zod dans les routes

```typescript
// src/schemas/todo.schema.ts
import { z } from 'zod';

export const createTodoSchema = z.object({
  title: z.string().min(1, 'Le titre ne peut pas etre vide').max(255),
  description: z.string().max(1000).optional(),
});

export const updateTodoSchema = createTodoSchema.partial();

export type CreateTodoInput = z.infer<typeof createTodoSchema>;
export type UpdateTodoInput = z.infer<typeof updateTodoSchema>;
```

```typescript
// Middleware de validation generique
import { ZodSchema } from 'zod';

function validateBody<T>(schema: ZodSchema<T>): RequestHandler {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);

    if (!result.success) {
      res.status(400).json({
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Donnees invalides',
          details: result.error.flatten().fieldErrors,
        },
      });
      return;
    }

    req.body = result.data; // Remplace req.body par les donnees validees et parsees
    next();
  };
}

// Utilisation
router.post(
  '/',
  validateBody(createTodoSchema),
  asyncHandler(async (req: Request<{}, {}, CreateTodoInput>, res: Response) => {
    // req.body est maintenant CreateTodoInput valide — garanti par Zod
    const todo = await todoService.create(req.body, req.user!.id);
    res.status(201).json(todo);
  })
);
```

---

## Carte Mentale

```
TYPESCRIPT BACKEND — VUE D'ENSEMBLE
=====================================

SETUP
├── tsconfig.json (target ES2022, module CommonJS, strict: true)
├── ts-node / tsx (execution dev)
├── @types/node (types des modules Node)
└── scripts: dev/build/start/test

EXPRESS + TYPES
├── Request<Params, ResBody, ReqBody, Query>
├── Response, NextFunction, RequestHandler
├── Augmentation declare global namespace Express
└── asyncHandler() — wrapper catch vers next()

ERREURS
├── AppError extends Error
│   ├── statusCode, code: ErrorCode, isOperational
│   └── static factories: notFound, unauthorized, validation...
└── errorHandler middleware (4 args — doit etre le dernier)

ENV
├── Pattern manuel — requireEnv() + parsePort()
└── Zod schema — parse() au demarrage, types derives

PRISMA
├── npx prisma generate — types auto depuis schema
├── Prisma.XCreateInput, Prisma.XUpdateInput
├── Prisma.XGetPayload<{ include/select }> — types requetes
└── Singleton PrismaClient — global.prisma en dev

API REST COMPLETE
├── DTOs types (CreateTodoDto, UpdateTodoDto)
├── Service (logique metier, AppError)
├── Router (validation, asyncHandler)
└── index.ts (bootstrap, errorHandler en dernier)

TESTS
├── jest + ts-jest (config jest.config.js)
├── jest.mock() pour prisma et services
└── supertest pour les routes HTTP

BUILD PROD
├── tsc — verifie les types, genere .d.ts
├── esbuild — bundle rapide, pas de check types
└── Pattern recommande: tsc --noEmit && esbuild
```

---

## Exercices Pratiques

### Exercice 1 — API User avec auth JWT

**Objectif** : Cree une API avec inscription, connexion et profil protege.

**Ce que tu dois implementer :**

1. `POST /auth/register` — cree un user, hache le mot de passe avec `bcrypt`, retourne un JWT
2. `POST /auth/login` — verifie email/mot de passe, retourne un JWT
3. `GET /auth/me` — route protegee, retourne l'utilisateur connecte
4. Middleware `authMiddleware` qui verifie le JWT et met `req.user`

**Types a creer :**
```typescript
interface RegisterDto { email: string; name: string; password: string; }
interface LoginDto { email: string; password: string; }
interface AuthResponse { token: string; user: Omit<User, 'password'>; }
```

**Challenge** : Utilise `Omit<User, 'password'>` pour ne jamais retourner le hash.

---

### Exercice 2 — Validation Zod complete

**Objectif** : Ajoute une validation Zod sur toutes les routes de l'API Todo.

**Schemas a creer :**
- `createTodoSchema` — title requis max 255 chars, description optionnelle max 1000
- `updateTodoSchema` — meme champs mais tous optionnels
- `todoParamsSchema` — id : string qui parse en nombre entier positif
- `todoQuerySchema` — page/limit en nombres avec min/max, completed en boolean

**Challenge** : Cree un middleware `validateParams(schema)` similaire a `validateBody(schema)` et applique-le sur les routes `/:id`.

---

### Exercice 3 — Tests d'integration avec une DB en memoire

**Objectif** : Ecrire des tests qui utilisent une vraie base de donnees SQLite en memoire.

**Setup :**
```bash
npm install --save-dev @prisma/client
# Changer provider en "sqlite" dans schema.prisma pour les tests
```

```typescript
// jest.setup.ts
import { execSync } from 'child_process';

beforeAll(() => {
  process.env.DATABASE_URL = 'file::memory:?cache=shared';
  execSync('npx prisma db push --force-reset');
});
```

**Tests a ecrire :**
1. Creer un utilisateur et un todo
2. Filtrer les todos par `completed`
3. Verifier que supprimer un todo d'un autre user renvoie 404
4. Verifier que la pagination retourne le bon `totalPages`

**Challenge** : Nettoyer la base entre chaque test avec `beforeEach` qui supprime tous les records.

---

### Exercice 4 — Pipeline de build production

**Objectif** : Configurer un pipeline de build complet avec check de types, esbuild, et Docker.

**Etapes :**

1. Ajoute ces scripts dans `package.json` :
   - `typecheck` — `tsc --noEmit`
   - `build` — `esbuild src/index.ts --bundle --platform=node --outfile=dist/index.js --external:@prisma/client`
   - `build:prod` — `npm run typecheck && npm run build`

2. Configure un `Dockerfile` multi-stage (builder + runner)

3. Ajoute un fichier `.dockerignore` :
   ```
   node_modules
   dist
   .env
   *.test.ts
   ```

4. Verifie que `docker build -t mon-api .` reussit et que `docker run -p 3000:3000 mon-api` demarre.

**Challenge** : Mesure le temps de build avec `esbuild` seul vs `tsc` seul. Avec un projet moyen (50 fichiers), la difference est souvent superieure a 10x.

---

> [!info] Prochaine etape
> Une fois a l'aise avec ces patterns, explore [[Node Express/01 - Express.js et NestJS]] pour voir comment NestJS pousse TypeScript encore plus loin avec les decorateurs, l'injection de dependances et la generation automatique de documentation Swagger. Beaucoup de patterns de cette note (DTOs, services, validation) se retrouvent directement dans NestJS.

---

*Notes connexes : [[TypeScript/01 - Introduction a TypeScript]] — [[TypeScript/02 - Types Avances]] — [[Node Express/01 - Express.js et NestJS]]*
