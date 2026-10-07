# Design Docs com IA — Webhooks de Notificação de Pedidos

## Sobre o desafio

Este repositório entrega o pacote de design docs de uma feature discutida em reunião técnica, mas nunca documentada formalmente: o **Sistema de Webhooks de Notificação de Pedidos** para um OMS Node.js + TypeScript já em produção. A única fonte narrativa era a [`TRANSCRICAO.md`](TRANSCRICAO.md); o código em `src/` e `prisma/` serviu de âncora para integração e para evitar inventar APIs ou padrões inexistentes.

O trabalho foi produzir PRD, RFC, FDD, ADRs e um Tracker de rastreabilidade — cada documento em uma “altura” diferente — sem alterar o código da aplicação. A IA foi a ferramenta principal de exploração e redação; o papel humano foi definir fronteiras, filtrar o que a reunião descartou e revisar até cada afirmação ter origem na transcrição ou no código.

> Enunciado original do desafio (repositório base): [devfullcycle/mba-ia-desafio-design-docs-com-ia](https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia)

---

## Ferramentas de IA utilizadas

| Ferramenta | Papel |
|------------|--------|
| **Cursor (Agent / Composer)** | Leitura do repositório, extração estruturada da transcrição, redação e refinamento dos Markdowns, montagem do Tracker |
| **Agentes de exploração (Explore)** | Varredura paralela da `TRANSCRICAO.md` (decisões, RFs, fora de escopo) e do código OMS (hooks de integração, erros, auth, Prisma) |
| **Revisão crítica iterativa no chat** | Correção de alucinações, remoção de duplicação RFC↔FDD, validação da checklist de aceite |

---

## Workflow adotado

Ordem seguida (alinhada ao enunciado):

1. **Contextualização** — explorar código + transcrição em paralelo; listar decisões fechadas vs. adiadas/descartadas.
2. **ADRs primeiro** — seis decisões principais em `docs/adrs/` (esqueleto do “como”).
3. **RFC** — proposta concisa para revisão, alternativas descartadas e questões em aberto; links para ADRs.
4. **FDD** — contratos, fluxos, erros `WEBHOOK_*`, integração com arquivos reais.
5. **PRD** — consolidação de produto (porquê/quê/métricas/escopo) com ADRs/RFC/FDD já estáveis.
6. **Tracker** — varredura de cada item → timestamp ou path.
7. **README do processo** — este arquivo, ao final.

Regra de ouro: se não dava para preencher a coluna **Localização** do Tracker, o trecho era removido ou reescrito.

---

## Prompts customizados

### Prompt 1 — Filtrar o que NÃO entra

```text
Leia TRANSCRICAO.md por completo. Extraia três listas mutuamente exclusivas:
1) Decisões FECHADAS (com [hh:mm] e falante)
2) Itens DESCARTADOS ou ADIADOS (com [hh:mm] e se foi descarte ou adiamento)
3) Questões EM ABERTO sem decisão

Regras:
- Não transforme item da lista 2 em requisito funcional.
- Não invente decisões que não tenham timestamp.
- Separe “mencionado” de “decidido”.
```

### Prompt 2 — Fronteira RFC vs FDD vs ADR

```text
Com base nas decisões fechadas da reunião e nos ADRs já escritos:

- Escreva o RFC em 2–4 páginas: proposta, alternativas descartadas (com trade-off),
  questões em aberto, links para ADRs. NÃO inclua payloads de exemplo nem matriz
  completa de erros.
- Reserve para o FDD: fluxos passo a passo, ≥4 endpoints com request/response,
  códigos WEBHOOK_*, seção "Integração com o sistema existente" citando apenas
  caminhos de arquivo que existam em src/ ou prisma/.

Se um parágrafo do RFC estiver no nível de implementação, mova-o para o FDD.
```

---

## Iterações e ajustes

Foram **4 iterações principais** até o pacote fechar a checklist:

1. **Primeira geração genérica** — rascunhos misturavam detalhe de payload no RFC e repetiam o resumo da Larissa em todos os docs. Ajuste: fronteiras explícitas (RFC = decisão; FDD = construção; ADR = uma decisão).
2. **Alucinação de escopo** — e-mail de alerta e dashboard apareceram como RF. Ajuste: prompt de filtragem (lista 2) + seção “Fora de escopo” no PRD ancorada em [09:37] e [09:40].
3. **Integração inventada** — menção a arquivos/helpers inexistentes. Ajuste: cruzar cada path com o tree real (`order.service.ts`, `auth.middleware.ts`, `app-error.ts`, etc.) e refletir no Tracker com `Fonte = CODIGO`.
4. **Tracker incompleto** — vários itens do FDD sem linha. Ajuste: varredura sistemática (PRD-FR/NFR, RFC-ALT/OPEN, FDD-CONTRATO/INT, ADR-00N) até cobertura ≥80% e ≥70% TRANSCRICAO.

---

## Como navegar a entrega

Ordem sugerida de leitura:

| Ordem | Arquivo | Pergunta que responde |
|------:|---------|------------------------|
| 1 | [docs/PRD.md](docs/PRD.md) | Por que e o quê? |
| 2 | [docs/RFC.md](docs/RFC.md) | Como pretendemos resolver? O que está em aberto? |
| 3 | [docs/adrs/](docs/adrs/) | Por que cada decisão pontual? |
| 4 | [docs/FDD.md](docs/FDD.md) | Como construir, em detalhe? |
| 5 | [docs/TRACKER.md](docs/TRACKER.md) | De onde veio cada coisa? |
| — | [TRANSCRICAO.md](TRANSCRICAO.md) | Fonte primária da reunião (não alterar) |

Estrutura do entregável:

```
.
├── README.md                 ← você está aqui (processo)
├── TRANSCRICAO.md            ← fonte (imutável)
├── docs/
│   ├── PRD.md
│   ├── RFC.md
│   ├── FDD.md
│   ├── TRACKER.md
│   └── adrs/
│       ├── ADR-001-outbox-no-mysql.md
│       ├── ADR-002-retry-backoff-e-dlq.md
│       ├── ADR-003-autenticacao-hmac-sha256.md
│       ├── ADR-004-entrega-at-least-once-x-event-id.md
│       ├── ADR-005-worker-polling-processo-separado.md
│       └── ADR-006-reuso-padroes-do-projeto.md
├── src/                      ← referência apenas (não alterado)
├── prisma/                   ← referência apenas (não alterado)
└── tests/                    ← referência apenas (não alterado)
```

---

## Nota sobre o código

A entrega é **puramente documental**. Nenhuma alteração foi feita em `src/`, `prisma/`, `tests/` ou configurações da aplicação. O código existe para contextualizar a seção de integração do FDD e o ADR-006.
