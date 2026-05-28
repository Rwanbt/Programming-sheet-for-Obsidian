# Express.js et NestJS

Ce cours couvre la construction de serveurs web et d'APIs REST avec Node.js, d'abord avec Express.js (minimaliste, flexible, omnipresent) puis avec NestJS (structure opiniate, TypeScript-first, inspire d'Angular). Les deux solutions dominent l'ecosysteme Node.js en production.

Voir aussi : [[08 - APIs REST avec Flask]] pour la comparaison avec l'approche Python, et [[04 - CI-CD avec GitHub Actions]] pour automatiser les tests et deploiements.

> [!tip] Express vs NestJS — Le bon choix
> **Express** : liberte totale, parfait pour des APIs simples, des microservices, ou quand vous voulez construire votre propre architecture. Courbe d'apprentissage quasi nulle.
> **NestJS** : structure imposee, ideal pour des applications d'entreprise de grande taille ou plusieurs developpeurs doivent cohabiter avec des conventions claires. Courbe d'apprentissage moderee.

---

## 1. Node.js — Rappels Essentiels

### 1.1 L'event loop

Node.js est **single-threaded** mais gere la concurrence via l'event loop et les callbacks asynchrones.

```
                    ┌───────────────────────────┐
                    │         Event Loop         │
                    │                           │
                ┌───▼────┐                      │
Timers          │ timers │ setTimeout, setInterval
                └───┬────┘                      │
                ┌───▼────────┐                  │
I/O callbacks   │  pending   │ callbacks des ops │
                │  callbacks │ I/O precedentes   │
                └───┬────────┘                  │
                ┌───▼───┐                       │
                │  idle │ usage interne Node.js  │
                └───┬───┘                       │
                ┌───▼────┐                      │
I/O polling     │  poll  │ lit les evenements I/O│
                └───┬────┘ (fs, net, etc.)      │
                ┌───▼────┐                      │
                │ check  │ setImmediate           │
                └───┬────┘                      │
                ┌───▼──────────┐                │
                │ close events │ socket.on('close')
                └──────────────┘                │
                        │                       │
                        └───────────────────────┘
```

> [!warning] Ne jamais bloquer l'event loop
> Des operations CPU-intensives (cryptographie lourde, traitement d'images) bloquent tout le serveur pour tous les clients. Deleguer au Worker Threads ou a des services specialises. Les operations I/O (lecture fichier, requete BDD) sont non-bloquantes par nature.

### 1.2 CommonJS vs ES Modules

```javascript
// CommonJS (ancien standard Node.js, extension .js ou .cjs)
const express = require('express')
const path = require('path')
const { readFile } = require('fs/promises')

module.exports = { maFonction }
module.exports.autreExport = valeur

// ES Modules (moderne, extension .mjs ou avec "type": "module" dans package.json)
import express from 'express'
import path from 'path'
import { readFile } from 'fs/promises'
import monModule, { maFonction } from './mon-module.js'

export function maFonction() { ... }
export default class MonService { ... }
```

```json
// package.json
{
  "name": "mon-api",
  "version": "1.0.0",
  "type": "module",      // "module" pour ESM, absent ou "commonjs" pour CJS
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js",
    "test": "vitest"
  }
}
```

### 1.3 npm — Commandes essentielles

```bash
npm init -y                     # Initialiser package.json
npm install express             # Installer en dependance production
npm install -D nodemon vitest   # Installer en devDependency
npm install                     # Installer depuis package.json
npm update                      # Mettre a jour les dependances
npm audit                       # Scanner les vulnerabilites
npm run start                   # Lancer un script defini
npx                             # Executer un package sans l'installer
```

---

## 2. Express.js

### 2.1 Installation et premier serveur

```bash
npm init -y
npm install express
npm install -D nodemon
```

```javascript
// src/index.js
import express from 'express'

const app = express()
const PORT = process.env.PORT || 3000

// Middleware : parsent le body JSON des requetes POST/PUT/PATCH
app.use(express.json())
app.use(express.urlencoded({ extended: true }))

// Route simple
app.get('/', (req, res) => {
  res.json({ message: 'Bienvenue sur l\'API', version: '1.0.0' })
})

// Demarrer le serveur
app.listen(PORT, () => {
  console.log(`Serveur demarre sur http://localhost:${PORT}`)
})
```

```json
// package.json scripts
{
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js"
  }
}
```

### 2.2 Routing — `req`, `res`, `next`

```javascript
// src/index.js
import express from 'express'

const app = express()
app.use(express.json())

// ─── Differentes methodes HTTP ───────────────────────────────────────

// GET : lire une ressource
app.get('/users', (req, res) => {
  // req.query : parametres de la query string (?page=2&limit=10)
  const { page = 1, limit = 10, search = '' } = req.query
  
  res.status(200).json({
    data: [],
    page: Number(page),
    limit: Number(limit),
    total: 0
  })
})

// GET avec parametre de chemin
app.get('/users/:id', (req, res) => {
  // req.params : parametres de route (/users/42 → { id: '42' })
  const { id } = req.params
  
  // Simuler une ressource non trouvee
  if (id === '999') {
    return res.status(404).json({ error: 'Utilisateur non trouve' })
  }
  
  res.json({ id, name: 'Alice', email: 'alice@example.com' })
})

// POST : creer une ressource
app.post('/users', (req, res) => {
  // req.body : corps de la requete (JSON parse par express.json())
  const { name, email } = req.body
  
  if (!name || !email) {
    return res.status(400).json({ error: 'name et email sont requis' })
  }
  
  const newUser = { id: Date.now(), name, email }
  res.status(201).json(newUser)
})

// PUT : remplacer completement une ressource
app.put('/users/:id', (req, res) => {
  const { id } = req.params
  const { name, email } = req.body
  
  res.json({ id, name, email, updatedAt: new Date().toISOString() })
})

// PATCH : modifier partiellement une ressource
app.patch('/users/:id', (req, res) => {
  const updates = req.body  // Seulement les champs a modifier
  res.json({ id: req.params.id, ...updates })
})

// DELETE : supprimer une ressource
app.delete('/users/:id', (req, res) => {
  // 204 No Content : succes sans corps de reponse
  res.status(204).send()
})
```

### 2.3 Middleware — Le concept central d'Express

Un middleware est une fonction qui a acces a `req`, `res`, et `next`. Il peut modifier la requete/reponse, appeler le middleware suivant, ou terminer la chaine.

```javascript
// Structure d'un middleware
function monMiddleware(req, res, next) {
  // Avant : traitement de la requete
  console.log(`${req.method} ${req.path}`)
  
  // Passer au middleware suivant (OBLIGATOIRE si on ne repond pas)
  next()
  
  // next(error) : passer au gestionnaire d'erreurs
  // next('route') : sauter les middlewares de cette route
}

// Middleware d'application (s'applique a toutes les routes)
app.use(monMiddleware)

// Middleware limite a un chemin
app.use('/api', monMiddleware)

// Middleware de route (s'applique a une route specifique)
app.get('/protected', monMiddleware, (req, res) => {
  res.json({ secret: 'data' })
})

// Middleware multiple sur une route
app.post('/users', validateBody, checkPermission, createUser)
```

```javascript
// Exemples de middlewares courants

// 1. Logger de requetes
function requestLogger(req, res, next) {
  const start = Date.now()
  res.on('finish', () => {
    const duration = Date.now() - start
    console.log(`${req.method} ${req.path} ${res.statusCode} — ${duration}ms`)
  })
  next()
}

// 2. Authentification JWT
import jwt from 'jsonwebtoken'

function authenticateToken(req, res, next) {
  const authHeader = req.headers['authorization']
  const token = authHeader?.split(' ')[1]  // "Bearer <token>"
  
  if (!token) {
    return res.status(401).json({ error: 'Token manquant' })
  }
  
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET)
    req.user = payload  // Attacher l'utilisateur a la requete
    next()
  } catch (err) {
    return res.status(403).json({ error: 'Token invalide ou expire' })
  }
}

// 3. Middleware de role
function requireRole(role) {
  // Retourne un middleware (factory)
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Non authentifie' })
    }
    if (!req.user.roles?.includes(role)) {
      return res.status(403).json({ error: 'Permission insuffisante' })
    }
    next()
  }
}

// Utilisation
app.delete('/users/:id', authenticateToken, requireRole('admin'), deleteUser)
```

### 2.4 Middlewares tiers essentiels

```bash
npm install cors helmet morgan express-rate-limit compression
```

```javascript
import cors from 'cors'
import helmet from 'helmet'
import morgan from 'morgan'
import rateLimit from 'express-rate-limit'
import compression from 'compression'

// CORS : autoriser les requetes cross-origin
app.use(cors({
  origin: ['http://localhost:5173', 'https://mon-app.com'],
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true  // Autoriser les cookies
}))

// Helmet : en-tetes de securite HTTP
app.use(helmet())
// Ajoute automatiquement :
// X-Content-Type-Options: nosniff
// X-Frame-Options: DENY
// Strict-Transport-Security: max-age=...
// Content-Security-Policy: ...

// Morgan : logging des requetes HTTP
app.use(morgan('dev'))         // Colorise pour le dev
app.use(morgan('combined'))    // Format Apache pour la prod

// Rate limiting : limiter le nombre de requetes
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                    // 100 requetes par fenetre par IP
  message: { error: 'Trop de requetes. Reessayez dans 15 minutes.' },
  standardHeaders: true,
  legacyHeaders: false
})
app.use('/api', limiter)

// Compression gzip
app.use(compression())
```

### 2.5 Gestion des erreurs

```javascript
// Middleware d'erreur : 4 parametres OBLIGATOIRES (err, req, res, next)
// Doit etre defini EN DERNIER dans l'application
function errorHandler(err, req, res, next) {
  console.error(err.stack)
  
  // Erreurs de validation
  if (err.name === 'ValidationError') {
    return res.status(400).json({
      error: 'Donnees invalides',
      details: err.details
    })
  }
  
  // Erreurs JWT
  if (err.name === 'JsonWebTokenError') {
    return res.status(401).json({ error: 'Token invalide' })
  }
  
  // Erreurs de syntaxe JSON dans le body
  if (err.type === 'entity.parse.failed') {
    return res.status(400).json({ error: 'JSON malformat' })
  }
  
  // Erreur generique
  const status = err.status || err.statusCode || 500
  res.status(status).json({
    error: err.message || 'Erreur serveur interne',
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  })
}

// Classe d'erreur personnalisee
class AppError extends Error {
  constructor(message, statusCode = 500) {
    super(message)
    this.statusCode = statusCode
    this.name = 'AppError'
  }
}

// Wrapper pour les fonctions async (evite les try/catch repetitifs)
function asyncHandler(fn) {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next)
  }
}

// Route avec gestion d'erreur propre
app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await db.findUser(req.params.id)
  if (!user) throw new AppError('Utilisateur non trouve', 404)
  res.json(user)
}))

// Toujours en dernier !
app.use(errorHandler)
```

### 2.6 Router — Organisation du code

```javascript
// src/routes/users.router.js
import { Router } from 'express'
import { UserController } from '../controllers/user.controller.js'
import { authenticateToken, requireRole } from '../middleware/auth.middleware.js'
import { validateCreateUser, validateUpdateUser } from '../middleware/validation.middleware.js'

const router = Router()
const userController = new UserController()

// Routes publiques
router.get('/', userController.getAll)
router.get('/:id', userController.getById)

// Routes protegees
router.post('/', authenticateToken, validateCreateUser, userController.create)
router.put('/:id', authenticateToken, validateUpdateUser, userController.update)
router.delete('/:id', authenticateToken, requireRole('admin'), userController.delete)

export default router
```

```javascript
// src/index.js — Montage des routers
import usersRouter from './routes/users.router.js'
import postsRouter from './routes/posts.router.js'
import authRouter from './routes/auth.router.js'

app.use('/api/v1/users', usersRouter)
app.use('/api/v1/posts', postsRouter)
app.use('/api/v1/auth', authRouter)
```

### 2.7 Organisation en couches (Architecture)

```
src/
├── index.js                ← Point d'entree, creation app Express
├── app.js                  ← Configuration middlewares (separee de index.js)
├── routes/                 ← Definition des routes et assemblage
│   ├── users.router.js
│   └── posts.router.js
├── controllers/            ← Gestion req/res, appel aux services
│   ├── user.controller.js
│   └── post.controller.js
├── services/               ← Logique metier pure (testable)
│   ├── user.service.js
│   └── post.service.js
├── repositories/           ← Acces aux donnees (BDD, API externe)
│   ├── user.repository.js
│   └── post.repository.js
├── middleware/             ← Middlewares reutilisables
│   ├── auth.middleware.js
│   ├── validation.middleware.js
│   └── error.middleware.js
├── models/                 ← Schemas/modeles de donnees
│   └── user.model.js
└── config/                 ← Configuration (BDD, JWT, etc.)
    └── database.js
```

```javascript
// src/controllers/user.controller.js
import { UserService } from '../services/user.service.js'
import { asyncHandler } from '../middleware/error.middleware.js'

export class UserController {
  constructor() {
    this.userService = new UserService()
  }
  
  getAll = asyncHandler(async (req, res) => {
    const { page = 1, limit = 20, search } = req.query
    const result = await this.userService.getAll({ page, limit, search })
    res.json(result)
  })
  
  getById = asyncHandler(async (req, res) => {
    const user = await this.userService.getById(req.params.id)
    res.json(user)  // userService lance AppError(404) si non trouve
  })
  
  create = asyncHandler(async (req, res) => {
    const user = await this.userService.create(req.body)
    res.status(201).json(user)
  })
  
  update = asyncHandler(async (req, res) => {
    const user = await this.userService.update(req.params.id, req.body)
    res.json(user)
  })
  
  delete = asyncHandler(async (req, res) => {
    await this.userService.delete(req.params.id)
    res.status(204).send()
  })
}
```

### 2.8 Validation avec express-validator

```bash
npm install express-validator
```

```javascript
// src/middleware/validation.middleware.js
import { body, param, query, validationResult } from 'express-validator'

// Regles de validation
export const validateCreateUser = [
  body('name')
    .trim()
    .notEmpty().withMessage('Le nom est requis')
    .isLength({ min: 2, max: 100 }).withMessage('Nom entre 2 et 100 caracteres'),
  
  body('email')
    .trim()
    .notEmpty().withMessage('L\'email est requis')
    .isEmail().withMessage('Format email invalide')
    .normalizeEmail(),
  
  body('password')
    .isLength({ min: 8 }).withMessage('Mot de passe minimum 8 caracteres')
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
    .withMessage('Le mot de passe doit contenir majuscule, minuscule et chiffre'),
  
  body('age')
    .optional()
    .isInt({ min: 0, max: 150 }).withMessage('Age invalide'),
  
  // Middleware de verification
  handleValidationErrors
]

function handleValidationErrors(req, res, next) {
  const errors = validationResult(req)
  if (!errors.isEmpty()) {
    return res.status(400).json({
      error: 'Validation echouee',
      details: errors.array().map(e => ({
        field: e.path,
        message: e.msg,
        value: e.value
      }))
    })
  }
  next()
}
```

### 2.9 Authentification JWT complete

```javascript
// src/services/auth.service.js
import jwt from 'jsonwebtoken'
import bcrypt from 'bcryptjs'
import { AppError } from '../middleware/error.middleware.js'

const JWT_SECRET = process.env.JWT_SECRET
const JWT_EXPIRES_IN = process.env.JWT_EXPIRES_IN || '7d'
const REFRESH_SECRET = process.env.REFRESH_SECRET
const REFRESH_EXPIRES_IN = '30d'

export class AuthService {
  async register(userData) {
    const existing = await this.userRepo.findByEmail(userData.email)
    if (existing) throw new AppError('Email deja utilise', 409)
    
    const hashedPassword = await bcrypt.hash(userData.password, 12)
    const user = await this.userRepo.create({
      ...userData,
      password: hashedPassword
    })
    
    return this.generateTokens(user)
  }
  
  async login(email, password) {
    const user = await this.userRepo.findByEmail(email)
    if (!user) throw new AppError('Identifiants incorrects', 401)
    
    const isValid = await bcrypt.compare(password, user.password)
    if (!isValid) throw new AppError('Identifiants incorrects', 401)
    
    return this.generateTokens(user)
  }
  
  generateTokens(user) {
    const payload = { sub: user.id, email: user.email, roles: user.roles }
    
    const accessToken = jwt.sign(payload, JWT_SECRET, {
      expiresIn: JWT_EXPIRES_IN
    })
    
    const refreshToken = jwt.sign({ sub: user.id }, REFRESH_SECRET, {
      expiresIn: REFRESH_EXPIRES_IN
    })
    
    return { accessToken, refreshToken, user: this.sanitizeUser(user) }
  }
  
  sanitizeUser(user) {
    const { password, ...safeUser } = user
    return safeUser  // Ne jamais retourner le mot de passe
  }
}
```

---

## 3. NestJS

NestJS est un framework Node.js progressif construit sur Express (ou Fastify) qui apporte une architecture modulaire inspire d'Angular.

### 3.1 Philosophie — Modules, Controleurs, Providers

```
Application NestJS
├── AppModule (root)
│   ├── UsersModule
│   │   ├── UsersController   ← Routes HTTP
│   │   ├── UsersService      ← Logique metier (Provider)
│   │   └── UsersRepository   ← Acces donnees (Provider)
│   ├── PostsModule
│   │   ├── PostsController
│   │   └── PostsService
│   └── AuthModule
│       ├── AuthController
│       ├── AuthService
│       └── JwtStrategy       ← Guard d'authentification
```

> [!info] Injection de dependances
> NestJS gere automatiquement les dependances entre classes via son conteneur IoC (Inversion of Control). Vous declarez quoi injecter avec `@Injectable()`, et NestJS instancie et injecte les services automatiquement. Plus besoin de `new UserService()` manuellement.

### 3.2 Installation et CLI

```bash
npm install -g @nestjs/cli

# Creer un projet
nest new mon-api
# Choisir npm ou yarn

# Generer des elements
nest generate module users        # ou nest g mo users
nest generate controller users    # ou nest g co users
nest generate service users       # ou nest g s users
nest generate resource products   # Genere module + controller + service + DTOs en un coup !

cd mon-api && npm run start:dev
```

Structure NestJS :

```
src/
├── app.module.ts          ← Module racine
├── main.ts                ← Point d'entree (bootstrap)
├── users/
│   ├── users.module.ts
│   ├── users.controller.ts
│   ├── users.controller.spec.ts   ← Tests auto-generes
│   ├── users.service.ts
│   ├── users.service.spec.ts
│   └── dto/
│       ├── create-user.dto.ts
│       └── update-user.dto.ts
└── auth/
    ├── auth.module.ts
    ├── auth.controller.ts
    ├── auth.service.ts
    ├── guards/
    │   └── jwt-auth.guard.ts
    └── strategies/
        └── jwt.strategy.ts
```

### 3.3 Module, Controleur, Service

```typescript
// src/users/users.module.ts
import { Module } from '@nestjs/common'
import { UsersController } from './users.controller'
import { UsersService } from './users.service'

@Module({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService]  // Pour utiliser UsersService dans d'autres modules
})
export class UsersModule {}
```

```typescript
// src/users/users.controller.ts
import {
  Controller, Get, Post, Put, Patch, Delete,
  Body, Param, Query, HttpCode, HttpStatus,
  ParseIntPipe, UseGuards, Request
} from '@nestjs/common'
import { ApiTags, ApiBearerAuth, ApiOperation } from '@nestjs/swagger'
import { UsersService } from './users.service'
import { CreateUserDto } from './dto/create-user.dto'
import { UpdateUserDto } from './dto/update-user.dto'
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard'

@ApiTags('users')              // Swagger : groupe les routes sous "users"
@Controller('users')           // Prefixe de route : /users
export class UsersController {
  constructor(private readonly usersService: UsersService) {}
  // NestJS injecte automatiquement UsersService

  @Get()
  @ApiOperation({ summary: 'Lister tous les utilisateurs' })
  findAll(
    @Query('page') page = '1',
    @Query('limit') limit = '20',
    @Query('search') search?: string
  ) {
    return this.usersService.findAll({
      page: Number(page),
      limit: Number(limit),
      search
    })
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    // ParseIntPipe : convertit le string param en number, lance 400 si invalide
    return this.usersService.findOne(id)
  }

  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() createUserDto: CreateUserDto) {
    // Validation automatique via class-validator (voir DTO)
    return this.usersService.create(createUserDto)
  }

  @Patch(':id')
  @UseGuards(JwtAuthGuard)       // Proteger la route avec un guard
  @ApiBearerAuth()               // Swagger : indique qu'un token est requis
  update(
    @Param('id', ParseIntPipe) id: number,
    @Body() updateUserDto: UpdateUserDto,
    @Request() req: any          // Acces a req.user (injecte par le guard)
  ) {
    return this.usersService.update(id, updateUserDto)
  }

  @Delete(':id')
  @UseGuards(JwtAuthGuard)
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id', ParseIntPipe) id: number) {
    return this.usersService.remove(id)
  }
}
```

```typescript
// src/users/users.service.ts
import { Injectable, NotFoundException, ConflictException } from '@nestjs/common'
import { CreateUserDto } from './dto/create-user.dto'
import { UpdateUserDto } from './dto/update-user.dto'

@Injectable()
export class UsersService {
  // En production : injecter un repository (TypeORM, Prisma)
  private users: any[] = []
  private nextId = 1

  findAll({ page, limit, search }: { page: number; limit: number; search?: string }) {
    let result = this.users
    
    if (search) {
      const q = search.toLowerCase()
      result = result.filter(u =>
        u.name.toLowerCase().includes(q) || u.email.toLowerCase().includes(q)
      )
    }
    
    const total = result.length
    const data = result.slice((page - 1) * limit, page * limit)
    
    return { data, total, page, limit, totalPages: Math.ceil(total / limit) }
  }

  findOne(id: number) {
    const user = this.users.find(u => u.id === id)
    if (!user) {
      throw new NotFoundException(`Utilisateur #${id} introuvable`)
    }
    return user
  }

  create(createUserDto: CreateUserDto) {
    const existing = this.users.find(u => u.email === createUserDto.email)
    if (existing) {
      throw new ConflictException('Un utilisateur avec cet email existe deja')
    }
    
    const user = {
      id: this.nextId++,
      ...createUserDto,
      createdAt: new Date().toISOString()
    }
    this.users.push(user)
    return user
  }

  update(id: number, updateUserDto: UpdateUserDto) {
    const userIndex = this.users.findIndex(u => u.id === id)
    if (userIndex === -1) {
      throw new NotFoundException(`Utilisateur #${id} introuvable`)
    }
    
    this.users[userIndex] = {
      ...this.users[userIndex],
      ...updateUserDto,
      updatedAt: new Date().toISOString()
    }
    return this.users[userIndex]
  }

  remove(id: number) {
    const userIndex = this.users.findIndex(u => u.id === id)
    if (userIndex === -1) {
      throw new NotFoundException(`Utilisateur #${id} introuvable`)
    }
    this.users.splice(userIndex, 1)
  }
}
```

### 3.4 DTOs avec class-validator

```bash
npm install class-validator class-transformer
```

```typescript
// src/users/dto/create-user.dto.ts
import {
  IsString, IsEmail, IsNotEmpty, MinLength, MaxLength,
  IsOptional, IsInt, Min, Max, IsArray, IsEnum, Matches
} from 'class-validator'
import { Transform } from 'class-transformer'
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger'

export enum UserRole {
  USER = 'user',
  ADMIN = 'admin',
  MODERATOR = 'moderator'
}

export class CreateUserDto {
  @ApiProperty({ example: 'Alice Dupont', description: 'Nom complet' })
  @IsString()
  @IsNotEmpty()
  @MinLength(2)
  @MaxLength(100)
  @Transform(({ value }) => value.trim())  // Supprimer espaces
  name: string

  @ApiProperty({ example: 'alice@example.com' })
  @IsEmail({}, { message: 'Format email invalide' })
  @IsNotEmpty()
  @Transform(({ value }) => value.toLowerCase().trim())
  email: string

  @ApiProperty({ minLength: 8 })
  @IsString()
  @MinLength(8, { message: 'Le mot de passe doit faire au moins 8 caracteres' })
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Le mot de passe doit contenir au moins une majuscule, une minuscule et un chiffre'
  })
  password: string

  @ApiPropertyOptional({ enum: UserRole, default: UserRole.USER })
  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole = UserRole.USER

  @ApiPropertyOptional({ minimum: 0, maximum: 150 })
  @IsOptional()
  @IsInt()
  @Min(0)
  @Max(150)
  age?: number
}
```

```typescript
// src/users/dto/update-user.dto.ts
import { PartialType, OmitType } from '@nestjs/swagger'
import { CreateUserDto } from './create-user.dto'

// PartialType : rend tous les champs optionnels
// OmitType : exclut des champs
export class UpdateUserDto extends PartialType(
  OmitType(CreateUserDto, ['password'] as const)
) {}
```

### 3.5 Pipes, Guards, Interceptors

```typescript
// main.ts — Activation globale de la validation
import { NestFactory } from '@nestjs/core'
import { ValidationPipe } from '@nestjs/common'
import { AppModule } from './app.module'

async function bootstrap() {
  const app = NestFactory.create(AppModule)
  
  // Validation globale des DTOs
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,           // Supprimer les proprietes non declarees dans le DTO
    forbidNonWhitelisted: true, // Lancer une erreur si proprietes inconnues
    transform: true,           // Transformer automatiquement les types (string → number)
    transformOptions: {
      enableImplicitConversion: true
    }
  }))
  
  // Prefixe global
  app.setGlobalPrefix('api/v1')
  
  // CORS
  app.enableCors({
    origin: process.env.ALLOWED_ORIGINS?.split(',') || '*'
  })
  
  await app.listen(process.env.PORT || 3000)
}
bootstrap()
```

```typescript
// src/auth/guards/jwt-auth.guard.ts
import { Injectable, ExecutionContext, UnauthorizedException } from '@nestjs/common'
import { AuthGuard } from '@nestjs/passport'
import { Reflector } from '@nestjs/core'
import { IS_PUBLIC_KEY } from '../decorators/public.decorator'

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super()
  }
  
  canActivate(context: ExecutionContext) {
    // Verifier si la route est marquee @Public()
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass()
    ])
    if (isPublic) return true
    
    return super.canActivate(context)
  }
}
```

```typescript
// src/common/interceptors/transform.interceptor.ts
import {
  Injectable, NestInterceptor, ExecutionContext, CallHandler
} from '@nestjs/common'
import { Observable } from 'rxjs'
import { map } from 'rxjs/operators'

// Interceptor : transforme toutes les reponses dans un format uniforme
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
        timestamp: new Date().toISOString()
      }))
    )
  }
}
```

### 3.6 Swagger — Documentation automatique

```bash
npm install @nestjs/swagger swagger-ui-express
```

```typescript
// main.ts
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger'

const config = new DocumentBuilder()
  .setTitle('Mon API')
  .setDescription('Documentation de l\'API REST')
  .setVersion('1.0')
  .addBearerAuth()
  .addTag('users', 'Gestion des utilisateurs')
  .addTag('posts', 'Gestion des articles')
  .build()

const document = SwaggerModule.createDocument(app, config)
SwaggerModule.setup('api/docs', app, document)
// Documentation disponible sur http://localhost:3000/api/docs
```

---

## 4. Comparaison Express vs NestJS

| Aspect | Express | NestJS |
|---|---|---|
| **Architecture** | Libre — vous definissez la structure | Imposee — modules/controllers/providers |
| **TypeScript** | Optionnel (types via @types) | Natif, recommande |
| **Learning curve** | Tres faible | Moderee (concepts Angular) |
| **Boilerplate** | Minimal | Moyen (decorateurs, modules) |
| **Flexibilite** | Maximale | Limitee par les conventions |
| **Maintenance** | Depends de vos choix | Facilitee par la structure |
| **Taille equipe** | Ideal solo ou petite equipe | Ideal grande equipe (conventions partagees) |
| **Performance** | Excellente | Tres bonne (legere surcharge decorateurs) |
| **Tests** | A configurer manuellement | Integre (spec files generes automatiquement) |
| **Swagger** | A configurer manuellement | Integre via decorateurs |
| **Validation** | A implementer (express-validator) | Integree (class-validator + Pipes) |
| **Use cases** | Microservices, APIs simples, prototypes | Applications enterprise, grandes APIs |

---

## 5. Exemple Complet — API REST avec Express

Application complete : API de blog avec auth JWT, validation, et organisation en couches.

```javascript
// src/app.js — Configuration Express
import express from 'express'
import cors from 'cors'
import helmet from 'helmet'
import morgan from 'morgan'
import { errorHandler } from './middleware/error.middleware.js'
import usersRouter from './routes/users.router.js'
import postsRouter from './routes/posts.router.js'
import authRouter from './routes/auth.router.js'

const app = express()

// Middlewares globaux
app.use(helmet())
app.use(cors({ origin: process.env.ALLOWED_ORIGINS || '*', credentials: true }))
app.use(morgan('dev'))
app.use(express.json({ limit: '10kb' }))

// Routes
app.use('/api/v1/auth', authRouter)
app.use('/api/v1/users', usersRouter)
app.use('/api/v1/posts', postsRouter)

// Route 404
app.use((req, res) => {
  res.status(404).json({ error: `Route ${req.method} ${req.path} introuvable` })
})

// Gestionnaire d'erreurs (TOUJOURS EN DERNIER)
app.use(errorHandler)

export default app
```

```javascript
// src/routes/posts.router.js
import { Router } from 'express'
import { PostsController } from '../controllers/posts.controller.js'
import { authenticateToken } from '../middleware/auth.middleware.js'
import { validateCreatePost } from '../middleware/validation.middleware.js'

const router = Router()
const postsController = new PostsController()

// Routes publiques
router.get('/', postsController.getAll)
router.get('/:id', postsController.getById)

// Routes protegees
router.post('/', authenticateToken, validateCreatePost, postsController.create)
router.put('/:id', authenticateToken, postsController.update)
router.delete('/:id', authenticateToken, postsController.delete)

export default router
```

---

## 6. Exercices Pratiques

> [!tip] Exercice 1 — API REST Express sans framework
> Construis une API CRUD complete pour des `books` (livres) :
> - `GET /api/books` avec pagination et filtre par auteur (`?author=Victor+Hugo`)
> - `GET /api/books/:id`
> - `POST /api/books` avec validation (titre, auteur, annee, isbn)
> - `PUT /api/books/:id`
> - `DELETE /api/books/:id`
> - Middleware de logging (timestamp, methode, chemin, duree, status)
> - Gestion d'erreurs centralisee avec `AppError`
> - Organisation : routes / controllers / services

> [!tip] Exercice 2 — Authentification JWT complete
> Ajoute un systeme d'authentification a l'API livres :
> - `POST /api/auth/register` : creer un compte (hashage bcrypt)
> - `POST /api/auth/login` : connexion, retourner access token (15min) + refresh token (7j)
> - `POST /api/auth/refresh` : echanger un refresh token contre un nouvel access token
> - `POST /api/auth/logout` : invalider le refresh token
> - Middleware `authenticateToken` pour proteger les routes
> - Les routes DELETE et PUT sont protegees, GET est public

> [!tip] Exercice 3 — API NestJS avec Swagger
> Reproduis l'API livres avec NestJS :
> - Generer le module Books avec `nest g resource books`
> - DTOs avec class-validator (`CreateBookDto`, `UpdateBookDto`)
> - `ValidationPipe` global avec `whitelist: true`
> - Swagger configure sur `/api/docs`
> - Un guard `JwtAuthGuard` pour les routes de modification
> - Tests unitaires du `BooksService` (spec genere automatiquement)

> [!tip] Exercice 4 — Middleware de rate limiting personnalise
> Implemente un middleware de rate limiting sans bibliotheque tierce :
> - Stocker les compteurs par IP dans un `Map` en memoire
> - Fenetre glissante de 60 secondes, max 30 requetes
> - Headers `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`
> - Reponse 429 avec `Retry-After` quand limite atteinte
> - Nettoyage des entrees expirees toutes les 5 minutes (`setInterval`)
> Bonus : rendre la fenetre et la limite configurables via des options.

---

## Liens Utiles

- Express.js documentation : https://expressjs.com/
- NestJS documentation : https://docs.nestjs.com/
- class-validator : https://github.com/typestack/class-validator
- jsonwebtoken : https://github.com/auth0/node-jsonwebtoken
- Voir aussi : [[08 - APIs REST avec Flask]], [[04 - CI-CD avec GitHub Actions]]
