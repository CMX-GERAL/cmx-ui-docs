# Plano: Design System da CMX como pacote npm (`@cmx-geral/ui`)

> **Objetivo:** parar de ter cada app da CMX com um design diferente. Empacotar os componentes + tokens
> num pacote npm **privado** (GitHub Packages) que todo app instala e usa. Fonte única, atualização por versão.
>
> **Contexto:** apps em **repositórios separados**. A fonte dos componentes já existe em
> `poc-agents/frontend/src/components/ui` (shadcn portado verbatim, Tailwind v4 + tokens OKLCH, fontes Geist).

---

## Visão geral

```
   ┌─────────────────────────┐         publica          ┌──────────────────┐
   │  repo  cmx-ui (fonte)   │  ───────────────────────▶ │  GitHub Packages │
   │  componentes + tokens   │   (CI no merge/changeset) │  @cmx-geral/ui   │
   └─────────────────────────┘                           └────────┬─────────┘
                                                                   │ npm install
                          ┌────────────────────────────────────────┼───────────────┐
                          ▼                    ▼                     ▼               ▼
                       app A                app B                 app C          poc-agents
                  (instala + usa)      (instala + usa)       (instala + usa)   (passa a consumir)
```

O pacote entrega **3 camadas** a partir da mesma fonte:
1. **Tema** (`@cmx-geral/ui/theme.css`) — variáveis OKLCH, `@theme` do Tailwind, fontes. *A camada de marca.*
2. **Componentes** — `.tsx` compilados (ESM) **+ `.d.ts` reais** (tipos de verdade).
3. **CSS pronto** (`@cmx-geral/ui/styles.css`) — estilos dos componentes já compilados (funciona até sem Tailwind no app).

> **Nota de escopo (GitHub Packages):** o escopo npm **tem que ser o nome da org em minúsculas** → `@cmx-geral`.

---

## Fase 0 — Decisões antes de codar
1. **Fonte canônica:** este repo (`cmx-ui`) vira a fonte; o `poc-agents` passa a **consumir** `@cmx-geral/ui` (dogfooding).
2. **O que entra na v1:** primitivas (Button, Card, Dialog, Select, Table, Input, Badge, Switch, …) + tokens + `Pagination`. Os "patterns" (DataTable, MetricCard, …) ficam como exemplos/Storybook por ora.
3. **Componentes com atenção:** `Avatar` (API custom `name`/`src`/`size`, traz `lib/user.initials`); `Toaster`/`Sonner` → **auto-detecta a classe `.dark` do `<html>`** (sem `ThemeContext`, sem provider, sem prop — zero config no app).

## Fase 1 — Extrair a biblioteca
```
cmx-ui/
├─ src/
│  ├─ components/        # os .tsx de ui/
│  ├─ lib/{utils.ts,user.ts}
│  ├─ styles/theme.css   # tokens OKLCH + @theme + @import "tailwindcss" + fontes
│  └─ index.ts           # barrel
├─ package.json · tsconfig.json · tsup.config.ts
└─ .npmrc · .github/workflows/publish.yml
```
- **deps:** `@radix-ui/*`, `class-variance-authority`, `clsx`, `tailwind-merge`, `lucide-react`, `sonner`, `tw-animate-css`.
- **peerDependencies:** `react`, `react-dom`.

## Fase 2 — `package.json`
```jsonc
{
  "name": "@cmx-geral/ui",
  "version": "0.1.0",
  "type": "module",
  "sideEffects": ["**/*.css"],
  "files": ["dist"],
  "exports": {
    ".":            { "types": "./dist/index.d.ts", "import": "./dist/index.js" },
    "./styles.css": "./dist/styles.css",
    "./theme.css":  "./dist/theme.css"
  },
  "peerDependencies": { "react": ">=19", "react-dom": ">=19" },
  "publishConfig": { "registry": "https://npm.pkg.github.com" }
}
```
`tsup.config.ts`: `entry: ['src/index.ts'], format: ['esm'], dts: true, external: ['react','react-dom']` → emite os `.d.ts` reais.

## Fase 3 — O CSS (Tailwind v4)
- **`theme.css`** = marca: `@import "tailwindcss"` + `:root`/`.dark` OKLCH + `@theme` + fontes.
- **`styles.css`** = estilos dos componentes compilados (o Tailwind processa a fonte da lib).
- App sem Tailwind: importa `styles.css`. App com Tailwind: importa `theme.css` + aponta `@source` para `node_modules/@cmx-geral/ui`.

## Fase 4 — Publicar no GitHub Packages
- `.npmrc`: `@cmx-geral:registry=https://npm.pkg.github.com`
- Action de publish no merge (`permissions: packages: write`, `NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}`).

## Fase 5 — Versionamento
- **Changesets** — cada PR descreve a mudança (patch/minor/major); no merge bumpa versão + CHANGELOG + publica. Semver é o contrato.

## Fase 6 — Como cada app consome
1. `.npmrc` no app autenticando no GitHub Packages (PAT com `read:packages`).
2. `npm install @cmx-geral/ui`
3. `import '@cmx-geral/ui/theme.css'` + `import '@cmx-geral/ui/styles.css'` no entry.
4. `import { Button } from '@cmx-geral/ui'`

## Fase 7 — Conexão com o design-sync
Mesma fonte, duas saídas: o pacote (devs) e o claude.ai/design (IA/designers). Quando este repo for a fonte, o `/design-sync` roda a partir daqui; o `.design-sync/`, previews e docs de desvio vêm junto. O build de lib (`dts:true`) emite `.d.ts` reais → o design-sync ganha contratos de tipo ricos.

## Decisões tomadas (lock — 2026-06-20)
| Item | Decisão |
|---|---|
| **Fonte canônica** | `cmx-ui` é a fonte; `poc-agents` migra pra consumir `@cmx-geral/ui` (dogfooding, passo posterior) |
| **Tema do Toaster** | **auto-detecta `.dark`** do `<html>` (sem provider/prop) |
| **Patterns** | **só exemplos/docs** (Storybook), **não** exports na v1 |
| **Ícones** | **`lucide-react` como `dependency`** do pacote (usado internamente; não re-exportado na v1) |
| **Storybook** | **depois da v0.1** (PoC foca no pipeline; depois vira vitrine + fonte do design-sync) |
| **CSS** | **enviar os dois**: `theme.css` (tokens/fontes) + `styles.css` (estilos compilados) |
| **v1** | primitivas + tokens + `Pagination` (componente real); patterns ficam de fora (exemplos) |

## Faseamento (esforço)
1. **PoC (1–2 dias):** 3–4 componentes + tokens, build tsup, publicar `0.0.1`, instalar num app de teste.
2. **v0.1:** todas as primitivas + tokens + Pagination, CI de publish, changesets.
3. **Adoção:** app piloto consome; ajusta; demais apps.
4. **Consolidar:** `poc-agents` consome o pacote; `/design-sync` roda daqui.
