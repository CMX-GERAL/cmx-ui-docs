# Contribuir — manter e evoluir o `@cmx-geral/ui`

Para quem **mantém** o pacote (não é necessário para quem só **usa** — esse é o [Playbook](PLAYBOOK.md)).

---

## Regra de ouro: shadcn **verbatim**
Componentes de UI são **portados literalmente** do shadcn oficial — nunca estilizar "parecido" à mão. Fontes:
- Registry: `https://ui.shadcn.com/r/styles/new-york-v4/<componente>.json` (campo `files[0].content`).
- Estilo radix-nova: `github.com/shadcn-ui/ui` → `apps/v4/styles/radix-nova/ui/*.tsx`.

**Adaptações permitidas (só infra):**
- `radix-ui` (meta-pacote) → pacote individual (`@radix-ui/react-dialog`, etc.) — e adicionar como **dependency**.
- `@/lib/utils` → `../lib/utils`; ícones via `lucide-react`.
- Remover `"use client"` (não é Next).

## Adicionar / atualizar um componente
1. Crie `src/components/<Nome>.tsx` (porte verbatim + adaptações acima).
2. Se usar um pacote Radix novo, adicione em `dependencies` no `package.json`.
3. Exporte no barrel `src/index.ts`: `export * from './components/<Nome>'`.
4. `npm run build` e confira `dist/` (ESM + `.d.ts` + as classes no `styles.css`).
5. Adicione um **changeset** (ver Release).

## Tokens (a marca)
- `src/styles/theme.css`: OKLCH em `:root`/`.dark`, mapeados no `@theme` do Tailwind. **Não** "neutralizar" o `--primary` (azul/roxo da marca).
- `src/styles/index.css` (entrada de build) faz `@import "tailwindcss"` + `@import "tw-animate-css"` + `@import "./theme.css"` + `@source "../components"`.

## Desvios conscientes (não regredir)
- **Avatar** é **custom** (não shadcn): props `name`/`src`/`size`. Não existe `AvatarFallback`/`AvatarImage`.
- **Toaster** auto-detecta a classe `.dark` do `<html>` (sem `ThemeContext`/prop).
- **Chart** fica **fora** do pacote (depende de `recharts`, é app-level).
- **Fonte** não é embutida (carregada pelo app) — `@import` remoto não fica estável no CSS compilado do Tailwind v4.

## Build (o que cada artefato é)
`npm run build` gera `dist/`:
- `index.js` (ESM) + `index.d.ts` (**tipos reais**) — via `tsup`.
- `styles.css` — Tailwind compila os componentes + tokens (o "just works").
- `theme.css` — só os tokens (pros apps que usam Tailwind).

## Release (Changesets)
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

## Conexão com o `/design-sync`
A mesma fonte alimenta o claude.ai/design. Rodando `/design-sync` a partir deste repo, o `.design-sync/` (config, previews, docs de desvio) vem junto, e o build de lib (`dts:true`) dá contratos de tipo ricos pro agente.

---

> Docs internos (não publicados): o **plano do projeto** e o **runbook de adoção/dogfooding** ficam no repositório privado `cmx-ui` (`PLAN.md`, `docs/dogfooding-poc-agents.md`).
