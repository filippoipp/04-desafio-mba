# ADR-004: Entrega at-least-once com X-Event-Id

## Status

Accepted

## Contexto

Com outbox + worker + retries, é possível que o mesmo evento seja entregue mais de uma vez (timeout após o cliente processar, crash entre HTTP 200 e marcação como entregue, replay de DLQ, etc.). Garantir exactly-once end-to-end exigiria coordenação complexa entre plataforma e cliente.

## Decisão

1. Adotar semântica **at-least-once**: o cliente pode receber o mesmo evento mais de uma vez e deve estar preparado.
2. Gerar um **UUID `event_id`** no momento da inserção na outbox e enviá-lo no header **`X-Event-Id`** (e no body do payload).
3. Documentar no portal de desenvolvedores que a deduplicação é responsabilidade do cliente, usando `event_id`.

## Alternativas Consideradas

1. **Exactly-once** — Eliminaria duplicatas do lado do cliente, mas exige coordenação dos dois lados e complexidade operacional elevada. Descartada; o padrão de mercado (Stripe, GitHub) é at-least-once com id de evento.

## Consequências

**Positivas**

- Implementação alinhada a práticas de mercado e ao modelo outbox/retry.
- Cliente consegue deduplicar de forma determinística com `X-Event-Id`.
- Evita protocolos de handshake/ack distribuídos nesta fase.

**Negativas / trade-offs**

- Empurra responsabilidade de idempotência para o integrador.
- Replay admin e retries aumentam a probabilidade de entrega duplicada — aceitável desde que documentado.
