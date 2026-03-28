# Dev Standards & Best Practices

> **Avertissement** : Ce document reflète les best practices au moment de sa rédaction (mars 2026). Les écosystèmes évoluent vite — **toujours vérifier rapidement** que ces recommandations sont encore à jour avant de les appliquer aveuglément. En cas de doute, la doc officielle fait foi.

---

## Next.js (App Router)

- **App Router par défaut** — ne pas utiliser Pages Router pour les nouveaux projets.
- **Server Components par défaut** — n'ajouter `"use client"` que quand c'est nécessaire (state, effects, event handlers, browser APIs).
- **Data fetching** : fetch dans les Server Components, pas de `useEffect` pour charger des données. Utiliser `async/await` directement dans le composant.
- **Route Handlers** (`app/api/`) pour les endpoints API. Typer `NextRequest` / `NextResponse`.
- **Server Actions** pour les mutations (formulaires, actions DB). Préférer aux route handlers quand c'est un form submit.
- **Metadata** : utiliser `generateMetadata()` ou l'export `metadata` pour le SEO, pas de `<Head>` manuel.
- **Loading / Error UI** : `loading.tsx`, `error.tsx`, `not-found.tsx` par segment de route.
- **Images** : toujours `next/image` avec `width`/`height` ou `fill`. Jamais de `<img>` brut.
- **Fonts** : `next/font/google` ou `next/font/local` dans le layout racine.
- **Parallel Routes & Intercepting Routes** quand le UX le justifie (modales, dashboards).

## React

- **Composants fonctionnels uniquement** — pas de classes.
- **Hooks** : garder les composants petits. Extraire la logique métier dans des hooks custom (`useXxx`).
- **State management** : commencer par `useState` / `useReducer` + context. N'ajouter Zustand ou autre que si c'est justifié.
- **Mémoïsation** : ne pas abuser de `useMemo` / `useCallback` / `React.memo`. Les utiliser uniquement quand un problème de perf est mesuré.
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

## Tailwind CSS

- **Utility-first** — pas de CSS custom sauf cas très spécifique (animations complexes, CSS grid avancé).
- **`cn()` helper** (clsx + tailwind-merge) pour merger les classes conditionnellement.
- **Design tokens** dans `tailwind.config.ts` : couleurs, spacing, fonts. Ne pas hardcoder des valeurs arbitraires (`text-[#F85A20]`) — utiliser le theme (`text-primary`).
- **Responsive** : mobile-first (`sm:`, `md:`, `lg:`). Toujours partir du mobile.
- **Dark mode** : `class` strategy. Utiliser `dark:` prefix.
- **Pas de `@apply`** sauf dans les cas où c'est inévitable (styles globaux, composants tiers).
- **Grouper visuellement** les classes : layout > sizing > spacing > typography > colors > effects.
- **Tailwind Merge** gère les conflits — ne pas hésiter à exposer un `className` prop sur les composants réutilisables.

## shadcn/ui

- **Ce n'est pas une dépendance** — c'est du code copié dans le projet. Le personnaliser librement.
- **CLI** : `npx shadcn@latest add <component>` pour ajouter un composant.
- **Composition** : utiliser le pattern compound (`<Select><SelectTrigger><SelectValue>...`).
- **Customisation** : modifier les fichiers dans `components/ui/` directement. C'est fait pour.
- **CSS variables** pour le theming dans `globals.css`. Respecter le naming shadcn (`--primary`, `--secondary`, etc.).
- **Radix primitives** sous le capot — consulter la doc Radix pour les props d'accessibilité et les edge cases.
- **Ne pas wrapper les composants shadcn** dans des abstractions inutiles. Les utiliser directement.
- **Formulaires** : combiner avec `react-hook-form` + `zod` via le pattern `<Form>` de shadcn.

## Neon (PostgreSQL serverless)

- **Connection pooling** : toujours utiliser le connection string avec pooler (`-pooler` suffix) en production.
- **Prisma** : configurer `directUrl` (sans pooler, pour migrations) + `url` (avec pooler, pour queries).
- **Branching** : utiliser les branches Neon pour les previews / staging. Pas de DB partagée entre envs.
- **Serverless driver** (`@neondatabase/serverless`) pour les edge functions si besoin. Sinon Prisma suffit.
- **Migrations** : `prisma migrate dev` en local, `prisma migrate deploy` en CI/CD.
- **Pas de raw queries** sauf pour des requêtes vraiment complexes que Prisma ne peut pas exprimer.

## NextAuth (Auth.js v5)

- **Auth.js v5** (`next-auth@beta` / `next-auth@5`) — la v5 est la version à utiliser avec App Router.
- **Config** dans `auth.ts` à la racine (ou `src/auth.ts`). Exporter `{ handlers, signIn, signOut, auth }`.
- **Route handler** : `app/api/auth/[...nextauth]/route.ts` qui réexporte `handlers`.
- **Middleware** : `auth` comme middleware pour protéger les routes. Configurer le `matcher`.
- **Session** : préférer la stratégie `jwt` pour le serverless. `database` si besoin de sessions persistantes.
- **Providers** : configurer dans `auth.ts`. Google, GitHub, Credentials, etc.
- **Callbacks** : `jwt` et `session` callbacks pour enrichir le token/session avec des données custom (rôle, orgId, etc.).
- **Prisma Adapter** : `@auth/prisma-adapter` pour persister users/accounts/sessions en DB.
- **Protéger côté serveur** : `const session = await auth()` dans les Server Components / Server Actions. Ne pas se fier uniquement au middleware.
- **Variables d'env** : `AUTH_SECRET` (obligatoire), `AUTH_URL` (auto-détecté en prod Vercel).

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
