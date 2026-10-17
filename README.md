# Respawn

> Foro de comunidad de videojuegos estilo Reddit. Guías, easter eggs, reviews y discusiones por juego.

**Demo en producción:** [forogaming.vercel.app](https://forogaming.vercel.app)

Respawn es una plataforma web de tipo foro especializada en videojuegos. Permite a usuarios registrados publicar guías paso a paso, compartir easter eggs, escribir reviews y mantener discusiones a través de comentarios anidados. Incluye sistema de votos, likes, favoritos, mensajería directa, sistema de amigos, bloqueos bidireccionales y un panel completo de moderación.

---

## Stack

| Capa | Tecnología |
|---|---|
| Framework | Next.js 16 (App Router) + Turbopack |
| Lenguaje | TypeScript (modo estricto) |
| Estilos | Tailwind CSS v3.4 |
| Base de datos | PostgreSQL 16 en Neon (serverless) |
| Auth | JWT con `jose` (HS256) + cookies httpOnly |
| Hashing | bcryptjs |
| Storage de medios | Uploadthing |
| Rate limiting | Upstash Redis (con fallback en memoria) |
| Catálogo de juegos | IGDB API |
| Hosting | Vercel |

---

## Características

**Contenido y feed**
- Posts con título, cuerpo HTML sanitizado, categoría (Guía / Easter Egg / Review / General), juego asociado y hasta 10 imágenes con drag-and-drop para reordenar.
- Comentarios anidados con threading mediante adjacency list.
- Algoritmo de trending con score temporal (vista SQL precalculada).
- Búsqueda full-text con filtros combinables por URL (`?q=`, `?category=`, `?game=`, `?sort=`).
- Sidebar persistente con navegación por juego y categoría.

**Social**
- Sistema de amigos con solicitudes pendientes.
- Mensajería directa con polling cada 15 segundos.
- Follows.
- Bloqueos bidireccionales totales con filtrado SQL declarativo (los bloqueados desaparecen de feeds, búsquedas, comentarios y perfil).

**Interacciones**
- Votos up/down polimórficos sobre posts y comentarios.
- Likes y favoritos.

**Moderación**
- Reportes con 4 target types: post, comment, user, message.
- Panel admin con acciones transaccionales: dismiss / remove content / ban user.
- Rate limiting de reportes (20/hora por usuario) y ventana anti-duplicado de 24h.

**Seguridad**
- SSR autenticado con cookies httpOnly y access tokens efímeros.
- CSP allowlist, HSTS, X-Frame-Options DENY, Permissions-Policy restrictiva.
- Sanitización HTML custom (sin dependencia de `jsdom`).
- Rate limiting en endpoints críticos (login, register, votes, post creation).
- Validación dual: Zod en servidor + bloqueo UI cliente.

---

## Estructura del proyecto

```
respawn/
├── src/
│   ├── app/              # Rutas (App Router): páginas, layouts, API routes
│   ├── components/       # Componentes React reutilizables
│   ├── lib/              # Helpers: auth, sanitize, blocks, db, etc.
│   ├── contexts/         # React contexts (Auth, Sidebar, AuthModal)
│   └── types/            # Tipos TypeScript compartidos
├── migrations/           # Migraciones SQL versionadas (001 a 006)
├── scripts/
│   └── migrate.js        # Runner de migraciones
├── public/               # Assets estáticos
└── SDD.md                # Software Design Document completo
```

---

## Setup local

### Requisitos

- Node.js 20+
- npm 10+
- Una base de datos PostgreSQL 16 (recomendado: cuenta gratuita en [Neon](https://neon.tech))
- Una cuenta en [Uploadthing](https://uploadthing.com) (plan free disponible)
- Opcional: una cuenta en [Upstash Redis](https://upstash.com) (plan free disponible) para el rate limiting distribuido
- Opcional: credenciales de [IGDB API](https://api-docs.igdb.com/) (vía Twitch) para el catálogo de juegos

### Instalación

```bash
# Clonar el repositorio
git clone https://github.com/Nikode17/respawn.git
cd respawn

# Instalar dependencias
npm install

# Copiar plantilla de variables de entorno
cp .env.example .env.local
```

### Variables de entorno

Editar `.env.local` con tus credenciales:

```bash
# Base de datos
DATABASE_URL=postgresql://user:password@host/database?sslmode=require

# JWT (generar un secret de al menos 32 caracteres)
JWT_SECRET=tu_secret_aleatorio_de_minimo_32_caracteres

# URL base de la app
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Uploadthing
UPLOADTHING_TOKEN=...

# Upstash Redis (rate limiting; opcional, sin estas claves usa memoria)
UPSTASH_REDIS_REST_URL=...
UPSTASH_REDIS_REST_TOKEN=...

# IGDB (opcional, para catálogo de juegos; usa credenciales OAuth de Twitch)
TWITCH_CLIENT_ID=...
TWITCH_CLIENT_SECRET=...
```

La lista completa, con valores por defecto, está en `.env.example`.

### Ejecutar migraciones

```bash
node scripts/migrate.js
```

El runner aplica las migraciones de `migrations/` en orden y registra cada una en la tabla `schema_migrations`. Es idempotente: si ya están aplicadas, no hace nada.

### Arrancar el servidor de desarrollo

```bash
npm run dev
```

La app estará disponible en [http://localhost:3000](http://localhost:3000).

### Build de producción

```bash
npm run build
npm start
```

---

## Documentación

- **SDD.md** — Software Design Document completo: arquitectura, modelo de datos, endpoints, decisiones de diseño (ADRs), seguridad y estado real construido.

---

## Autores

**Nicolas Mefano Fioretti** — [@Nikode17](https://github.com/Nikode17)  
**Javier Condeza Condeza** — [@MuckMaster](https://github.com/MuckMaster)

Proyecto final del CFGS Desarrollo de Aplicaciones Web (DAW) — CEAC Barcelona, 2026.

---

## Licencia

Código fuente disponible para consulta. Para usos derivados, contactar al autor.
