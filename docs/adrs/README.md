# Architectural Decision Records

Decisões arquiteturais do **Sistema de Webhooks de Notificação de Pedidos**, derivadas da reunião técnica registrada em [`TRANSCRICAO.md`](../../TRANSCRICAO.md) e do código base do OMS.

| ADR | Título | Status |
|-----|--------|--------|
| [ADR-001](ADR-001-outbox-no-mysql.md) | Outbox no MySQL | Accepted |
| [ADR-002](ADR-002-retry-backoff-e-dlq.md) | Retry com backoff e DLQ | Accepted |
| [ADR-003](ADR-003-autenticacao-hmac-sha256.md) | Autenticação HMAC-SHA256 com secret por endpoint | Accepted |
| [ADR-004](ADR-004-entrega-at-least-once-x-event-id.md) | Entrega at-least-once com X-Event-Id | Accepted |
| [ADR-005](ADR-005-worker-polling-processo-separado.md) | Worker em processo separado com polling | Accepted |
| [ADR-006](ADR-006-reuso-padroes-do-projeto.md) | Reuso dos padrões existentes do projeto | Accepted |

Cada ADR segue o formato MADR (Status, Contexto, Decisão, Alternativas Consideradas, Consequências).
