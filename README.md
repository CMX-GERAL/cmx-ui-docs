# @cmx-geral/ui — Documentação

Documentação do **design system da CMX**. Componentes React, tokens (OKLCH) e tema compartilhados entre os apps, distribuídos como pacote npm privado (`@cmx-geral/ui`).

> Esta é a documentação **pública** (processo/guia). O código-fonte (componentes e tokens) vive num **repositório privado** — o acesso ao pacote é via GitHub Packages.

## Comece por aqui

- **[📘 Playbook](PLAYBOOK.md)** — o guia central do time, em 3 frentes:
  - **Consumir** — instalar e usar o pacote num app
  - **Manter / evoluir** — portar componentes, tokens, release
  - **Adotar** — migrar um app existente pro pacote
- **[🗺️ Plano do projeto](PLAN.md)** — roteiro e decisões de arquitetura.
- **[🔄 Adoção (dogfooding)](dogfooding-poc-agents.md)** — runbook de migração (exemplo: poc-agents).

## Em uma linha
```bash
npm install @cmx-geral/ui
```
```tsx
import '@cmx-geral/ui/styles.css'
import { Button, Card } from '@cmx-geral/ui'
```
Detalhes de autenticação, CSS, fonte e dark mode no [Playbook](PLAYBOOK.md#parte-1--consumir-devs-de-app).
