# Playbook — guia de uso do `@cmx-geral/ui`

Como usar o design system da CMX num app: instalar, os componentes, os padrões e os tokens.

> Quem **mantém** o pacote (portar componente, release): ver **[Contribuir](CONTRIBUIR.md)**.

---

## 1. Começar

### Autenticar (pacote privado no GitHub Packages)
Crie um **PAT (classic)** com escopo `read:packages` e configure o `.npmrc` na raiz do app:
```
@cmx-geral:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${NODE_AUTH_TOKEN}
```
Local: `NODE_AUTH_TOKEN=<PAT>` no ambiente. CI: `${{ secrets.GITHUB_TOKEN }}`. Nunca commite o token.

### Instalar e ligar
```bash
npm install @cmx-geral/ui
```
```ts
// entry do app (main.tsx)
import '@cmx-geral/ui/styles.css'   // estilos dos componentes + tokens
```
```html
<!-- index.html: a fonte Geist é carregada pelo app -->
<link href="https://fonts.googleapis.com/css2?family=Geist:wght@300..700&family=Geist+Mono:wght@400..600&display=swap" rel="stylesheet" />
```

### Dark mode
Ligue a classe `dark` no `<html>` — todos os tokens trocam pros valores escuros:
```ts
document.documentElement.classList.toggle('dark', isDark)
```

### Primeiro componente
```tsx
import { Button } from '@cmx-geral/ui'

<Button variant="outline" size="sm">Salvar</Button>
```
Tipos vêm com o pacote (autocomplete de `variant`/`size`).

---

## 2. Componentes

### Button
```tsx
import { Button } from '@cmx-geral/ui'
<Button>Salvar</Button>
<Button variant="outline" size="sm">Editar</Button>
<Button variant="destructive">Excluir</Button>
<Button variant="ghost" size="icon-sm"><Icon /></Button>
```
**variant:** `default` · `secondary` · `outline` · `ghost` · `destructive` · `link`
**size:** `default` · `sm` · `lg` · `xs` · `icon` · `icon-sm` · `icon-xs` · `icon-lg`
`asChild` para envolver um link/outro elemento.

### Badge
```tsx
import { Badge } from '@cmx-geral/ui'
<Badge>Novo</Badge>
<Badge variant="secondary">Beta</Badge>
// Status (padrão do app): outline + dot
<Badge variant="outline"><span className="size-1.5 rounded-full bg-emerald-500" /> Ativo</Badge>
```
**variant:** `default` · `secondary` · `outline` · `destructive` · `ghost` · `link`

### Card
```tsx
import { Card, CardHeader, CardTitle, CardDescription, CardContent, CardFooter } from '@cmx-geral/ui'
<Card>
  <CardHeader><CardTitle>…</CardTitle><CardDescription>…</CardDescription></CardHeader>
  <CardContent>…</CardContent>
  <CardFooter><Button size="sm">Salvar</Button></CardFooter>
</Card>
```

### Avatar ⚠️ API custom
**Não é o Avatar do shadcn** — não existe `AvatarFallback`/`AvatarImage`.
```tsx
import { Avatar } from '@cmx-geral/ui'
<Avatar name="Edmilson Junior" />           // gera as iniciais
<Avatar name="Empresa X" src="/logo.png" /> // imagem (fallback p/ iniciais)
<Avatar name="X" size={48} />               // tamanho em px (default 32)
```
Props: `name` · `src` · `size`. Circular por padrão (`style={{ borderRadius: 'var(--radius-lg)' }}` p/ quadrado).

### Spinner
```tsx
import { Spinner } from '@cmx-geral/ui'
<Spinner />
<Button disabled><Spinner /> Carregando…</Button>
```

### Campos de formulário (Input, Label, Textarea)
```tsx
import { Input, Label, Textarea } from '@cmx-geral/ui'
<div className="grid gap-3">
  <Label htmlFor="nome">Nome</Label>
  <Input id="nome" placeholder="Ex: Agente de Vendas" />
</div>
<Textarea placeholder="Descrição…" />
```

### SecretInput ⚠️ use p/ credenciais
```tsx
import { SecretInput } from '@cmx-geral/ui'
<SecretInput value={key} onChange={…} placeholder="sk-…" />
```
**Não** use `<Input type="password">` para tokens/keys — o `SecretInput` mascara, tem mostrar/ocultar e evita autofill/gerenciador de senha. Mesma API do `Input`.

### Checkbox e Switch
```tsx
import { Checkbox, Switch, Label } from '@cmx-geral/ui'
<Checkbox id="aceito" /> <Label htmlFor="aceito">Aceito os termos</Label>
<Switch checked={on} onCheckedChange={setOn} />
```

### Select ⚠️ sem `value=""`
```tsx
import { Select, SelectTrigger, SelectValue, SelectContent, SelectItem, SelectGroup, SelectLabel } from '@cmx-geral/ui'
<Select value={x || undefined} onValueChange={setX}>
  <SelectTrigger><SelectValue placeholder="Selecione…" /></SelectTrigger>
  <SelectContent>
    <SelectGroup>
      <SelectLabel>OpenAI</SelectLabel>
      <SelectItem value="gpt-4o">GPT-4o</SelectItem>
    </SelectGroup>
  </SelectContent>
</Select>
```
**Desvio:** `SelectItem` **não aceita `value=""`**. Caso "Selecione…" → `placeholder` no `SelectValue` + `value={x || undefined}`.

### Dialog (modal)
```tsx
import { Dialog, DialogTrigger, DialogContent, DialogHeader, DialogTitle, DialogDescription, DialogFooter } from '@cmx-geral/ui'
<Dialog>
  <DialogTrigger asChild><Button>Novo</Button></DialogTrigger>
  <DialogContent className="sm:max-w-[425px]">
    <DialogHeader><DialogTitle>Criar agente</DialogTitle><DialogDescription>…</DialogDescription></DialogHeader>
    <form className="grid gap-4">…</form>
    <DialogFooter><Button variant="outline">Cancelar</Button><Button>Salvar</Button></DialogFooter>
  </DialogContent>
</Dialog>
```
**Desvios:** `DialogContent` usa `bg-card` + `rounded-xl`; overlay sem dim (só blur); `DialogFooter` é uma faixa. Form longo: nunca `overflow-y-auto` no content (a faixa do footer rola junto). Aberto a partir de um `DropdownMenu` → passe `modal={false}`.

### AlertDialog (confirmação)
```tsx
import { AlertDialog, AlertDialogTrigger, AlertDialogContent, AlertDialogHeader, AlertDialogTitle, AlertDialogDescription, AlertDialogFooter, AlertDialogCancel, AlertDialogAction } from '@cmx-geral/ui'
<AlertDialog>
  <AlertDialogTrigger asChild><Button variant="destructive">Excluir</Button></AlertDialogTrigger>
  <AlertDialogContent>
    <AlertDialogHeader><AlertDialogTitle>Confirmar?</AlertDialogTitle><AlertDialogDescription>…</AlertDialogDescription></AlertDialogHeader>
    <AlertDialogFooter><AlertDialogCancel>Cancelar</AlertDialogCancel><AlertDialogAction>Excluir</AlertDialogAction></AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>
```
Use para confirmações destrutivas em vez de `window.confirm`.

### DropdownMenu
```tsx
import { DropdownMenu, DropdownMenuTrigger, DropdownMenuContent, DropdownMenuItem, DropdownMenuSeparator, DropdownMenuLabel } from '@cmx-geral/ui'
<DropdownMenu>
  <DropdownMenuTrigger asChild><Button variant="ghost" size="icon-sm"><MoreHorizontal /></Button></DropdownMenuTrigger>
  <DropdownMenuContent align="end">
    <DropdownMenuItem>Editar</DropdownMenuItem>
    <DropdownMenuItem variant="destructive">Excluir</DropdownMenuItem>
  </DropdownMenuContent>
</DropdownMenu>
```

### Tooltip (precisa do Provider)
```tsx
import { Tooltip, TooltipTrigger, TooltipContent, TooltipProvider } from '@cmx-geral/ui'
<TooltipProvider>
  <Tooltip>
    <TooltipTrigger asChild><Button variant="ghost" size="icon-sm"><Info /></Button></TooltipTrigger>
    <TooltipContent>Copiar ID</TooltipContent>
  </Tooltip>
</TooltipProvider>
```

### Tabs
```tsx
import { Tabs, TabsList, TabsTrigger, TabsContent } from '@cmx-geral/ui'
<Tabs defaultValue="agentes">
  <TabsList><TabsTrigger value="agentes">Agentes</TabsTrigger><TabsTrigger value="tools">Ferramentas</TabsTrigger></TabsList>
  <TabsContent value="agentes">…</TabsContent>
  <TabsContent value="tools">…</TabsContent>
</Tabs>
```

### Table
```tsx
import { Table, TableHeader, TableBody, TableRow, TableHead, TableCell } from '@cmx-geral/ui'
<Table>
  <TableHeader><TableRow><TableHead>Agente</TableHead><TableHead>Status</TableHead></TableRow></TableHeader>
  <TableBody>
    <TableRow><TableCell>…</TableCell><TableCell>…</TableCell></TableRow>
  </TableBody>
</Table>
```
Estado vazio: `TableRow` com `TableCell colSpan={n}` `className="h-24 text-center text-muted-foreground"`. Ver o padrão **tabela** em [Padrões](#3-padrões).

### Pagination
```tsx
import { Pagination } from '@cmx-geral/ui'
<Pagination page={page} pageSize={20} total={total} onPageChange={setPage} />
```
Rodapé "Mostrando X–Y de N" + Anterior/Próxima. Não renderiza nada quando cabe numa página.

### ScrollArea
```tsx
import { ScrollArea } from '@cmx-geral/ui'
<ScrollArea className="h-72">…lista longa…</ScrollArea>
```

### Toaster (notificações)
Monte o `<Toaster />` uma vez (no topo do app) e dispare com `toast` do `sonner`:
```tsx
import { Toaster } from '@cmx-geral/ui'
import { toast } from 'sonner'

<Toaster />                          // uma vez, no root
toast.success('Agente criado')       // em qualquer lugar
toast.error('Falha ao salvar')
```
Auto-detecta o tema pela classe `.dark` do `<html>` (sem provider).

---

## 3. Padrões

Receitas que combinam os componentes — repita-as para manter as telas consistentes.

- **Status em tabela:** `Badge variant="outline"` + dot `size-1.5 rounded-full` (`bg-emerald-500` ativo · `bg-amber-500` pendente · `bg-red-500` erro).
- **Ação de linha:** última coluna com `DropdownMenu` (trigger `Button ghost icon-sm` + `MoreHorizontal`); item destrutivo `variant="destructive"`.
- **Formulário em modal:** `DialogContent sm:max-w-[425px]`, form `grid gap-4`, cada campo `grid gap-3` (Label + controle), `DialogFooter` com Cancelar `outline` + submit.
- **Card de métrica:** `Card` com label `text-sm text-muted-foreground`, número `text-3xl font-bold`, delta `text-xs`.
- **Linha de configurações:** seção titulada + `Card` dividido; cada linha = label+descrição à esquerda, controle (`Switch`) numa coluna fixa à direita.
- **Tabela full-bleed:** wrapper `-mx-6` (anula o padding do layout) + `pl-6`/`pr-6` nas colunas das pontas; header `hover:bg-transparent`.

---

## 4. Tokens

Sempre use os tokens semânticos (nunca cor/tamanho fixo).

**Cor** — `bg-background` · `text-foreground` · `bg-card` · `text-muted-foreground` · `border` · `bg-primary` (marca) · `bg-secondary` · `bg-accent` · `bg-destructive`. Todos têm versão dark (via `.dark` no `<html>`).

**Tipografia** — fontes `font-sans` (Geist) e `font-mono` (Geist Mono); escala `text-xs`→`text-3xl`; pesos `font-normal`→`font-bold`.

**Espaçamento e raio** — espaçamento base `0.25rem` (`gap-2`, `p-4`…); raio `rounded-sm`→`rounded-xl` (base `0.625rem`) + `rounded-full`.

**Elevação** — `shadow-xs` (repouso) · `shadow-sm` (hover) · `shadow-md` (dropdown) · `shadow-lg` (dialog). No dark a separação vem da borda + superfície, não da sombra.
