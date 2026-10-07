# ADR-001: Outbox no MySQL

## Status

Accepted

## Contexto

Clientes B2B precisam ser notificados quando o status de um pedido muda. A mudança de status no OMS já ocorre em uma transação SQL pesada que atualiza `orders`, insere em `order_status_history` e ajusta `stock_quantity`. Incluir uma chamada HTTP síncrona nessa transação degradaria latência e acoplaria o sucesso da mudança de status à disponibilidade do endpoint do cliente.

O time também considerou introduzir uma fila externa (Redis Streams / Redis Cluster), o que exigiria nova infraestrutura operacional.

## Decisão

Adotar o padrão **Transactional Outbox** no MySQL já usado pelo projeto:

1. Na mesma transação que altera o status do pedido, inserir uma linha na tabela `webhook_outbox` com o evento (payload snapshot, `event_id`, status pendente).
2. Se a transação fizer commit, o evento fica persistido; se fizer rollback, o evento some junto — sem inconsistência entre estado do pedido e presença do evento.
3. Um worker separado lê a outbox e dispara o HTTP outbound.

## Alternativas Consideradas

1. **HTTP síncrono no service de orders** — Cliente lento trava a mudança de status para outros pedidos; se o cliente estiver offline, rollback da mudança de status seria incorreto. Descartada.
2. **Redis Streams / Redis Cluster** — Resolve desacoplamento, mas exige subir e operar infra adicional. Para um time pequeno, foi considerado overengineering frente ao MySQL já existente.

## Consequências

**Positivas**

- Consistência atômica entre mudança de status e registro do evento.
- Sem nova infraestrutura de mensageria na fase 1.
- Worker pode falhar/reiniciar sem perder eventos commitados.

**Negativas / trade-offs**

- A outbox cresce no mesmo banco de pedidos; exige índice em status + `created_at` e política futura de arquivamento (fora do escopo desta feature).
- Throughput limitado pelo MySQL e pelo polling do worker, não por um broker dedicado.
