# Dev Standards & Best Practices

> **Avertissement** : Ce document reflète les best practices au moment de sa rédaction (mars 2026). Les écosystèmes évoluent vite — **toujours vérifier rapidement** que ces recommandations sont encore à jour avant de les appliquer aveuglément. En cas de doute, la doc officielle fait foi.

---

## Next.js 15 (App Router)

- **App Router par défaut** — ne pas utiliser Pages Router pour les nouveaux projets.
- **Server Components par défaut** — n'ajouter `"use client"` que quand c'est nécessaire (state, effects, event handlers, browser APIs).
- **Data fetching** : fetch dans les Server Components avec `async/await` directement dans le composant. Pas de `useEffect` pour charger des données. Attention : les fetch sont **non-cachés par défaut** depuis Next 15 — opt-in explicite avec `cache: 'force-cache'` si besoin.
- **APIs de requête async** : `cookies()`, `headers()`, `params`, `searchParams` sont **async** depuis Next 15 — toujours les `await`.
- **Server Actions** pour les mutations (formulaires, actions DB). Utiliser avec `useActionState` + `useFormStatus` + le composant `<Form>` de `next/form` pour le progressive enhancement.
- **Route Handlers** (`app/api/`) pour les endpoints API REST. Typer `NextRequest` / `NextResponse`.
- **Metadata** : utiliser `generateMetadata()` ou l'export `metadata` pour le SEO, pas de `<Head>` manuel.
- **Loading / Error UI** : `loading.tsx`, `error.tsx`, `not-found.tsx` par segment de route.
- **Images** : toujours `next/image` avec `width`/`height` ou `fill`. Jamais de `<img>` brut.
- **Fonts** : `next/font/google` ou `next/font/local` dans le layout racine.
- **Turbopack** : utiliser `next dev --turbo` pour un dev server plus rapide (stable depuis Next 15).
- **`after()`** : pour exécuter du code après l'envoi de la réponse (logging, analytics) sans bloquer.
- **Config TypeScript** : `next.config.ts` supporté nativement.
- **Parallel Routes & Intercepting Routes** quand le UX le justifie (modales, dashboards).

## React 19

- **Composants fonctionnels uniquement** — pas de classes.
- **React Compiler** (expérimental, utilisable en production) : mémoïse automatiquement composants et valeurs. Quand activé, `useMemo`, `useCallback`, `React.memo` deviennent inutiles dans le code compilé. Adoption progressive via le plugin Babel — tester sur un projet avant de généraliser.
- **`use()`** : nouveau hook pour lire des promesses et du context pendant le render. Remplace `useContext` — préférer `use(MyContext)` dans le nouveau code. Peut être appelé conditionnellement.
- **`ref` comme prop** : `forwardRef` est déprécié. Accepter `ref` directement dans les props du composant.
- **Context provider simplifié** : `<MyContext value={...}>` directement, plus besoin de `<MyContext.Provider>`.
- **Formulaires** : `useActionState` (ex-`useFormState`) + `useFormStatus` + `useOptimistic` pour les Server Actions.
- **Hooks custom** : garder les composants petits. Extraire la logique métier dans des hooks custom (`useXxx`).
- **State management** : commencer par `useState` / `useReducer` + context. N'ajouter Zustand ou autre que si c'est justifié.
- **Keys** : toujours une key stable et unique sur les listes. Jamais l'index sauf liste statique.
- **Props** : destructurer dans la signature. Typer avec une `interface` (pas `type` sauf union/intersection).
- **Composition over configuration** : préférer `children` et le pattern composé (`<Dialog><DialogTrigger>...`) aux props massives.
- **Pas de logique dans le JSX** : extraire les conditions complexes dans des variables ou des sous-composants.

## TypeScript

- **`strict: true`** dans le `tsconfig.json` — non négociable.
- **Interfaces** pour les shapes d'objet, **types** pour les unions, intersections, et utilitaires.
- **Pas de `any`** — utiliser `unknown` si le type n'est pas connu, puis narrower.
- **Inférence** : laisser TS inférer quand c'est évident. Typer explicitement les signatures de fonctions publiques et les retours de fonctions complexes.
- **Enums** : préférer `as const` satisfies ou les union types aux `enum` classiques.
- **Zod** pour la validation runtime aux frontières (API inputs, form data, env vars). Inférer le type TS depuis le schema Zod (`z.infer<typeof schema>`).
- **Pas de assertions non-null (`!`)** sauf cas exceptionnel documenté.
- **Generics** : utiliser quand ça apporte de la réutilisabilité, pas pour montrer qu'on sait faire.

## Tailwind CSS v4

- **CSS-first config** : plus de `tailwind.config.js`. La configuration se fait dans le CSS avec la directive `@theme { ... }` pour les design tokens (couleurs, fonts, spacing).
- **Import unique** : `@import "tailwindcss"` remplace les anciennes directives `@tailwind base/components/utilities`.
- **Détection automatique** : plus besoin de configurer `content` — Tailwind v4 scanne automatiquement le projet.
- **`cn()` helper** (clsx + tailwind-merge) pour merger les classes conditionnellement.
- **Design tokens** dans `@theme` : couleurs, spacing, fonts. Ne pas hardcoder des valeurs arbitraires — utiliser le theme (`text-primary`).
- **Responsive** : mobile-first (`sm:`, `md:`, `lg:`). Toujours partir du mobile.
- **Dark mode** : utiliser le prefix `dark:`.
- **Opacité** : syntaxe slash uniquement (`bg-black/50`), les anciennes classes `bg-opacity-*` n'existent plus.
- **Pas de `@apply`** sauf dans les cas où c'est inévitable (styles globaux, composants tiers).
- **Grouper visuellement** les classes : layout > sizing > spacing > typography > colors > effects.
- **Tailwind Merge** gère les conflits — ne pas hésiter à exposer un `className` prop sur les composants réutilisables.
- **Migration** : utiliser `@tailwindcss/upgrade` pour migrer un projet v3 vers v4.

## shadcn/ui

- **Ce n'est pas une dépendance** — c'est du code copié dans le projet. Le personnaliser librement.
- **CLI** : `npx shadcn@latest add <component>` (le package s'appelle `shadcn`, plus `shadcn-ui`). Supporte Tailwind v4 nativement.
- **Init** : `npx shadcn@latest init` pour setup un nouveau projet.
- **Composition** : utiliser le pattern compound (`<Select><SelectTrigger><SelectValue>...`).
- **Customisation** : modifier les fichiers dans `components/ui/` directement. C'est fait pour.
- **CSS variables** pour le theming dans `globals.css` (ou via `@theme` en Tailwind v4). Respecter le naming shadcn (`--primary`, `--secondary`, etc.).
- **Radix primitives** sous le capot — consulter la doc Radix pour les props d'accessibilité et les edge cases.
- **Ne pas wrapper les composants shadcn** dans des abstractions inutiles. Les utiliser directement.
- **Formulaires** : combiner avec `react-hook-form` + `zod` via le pattern `<Form>` de shadcn.

## Neon (PostgreSQL serverless)

- **Connection pooling** : toujours utiliser le connection string avec pooler (`-pooler` suffix) en production.
- **Prisma** : configurer `directUrl` (sans pooler, pour migrations) + `url` (avec pooler, pour queries).
- **Prisma adapter Neon** : `@prisma/adapter-neon` avec `@neondatabase/serverless` pour les edge runtimes (WebSocket/HTTP).
- **Branching** : utiliser les branches Neon pour les previews / staging. Pas de DB partagée entre envs.
- **Serverless driver** (`@neondatabase/serverless`) pour les edge functions si besoin. Sinon Prisma suffit.
- **Migrations** : `prisma migrate dev` en local, `prisma migrate deploy` en CI/CD.
- **Pas de raw queries** sauf pour des requêtes vraiment complexes que Prisma ne peut pas exprimer.

## Auth.js v5 (NextAuth)

- **Auth.js v5 est en beta** — installer `next-auth@beta` (la v5 n'est pas encore en release stable, `next-auth@latest` reste la v4). Malgré le tag beta, v5 est largement utilisé en production et c'est la version recommandée pour App Router.
- **Config** dans `auth.ts` à la racine (ou `src/auth.ts`). Exporter `{ handlers, signIn, signOut, auth }`.
- **Route handler** : `app/api/auth/[...nextauth]/route.ts` qui réexporte `handlers` (`GET`, `POST`).
- **Middleware** : `auth` comme wrapper middleware pour protéger les routes. Configurer le `matcher`.
- **Session** : préférer la stratégie `jwt` pour le serverless. `database` si besoin de sessions persistantes (défaut quand un adapter est configuré).
- **Providers** : configurer dans `auth.ts`. Google, GitHub, Credentials, etc.
- **Callbacks** : `jwt` et `session` callbacks pour enrichir le token/session avec des données custom (rôle, orgId, etc.).
- **Prisma Adapter** : `@auth/prisma-adapter` pour persister users/accounts/sessions en DB.
- **Protéger côté serveur** : `const session = await auth()` dans les Server Components / Server Actions. Ne pas se fier uniquement au middleware.
- **Variables d'env** : `AUTH_SECRET` (obligatoire), `AUTH_URL` (auto-détecté en prod Vercel). Les anciens noms `NEXTAUTH_*` sont dépréciés.
- **Edge compatible** : conçu pour fonctionner sur les edge runtimes.

---

## Architecture full-stack (Vertical Slices + Hybride pragmatique)

Next.js est volontairement agnostique sur l'architecture. Le pattern ci-dessous combine **Vertical Slices** (chaque feature est autonome) avec une organisation hybride inspirée de FSD et DDD, adaptée aux contraintes de l'App Router.

### Structure recommandée

```
app/                              # Routing UNIQUEMENT — pages fines
├── (public)/
│   └── page.tsx                  # Compose depuis src/features/
├── (dashboard)/
│   ├── layout.tsx
│   ├── campaigns/
│   │   ├── page.tsx              # Import depuis features/campaigns
│   │   ├── [id]/
│   │   │   └── page.tsx
│   │   ├── loading.tsx
│   │   └── error.tsx
│   └── bilans/
│       └── page.tsx
├── api/                          # Route Handlers (intégrations externes)
│   ├── webhooks/
│   └── auth/[...nextauth]/
└── layout.tsx

src/
├── features/                     # Vertical slices par feature
│   ├── campaigns/
│   │   ├── components/           # UI spécifique à la feature
│   │   │   ├── campaign-list.tsx
│   │   │   ├── campaign-card.tsx
│   │   │   └── create-campaign-dialog.tsx
│   │   ├── actions/              # Server Actions (THIN)
│   │   │   └── create-campaign.ts
│   │   ├── services/             # Logique métier (THICK, testable)
│   │   │   ├── campaign.service.ts
│   │   │   └── campaign.service.test.ts
│   │   ├── queries/              # Data fetching (Server Components)
│   │   │   └── get-campaigns.ts
│   │   ├── schemas/              # Validation Zod
│   │   │   └── campaign.schema.ts
│   │   ├── types.ts
│   │   └── index.ts              # Barrel — API publique de la feature
│   └── bilans/
│       └── ...
│
├── entities/                     # Modèles de domaine partagés
│   ├── campaign/
│   │   ├── types.ts              # Types/interfaces du domaine
│   │   └── utils.ts              # Helpers purement liés au domaine
│   └── user/
│       └── types.ts
│
├── shared/                       # Code réutilisable cross-feature
│   ├── ui/                       # shadcn/ui + composants design system
│   ├── hooks/                    # Hooks partagés (useDebounce, etc.)
│   ├── lib/                      # Utilitaires (cn(), formatDate, etc.)
│   └── config/                   # Constantes, env validation (Zod)
│
└── infrastructure/               # Services externes et adapters
    ├── database/                 # Prisma client, helpers
    ├── auth/                     # Config Auth.js (auth.ts)
    └── api-clients/              # Clients API externes (GAM, Stripe, etc.)
```

### Règles clés

- **`app/` = routing seulement.** Les `page.tsx` sont des compositions fines qui importent depuis `src/features/`. Pas de logique métier dans `app/`.
- **Thin Actions, Thick Services.** Les Server Actions valident (Zod), appellent un service, et revalidate. La logique métier vit dans `services/` — testable, réutilisable (webhooks, cron, API routes).
- **Un barrel `index.ts` par feature.** Il expose l'API publique du slice. Les autres features n'importent que via le barrel, jamais dans les fichiers internes.
- **Pas d'imports cross-features.** Si deux features partagent un type ou une logique, l'extraire dans `entities/` ou `shared/`.
- **`entities/` = domaine pur.** Types, interfaces, et helpers qui décrivent le métier sans dépendre de React ou Next.js.
- **`infrastructure/` = le monde extérieur.** DB, auth, APIs tierces. Isolé pour être substituable.

### Thin Action → Thick Service (exemple)

```typescript
// src/features/campaigns/actions/create-campaign.ts
"use server"

import { auth } from "@/infrastructure/auth"
import { createCampaignSchema } from "../schemas/campaign.schema"
import { createCampaign } from "../services/campaign.service"
import { revalidatePath } from "next/cache"

export async function createCampaignAction(formData: FormData) {
  const session = await auth()
  if (!session) throw new Error("Unauthorized")

  const data = createCampaignSchema.parse(Object.fromEntries(formData))
  await createCampaign(data, session.user.id)  // ← toute la logique est ici
  revalidatePath("/campaigns")
}
```

```typescript
// src/features/campaigns/services/campaign.service.ts
import { prisma } from "@/infrastructure/database"
import type { CreateCampaignInput } from "../schemas/campaign.schema"

export async function createCampaign(data: CreateCampaignInput, userId: string) {
  // Logique métier pure — testable sans Next.js
  return prisma.campaign.create({
    data: { ...data, createdById: userId },
  })
}
```

### Quand utiliser quoi

| Besoin | Emplacement |
|---|---|
| UI spécifique à une feature | `src/features/<name>/components/` |
| Mutation depuis un formulaire | `src/features/<name>/actions/` → `services/` |
| Data fetching (Server Component) | `src/features/<name>/queries/` |
| Type partagé entre features | `src/entities/<domain>/types.ts` |
| Composant UI générique | `src/shared/ui/` |
| Hook réutilisable | `src/shared/hooks/` |
| Client API externe | `src/infrastructure/api-clients/` |
| Config DB / Prisma | `src/infrastructure/database/` |

---

## Principes transversaux

1. **Colocation** — garder les fichiers proches de là où ils sont utilisés.
2. **Pas d'abstraction prématurée** — dupliquer 2-3 fois avant d'extraire.
3. **Fail fast** — valider les inputs tôt (Zod), crash plutôt que silently fail.
4. **Conventions de nommage** : `PascalCase` composants, `camelCase` fonctions/variables, `kebab-case` fichiers/dossiers.
5. **Un fichier = une responsabilité** — si un fichier dépasse ~200 lignes, se demander s'il faut split.
6. **Accessibilité** — utiliser les éléments HTML sémantiques, tester au clavier, aria-labels quand nécessaire.
7. **Env vars** — valider avec Zod au boot (`env.ts`). Jamais d'accès direct à `process.env` éparpillé dans le code.

---

*Dernière mise à jour : mars 2026*
