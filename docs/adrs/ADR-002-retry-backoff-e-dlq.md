# ADR-002: Retry com backoff exponencial e DLQ

## Status

Accepted

## Contexto

Endpoints de clientes B2B podem ficar indisponíveis (manutenção, incidente, timeout). Sem política de retry, notificações se perdem. Com retry infinito, eventos ficam pendurados para sempre se o cliente sumir. Já houve caso de cliente com indisponibilidade planejada de cerca de duas horas — três tentativas em ~30 minutos seriam insuficientes.

Também é necessário um destino claro para falhas permanentes, com evidência para debug e reprocessamento manual.

## Decisão

1. Após falha de entrega HTTP, aplicar **backoff exponencial** com **5 tentativas** nos intervalos **1m / 5m / 30m / 2h / 12h** (~15h entre a primeira falha e a última tentativa).
2. Esgotadas as tentativas, mover o evento para a tabela separada **`webhook_dead_letter`** (payload, motivo da falha, timestamp).
3. Disponibilizar replay manual via endpoint admin `POST /admin/webhooks/dead-letter/:id/replay`, que recoloca o evento na outbox como pendente.

## Alternativas Consideradas

1. **Retry indefinido com backoff** — Cobre indisponibilidades longas, mas deixa eventos pendurados indefinidamente quando o cliente abandona o endpoint. Descartada.
2. **Apenas 3 tentativas** — Mais agressivo, porém insuficiente para manutenções de ~2h já observadas. Descartada.
3. **Marcar `failed` na própria outbox em vez de tabela DLQ** — Simplifica o schema, mas polui a leitura da outbox operacional e enfraquece o papel de “evidence” para reprocessamento. Preferida a tabela separada.

## Consequências

**Positivas**

- Janela de recuperação compatível com manutenções típicas do cliente.
- DLQ isolada facilita operação, auditoria e replay.
- Evita crescimento infinito de filas de retry.

**Negativas / trade-offs**

- Cliente offline por mais de ~15h perde entrega automática (precisa de replay manual ou nova mudança de status).
- Replay exige processo operacional e role ADMIN; não é self-service do cliente nesta fase.
