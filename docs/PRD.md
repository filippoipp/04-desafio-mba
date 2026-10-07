# PRD — Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
|-------|--------|
| **Produto** | Order Management System (OMS) |
| **Feature** | Webhooks outbound de mudança de status de pedidos |
| **PM** | Marcos |
| **Tech Lead** | Larissa |
| **Status** | Aprovado em reunião técnica; documentação para implementação |
| **Documentos técnicos** | [RFC](RFC.md), [FDD](FDD.md), [ADRs](adrs/README.md) |

---

## 1. Resumo e contexto da feature

Clientes B2B precisam saber, quase em tempo real, quando o status dos pedidos deles muda na plataforma. Hoje dependem de polling em `GET /orders`, o que encarece e atrasa a integração. Esta feature entrega **webhooks outbound**: a plataforma chama um endpoint HTTPS do cliente quando o status muda, com autenticação por assinatura e histórico de entregas consultável via API.

A decisão técnica já foi tomada em reunião entre produto, engenharia e segurança; este PRD consolida o **porquê** e o **quê**. O **como** está no RFC/FDD/ADRs.

---

## 2. Problema e motivação

- **Dor atual:** Atlas Comercial, MaxDistribuição e Nova Cargo fazem polling periódico para detectar mudanças de status.
- **Pressão comercial:** a Atlas indicou possível migração para concorrente se a capacidade não chegar até o fim do trimestre.
- **Definição de tempo real do cliente:** qualquer latência **abaixo de 10 segundos** já é aceitável; o problema é ficar “pendurado” atualizando manualmente.
- **Restrição de produto:** apenas notificações **saindo** da plataforma para o cliente (outbound).

---

## 3. Público-alvo e cenários de uso

| Público | Necessidade |
|---------|-------------|
| Integradores B2B (ex.: Atlas, MaxDistribuição, Nova Cargo) | Receber evento HTTP quando pedido muda de status |
| Operadores / usuários autenticados da API | Cadastrar e gerenciar webhooks do customer que representam |
| Admins da plataforma | Reprocessar entregas que foram para DLQ |
| Time de engenharia OMS | Implementar sem degradar o fluxo de pedidos em produção |

**Cenários**

1. Cliente cadastra URL HTTPS e escolhe status (`SHIPPED`, `DELIVERED`, …).
2. Operador muda pedido para `SHIPPED` → cliente recebe POST assinado em poucos segundos.
3. Endpoint do cliente está em manutenção → plataforma retenta com backoff e, se necessário, move para DLQ; admin pode reprocessar.
4. Cliente rotaciona secret sem downtime na verificação (grace 24h).

---

## 4. Objetivos e métricas de sucesso

| Objetivo | Métrica | Meta |
|----------|---------|------|
| Notificação percebida como tempo real | Tempo entre commit da mudança de status e tentativa de entrega HTTP (caminho feliz) | **&lt; 10 segundos** |
| Reduzir dependência de polling | Clientes piloto passam a consumir webhook como fonte primária de mudança de status | 3 clientes B2B onboarding até fim da entrega estimada |
| Confiabilidade de entrega | Eventos sem sucesso após retries ficam rastreáveis | 100% das falhas permanentes na DLQ com motivo |
| Segurança do canal | Endpoints HTTPS + HMAC verificável | 0 entregas para URL não-HTTPS; secret única por endpoint |

Prazo interno estimado: **3 sprints**, incluindo revisão de segurança (≥2 dias úteis antes do deploy).

---

## 5. Escopo

### 5.1 Incluso

- Cadastro/CRUD de webhooks por customer (URL, filtro de status, active).
- Geração e rotação de secret (grace 24h).
- Disparo assíncrono na mudança de status (outbox + worker).
- Retry com backoff, DLQ e replay admin.
- Histórico de entregas via API (~últimos 100).
- Assinatura HMAC-SHA256 e headers de correlação (`X-Event-Id`, etc.).
- Documentação de integração no portal (responsabilidade de produto; API-only nesta fase).

### 5.2 Fora de escopo

Itens **explicitamente descartados ou adiados** na reunião:

1. **E-mail de alerta** quando o webhook falha N vezes seguidas — adiado para próxima fase.
2. **Dashboard / painel visual** para o cliente gerenciar webhooks — fora de escopo; apenas endpoints (frontend é projeto separado).
3. **Redis Streams / broker externo** — descartado (overengineering nesta fase).
4. **Disparo HTTP síncrono** na mudança de status — descartado.
5. **Exactly-once** de entrega — descartado em favor de at-least-once.
6. **Rate limiting de saída** — adiado (“observar e decidir depois”).
7. **Arquivamento automático** de linhas entregues da outbox (~30 dias) — fora do escopo desta feature.
8. **Multi-worker / particionamento** — adiado.

---

## 6. Requisitos funcionais

| ID | Requisito |
|----|-----------|
| RF-01 | Notificar clientes B2B via HTTP outbound quando o status do pedido muda, substituindo a necessidade de polling contínuo em `GET /orders`. |
| RF-02 | Permitir **criar** webhook (`POST`) com URL, lista de status desejados; **secret gerada pelo sistema** e devolvida na criação; `customer_id` no path/body (não implícito do JWT). |
| RF-03 | Suportar **CRUD completo**: listar, editar (`PATCH`), remover (`DELETE`) webhooks de um customer. |
| RF-04 | Filtrar eventos por lista de status no cadastro; filtrar **na inserção da outbox** (não inserir se nenhum webhook estiver interessado). |
| RF-05 | Expor histórico de entregas `GET /webhooks/:id/deliveries` (últimos ~100: sucesso/falha, payload, response, tempo). |
| RF-06 | Permitir **replay** de itens da DLQ via `POST /admin/webhooks/dead-letter/:id/replay` exclusivo para role **ADMIN**, com auditoria de quem reprocessou. |
| RF-07 | Suportar **rotação de secret** via API com grace period de **24h**. |
| RF-08 | Publicar evento na outbox **na mesma transação** da mudança de status (`changeStatus`); falha na outbox implica rollback. |
| RF-09 | Entregar com semântica **at-least-once** e identificar eventos com `X-Event-Id` para deduplicação no cliente. |
| RF-10 | Escopo **somente outbound** (plataforma → cliente). |

---

## 7. Requisitos não funcionais

| ID | Requisito |
|----|-----------|
| RNF-01 | Latência de notificação (caminho feliz) **&lt; 10s**. |
| RNF-02 | Polling do worker a cada **2s**; timeout HTTP de entrega **10s**. |
| RNF-03 | Não degradar a transação de mudança de status com I/O HTTP síncrono. |
| RNF-04 | HTTPS obrigatório na URL do webhook. |
| RNF-05 | HMAC-SHA256 sobre o corpo; secret **por endpoint**. |
| RNF-06 | Limite de payload **64KB**; erro se ultrapassar (não truncar). |
| RNF-07 | Retry: 5 tentativas com backoff 1m/5m/30m/2h/12h; depois DLQ. |
| RNF-08 | Reuso dos padrões do OMS (módulo, AppError, Zod, Pino, JWT). |
| RNF-09 | Revisão de segurança (HMAC/secret) com ≥2 dias úteis antes do deploy. |

---

## 8. Decisões e trade-offs principais

| Decisão | Trade-off |
|---------|-----------|
| Outbox MySQL vs Redis | Consistência e simplicidade operacional vs. capacidade de mensageria dedicada |
| At-least-once vs exactly-once | Simplicidade e padrão de mercado vs. possíveis duplicatas no cliente |
| Single-worker + polling 2s | Atende &lt;10s e ordering por pedido vs. escala horizontal limitada |
| Secret por endpoint + grace 24h | Menor blast radius e rotação segura vs. duas keys ativas temporariamente |
| API-only (sem dashboard) | Entrega mais rápida vs. UX self-service visual |

Detalhes: [ADR-001](adrs/ADR-001-outbox-no-mysql.md)–[ADR-006](adrs/ADR-006-reuso-padroes-do-projeto.md).

---

## 9. Dependências

- OMS em produção: módulos de orders, auth JWT, Prisma/MySQL.
- Extensão do método `changeStatus` e schema Prisma (novas tabelas).
- Novo processo worker em deploy (além da API).
- Clientes B2B: endpoint HTTPS, verificação HMAC, deduplicação por `event_id`.
- Janela de revisão de segurança (Sofia) antes do go-live.
- Portal/docs de integração (Marcos) para onboarding Atlas e demais.

---

## 10. Riscos e mitigação

| Risco | Probabilidade | Impacto | Mitigação |
|-------|---------------|---------|-----------|
| Atraso na entrega vs. expectativa Atlas (fim do trimestre) | Média | Alto (risco de churn) | Escopo fechado; 3 sprints; ADRs/FDD prontos antes do código; sem dashboard/e-mail nesta fase |
| Endpoint do cliente offline por período longo | Média | Médio (notificações atrasadas) | Backoff ~15h + DLQ + replay admin; documentar responsabilidade do cliente |
| Vazamento de secret no lado do cliente | Baixa–Média | Alto | Secret por endpoint; rotação 24h; revisão de segurança pré-deploy |
| Crescimento da tabela outbox | Baixa | Médio | Índices + batch; arquivamento como trabalho futuro monitorado por métricas |

---

## 11. Critérios de aceitação

1. Os 3 clientes piloto conseguem cadastrar webhook HTTPS e receber evento `order.status_changed` após mudança de status, em &lt;10s no caminho feliz.
2. Secret é gerada na criação e rotacionável com overlap de 24h.
3. Webhook com filtro só `DELIVERED` **não** gera outbox em transição para `SHIPPED`.
4. Após esgotar retries, evento está na DLQ; admin consegue replay; operador não-admin recebe 403.
5. Cliente consegue listar histórico de deliveries via API.
6. URL `http://` é rejeitada na validação.
7. Documentação de integração explica at-least-once e deduplicação por `X-Event-Id`.
8. Itens de “Fora de escopo” não são entregues nesta fase.

---

## 12. Estratégia de testes e validação

| Camada | Foco |
|--------|------|
| Unitário | Filtro de status na publicação; cálculo de backoff; validação Zod HTTPS; HMAC |
| Integração | `changeStatus` + outbox na mesma tx (incl. rollback); worker marca entregue/falha |
| Contrato API | CRUD, deliveries, rotate-secret, replay ADMIN |
| E2E / sandbox | Endpoint mock do cliente; latência &lt;10s; duplicata com mesmo `event_id` |
| Segurança | Revisão Sofia (HMAC, secrets, HTTPS); testes de role no replay |
| Aceite produto | Validação com Atlas (ou ambiente piloto) após docs de integração |

---

## 13. Navegação

- Proposta para revisão: [RFC](RFC.md)
- Como implementar: [FDD](FDD.md)
- Decisões: [ADRs](adrs/README.md)
- Origem de cada item: [TRACKER](TRACKER.md)
