# GraphQL Complet

> [!info] GraphQL — Query Language for your API
> GraphQL est un langage de requête pour les APIs et un runtime pour exécuter ces requêtes, créé par Facebook en 2012 et open-sourcé en 2015. Il permet au client de demander exactement les données dont il a besoin — ni plus, ni moins.

## Table des matières
1. [[#GraphQL vs REST]]
2. [[#Schema Definition Language (SDL)]]
3. [[#Queries]]
4. [[#Mutations]]
5. [[#Subscriptions]]
6. [[#Serveur Python — Strawberry]]
7. [[#Serveur JavaScript — Apollo Server]]
8. [[#Client Apollo React]]
9. [[#Introspection et GraphiQL]]
10. [[#Authentification et autorisation]]
11. [[#Performance — DataLoader et N+1]]
12. [[#Pagination]]
13. [[#GraphQL Federation]]
14. [[#Hasura]]
15. [[#Exemple complet]]
16. [[#Exercices Pratiques]]

---

## GraphQL vs REST

### Les problèmes de REST que GraphQL résout

**Over-fetching** : REST retourne plus de données que nécessaire.
```
GET /api/users/42
→ { id, name, email, phone, address, avatar, bio, settings, ... }
// Vous avez besoin seulement de { name, email }
```

**Under-fetching (N+1)** : REST nécessite plusieurs requêtes pour des données liées.
```
GET /api/posts          → [{ id, title, authorId }]
GET /api/users/1        → auteur du post 1
GET /api/users/7        → auteur du post 2
GET /api/users/3        → auteur du post 3
// 4 requêtes pour afficher une liste de posts avec auteurs
```

**Versioning** : REST nécessite `/v1/`, `/v2/`, `/v3/` quand l'API évolue.

**GraphQL solution** :
```graphql
# Une seule requête, exactement ce dont on a besoin
query {
    posts {
        title
        author {
            name
            email
        }
    }
}
```

### Tableau comparatif

| Aspect | REST | GraphQL |
|--------|------|---------|
| **Endpoints** | Multiples (`/users`, `/posts`, ...) | Un seul (`/graphql`) |
| **Sélection des champs** | Non (retourne tout) | Oui (le client choisit) |
| **Relations** | Multiples requêtes ou endpoints spéciaux | Une requête imbriquée |
| **Type system** | Non (OpenAPI optionnel) | Oui (SDL, obligatoire) |
| **Over-fetching** | Oui | Non |
| **Under-fetching** | Oui | Non |
| **Caching HTTP** | Natif (GET cacheable) | Plus complexe (tout en POST) |
| **Upload fichiers** | Natif (multipart) | Via lib spéciale |
| **Temps réel** | SSE / WebSocket séparé | Subscriptions intégrées |
| **Découverte** | Swagger/OpenAPI | Introspection native + GraphiQL |
| **Courbe apprentissage** | Faible | Moyenne |
| **Idéal pour** | APIs simples, publiques | Apps complexes, clients multiples |

> [!tip] Quand choisir GraphQL
> - Application avec **plusieurs clients** (mobile, web, desktop) aux besoins différents
> - Données **fortement relationnelles** (graph de données)
> - Équipe qui veut **découpler** frontend et backend
> - **Pas idéal** pour : APIs publiques simples, upload massif de fichiers, système de cache HTTP agressif

---

## Schema Definition Language (SDL)

Le schema GraphQL est le **contrat** entre client et serveur. Il définit tous les types, queries, mutations et subscriptions disponibles.

### Types de base

```graphql
# Scalaires primitifs
String    # Chaîne UTF-8
Int       # Entier 32 bits signé
Float     # Nombre flottant
Boolean   # true / false
ID        # Identifiant unique (sérialisé comme String)

# Modificateurs
String!   # Non-null (obligatoire)
[String]  # Liste nullable de Strings nullables
[String!] # Liste nullable de Strings non-nulls
[String]! # Liste non-null de Strings nullables
[String!]! # Liste non-null de Strings non-nulls
```

### Définir des types

```graphql
# Type objet
type User {
    id: ID!
    email: String!
    username: String!
    displayName: String
    bio: String
    createdAt: String!
    posts: [Post!]!
    followers: [User!]!
}

type Post {
    id: ID!
    title: String!
    content: String!
    published: Boolean!
    author: User!
    tags: [Tag!]!
    comments: [Comment!]!
    createdAt: String!
    updatedAt: String!
}

type Tag {
    id: ID!
    name: String!
    slug: String!
    posts: [Post!]!
}

type Comment {
    id: ID!
    content: String!
    author: User!
    post: Post!
    createdAt: String!
}

# Enum
enum PostStatus {
    DRAFT
    PUBLISHED
    ARCHIVED
}

# Interface
interface Node {
    id: ID!
}

type Article implements Node {
    id: ID!
    title: String!
}

# Union — retourne un type OU un autre
union SearchResult = Post | User | Tag

# Type input (pour mutations)
input CreatePostInput {
    title: String!
    content: String!
    tagIds: [ID!]
    published: Boolean = false
}

input UpdatePostInput {
    title: String
    content: String
    tagIds: [ID!]
    published: Boolean
}

input PaginationInput {
    first: Int = 10
    after: String
}

# Types de pagination (Relay spec)
type PageInfo {
    hasNextPage: Boolean!
    hasPreviousPage: Boolean!
    startCursor: String
    endCursor: String
}

type PostEdge {
    node: Post!
    cursor: String!
}

type PostConnection {
    edges: [PostEdge!]!
    pageInfo: PageInfo!
    totalCount: Int!
}

# Scalaire custom
scalar DateTime
scalar JSON
scalar Upload
```

### Query, Mutation, Subscription

```graphql
# Point d'entrée pour les lectures
type Query {
    # Requêtes simples
    me: User
    user(id: ID!): User
    users: [User!]!
    
    # Avec arguments
    post(id: ID, slug: String): Post
    posts(
        status: PostStatus
        authorId: ID
        tagSlug: String
        pagination: PaginationInput
    ): PostConnection!
    
    # Recherche unifiée
    search(query: String!): [SearchResult!]!
}

# Point d'entrée pour les modifications
type Mutation {
    # Auth
    register(email: String!, password: String!, username: String!): AuthPayload!
    login(email: String!, password: String!): AuthPayload!
    logout: Boolean!
    
    # Posts
    createPost(input: CreatePostInput!): Post!
    updatePost(id: ID!, input: UpdatePostInput!): Post!
    deletePost(id: ID!): Boolean!
    publishPost(id: ID!): Post!
    
    # Comments
    addComment(postId: ID!, content: String!): Comment!
    deleteComment(id: ID!): Boolean!
}

# Temps réel via WebSocket
type Subscription {
    postPublished: Post!
    commentAdded(postId: ID!): Comment!
    userOnline(userId: ID!): Boolean!
}

# Types de retour auth
type AuthPayload {
    token: String!
    user: User!
}
```

### Directives

```graphql
# Directives built-in
type Query {
    post(id: ID!, includeComments: Boolean = false): Post
}

# Usage côté client
query GetPost($id: ID!, $withComments: Boolean!) {
    post(id: $id) {
        title
        content
        comments @include(if: $withComments) {
            content
            author { username }
        }
    }
}

# @skip (inverse de @include)
query GetPost($id: ID!, $skipMeta: Boolean!) {
    post(id: $id) {
        title
        createdAt @skip(if: $skipMeta)
        updatedAt @skip(if: $skipMeta)
    }
}

# @deprecated
type User {
    fullName: String @deprecated(reason: "Use displayName instead")
    displayName: String!
}
```

---

## Queries

```graphql
# Query nommée simple
query GetUser {
    me {
        id
        email
        username
    }
}

# Query avec arguments
query GetPost($id: ID!) {
    post(id: $id) {
        id
        title
        content
        author {
            username
            avatar
        }
        tags {
            name
            slug
        }
    }
}

# Alias — requêter deux fois le même champ
query ComparePosts {
    firstPost: post(id: "1") {
        title
    }
    secondPost: post(id: "2") {
        title
    }
}

# Fragments — réutiliser des sélections de champs
fragment UserCard on User {
    id
    username
    displayName
    avatar
}

query GetPostWithAuthorAndCommenters {
    post(id: "1") {
        title
        author {
            ...UserCard
        }
        comments {
            content
            author {
                ...UserCard
            }
        }
    }
}

# Inline fragments pour unions/interfaces
query Search($q: String!) {
    search(query: $q) {
        ... on Post {
            title
            author { username }
        }
        ... on User {
            username
            displayName
        }
        ... on Tag {
            name
            slug
        }
    }
}
```

---

## Mutations

```graphql
# Mutation basique
mutation CreatePost($input: CreatePostInput!) {
    createPost(input: $input) {
        id
        title
        published
        author {
            username
        }
    }
}

# Variables envoyées séparément (JSON)
{
    "input": {
        "title": "Mon article GraphQL",
        "content": "Contenu complet...",
        "tagIds": ["tag1", "tag2"],
        "published": false
    }
}

# Mutation avec erreurs typées (pattern recommandé)
type CreatePostResult {
    post: Post
    errors: [UserError!]!
}

type UserError {
    field: String
    message: String!
    code: String!
}

type Mutation {
    createPost(input: CreatePostInput!): CreatePostResult!
}
```

---

## Subscriptions

```graphql
# Côté schema
type Subscription {
    commentAdded(postId: ID!): Comment!
}

# Côté client (avec Apollo)
subscription OnCommentAdded($postId: ID!) {
    commentAdded(postId: $postId) {
        id
        content
        author {
            username
            avatar
        }
        createdAt
    }
}
```

---

## Serveur Python — Strawberry

```bash
pip install strawberry-graphql[fastapi] uvicorn
```

```python
# schema.py
import strawberry
from typing import Optional, List
from datetime import datetime


@strawberry.type
class User:
    id: strawberry.ID
    email: str
    username: str
    display_name: Optional[str] = None
    
    @strawberry.field
    async def posts(self) -> List["Post"]:
        # Charger les posts de l'utilisateur
        return await Post.get_by_author(self.id)


@strawberry.type
class Post:
    id: strawberry.ID
    title: str
    content: str
    published: bool
    created_at: datetime
    
    @strawberry.field
    async def author(self) -> User:
        return await User.get_by_id(self.author_id)
    
    @classmethod
    async def get_by_author(cls, author_id: str) -> List["Post"]:
        # Simulation — remplacer par votre ORM
        return []


@strawberry.input
class CreatePostInput:
    title: str
    content: str
    published: bool = False
    tag_ids: Optional[List[strawberry.ID]] = None


@strawberry.type
class UserError:
    field: Optional[str]
    message: str
    code: str


@strawberry.type
class CreatePostResult:
    post: Optional[Post] = None
    errors: List[UserError] = strawberry.field(default_factory=list)


# Info de contexte (auth, DataLoader, etc.)
from strawberry.types import Info
from .context import Context


@strawberry.type
class Query:
    @strawberry.field
    async def me(self, info: Info[Context, None]) -> Optional[User]:
        if not info.context.user:
            return None
        return info.context.user
    
    @strawberry.field
    async def post(self, id: strawberry.ID) -> Optional[Post]:
        # Utilisation du DataLoader
        return await info.context.post_loader.load(id)
    
    @strawberry.field
    async def posts(
        self,
        info: Info[Context, None],
        published_only: bool = True
    ) -> List[Post]:
        # Appel base de données
        return await fetch_posts(published_only=published_only)


@strawberry.type
class Mutation:
    @strawberry.mutation
    async def create_post(
        self,
        input: CreatePostInput,
        info: Info[Context, None]
    ) -> CreatePostResult:
        user = info.context.user
        if not user:
            return CreatePostResult(errors=[
                UserError(field=None, message="Non authentifié", code="UNAUTHENTICATED")
            ])
        
        if len(input.title) < 5:
            return CreatePostResult(errors=[
                UserError(field="title", message="Titre trop court", code="VALIDATION_ERROR")
            ])
        
        # Créer en base
        post = await create_post_in_db(
            title=input.title,
            content=input.content,
            author_id=user.id,
            published=input.published,
        )
        return CreatePostResult(post=post)
    
    @strawberry.mutation
    async def publish_post(
        self,
        id: strawberry.ID,
        info: Info[Context, None]
    ) -> Post:
        user = info.context.user
        post = await get_post(id)
        
        if post.author_id != user.id:
            raise Exception("Non autorisé")
        
        return await update_post(id, published=True)


# Subscriptions
import asyncio
from typing import AsyncGenerator

@strawberry.type
class Subscription:
    @strawberry.subscription
    async def comment_added(
        self,
        post_id: strawberry.ID
    ) -> AsyncGenerator[Comment, None]:
        # Pattern avec pubsub
        async for comment in pubsub.subscribe(f"comments:{post_id}"):
            yield comment


# Schéma final
schema = strawberry.Schema(
    query=Query,
    mutation=Mutation,
    subscription=Subscription,
)
```

```python
# main.py — Intégration FastAPI
from fastapi import FastAPI, Request, Depends
from strawberry.fastapi import GraphQLRouter
from .schema import schema
from .context import get_context

graphql_app = GraphQLRouter(
    schema,
    context_getter=get_context,
    graphiql=True,  # Interface graphique de débogage
)

app = FastAPI(title="Blog API")
app.include_router(graphql_app, prefix="/graphql")
```

```python
# context.py
from strawberry.fastapi import BaseContext
from fastapi import Request, Depends
from strawberry.dataloader import DataLoader
from .models import User, Post


class Context(BaseContext):
    user: Optional[User]
    post_loader: DataLoader
    user_loader: DataLoader


async def get_context(request: Request) -> Context:
    # Extraire et valider le token JWT
    token = request.headers.get("Authorization", "").replace("Bearer ", "")
    user = await verify_jwt_token(token) if token else None
    
    # DataLoaders pour éviter le N+1
    post_loader = DataLoader(load_fn=batch_load_posts)
    user_loader = DataLoader(load_fn=batch_load_users)
    
    return Context(user=user, post_loader=post_loader, user_loader=user_loader)
```

---

## Serveur JavaScript — Apollo Server

```bash
npm install @apollo/server graphql graphql-tag
```

```javascript
// schema.js
import { gql } from 'graphql-tag'

export const typeDefs = gql`
    type User {
        id: ID!
        email: String!
        username: String!
        posts: [Post!]!
    }
    
    type Post {
        id: ID!
        title: String!
        content: String!
        author: User!
        tags: [Tag!]!
    }
    
    type Tag {
        id: ID!
        name: String!
    }
    
    type Query {
        me: User
        post(id: ID!): Post
        posts: [Post!]!
    }
    
    type Mutation {
        createPost(title: String!, content: String!): Post!
    }
`

// resolvers.js
import { UserAPI } from './datasources/UserAPI.js'
import { PostAPI } from './datasources/PostAPI.js'
import DataLoader from 'dataloader'

export const resolvers = {
    Query: {
        me: (_, __, { user }) => {
            if (!user) throw new GraphQLError('Not authenticated', {
                extensions: { code: 'UNAUTHENTICATED' }
            })
            return user
        },
        post: (_, { id }, { dataSources }) => {
            return dataSources.postAPI.getPost(id)
        },
        posts: (_, __, { dataSources }) => {
            return dataSources.postAPI.getPosts()
        },
    },
    
    Mutation: {
        createPost: async (_, { title, content }, { user, dataSources }) => {
            if (!user) throw new GraphQLError('Not authenticated', {
                extensions: { code: 'UNAUTHENTICATED' }
            })
            return dataSources.postAPI.createPost({ title, content, authorId: user.id })
        },
    },
    
    // Resolvers de champs (relations)
    Post: {
        // DataLoader pour éviter N+1
        author: (post, _, { userLoader }) => {
            return userLoader.load(post.authorId)
        },
        tags: (post, _, { dataSources }) => {
            return dataSources.postAPI.getPostTags(post.id)
        },
    },
    
    User: {
        posts: (user, _, { dataSources }) => {
            return dataSources.postAPI.getPostsByAuthor(user.id)
        },
    },
}

// server.js
import { ApolloServer } from '@apollo/server'
import { startStandaloneServer } from '@apollo/server/standalone'
import { typeDefs } from './schema.js'
import { resolvers } from './resolvers.js'
import DataLoader from 'dataloader'
import { batchLoadUsers } from './loaders.js'

const server = new ApolloServer({ typeDefs, resolvers })

const { url } = await startStandaloneServer(server, {
    context: async ({ req }) => {
        const token = req.headers.authorization?.replace('Bearer ', '')
        const user = token ? await verifyToken(token) : null
        
        // DataLoader créé par requête (pas global !)
        const userLoader = new DataLoader(batchLoadUsers)
        
        return { user, userLoader }
    },
    listen: { port: 4000 },
})

console.log(`GraphQL server ready at ${url}`)
```

---

## Client Apollo React

```bash
npm install @apollo/client graphql
```

```tsx
// apollo-client.ts
import { ApolloClient, InMemoryCache, createHttpLink, ApolloLink } from '@apollo/client'
import { setContext } from '@apollo/client/link/context'

const httpLink = createHttpLink({ uri: 'http://localhost:4000/graphql' })

const authLink = setContext((_, { headers }) => {
    const token = localStorage.getItem('token')
    return {
        headers: {
            ...headers,
            authorization: token ? `Bearer ${token}` : '',
        }
    }
})

export const client = new ApolloClient({
    link: authLink.concat(httpLink),
    cache: new InMemoryCache({
        typePolicies: {
            Post: {
                keyFields: ['id'],
            },
        },
    }),
})

// App.tsx
import { ApolloProvider } from '@apollo/client'
import { client } from './apollo-client'

function App() {
    return (
        <ApolloProvider client={client}>
            <Router />
        </ApolloProvider>
    )
}
```

```tsx
// hooks/usePosts.ts
import { gql, useQuery, useMutation } from '@apollo/client'

const GET_POSTS = gql`
    query GetPosts {
        posts {
            id
            title
            content
            author {
                username
                displayName
            }
            tags {
                name
                slug
            }
            createdAt
        }
    }
`

const CREATE_POST = gql`
    mutation CreatePost($title: String!, $content: String!) {
        createPost(title: $title, content: $content) {
            id
            title
            content
            author { username }
        }
    }
`

// Composant PostList
import React from 'react'

function PostList() {
    const { data, loading, error, refetch } = useQuery(GET_POSTS, {
        variables: {},
        fetchPolicy: 'cache-and-network',
        pollInterval: 30000,  // Rafraîchir toutes les 30s
    })
    
    const [createPost, { loading: creating }] = useMutation(CREATE_POST, {
        // Mettre à jour le cache après mutation
        update(cache, { data: { createPost } }) {
            const existing = cache.readQuery({ query: GET_POSTS })
            cache.writeQuery({
                query: GET_POSTS,
                data: {
                    posts: [createPost, ...existing.posts]
                }
            })
        },
        // Ou plus simple : refetch
        refetchQueries: [{ query: GET_POSTS }],
        // Optimistic response (UI instantané)
        optimisticResponse: {
            createPost: {
                __typename: 'Post',
                id: 'temp-id',
                title: 'Nouveau post',
                content: '...',
                author: { username: 'moi', __typename: 'User' },
            }
        }
    })
    
    if (loading) return <div>Chargement...</div>
    if (error) return <div>Erreur : {error.message}</div>
    
    return (
        <div>
            {data.posts.map(post => (
                <article key={post.id}>
                    <h2>{post.title}</h2>
                    <p>Par {post.author.username}</p>
                </article>
            ))}
        </div>
    )
}

// Subscription
import { useSubscription } from '@apollo/client'

const COMMENT_ADDED = gql`
    subscription OnCommentAdded($postId: ID!) {
        commentAdded(postId: $postId) {
            id
            content
            author { username }
            createdAt
        }
    }
`

function CommentSection({ postId }) {
    const [comments, setComments] = React.useState([])
    
    useSubscription(COMMENT_ADDED, {
        variables: { postId },
        onData: ({ data }) => {
            setComments(prev => [...prev, data.data.commentAdded])
        },
    })
    
    return (
        <ul>
            {comments.map(c => <li key={c.id}>{c.content}</li>)}
        </ul>
    )
}
```

---

## Introspection et GraphiQL

```graphql
# Introspection — le schema se décrit lui-même
query IntrospectTypes {
    __schema {
        types {
            name
            kind
            description
        }
    }
}

query IntrospectType {
    __type(name: "Post") {
        name
        fields {
            name
            type {
                name
                kind
                ofType {
                    name
                    kind
                }
            }
            description
        }
    }
}
```

> [!tip] GraphiQL — IDE dans le navigateur
> GraphiQL est disponible à `/graphql` quand `graphiql=True` dans Strawberry ou Apollo. Il offre : autocomplétion, documentation auto-générée, historique des requêtes, variables, headers.
>
> Pour la production : désactiver l'introspection (`introspection=False`) pour ne pas exposer la structure interne de l'API.

---

## Authentification et autorisation

### Context avec JWT

```python
# Strawberry — middleware auth
import jwt
from fastapi import Request

async def get_context(request: Request) -> Context:
    token = request.headers.get("Authorization", "").replace("Bearer ", "")
    
    user = None
    if token:
        try:
            payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
            user = await User.get_by_id(payload["sub"])
        except jwt.InvalidTokenError:
            pass  # Token invalide — user reste None
    
    return Context(user=user, ...)
```

### Permission au niveau des resolvers

```python
# strawberry_permissions.py
import strawberry
from strawberry.types import Info

def is_authenticated(info: Info) -> bool:
    return info.context.user is not None

def is_owner(info: Info, resource_user_id: str) -> bool:
    return (
        info.context.user is not None and
        str(info.context.user.id) == str(resource_user_id)
    )

# Décorateur
from functools import wraps

def require_auth(func):
    @wraps(func)
    async def wrapper(*args, info: Info, **kwargs):
        if not info.context.user:
            raise strawberry.exceptions.StrawberryException("Non authentifié")
        return await func(*args, info=info, **kwargs)
    return wrapper

@strawberry.type
class Mutation:
    @strawberry.mutation
    @require_auth
    async def create_post(self, input: CreatePostInput, info: Info) -> Post:
        ...
```

---

## Performance — DataLoader et N+1

### Le problème N+1 en GraphQL

```graphql
query {
    posts {           # 1 requête → 100 posts
        author {      # 100 requêtes pour charger chaque auteur !
            username
        }
    }
}
```

### Solution : DataLoader (batching + caching)

```python
# Python — DataLoader avec Strawberry
from strawberry.dataloader import DataLoader
from typing import List

async def batch_load_users(user_ids: List[str]) -> List[Optional[User]]:
    """Une seule requête pour TOUS les user_ids."""
    # SELECT * FROM users WHERE id IN (id1, id2, id3, ...)
    users = await User.filter(id__in=user_ids)
    
    # Mapper user_id → User (dans le même ordre que user_ids !)
    user_map = {str(u.id): u for u in users}
    return [user_map.get(uid) for uid in user_ids]

# Dans le contexte (créé par requête, pas global)
async def get_context(request: Request) -> Context:
    user_loader = DataLoader(load_fn=batch_load_users)
    post_loader = DataLoader(load_fn=batch_load_posts)
    return Context(user_loader=user_loader, post_loader=post_loader, ...)

# Dans les resolvers
@strawberry.type
class Post:
    author_id: strawberry.Private[str]  # champ interne, non exposé
    
    @strawberry.field
    async def author(self, info: Info) -> User:
        # DataLoader accumule les appels et fait UNE seule requête batch
        return await info.context.user_loader.load(self.author_id)
```

```javascript
// JavaScript — DataLoader
import DataLoader from 'dataloader'

// Fonction de batch : reçoit un tableau d'IDs, retourne un tableau de Users
async function batchLoadUsers(userIds) {
    const users = await db.users.findMany({
        where: { id: { in: userIds } }
    })
    
    // Retourner dans le même ordre que userIds
    const userMap = Object.fromEntries(users.map(u => [u.id, u]))
    return userIds.map(id => userMap[id] ?? new Error(`User ${id} not found`))
}

// Créé par requête dans le contexte Apollo
context: async ({ req }) => ({
    userLoader: new DataLoader(batchLoadUsers),
    // DataLoader cache automatiquement les résultats pendant la requête
})
```

> [!warning] DataLoader par requête, pas global
> Le DataLoader doit être créé **à chaque requête** (dans le `context`), jamais en singleton global. Le cache interne est per-requête — un singleton causerait des fuites de données entre requêtes utilisateurs.

---

## Pagination

### Cursor-based (Relay spec — recommandée)

```graphql
type PostConnection {
    edges: [PostEdge!]!
    pageInfo: PageInfo!
    totalCount: Int!
}

type PostEdge {
    node: Post!
    cursor: String!
}

type PageInfo {
    hasNextPage: Boolean!
    hasPreviousPage: Boolean!
    startCursor: String
    endCursor: String
}

type Query {
    posts(
        first: Int,       # Nombre d'éléments à récupérer (forward)
        after: String,    # Curseur de départ (forward)
        last: Int,        # Nombre d'éléments (backward)
        before: String,   # Curseur de départ (backward)
    ): PostConnection!
}
```

```graphql
# Première page
query {
    posts(first: 10) {
        edges {
            node { id title }
            cursor
        }
        pageInfo {
            hasNextPage
            endCursor
        }
        totalCount
    }
}

# Page suivante (après le dernier curseur reçu)
query {
    posts(first: 10, after: "Y3Vyc29yMTA=") {
        edges {
            node { id title }
        }
        pageInfo { hasNextPage endCursor }
    }
}
```

### Implémentation serveur Python

```python
import base64
from typing import Optional, List

def encode_cursor(id: str) -> str:
    return base64.b64encode(f"cursor:{id}".encode()).decode()

def decode_cursor(cursor: str) -> str:
    decoded = base64.b64decode(cursor).decode()
    return decoded.replace("cursor:", "")

@strawberry.type
class PostEdge:
    node: Post
    cursor: str

@strawberry.type  
class PageInfo:
    has_next_page: bool
    has_previous_page: bool
    start_cursor: Optional[str]
    end_cursor: Optional[str]

@strawberry.type
class PostConnection:
    edges: List[PostEdge]
    page_info: PageInfo
    total_count: int

@strawberry.type
class Query:
    @strawberry.field
    async def posts(
        self,
        first: Optional[int] = 10,
        after: Optional[str] = None,
    ) -> PostConnection:
        limit = min(first or 10, 100)
        
        query = Post.filter(published=True).order_by('created_at')
        total = await query.count()
        
        if after:
            after_id = decode_cursor(after)
            query = query.filter(id__gt=after_id)
        
        posts = await query.limit(limit + 1)  # +1 pour détecter hasNextPage
        has_next = len(posts) > limit
        posts = posts[:limit]
        
        edges = [
            PostEdge(node=p, cursor=encode_cursor(str(p.id)))
            for p in posts
        ]
        
        return PostConnection(
            edges=edges,
            page_info=PageInfo(
                has_next_page=has_next,
                has_previous_page=after is not None,
                start_cursor=edges[0].cursor if edges else None,
                end_cursor=edges[-1].cursor if edges else None,
            ),
            total_count=total,
        )
```

---

## GraphQL Federation

Apollo Federation permet de **composer plusieurs sous-graphes** GraphQL en un seul graphe unifié — l'architecture microservices pour GraphQL.

```graphql
# user-service/schema.graphql
type User @key(fields: "id") {
    id: ID!
    email: String!
    username: String!
}

# post-service/schema.graphql
type Post @key(fields: "id") {
    id: ID!
    title: String!
    authorId: ID!
    author: User!    # Référence vers User Service !
}

extend type User @key(fields: "id") {
    id: ID! @external
    posts: [Post!]!  # Posts définis dans Post Service
}

# Apollo Router combine les deux services automatiquement
```

---

## Hasura

Hasura génère **automatiquement une API GraphQL** depuis une base de données PostgreSQL.

```yaml
# docker-compose.yml
services:
    postgres:
        image: postgres:15
        environment:
            POSTGRES_PASSWORD: password
    
    hasura:
        image: hasura/graphql-engine:v2.36.0
        ports:
            - "8080:8080"
        environment:
            HASURA_GRAPHQL_DATABASE_URL: postgres://postgres:password@postgres/postgres
            HASURA_GRAPHQL_ENABLE_CONSOLE: "true"
            HASURA_GRAPHQL_ADMIN_SECRET: myadminsecretkey
```

Hasura offre automatiquement :
- Queries avec filtres, tri, pagination
- Mutations insert/update/delete
- Subscriptions temps réel
- Row-level security (permissions par rôle)
- Actions (logique métier custom)
- Event triggers (webhooks sur mutations)

---

## Exemple complet

API Blog avec Strawberry + FastAPI.

```python
# Projet complet : blog_api/
# ├── main.py
# ├── schema.py
# ├── models.py (SQLAlchemy)
# ├── context.py
# └── loaders.py

# models.py
from sqlalchemy import Column, String, Boolean, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from database import Base
import uuid
from datetime import datetime

class UserModel(Base):
    __tablename__ = "users"
    id = Column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    email = Column(String, unique=True, nullable=False)
    username = Column(String, unique=True, nullable=False)
    hashed_password = Column(String, nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    posts = relationship("PostModel", back_populates="author")

class PostModel(Base):
    __tablename__ = "posts"
    id = Column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    title = Column(String, nullable=False)
    content = Column(String, nullable=False)
    published = Column(Boolean, default=False)
    author_id = Column(String, ForeignKey("users.id"), nullable=False)
    author = relationship("UserModel", back_populates="posts")
    created_at = Column(DateTime, default=datetime.utcnow)
```

```python
# schema.py — Schéma Strawberry complet
import strawberry
from strawberry.fastapi import GraphQLRouter
from typing import Optional, List, Annotated
from datetime import datetime

@strawberry.type
class UserType:
    id: strawberry.ID
    email: str
    username: str
    created_at: datetime

@strawberry.type
class PostType:
    id: strawberry.ID
    title: str
    content: str
    published: bool
    created_at: datetime
    
    @strawberry.field
    async def author(self, info) -> UserType:
        return await info.context.user_loader.load(self.author_id)

@strawberry.input
class RegisterInput:
    email: str
    username: str
    password: str

@strawberry.type
class AuthPayload:
    token: str
    user: UserType

@strawberry.type
class Query:
    @strawberry.field
    async def posts(self, published_only: bool = True) -> List[PostType]:
        async with get_db() as db:
            query = select(PostModel)
            if published_only:
                query = query.where(PostModel.published == True)
            result = await db.execute(query)
            return [map_to_post_type(p) for p in result.scalars()]

@strawberry.type
class Mutation:
    @strawberry.mutation
    async def register(self, input: RegisterInput) -> AuthPayload:
        # Créer utilisateur, générer token JWT
        ...
    
    @strawberry.mutation
    async def login(self, email: str, password: str) -> AuthPayload:
        ...
    
    @strawberry.mutation
    async def create_post(
        self,
        title: str,
        content: str,
        info: strawberry.types.Info
    ) -> PostType:
        user = info.context.user
        if not user:
            raise Exception("Non authentifié")
        ...

schema = strawberry.Schema(query=Query, mutation=Mutation)
graphql_app = GraphQLRouter(schema, context_getter=get_context, graphiql=True)
```

---

## Exercices Pratiques

### Exercice 1 — Schema SDL : E-commerce

Concevez le schema GraphQL complet pour une application e-commerce :
- Types : Product, Category, Cart, CartItem, Order, OrderItem, User, Review
- Query : products (avec filtres : catégorie, prix, disponibilité, pagination), product(id), cart, orders
- Mutations : addToCart, removeFromCart, checkout, addReview
- Toutes les multiplicités et les types non-null correctement définis

### Exercice 2 — Serveur Strawberry

Implémentez un serveur GraphQL pour une app de gestion de tâches :
- Types : User, Project, Task (avec enum Status : TODO/IN_PROGRESS/DONE)
- Authentification JWT dans le context
- CRUD complet sur Task (avec permissions : seul le créateur peut modifier)
- DataLoader pour charger les projets par tâche (éviter N+1)
- Tests avec le client de test Strawberry

### Exercice 3 — Client Apollo React

Créez une interface de blog avec Apollo Client :
- Liste de posts avec pagination cursor-based (bouton "Charger plus")
- Détail d'un post avec commentaires
- Formulaire d'ajout de commentaire avec mutation + mise à jour du cache
- Subscription temps réel sur les nouveaux commentaires
- Gestion des états loading/error dans chaque composant

### Exercice 4 — Optimisation N+1

Analysez cette query et implémentez les DataLoaders nécessaires :
```graphql
query {
    orders {
        id
        user { name email }
        items {
            product { name price }
            quantity
        }
    }
}
```
Identifier tous les N+1 potentiels et implémenter les batch functions.

### Exercice 5 — API complète

Construire une API GraphQL de réseau social avec :
- Auth JWT (register/login/me)
- Users avec follow/unfollow
- Posts avec likes et commentaires
- Feed personnalisé (posts des utilisateurs suivis)
- Subscriptions sur les nouveaux posts du feed
- Rate limiting (max 5 mutations par minute par user)
- Tests d'intégration avec httpx (Python) ou supertest (JS)

---

## Liens et Références

- [[08 - APIs REST avec Flask]] — APIs REST pour comparer avec GraphQL
- [[09 - APIs REST avec FastAPI]] — FastAPI comme base serveur GraphQL
- [[03 - JavaScript Asynchrone]] — Async/await pour les resolvers
- [[05 - WebSockets]] — Subscriptions GraphQL utilisent WebSocket
