> ⚠️ **ARCHIVÉ le 03/10/2026.** Ce document n'est plus maintenu (il date de mars 2026 : React Compiler « expérimental », Prisma 6…).
> Source unique à jour : le skill Claude `dev-standards` (`~/.claude/skills/dev-standards/`, repo `claude-config`).

# Dev Standards & Best Practices

> Ce document definit les standards a suivre. Il est aligne sur les conventions **Next.js 16 + React 19 + Tailwind v4 + shadcn v4** (mars 2026). En cas de doute, la doc officielle fait foi : [nextjs.org/docs](https://nextjs.org/docs), [react.dev](https://react.dev).

---

## Structure du projet

Le projet suit la structure **stock Next.js 16 + shadcn v4** :

```
/                              # Racine — PAS de src/
├── app/                       # App Router — routing + pages fines
│   ├── globals.css            # @import "tailwindcss" + @theme inline
│   ├── layout.tsx             # Root layout (Server Component)
│   ├── page.tsx               # Page d'accueil
│   ├── (dashboard)/           # Route group pour les pages auth
│   │   ├── rapports/
│   │   │   ├── page.tsx       # SC fin → importe depuis features/
│   │   │   ├── loading.tsx
│   │   │   └── error.tsx
│   │   └── brief/
│   │       └── page.tsx
│   └── api/                   # Route Handlers (API REST externes)
├── features/                  # Vertical slices par domaine
│   ├── rapports/
│   │   ├── components/        # UI specifique
│   │   ├── hooks/             # Hooks custom
│   │   ├── actions/           # Server Actions ("use server")
│   │   ├── services/          # Logique metier (testable)
│   │   ├── queries/           # Data fetching (appeles depuis les SC)
│   │   └── index.ts           # Barrel — API publique
│   └── brief/
│       └── ...
├── components/                # Composants partages cross-feature
│   ├── ui/                    # shadcn/ui
│   └── icons/                 # Icones custom
├── lib/                       # Utilitaires partages
│   └── utils.ts               # cn()
├── hooks/                     # Hooks partages (useDebounce, etc.)
├── eslint.config.mjs          # ESLint v9 flat config
├── postcss.config.mjs         # @tailwindcss/postcss
├── next.config.mjs            # ESM
├── components.json            # shadcn config
└── tsconfig.json              # paths: @/* → ./*
```

### Regles de structure

- **Pas de `src/` directory** — tout est a la racine, comme le standard Next.js 16 + shadcn.
- **`app/` = routing uniquement** — les `page.tsx` sont des Server Components fins (~10-20 lignes) qui importent depuis `features/`.
- **`features/` = logique par domaine** — chaque feature est autonome avec ses composants, hooks, actions, services.
- **`components/` = UI partagee** — shadcn/ui, icones, theme provider.
- **`lib/` = utilitaires** — `cn()`, constantes, validation env.

---

## Next.js (App Router) — Patterns modernes obligatoires

> **Regle fondamentale** : toujours utiliser les patterns documentes dans la derniere version stable de Next.js. Ne pas ecrire de code legacy quand un pattern serveur natif existe.

- **Server Components par defaut** — n'ajouter `"use client"` que quand c'est strictement necessaire.
- **Data fetching dans les Server Components** — `async/await` directement. **Interdit** d'utiliser `useEffect` + `fetch()` pour charger des donnees.
- **Streaming + `<Suspense>`** — wrapper les parties lentes dans `<Suspense fallback={...}>`.
- **Server Actions pour toutes les mutations** :
  - `useActionState` pour gerer l'etat formulaire
  - `useFormStatus` pour les boutons submit
  - `useOptimistic` pour les updates optimistes
  - `<form action={serverAction}>` pour le progressive enhancement
- **Route Handlers** (`app/api/`) reserves aux endpoints REST consommes par des clients externes ou webhooks. Pas pour les mutations internes.
- **`after()`** pour executer du code apres la reponse (notifications, logging).
- **Metadata** : `generateMetadata()` ou export `metadata`.
- **`loading.tsx` / `error.tsx` / `not-found.tsx`** par segment de route.
- **`next/image`** partout. Jamais de `<img>` brut.
- **`next/font`** dans le root layout.
- **`next.config.mjs`** en ESM.

### Anti-patterns Next.js (INTERDIT)

| Anti-pattern | Pattern moderne |
|---|---|
| `"use client"` + `useEffect` + `fetch()` pour charger des donnees | Server Component async |
| `useState` + `onSubmit` + `fetch()` pour les mutations | Server Action + `useActionState` |
| `loading` state manuel | `loading.tsx` + `<Suspense>` |
| Route Handler POST pour mutation formulaire | Server Action + `revalidatePath` |
| Page entiere en `"use client"` | SC parent + CC enfant interactif |

---

## React 19 — Patterns modernes obligatoires

- **`ref` comme prop** — `forwardRef` est **interdit**. Utiliser `React.ComponentProps<"element">`.
- **`use()` hook** — **obligatoire** pour lire du context (remplace `useContext`).
- **Context simplifie** — `<MyContext value={...}>` directement, pas de `.Provider`.
- **Formulaires** : `useActionState` + `useFormStatus` + `useOptimistic` + Server Actions.
- **React Compiler** : memore automatiquement. `useMemo`/`useCallback`/`React.memo` deviennent inutiles.
- **Composants petits** — extraire dans des hooks custom. Un fichier ne depasse pas ~200 lignes.

### Anti-patterns React (INTERDIT)

| Anti-pattern | Pattern moderne React 19 |
|---|---|
| `React.forwardRef()` | `ref` via `React.ComponentProps` |
| `useContext(MyContext)` | `use(MyContext)` |
| `<MyContext.Provider>` | `<MyContext value={...}>` |
| `useState` + `onSubmit` + `fetch()` | `useActionState` + Server Action |
| `useMemo`/`useCallback` partout | React Compiler |

---

## TypeScript

- **`strict: true`** — non negociable.
- **Pas de `any`** — utiliser `unknown` + narrowing.
- **Zod** pour la validation aux frontieres (API inputs, env vars).
- **Pas d'enums** — `as const` ou union types.
- **Pas d'assertions non-null (`!`)** sauf cas documente.

---

## Tailwind CSS v4

- **`@import "tailwindcss"`** + **`@theme inline`** pour les design tokens.
- **`oklch()`** pour les couleurs (pas `hsl()`).
- **`tw-animate-css`** pour les animations (pas `tailwindcss-animate`).
- **`cn()` helper** (clsx + tailwind-merge).
- **Opacité slash** : `bg-black/50`. Les classes `bg-opacity-*` n'existent plus.
- **Pas de `@apply`** — CSS natif uniquement.
- **Pas de `tailwind.config.*`** — configuration dans le CSS via `@theme`.
- **`postcss.config.mjs`** en ESM avec `@tailwindcss/postcss`.

### Couleurs — tokens obligatoires (INTERDIT de hardcoder)

**Toutes les couleurs DOIVENT utiliser les tokens du thème.** Jamais de classes Tailwind avec numéros (`text-gray-500`, `bg-orange-50`) ni de hex inline (`from-[#F85A20]`).

| Usage | Token Tailwind | INTERDIT |
|-------|----------------|----------|
| Texte principal | `text-foreground` | `text-gray-900` |
| Texte secondaire | `text-muted-foreground` | `text-gray-500` |
| Texte moyen | `text-ev-gray-400` | `text-gray-700` |
| Texte léger | `text-ev-gray-300` | `text-gray-600` |
| Texte placeholder | `text-ev-gray-200` | `text-gray-400` |
| Fond page | `bg-ev-gray-50` | `bg-gray-50`, `bg-gray-100` |
| Bordure standard | `border-border` | `border-gray-200` |
| Bordure légère | `border-ev-gray-100` | `border-gray-300` |
| Orange principal | `text-orange` / `bg-orange` | `text-orange-500`, `bg-orange-500` |
| Orange foncé | `text-orange-dark` / `bg-orange-dark` | `text-orange-700`, `bg-orange-600` |
| Orange clair (fond) | `bg-orange-light` | `bg-orange-50`, `bg-orange-100` |
| Gradient orange | `from-orange to-orange-dark` | `from-orange-500 to-orange-600` |
| Bordure orange | `border-orange/30` | `border-orange-200` |

**Pour les graphiques (Recharts)** : utiliser les constantes de `lib/constants/platform-colors.ts` (`CHART_COLORS.primary`, etc.) — seul endroit où le hex est toléré car Recharts ne supporte pas les CSS variables.

**Pour les librairies tierces** : configurer via `var(--primary)` et les CSS variables du thème dans `globals.css`, jamais de hex inline.

---

## shadcn/ui v4

- **Package `shadcn` v4** installé en dépendance.
- **`radix-ui`** unifié (pas les packages `@radix-ui/*` séparés).
- **Composants dans `components/ui/`** — les personnaliser directement.
- **Pattern `React.ComponentProps<"element">`** — pas de `forwardRef`.
- **Formulaires** : `react-hook-form` + `zod` + composant `<Form>` shadcn.
- **CSS variables** via `@theme inline` dans `globals.css`.

### Composants shadcn obligatoires (INTERDIT de créer des composants maison)

**Toujours utiliser les composants shadcn existants** (`components/ui/`) pour les éléments d'interface. Ne JAMAIS recréer un composant qui existe déjà dans shadcn.

| Besoin | Composant shadcn | INTERDIT |
|--------|------------------|----------|
| Bouton | `<Button>` | `<button className="...">` custom |
| Champ texte | `<Input>` | `<input className="...">` custom |
| Zone de texte | `<Textarea>` | `<textarea className="...">` custom |
| Case à cocher | `<Checkbox>` | `<input type="checkbox">` custom |
| Sélecteur | `<Select>` | `<select>` natif ou custom dropdown |
| Dialogue | `<Dialog>` | Modal custom avec portail |
| Popover | `<Popover>` | Dropdown custom |
| Badge | `<Badge>` | `<span className="badge...">` custom |
| Card | `<Card>` | `<div className="card...">` custom |
| Tableau | `<Table>` | `<table className="...">` custom |
| Formulaire | `<Form>` + `<FormField>` | `<form>` avec `useState` manuel |
| Toast | `toast()` (sonner) | `alert()` ou notification custom |
| Tabs | `<Tabs>` | Tabs custom avec `useState` |
| Alert Dialog | `<AlertDialog>` | `window.confirm()` |

**Règles :**
- **Personnaliser** les composants shadcn via leurs `className` prop, pas en les recréant
- **Régénérer** avec `npx shadcn@latest add <component> --overwrite` si un composant est corrompu
- Après régénération, adapter au projet : `radix-ui` unifié (pas `@radix-ui/*`), `React.ComponentProps` (pas `forwardRef`)
- Les variantes custom (ex: `variant="sold"` sur Badge) se définissent **dans** le composant shadcn, pas dans un composant séparé

---

## Auth.js v5

- **Config** : `NextAuth()` → `{ handlers, auth, signIn, signOut }`.
- **`auth()`** dans les Server Components et Server Actions.
- **Middleware** : `auth` comme wrapper.
- **`AUTH_SECRET` / `AUTH_URL`** (pas les anciens `NEXTAUTH_*`).
- **JWT strategy** pour le serverless.

---

## Prisma

- **`directUrl` + `url`** configures.
- **Client singleton** (global pattern).
- **Pas de raw queries** sauf cas complexe justifie.

---

## Qualite de code

- **ESLint v9** flat config (`eslint.config.mjs`) avec `eslint-config-next`.
- **Prettier** avec `prettier-plugin-tailwindcss`.
- **`pnpm`** comme package manager (jamais npm/yarn).
- **`"type": "module"`** dans package.json.

---

## Principes transversaux

1. **Colocation** — fichiers proches de ou ils sont utilises.
2. **Pas d'abstraction prematuree** — dupliquer 2-3 fois avant d'extraire.
3. **Fail fast** — valider avec Zod, crash plutot que silently fail.
4. **Nommage** : `PascalCase` composants, `camelCase` fonctions, `kebab-case` fichiers.
5. **~200 lignes max** par fichier.
6. **Env vars** — valider avec Zod. Jamais `process.env` direct.

---

*Derniere mise a jour : mars 2026*
