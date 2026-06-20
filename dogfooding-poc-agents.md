# Dogfooding: migrar o `poc-agents` para consumir `@cmx-geral/ui`

> Plano passo a passo para o `poc-agents` parar de usar a cópia local em
> `frontend/src/components/ui` e passar a consumir o pacote `@cmx-geral/ui`.
> Serve de **template** para qualquer app da CMX adotar o design system.
>
> **Estado:** planejado, **não executado** (a pedido). Nada foi mexido no poc-agents.

---

## Impacto medido (poc-agents/frontend)

- **48 arquivos** importam de `components/ui` (fora do próprio `ui/`).
- Imports são **relativos** (`./ui/Button`, `../ui/Button`) — não há alias `@/`.
- **Todos** os componentes usados estão no pacote, **exceto `Chart`** (1 uso) — depende de `recharts`, **fica local**.
- Uso (contagem): Button 35 · Input 25 · Spinner 23 · Label 21 · Badge 19 · Select 17 · DropdownMenu 15 · Dialog 15 · Textarea 11 · Table 11 · Card 10 · Switch 7 · SecretInput 6 · Tabs 5 · Checkbox 4 · Avatar 4 · AlertDialog 2 · Tooltip/Sonner/ScrollArea 1 · **Chart 1 (não migra)**.

---

## Estratégia: shims de re-export (menor risco)

Em vez de varrer os 48 arquivos consumidores, **mantém o diretório `ui/`** e converte cada componente num re-export do pacote. Os 48 arquivos **não mudam** (continuam `import { Button } from './ui/Button'`).

```tsx
// frontend/src/components/ui/Button.tsx  (vira um shim)
export * from '@cmx-geral/ui'
```
Funciona porque o consumidor importa um nome específico (`Button`) que está no pacote. (Opcional, mais limpo: `export { Button, buttonVariants } from '@cmx-geral/ui'` por arquivo.)

**Exceções:**
- **`Chart.tsx`** — NÃO virar shim. Continua o arquivo local real (usa `recharts`, não está no pacote).
- **`Sonner.tsx`** — shim `export * from '@cmx-geral/ui'` (o consumidor faz `import { Toaster } from './ui/Sonner'`; o `Toaster` do pacote auto-detecta `.dark`).
- Se algum consumidor importar um **tipo** que o pacote não exporta, tratar aquele arquivo individualmente (named re-export + o tipo, ou ajustar o import).

Vantagem: superfície mínima, reversível (basta restaurar os arquivos `ui/`). Depois, opcionalmente, varrer os imports para `@cmx-geral/ui` direto e apagar os shims.

---

## CSS (atenção — é o ponto técnico)

Hoje o Tailwind do poc acha as classes dos componentes em `ui/*.tsx`. Com os shims, essas classes **somem da fonte do poc** → os componentes ficariam sem estilo.

**Correção:** no `frontend/src/main.tsx`, importar o CSS do pacote:
```ts
import '@cmx-geral/ui/styles.css'   // classes dos componentes + tokens, já compilados
import './index.css'                // mantém o CSS do poc (suas páginas + app-specific)
```
- Os dois usam **os mesmos tokens OKLCH**, então coexistem sem conflito (só duplica alguns bytes).
- Manter o `index.css` do poc (ele tem o `login-aurora`, scrollbar, e compila as classes das **páginas** do poc).
- A **fonte Geist**: o poc já carrega via `@import` no `index.css` — mantém. (O pacote não embute fonte.)

> Alternativa (sem duplicação, mais avançada): em vez de importar `styles.css`, apontar o Tailwind do poc para o pacote com `@source "../node_modules/@cmx-geral/ui/dist"` no `index.css`. Pipeline único. Deixar para depois — `import styles.css` é o caminho de menor risco.

---

## Autenticação (como o poc instala o pacote privado)

O `@cmx-geral/ui` é privado no GitHub Packages. Dois caminhos:

### a) Tarball local — para provar localmente (sem PAT)
```jsonc
// frontend/package.json
"dependencies": { "@cmx-geral/ui": "file:../../cmx-ui/cmx-geral-ui-0.1.0.tgz" }
```
Gera o tarball com `npm pack` no `cmx-ui`. Bom para validar o `npm run build` local. **Não** serve para Docker/CI/outros devs.

### b) Registry + PAT — caminho de produção
```
# frontend/.npmrc
@cmx-geral:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${NODE_AUTH_TOKEN}
```
- Dev local: `NODE_AUTH_TOKEN` = PAT (classic) com escopo **`read:packages`** no ambiente.
- **Docker** (o poc builda o frontend em container): passar o token como **build secret** e usar no `npm ci`:
  ```dockerfile
  # frontend Dockerfile
  RUN --mount=type=secret,id=npm_token \
      NODE_AUTH_TOKEN=$(cat /run/secrets/npm_token) npm ci
  ```
  e no compose/CI, fornecer o secret. O `.npmrc` acima referencia `${NODE_AUTH_TOKEN}`.
- **CI (deploy.yml)**: o `GITHUB_TOKEN` da Action já tem `read:packages` na própria org — basta `registry-url`/`scope` no setup-node ou o `.npmrc` com `${{ secrets.GITHUB_TOKEN }}`.

---

## Passo a passo (na ordem)

1. **Branch** a partir do remoto limpo (regra do poc):
   ```bash
   git fetch origin && git checkout -b chore/dogfood-cmx-ui origin/master
   ```
2. **Instalar o pacote** (escolher a/b acima). Para o 1º proof: tarball local.
3. **Converter os componentes em shims** (todos os `ui/*.tsx` **menos `Chart.tsx`**) para `export * from '@cmx-geral/ui'`.
4. **CSS**: no `main.tsx`, adicionar `import '@cmx-geral/ui/styles.css'` antes do `./index.css`.
5. **Validar**: `cd frontend && npm run build` (`tsc -b && vite build`) — pega erros de tipo e de CSS. Subir o app e conferir visualmente (no seu navegador, como manda o poc).
6. **Docker** (quando for além do local): ajustar o Dockerfile do frontend para a auth do registry (build secret) e testar `docker compose build frontend`.
7. **PR** contra o master (regra do poc): `git push -u origin chore/dogfood-cmx-ui && gh pr create`.

---

## Riscos & rollback
- **Reversível**: os shims são triviais de desfazer (restaurar os arquivos `ui/` originais via git). Nada é deletado de forma destrutiva.
- **Risco de CSS**: cobrir no passo 5 (build + visual). Se algo ficar sem estilo, conferir se o `styles.css` do pacote foi importado.
- **Risco de tipo**: o build (`tsc -b`) acusa qualquer import quebrado antes de subir.

## Follow-ups (depois do proof local)
1. **Registry + PAT no Docker/CI** (sair do tarball) — é o que torna o consumo "de verdade".
2. **Varrer imports** `./ui/X` → `@cmx-geral/ui` e **remover os shims** (end-state limpo).
3. **Self-host da fonte Geist** no pacote (`@font-face` + woff2) — tira a dependência do Google Fonts no `<link>`.
4. **`/design-sync` a partir do `cmx-ui`** — a fonte canônica passa a ser este repo.
