# ADR-003: Autenticação HMAC-SHA256 com secret por endpoint

## Status

Accepted

## Contexto

Webhooks outbound enviam dados de pedidos para endpoints fora da nossa infraestrutura. O receptor precisa validar autenticidade (a requisição veio de nós) e integridade (o payload não foi adulterado). Uma secret global da plataforma ampliaria o blast radius em caso de vazamento — cenário já vivido por cliente que vazou secret em log.

Também é necessário permitir rotação de secret sem downtime na verificação do cliente.

## Decisão

1. Assinar o **corpo** do request com **HMAC-SHA256** e enviar a assinatura no header **`X-Signature`**.
2. Cada endpoint de webhook cadastrado possui **secret própria** (não global).
3. Suportar **rotação via API** com **grace period de 24h**: secret antiga e nova válidas em paralelo; após 24h a antiga expira.
4. Exigir **HTTPS** na URL do webhook (validação no schema Zod); `http` é rejeitado.

## Alternativas Consideradas

1. **Secret global da plataforma** — Mais simples de operar, mas vazamento de uma secret compromete todos os clientes. Descartada.
2. **mTLS mútuo** — Forte, porém aumenta complexidade de onboarding e gestão de certificados para clientes B2B; fora do perfil desta fase. Não adotada.

## Consequências

**Positivas**

- Padrão de mercado (verificável com bibliotecas comuns no cliente).
- Blast radius limitado por endpoint.
- Rotação com overlap reduz risco de downtime na migração.

**Negativas / trade-offs**

- Cliente precisa implementar verificação HMAC e, opcionalmente, checagem de `X-Timestamp` contra replay.
- Durante o grace period, duas secrets válidas aumentam ligeiramente a superfície de chave ativa.
- Revisão de segurança do código de HMAC/secret deve ocorrer ≥2 dias úteis antes do deploy.
