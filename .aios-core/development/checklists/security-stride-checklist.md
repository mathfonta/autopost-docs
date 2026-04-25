# Security Checklist — STRIDE Threat Model (AutoPost)

**Agente:** @security (Sage)
**Uso:** Executado durante `*threat-model`. Aplicar por componente/Epic/Story.

---

## Como usar este checklist

Para cada componente sendo analisado, percorra as 6 categorias STRIDE.
Registre cada ameaça identificada com: descrição, impacto, mitigação existente e status.

---

## S — Spoofing (Falsificação de Identidade)

> "Um atacante pode se passar por outro usuário ou sistema?"

- [ ] Tokens JWT verificados em cada request protegido (não apenas no login)
- [ ] OAuth state parameter validado no callback Meta (previne CSRF no OAuth)
- [ ] Webhook de Meta (se implementado) valida assinatura HMAC
- [ ] Chaves de API de serviços externos não expostas a usuários comuns
- [ ] Presigned URLs do R2 têm TTL curto (não reutilizáveis indefinidamente)
- [ ] Identidade do usuário no Celery task verificada (não apenas o job ID)

**Ameaças identificadas:**
| ID | Descrição | Componente | Impacto | Mitigação | Status |
|----|-----------|-----------|---------|-----------|--------|
| | | | | | |

---

## T — Tampering (Adulteração de Dados)

> "Dados podem ser modificados sem autorização?"

- [ ] Transições de status de `content_request` validadas server-side (ex: não pode ir de `pending` direto para `published`)
- [ ] Caption editada rejeitada se status ≠ `awaiting_approval`
- [ ] IDs de objetos R2 gerados server-side (não controlados pelo usuário)
- [ ] Campos `user_id` em criação de registros sempre definidos pelo servidor (nunca pelo body da request)
- [ ] Migrations Alembic não alteráveis por usuários do sistema
- [ ] Dados do brand_profile não alteráveis por outros usuários

**Ameaças identificadas:**
| ID | Descrição | Componente | Impacto | Mitigação | Status |
|----|-----------|-----------|---------|-----------|--------|
| | | | | | |

---

## R — Repudiation (Repúdio de Ações)

> "Uma ação pode ser negada sem trilha de auditoria?"

- [ ] Publicações no Instagram logadas com: user_id, content_request_id, timestamp, resposta da Meta API
- [ ] Aprovações e rejeições de content_request logadas com user_id e timestamp
- [ ] Falhas de autenticação logadas com timestamp e IP
- [ ] Geração de copy logada com parâmetros usados (para reproduced audits)
- [ ] Ações de admin (se existirem) com log separado

**Ameaças identificadas:**
| ID | Descrição | Componente | Impacto | Mitigação | Status |
|----|-----------|-----------|---------|-----------|--------|
| | | | | | |

---

## I — Information Disclosure (Exposição de Informação)

> "Dados sensíveis podem ser expostos a quem não deveria ver?"

- [ ] Supabase RLS impede leitura cross-tenant em todas as tabelas com dados de usuário
- [ ] Tokens Meta retornados apenas para o dono da conta (nunca em listagens gerais)
- [ ] Erros 500 retornam mensagem genérica (não stack trace com paths internos)
- [ ] Logs do Railway não contêm: tokens, senhas, connection strings
- [ ] Imagens R2 acessíveis apenas via presigned URL (bucket privado)
- [ ] Endpoint `/meta/status` retorna apenas dados do usuário autenticado
- [ ] brand_profile de um usuário inacessível a outros usuários

**Ameaças identificadas:**
| ID | Descrição | Componente | Impacto | Mitigação | Status |
|----|-----------|-----------|---------|-----------|--------|
| | | | | | |

---

## D — Denial of Service (Negação de Serviço)

> "O serviço pode ser tornado indisponível?"

- [ ] Rate limiting em endpoints de geração de IA (custo + disponibilidade)
- [ ] Rate limiting em endpoints de autenticação (brute force prevention)
- [ ] Upload de arquivo com limite de tamanho (evitar esgotamento de storage)
- [ ] Tasks Celery com timeout configurado (evitar workers presos)
- [ ] Fila Celery não pode ser inundada por um único usuário
- [ ] Retry de geração de copy tem limite (não loop infinito)

**Ameaças identificadas:**
| ID | Descrição | Componente | Impacto | Mitigação | Status |
|----|-----------|-----------|---------|-----------|--------|
| | | | | | |

---

## E — Elevation of Privilege (Escalada de Privilégio)

> "Um usuário com baixo privilégio pode obter acesso maior?"

- [ ] Não existe endpoint de admin acessível por usuários normais
- [ ] `user_id` nunca aceito como parâmetro de request para dados próprios (sempre extraído do JWT)
- [ ] IDOR impossível: content_request de usuário A inacessível por usuário B mesmo com ID correto
- [ ] Celery tasks não executam com permissões além do usuário solicitante
- [ ] Tokens Meta de um usuário não acessíveis por outro usuário no sistema
- [ ] Sem bypass de RLS via parâmetros de query diretos ao Supabase

**Ameaças identificadas:**
| ID | Descrição | Componente | Impacto | Mitigação | Status |
|----|-----------|-----------|---------|-----------|--------|
| | | | | | |

---

## Resumo STRIDE

| Categoria | Ameaças Abertas | Maior Impacto | Status |
|-----------|----------------|---------------|--------|
| Spoofing | | | |
| Tampering | | | |
| Repudiation | | | |
| Information Disclosure | | | |
| Denial of Service | | | |
| Elevation of Privilege | | | |
