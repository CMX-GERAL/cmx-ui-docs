# Playbook — `@cmx-geral/ui`

Guia central do design system da CMX. Três frentes:
- **[Parte 1 — Consumir](#parte-1--consumir-devs-de-app)** (devs de qualquer app)
- **[Parte 2 — Manter / evoluir](#parte-2--manter--evoluir-donos-do-design-system)** (donos do `cmx-ui`)
- **[Parte 3 — Adotar](#parte-3--adotar-migrar-um-app-existente)** (migrar um app existente)

> **Princípio:** uma fonte (`cmx-ui`), duas saídas — o **pacote npm** (engenheiros) e o **claude.ai/design** via `/design-sync` (IA + designers). Mudou um componente aqui, atualiza os dois canais.

**Stack:** React 19 · Tailwind v4 (tokens OKLCH + `@theme`) · shadcn/ui (portado verbatim) · Radix · fontes Geist. Pacote **privado** no GitHub Packages, escopo `@cmx-geral`.

---

## Parte 1 — Consumir (devs de app)

### 1.1 Autenticar no GitHub Packages
O pacote é privado. Crie um **PAT (classic)** com escopo **`read:packages`** e configure o `.npmrc` do app:
```
# .npmrc (na raiz do app)
@cmx-geral:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${NODE_AUTH_TOKEN}
```
- **Local:** exporte `NODE_AUTH_TOKEN=<seu PAT>` no ambiente (ou `.env` lido pelo shell). Nunca commite o token.
- **CI (GitHub Actions):** use `${{ secrets.GITHUB_TOKEN }}` — já tem `read:packages` na org.
- **Docker:** passe como build secret (ver [Parte 3](#34-docker)).

### 1.2 Instalar
```bash
npm install @cmx-geral/ui
```

### 1.3 CSS + fonte (no entry do app, ex. `main.tsx`)
```ts
import '@cmx-geral/ui/styles.css'   // estilos dos componentes + tokens (funciona sozinho)
```
A **fonte Geist** é carregada pelo app — um `<link>` no `index.html`:
```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link href="https://fonts.googleapis.com/css2?family=Geist:wght@300..700&family=Geist+Mono:wght@400..600&display=swap" rel="stylesheet" />
```

### 1.4 Usar
```tsx
import { Button, Card, CardHeader, CardTitle, Dialog, Select } from '@cmx-geral/ui'

<Button variant="outline" size="sm">Salvar</Button>
```
Os tipos vêm com o pacote (autocomplete de `variant`/`size` etc.). Catálogo: AlertDialog, Avatar, Badge, Button, Card, Checkbox, Dialog, DropdownMenu, Input, Label, Pagination, ScrollArea, SecretInput, Select, Spinner, Switch, Table, Tabs, Textarea, Toaster, Tooltip + o helper `cn`.

### 1.5 Dark mode
Os tokens têm tema claro/escuro. Para o tema escuro, **ligue a classe `dark` no `<html>`** (elemento raiz):
```ts
document.documentElement.classList.toggle('dark', isDark)
```
Todos os utilitários (`bg-card`, `text-foreground`…) resolvem os valores escuros. O `Toaster` **auto-detecta** essa classe (não precisa de provider nem prop).

### 1.6 App que usa Tailwind v4 (opcional)
Se o app também escreve utilitários da marca (`bg-primary`) no próprio código, importe os tokens e aponte o Tailwind pro pacote:
```css
/* no CSS do app */
@import "tailwindcss";
@import "@cmx-geral/ui/theme.css";       /* tokens + @theme */
@source "../node_modules/@cmx-geral/ui/dist";  /* gera as classes dos componentes */
```
(Nesse caminho, **não** importe o `styles.css` — o pipeline do app gera tudo.)

---

## Parte 2 — Manter / evoluir (donos do design system)

### 2.1 Regra de ouro: shadcn **verbatim**
Componentes de UI são **portados literalmente** do shadcn oficial — nunca estilizar "parecido" à mão. Fontes:
- Registry: `https://ui.shadcn.com/r/styles/new-york-v4/<componente>.json` (campo `files[0].content`).
- Estilo radix-nova: `github.com/shadcn-ui/ui` → `apps/v4/styles/radix-nova/ui/*.tsx`.

**Adaptações permitidas (só infra):**
- `radix-ui` (meta-pacote) → pacote individual (`@radix-ui/react-dialog`, etc.) — e adicionar como **dependency**.
- `@/lib/utils` → `../lib/utils`; ícones via `lucide-react`.
- Remover `"use client"` (não é Next).

### 2.2 Adicionar / atualizar um componente
1. Crie `src/components/<Nome>.tsx` (porte verbatim + adaptações acima).
2. Se usar um pacote Radix novo, adicione em `dependencies` no `package.json`.
3. Exporte no barrel `src/index.ts`: `export * from './components/<Nome>'`.
4. `npm run build` e confira `dist/` (ESM + `.d.ts` + as classes no `styles.css`).
5. Adicione um **changeset** (ver 2.5).

### 2.3 Tokens (a marca)
- Em `src/styles/theme.css`: OKLCH em `:root`/`.dark`, mapeados no `@theme` do Tailwind. **Não** "neutralizar" o `--primary` (roxo/azul da marca).
- O `index.css` (entrada de build) faz `@import "tailwindcss"` + `@import "tw-animate-css"` + `@import "./theme.css"` + `@source "../components"`.

### 2.4 Desvios conscientes (não regredir)
- **Avatar** é **custom** (não shadcn): props `name`/`src`/`size`, gera iniciais. **Não** existe `AvatarFallback`/`AvatarImage`.
- **Toaster** auto-detecta a classe `.dark` do `<html>` (sem `ThemeContext`/prop).
- **Chart** fica **fora** do pacote (depende de `recharts`, é app-level).
- **Fonte** não é embutida (carregada pelo app) — `@import` remoto não fica estável no CSS compilado do Tailwind v4.

### 2.5 Lançar uma versão (Changesets)
O bump de versão + CHANGELOG saem dos changesets; a publicação é automática no CI. **Não** usamos a PR automática "Version Packages" (a org não libera a Action abrir PRs), então o bump é um comando local:
```bash
# 1. em cada PR que muda a UI:
npx changeset            # patch/minor/major + resumo → commita o .changeset/

# 2. pra lançar (mantenedor, local):
git checkout main && git pull
npm run version          # consome os changesets → bump + CHANGELOG
git add -A && git commit -m "Version Packages" && git push

# 3. o CI publica sozinho no push da main (changeset publish + tag)
```
Semver é o contrato: visual sem quebrar API = `minor`; renomear/remover prop = `major`; correção = `patch`.

### 2.6 Build (o que cada artefato é)
`npm run build` gera `dist/`:
- `index.js` (ESM) + `index.d.ts` (**tipos reais**) — via `tsup`.
- `styles.css` — Tailwind compila os componentes + tokens (o "just works").
- `theme.css` — só os tokens (pros apps Tailwind).

### 2.7 Conexão com o `/design-sync`
A mesma fonte alimenta o claude.ai/design. Quando rodar `/design-sync` a partir deste repo, o `.design-sync/` (config, previews, docs de desvio) vem junto, e o build de lib (`dts:true`) dá contratos de tipo ricos pro agente.

---

## Parte 3 — Adotar (migrar um app existente)

Runbook repetível pra um app sair da cópia local de componentes e consumir `@cmx-geral/ui`. (Exemplo trabalhado: `dogfooding-poc-agents.md`.)

### 3.1 Medir o impacto
```bash
# quantos arquivos importam dos componentes locais, e quais componentes
grep -rl "components/ui/" src | wc -l
```
Liste os componentes usados e veja quais **não** estão no pacote (ex.: Chart) — esses ficam locais.

### 3.2 Estratégia de menor risco: shims
Mantém o diretório de componentes local e converte cada um num re-export — **os arquivos consumidores não mudam**:
```tsx
// src/components/ui/Button.tsx  → vira shim
export * from '@cmx-geral/ui'
```
Exceções: componentes fora do pacote (Chart) ficam reais; `Sonner`/`Toaster` re-exporta o `Toaster` do pacote. Reversível via git.

### 3.3 CSS
Os componentes precisam das classes compiladas. No entry do app:
```ts
import '@cmx-geral/ui/styles.css'
import './index.css'   // mantém o CSS do app (páginas + app-specific + fonte)
```
Coexistem porque usam os **mesmos tokens**. (Alternativa avançada: `@source` o pacote no Tailwind do app — pipeline único, sem duplicação.)

### 3.4 Docker
Se o app builda em container, autentique o `npm ci` com o PAT como **build secret**:
```dockerfile
RUN --mount=type=secret,id=npm_token \
    NODE_AUTH_TOKEN=$(cat /run/secrets/npm_token) npm ci
```
e forneça o secret no compose/CI.

### 3.5 Validar e entregar
1. `npm run build` (type-check + bundle + CSS).
2. Subir o app e conferir visualmente.
3. Branch + PR (siga as regras do app — ex.: o poc exige branch de `origin/master` e entrega por PR).

### Checklist de adoção
- [ ] `.npmrc` + PAT configurados
- [ ] `npm install @cmx-geral/ui`
- [ ] componentes locais → shims (menos os fora do pacote)
- [ ] `import '@cmx-geral/ui/styles.css'` no entry
- [ ] fonte Geist no `index.html`
- [ ] dark mode liga `.dark` no `<html>`
- [ ] `npm run build` passa + conferência visual
- [ ] Docker/CI com a auth do registry
- [ ] PR

---

## Troubleshooting

| Sintoma | Causa provável | Correção |
|---|---|---|
| **401/403 no `npm install`** | PAT ausente/sem `read:packages` | Conferir o `.npmrc` + `NODE_AUTH_TOKEN` (Parte 1.1) |
| **Componentes sem estilo** | Faltou o CSS do pacote | `import '@cmx-geral/ui/styles.css'` no entry |
| **Fonte errada (system-ui)** | Geist não carregada | Adicionar o `<link>` da fonte no `index.html` |
| **Dark mode não muda** | `.dark` não está no `<html>` | `document.documentElement.classList.toggle('dark', …)` |
| **`AvatarFallback is not exported`** | Avatar é custom | Usar `<Avatar name="..." size={n} />` (Parte 2.4) |
| **Estilos duplicados/inflados** | Dois builds Tailwind (app + pacote) | OK se os tokens batem; pra zerar, usar o caminho `@source` (Parte 1.6) |
| **CI não publica** | Sem versão nova | Rodar `npm run version` local antes (Parte 2.5) |

---

## Referência rápida
```bash
# Consumir
npm install @cmx-geral/ui

# Manter (no cmx-ui)
npm run build            # dist/ (ESM + .d.ts + CSS)
npx changeset            # registrar uma mudança
npm run version          # bump + CHANGELOG (local, antes de lançar)

# Links
# Repo:     github.com/CMX-GERAL/cmx-ui
# Plano:    PLAN.md
# Dogfood:  dogfooding-poc-agents.md
```
