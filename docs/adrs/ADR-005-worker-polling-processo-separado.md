# ADR-005: Worker em processo separado com polling

## Status

Accepted

## Contexto

Eventos na outbox precisam ser disparados de forma assíncrona. Clientes aceitam latência percebida como “tempo real” abaixo de **10 segundos**. MySQL não oferece listener nativo equivalente ao `LISTEN/NOTIFY` do Postgres; triggers no banco não notificam processo externo de forma limpa.

Se o worker rodar no mesmo processo da API HTTP, reinícios da API interrompem o processamento de entregas.

## Decisão

1. Executar o worker como **processo Node separado** (`src/worker.ts` + script `npm run worker`), usando o mesmo banco/Prisma stack, mas **outra instância de processo**.
2. Consumir a outbox por **polling a cada 2 segundos**: buscar batch pequeno dos eventos pendentes mais antigos, processar e marcar status.
3. Fase 1: **single-worker**; ordering implícita por `created_at` / `order_id`. Escalabilidade multi-worker (partição por `order_id` ou lock pessimista) fica como limitação conhecida / trabalho futuro.

## Alternativas Consideradas

1. **Worker embutido no processo da API** (`src/server.ts`) — Menos deploy, mas perde o worker quando a API reinicia. Descartada.
2. **Trigger MySQL / improvisar notificação a processo externo** — Sem listener nativo; ficaria gambiarra (arquivo, HTTP interno, etc.). Polling de 2s atende o SLA de &lt;10s. Descartada.

## Consequências

**Positivas**

- Latência típica compatível com o requisito (&lt;10s; piso ~2s pelo intervalo de poll).
- Ciclo de vida do worker independente da API.
- Modelo operacional simples para time pequeno.

**Negativas / trade-offs**

- Polling introduz atraso mínimo e carga periódica no banco (mitigada por índice e batch pequeno).
- Single-worker limita throughput horizontal; multi-worker sem particionamento pode quebrar ordering por pedido.
