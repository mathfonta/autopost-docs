# Epic 4 — Frontend Dashboard

**Status:** Planning  
**Início estimado:** 2026-04-18  
**Stack:** Next.js 14 + TypeScript + shadcn/ui + Tailwind CSS  
**Deploy:** Vercel  
**Depende de:** Epic 1, Epic 2 (Backend MVP 100% em produção)

---

## Visão

O dashboard AutoPost deve **imitar a familiaridade do Instagram** — o cliente já sabe usar Instagram,
então a interface deve ser intuitiva sem treinamento. Mobile-first, funciona como app no celular (PWA).

O objetivo não é um painel de admin genérico. É uma interface que o dono de uma construtora pequena
abre no celular, vê a foto que o funcionário tirou, aprova com um toque, e o post vai ao ar.

---

## Proposta de Valor

| Usuário | Problema Atual | Solução |
|---------|---------------|---------|
| Cliente SMB (ex: Espectra Construção) | Precisa abrir Postman/curl para aprovar posts | Toque em "Aprovar" no celular |
| Cliente SMB | Não vê como o post vai ficar antes de publicar | Preview estilo Instagram |
| Cliente SMB | Não sabe se o sistema está funcionando | Dashboard com status em tempo real |

---

## Stories

| Story | Título | Pontos | Prioridade |
|-------|--------|--------|------------|
| 4.1 | Setup Next.js + Vercel + Autenticação | 3 | 🔴 Must Have |
| 4.2 | Dashboard Principal + Fila de Aprovação | 5 | 🔴 Must Have |
| 4.3 | Preview do Post (estilo Instagram) | 3 | 🔴 Must Have |
| 4.4 | Onboarding Wizard (conectar Instagram) | 3 | 🔴 Must Have |
| 4.5 | Histórico de Posts + Métricas | 2 | 🟡 Should Have |
| 4.6 | PWA + Notificações Push | 2 | 🟡 Should Have |

**Total MVP (4.1–4.4):** 14 pontos  
**Total completo:** 18 pontos

---

## Princípios de Design

1. **Mobile-first** — 90% dos clientes usam celular
2. **Imitar Instagram** — card de post, stories-like, paleta familiar
3. **Zero fricção na aprovação** — botão grande, visível, uma tela
4. **Feedback imediato** — loading states, toast notifications, status em tempo real
5. **PWA** — pode ser instalado na home screen do celular

---

## Arquitetura Frontend

```
Next.js 14 (App Router)
├── /app
│   ├── (auth)/login          # Login/Register
│   ├── (auth)/onboarding     # Wizard pós-registro
│   ├── dashboard/            # Fila de aprovação + status
│   ├── posts/[id]/           # Preview do post individual
│   └── history/              # Histórico + métricas
├── components/
│   ├── PostCard              # Card estilo Instagram
│   ├── ApprovalQueue         # Lista de posts pendentes
│   ├── PostPreview           # Preview 1:1 do post final
│   └── MetricsChart          # Gráfico de alcance/likes
└── lib/
    ├── api.ts                # Cliente da Railway API
    └── auth.ts               # Auth via Supabase client
```

---

## Integração com Backend

- **Auth:** JWT via `POST /auth/login` → token no cookie httpOnly
- **Posts:** `GET /content-requests` → lista com status
- **Aprovação:** `POST /content-requests/{id}/approve`
- **Status real-time:** polling a cada 5s (WebSocket em v1.1)
- **Onboarding:** `POST /onboarding/start` + `POST /onboarding/message`
- **OAuth Meta:** redirect para `GET /meta/connect`

---

## Backlog v1.1

| Item | Descrição | Origem |
|------|-----------|--------|
| Cookie httpOnly | Migrar auth de `js-cookie` para Route Handler Next.js + cookie httpOnly para eliminar risco XSS | @architect review Story 4.1 |
| Imagens R2 no browser | `r2.cloudflarestorage.com` é endpoint S3 privado — browser não carrega. Backend deve retornar presigned URLs no `GET /content-requests`, ou habilitar R2 Public Dev URL. Frontend já tem placeholder `ImageOff` como fallback. | Confirmado Story 4.2 |
| WebSocket | Substituir polling 5s por WebSocket para reduzir chamadas e melhorar UX de status em tempo real | Planejado desde Epic 4 |
| Token Meta auto-renovação | Renovação automática do token Meta (60 dias) | Planejado desde Epic 2 |

---

## Variáveis de Ambiente (Vercel)

```env
NEXT_PUBLIC_API_URL=https://espectra-api-production.up.railway.app
NEXT_PUBLIC_SUPABASE_URL=...
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
```
