# TRACKER — Rastreabilidade da documentação

Mapeia itens dos design docs à origem na [`TRANSCRICAO.md`](../TRANSCRICAO.md) ou no código-fonte. Se um item não tiver localização preenchível, não deve permanecer nos documentos.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|----|-----------|------|-------------------|-------|-------------|
| PRD-CTX-01 | docs/PRD.md | Contexto | Clientes B2B Atlas, MaxDistribuição, Nova Cargo pedem notificação em tempo real | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-02 | docs/PRD.md | Contexto | Hoje clientes fazem polling em GET /orders | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-03 | docs/PRD.md | Risco de negócio | Atlas pode migrar se não entregar até fim do trimestre | TRANSCRICAO | [09:00] Marcos |
| PRD-MET-01 | docs/PRD.md | Métrica | Latência &lt; 10s aceita como tempo real | TRANSCRICAO | [09:02] Marcos |
| PRD-ESC-01 | docs/PRD.md | Escopo | Apenas webhooks outbound (plataforma → cliente) | TRANSCRICAO | [09:02] Sofia / [09:02] Marcos |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Notificar mudança de status via webhook outbound | TRANSCRICAO | [09:00] Marcos |
| PRD-FR-02 | docs/PRD.md | Requisito Funcional | POST criar webhook; secret gerada pelo sistema; status list | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-03 | docs/PRD.md | Requisito Funcional | CRUD: PATCH, DELETE, GET listar webhooks do customer | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-04 | docs/PRD.md | Requisito Funcional | Filtro de eventos por status na inserção da outbox | TRANSCRICAO | [09:33] Marcos / [09:34] Bruno |
| PRD-FR-05 | docs/PRD.md | Requisito Funcional | GET /webhooks/:id/deliveries (~últimos 100) | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-06 | docs/PRD.md | Requisito Funcional | Replay DLQ admin POST .../dead-letter/:id/replay | TRANSCRICAO | [09:18] Diego / [09:35] Diego |
| PRD-FR-07 | docs/PRD.md | Requisito Funcional | Rotação de secret com grace 24h | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-08 | docs/PRD.md | Requisito Funcional | Outbox na mesma transação de changeStatus | TRANSCRICAO | [09:40] Bruno |
| PRD-FR-09 | docs/PRD.md | Requisito Funcional | At-least-once com X-Event-Id | TRANSCRICAO | [09:24] Diego / [09:25] Diego |
| PRD-FR-10 | docs/PRD.md | Requisito Funcional | customer_id no body/path, não implícito do JWT | TRANSCRICAO | [09:32] Larissa |
| PRD-NFR-01 | docs/PRD.md | Requisito Não Funcional | Latência &lt; 10s | TRANSCRICAO | [09:02] Marcos |
| PRD-NFR-02 | docs/PRD.md | Requisito Não Funcional | Polling 2s; timeout HTTP 10s | TRANSCRICAO | [09:09] Diego / [09:42] Diego |
| PRD-NFR-03 | docs/PRD.md | Requisito Não Funcional | Não degradar transação de status com HTTP síncrono | TRANSCRICAO | [09:04] Bruno |
| PRD-NFR-04 | docs/PRD.md | Requisito Não Funcional | HTTPS obrigatório | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-05 | docs/PRD.md | Requisito Não Funcional | HMAC-SHA256; secret por endpoint | TRANSCRICAO | [09:20] Sofia / [09:21] Sofia |
| PRD-NFR-06 | docs/PRD.md | Requisito Não Funcional | Payload máx. 64KB; erro se ultrapassar | TRANSCRICAO | [09:24] Diego / [09:24] Larissa |
| PRD-NFR-07 | docs/PRD.md | Requisito Não Funcional | 5 retries 1m/5m/30m/2h/12h + DLQ | TRANSCRICAO | [09:17] Diego / [09:17] Larissa |
| PRD-NFR-08 | docs/PRD.md | Requisito Não Funcional | Reuso AppError, Zod, Pino, módulo webhooks | TRANSCRICAO | [09:30] Larissa |
| PRD-NFR-09 | docs/PRD.md | Requisito Não Funcional | Revisão segurança ≥2 dias úteis pré-deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-OOS-01 | docs/PRD.md | Fora de escopo | E-mail de alerta por falhas consecutivas adiado | TRANSCRICAO | [09:37] Larissa |
| PRD-OOS-02 | docs/PRD.md | Fora de escopo | Dashboard visual deferred / projeto frontend | TRANSCRICAO | [09:40] Larissa |
| PRD-OOS-03 | docs/PRD.md | Fora de escopo | Redis Streams descartado | TRANSCRICAO | [09:07] Diego |
| PRD-OOS-04 | docs/PRD.md | Fora de escopo | Rate limiting de saída adiado | TRANSCRICAO | [09:39] Larissa |
| PRD-OOS-05 | docs/PRD.md | Fora de escopo | Arquivamento outbox ~30 dias fora desta feature | TRANSCRICAO | [09:08] Diego |
| PRD-RISK-01 | docs/PRD.md | Risco | Atraso vs Atlas / churn | TRANSCRICAO | [09:00] Marcos / [09:45] Marcos |
| PRD-RISK-02 | docs/PRD.md | Risco | Cliente offline longo → DLQ | TRANSCRICAO | [09:15] Diego / [09:17] Marcos |
| PRD-DEP-01 | docs/PRD.md | Dependência | Prazo estimado 3 sprints | TRANSCRICAO | [09:46] Larissa / [09:47] Larissa |
| PRD-TRADE-01 | docs/PRD.md | Trade-off | Outbox MySQL vs Redis | TRANSCRICAO | [09:07] Larissa / [09:07] Diego |
| PRD-TRADE-02 | docs/PRD.md | Trade-off | At-least-once vs exactly-once | TRANSCRICAO | [09:25] Diego |
| RFC-META-01 | docs/RFC.md | Metadados | Revisores = participantes da reunião | TRANSCRICAO | [09:00] Larissa |
| RFC-PROP-01 | docs/RFC.md | Decisão | Proposta: outbox + worker + HMAC + at-least-once | TRANSCRICAO | [09:48] Larissa |
| RFC-ALT-01 | docs/RFC.md | Alternativa descartada | HTTP síncrono em changeStatus | TRANSCRICAO | [09:04] Bruno / [09:06] Diego |
| RFC-ALT-02 | docs/RFC.md | Alternativa descartada | Redis Streams / Cluster | TRANSCRICAO | [09:07] Larissa / [09:07] Diego |
| RFC-OPEN-01 | docs/RFC.md | Questão em aberto | Rate limiting de saída | TRANSCRICAO | [09:38] Diego / [09:39] Larissa |
| RFC-OPEN-02 | docs/RFC.md | Questão em aberto | Endurecer auth do CRUD depois | TRANSCRICAO | [09:37] Sofia |
| RFC-OPEN-03 | docs/RFC.md | Questão em aberto | Escala multi-worker futura | TRANSCRICAO | [09:13] Diego |
| RFC-RISK-01 | docs/RFC.md | Risco | Secret leak / rotação | TRANSCRICAO | [09:22] Diego / [09:21] Sofia |
| FDD-FLOW-01 | docs/FDD.md | Fluxo | publishWebhookEvent na mesma tx de changeStatus | TRANSCRICAO | [09:40] Bruno / [09:41] Diego |
| FDD-FLOW-02 | docs/FDD.md | Fluxo | Worker polling 2s batch pendentes | TRANSCRICAO | [09:09] Diego |
| FDD-FLOW-03 | docs/FDD.md | Fluxo | Retry backoff e DLQ tabela separada | TRANSCRICAO | [09:17] Diego / [09:18] Diego |
| FDD-FLOW-04 | docs/FDD.md | Fluxo | Snapshot de payload na inserção | TRANSCRICAO | [09:52] Larissa / [09:52] Diego |
| FDD-FLOW-05 | docs/FDD.md | Fluxo | Filtro na inserção da outbox | TRANSCRICAO | [09:34] Bruno |
| FDD-CONTRATO-01 | docs/FDD.md | Contrato | POST criar webhook + secret na resposta | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-02 | docs/FDD.md | Contrato | GET listar webhooks do customer | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-03 | docs/FDD.md | Contrato | GET /webhooks/:id/deliveries | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-04 | docs/FDD.md | Contrato | POST rotate-secret | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-05 | docs/FDD.md | Contrato | POST admin dead-letter replay | TRANSCRICAO | [09:18] Diego / [09:36] Sofia |
| FDD-CONTRATO-06 | docs/FDD.md | Contrato | Payload order.status_changed sem items | TRANSCRICAO | [09:43] Diego |
| FDD-CONTRATO-07 | docs/FDD.md | Contrato | Headers X-Event-Id, X-Signature, X-Timestamp, X-Webhook-Id | TRANSCRICAO | [09:44] Diego / [09:44] Sofia |
| FDD-ERR-01 | docs/FDD.md | Erro | Prefixo WEBHOOK_ nos códigos | TRANSCRICAO | [09:28] Bruno / [09:29] Larissa |
| FDD-ERR-02 | docs/FDD.md | Erro | WEBHOOK_NOT_FOUND / WEBHOOK_INVALID_URL | TRANSCRICAO | [09:28] Bruno |
| FDD-RES-01 | docs/FDD.md | Resiliência | Timeout HTTP 10s | TRANSCRICAO | [09:42] Diego |
| FDD-OBS-01 | docs/FDD.md | Observabilidade | Logs Pino estruturados | TRANSCRICAO | [09:29] Bruno |
| FDD-OBS-02 | docs/FDD.md | Observabilidade | Auditoria de quem fez replay | TRANSCRICAO | [09:36] Sofia |
| FDD-INT-01 | docs/FDD.md | Integração | Extender changeStatus em order.service.ts | CODIGO | src/modules/orders/order.service.ts |
| FDD-INT-02 | docs/FDD.md | Integração | Reusar OrderStatus / canTransition | CODIGO | src/modules/orders/order.status.ts |
| FDD-INT-03 | docs/FDD.md | Integração | AppError / http-errors para WEBHOOK_* | CODIGO | src/shared/errors/http-errors.ts |
| FDD-INT-04 | docs/FDD.md | Integração | authenticate + requireRole ADMIN | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-INT-05 | docs/FDD.md | Integração | Wiring de rotas em routes/index.ts | CODIGO | src/routes/index.ts |
| FDD-INT-06 | docs/FDD.md | Integração | Entry API server.ts como modelo do worker | CODIGO | src/server.ts |
| FDD-INT-07 | docs/FDD.md | Integração | Novos models no schema Prisma | CODIGO | prisma/schema.prisma |
| FDD-INT-08 | docs/FDD.md | Integração | Logger Pino compartilhado | CODIGO | src/shared/logger/index.ts |
| FDD-INT-09 | docs/FDD.md | Integração | validate middleware + Zod schemas | CODIGO | src/middlewares/validate.middleware.ts |
| FDD-MOD-01 | docs/FDD.md | Estrutura | Módulo src/modules/webhooks | TRANSCRICAO | [09:27] Bruno |
| FDD-MOD-02 | docs/FDD.md | Estrutura | src/worker.ts + npm run worker | TRANSCRICAO | [09:11] Larissa |
| FDD-AUTH-01 | docs/FDD.md | Restrição | Replay exige role ADMIN | TRANSCRICAO | [09:36] Sofia / [09:36] Larissa |
| FDD-ID-01 | docs/FDD.md | Decisão | IDs UUID na outbox | TRANSCRICAO | [09:51] Larissa |
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Outbox no MySQL na mesma transação | TRANSCRICAO | [09:06] Diego / [09:08] Larissa |
| ADR-001-ALT-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Alternativa | Síncrono descartado | TRANSCRICAO | [09:04] Bruno |
| ADR-001-ALT-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Alternativa | Redis descartado | TRANSCRICAO | [09:07] Diego |
| ADR-002 | docs/adrs/ADR-002-retry-backoff-e-dlq.md | Decisão | 5 retries + DLQ separada | TRANSCRICAO | [09:17] Larissa / [09:18] Diego |
| ADR-002-ALT-01 | docs/adrs/ADR-002-retry-backoff-e-dlq.md | Alternativa | Só 3 tentativas insuficiente | TRANSCRICAO | [09:16] Diego |
| ADR-003 | docs/adrs/ADR-003-autenticacao-hmac-sha256.md | Decisão | HMAC-SHA256, secret por endpoint, grace 24h | TRANSCRICAO | [09:22] Sofia |
| ADR-003-ALT-01 | docs/adrs/ADR-003-autenticacao-hmac-sha256.md | Alternativa | Secret global descartada | TRANSCRICAO | [09:21] Sofia |
| ADR-004 | docs/adrs/ADR-004-entrega-at-least-once-x-event-id.md | Decisão | At-least-once + X-Event-Id | TRANSCRICAO | [09:26] Larissa |
| ADR-004-ALT-01 | docs/adrs/ADR-004-entrega-at-least-once-x-event-id.md | Alternativa | Exactly-once descartado | TRANSCRICAO | [09:25] Diego |
| ADR-005 | docs/adrs/ADR-005-worker-polling-processo-separado.md | Decisão | Worker separado polling 2s | TRANSCRICAO | [09:10] Larissa / [09:11] Diego |
| ADR-005-ALT-01 | docs/adrs/ADR-005-worker-polling-processo-separado.md | Alternativa | Trigger MySQL descartado | TRANSCRICAO | [09:09] Diego |
| ADR-006 | docs/adrs/ADR-006-reuso-padroes-do-projeto.md | Decisão | Reuso padrões OMS + módulo webhooks | TRANSCRICAO | [09:30] Larissa |
| ADR-006-CODE-01 | docs/adrs/ADR-006-reuso-padroes-do-projeto.md | Integração | Referência changeStatus / AppError / requireRole | CODIGO | src/modules/orders/order.service.ts |
| ADR-006-CODE-02 | docs/adrs/ADR-006-reuso-padroes-do-projeto.md | Integração | AppError base | CODIGO | src/shared/errors/app-error.ts |
| ADR-006-CODE-03 | docs/adrs/ADR-006-reuso-padroes-do-projeto.md | Integração | requireRole existente | CODIGO | src/middlewares/auth.middleware.ts |
| ADR-006-CODE-04 | docs/adrs/ADR-006-reuso-padroes-do-projeto.md | Integração | Entry server.ts | CODIGO | src/server.ts |
| SUM-01 | docs/RFC.md / docs/PRD.md | Decisão | Resumo fechado das 6 decisões principais | TRANSCRICAO | [09:48] Larissa |
| SUM-02 | docs/PRD.md | Fora de escopo | Email futura + rate limit observar + dashboard fora | TRANSCRICAO | [09:48] Larissa |

## Notas de cobertura

- Linhas com `Fonte = TRANSCRICAO`: maioria da tabela (≥70%), com timestamp `[hh:mm]` e nome do falante.
- Linhas com `Fonte = CODIGO`: FDD-INT-01…09 e ADR-006-CODE-01…04 (paths reais do repositório).
- Itens de implementação detalhada no FDD (ex.: nomes exatos de métricas Prometheus) são derivados operacionais das decisões de observabilidade/logging da reunião e dos padrões do logger existente; quando não há timestamp literal, a origem ancora em Pino ([09:29] Bruno) + arquivo de logger.
