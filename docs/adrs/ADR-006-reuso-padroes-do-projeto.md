# ADR-006: Reuso dos padrões existentes do projeto

## Status

Accepted

## Contexto

O OMS já possui convenções estáveis de módulo, erros, autenticação, validação e logging. Introduzir stack paralela para webhooks aumentaria custo de manutenção e divergiria do que o time já opera em produção.

Arquivos e padrões de referência no código atual:

- Módulos de domínio em `src/modules/*` (controller, service, repository, routes, schemas) — ex.: `src/modules/orders/`.
- Entrypoint HTTP em [`src/server.ts`](../../src/server.ts); composição em [`src/app.ts`](../../src/app.ts) e rotas em [`src/routes/index.ts`](../../src/routes/index.ts).
- Erros via [`src/shared/errors/app-error.ts`](../../src/shared/errors/app-error.ts) e [`src/shared/errors/http-errors.ts`](../../src/shared/errors/http-errors.ts) (`AppError`, `InvalidStatusTransitionError`, códigos como `INSUFFICIENT_STOCK`).
- Auth JWT e `requireRole` em [`src/middlewares/auth.middleware.ts`](../../src/middlewares/auth.middleware.ts) (já usado em `src/modules/users/user.routes.ts` para ADMIN).
- Logger Pino em [`src/shared/logger/index.ts`](../../src/shared/logger/index.ts); validação Zod + middleware em [`src/middlewares/validate.middleware.ts`](../../src/middlewares/validate.middleware.ts).
- Ponto de integração de negócio: método `changeStatus` em [`src/modules/orders/order.service.ts`](../../src/modules/orders/order.service.ts), que já agrupa update de pedido, histórico e estoque em `prisma.$transaction`.

## Decisão

1. Criar o domínio em **`src/modules/webhooks`** seguindo o mesmo layout dos demais módulos.
2. Colocar a lógica de processamento no módulo (ex.: `webhook.worker.ts` / `webhook.processor.ts`) e o entrypoint em **`src/worker.ts`**.
3. Reutilizar **`AppError`** / hierarquia HTTP existente; códigos de erro com prefixo **`WEBHOOK_`** (ex.: `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`).
4. Reutilizar Pino, error middleware, schemas Zod e `authenticate` / `requireRole('ADMIN')` no replay de DLQ.
5. Expor `publishWebhookEvent(tx, order, fromStatus, toStatus)` para ser chamada de dentro da transação de `OrderService.changeStatus`, recebendo o `TransactionClient` — sem duplicar orquestração de status.

## Alternativas Consideradas

1. **Pacote/serviço isolado com stack própria** — Isolamento forte, porém dobra padrões de erro, auth e deploy para uma feature que compartilha o mesmo banco e o mesmo domínio de pedidos. Descartada nesta fase.

## Consequências

**Positivas**

- Curva de onboarding baixa para o time de Pedidos/Plataforma.
- Error middleware atual já serializa `AppError` sem mudanças estruturais.
- Integração no `changeStatus` preserva a garantia atômica da outbox.

**Negativas / trade-offs**

- `OrderService` passa a depender de uma função do módulo de webhooks (acoplamento controlado via `publishWebhookEvent(tx, ...)`).
- Worker e API compartilham schema Prisma; migrações precisam considerar ambos os processos.
