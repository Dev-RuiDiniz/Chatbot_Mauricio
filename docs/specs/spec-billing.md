# Especificação: Billing e Planos

**Código:** SPEC-BILLING  
**Versão:** 1.0  
**Status:** MVP / Parcial  
**Atualizado em:** 2026-07-02

---

## 1. Objetivo

Gerenciar planos de assinatura, limites de uso, faturas e transações, permitindo que o SaaS cobre clientes finais e revendas de forma automatizada.

---

## 2. Contexto

Cada tenant tem uma assinatura vinculada a um plano. Planos definem limites de bots, agentes e mensagens. O sistema gera faturas e registra transações. Um heartbeat de billing verifica assinaturas periodicamente.

### Usuários envolvidos

- Admin master (gerencia planos e faturas).
- Revenda (gerencia planos de subclientes).
- Cliente final (visualiza assinatura e faturas).
- Sistema (executa heartbeat de billing).

---

## 3. Regras de Negócio

| ID | Regra | Origem |
|---|---|---|
| RN-BILL-001 | Cada plano define limites de bots, agentes e mensagens mensais. | `plans` |
| RN-BILL-002 | Cada tenant tem no máximo uma assinatura ativa. | `subscriptions` (`tenant_id` UNIQUE) |
| RN-BILL-003 | Assinatura expirada ou suspensa bloqueia uso do bot. | HTTP 402 |
| RN-BILL-004 | Fatura é gerada para cada período de cobrança. | `invoices` |
| RN-BILL-005 | Transação registra pagamento recebido de um gateway. | `transactions` |
| RN-BILL-006 | Heartbeat de billing executa a cada 12 horas. | `server.js` |
| RN-BILL-007 | Revenda pode ter plano próprio e limites de sub-tenants. | `resellers` |
| RN-BILL-008 | Cliente final pode ter plano atribuído pela revenda. | `reseller_sub_tenants` |

---

## 4. Fluxo Esperado

### 4.1 Criar plano

1. Admin master define nome, preço, limites e recursos.
2. Sistema salva plano em `plans`.
3. Plano fica disponível para assinaturas.

### 4.2 Assinar plano

1. Cliente final ou revenda seleciona plano.
2. Sistema cria assinatura vinculada ao tenant e plano.
3. Sistema agenda próxima cobrança.

### 4.3 Gerar fatura

1. Heartbeat de billing identifica assinatura a vencer.
2. Sistema gera fatura com valor do plano.
3. Sistema notifica cliente (email/painel) — A CONFIRMAR.

### 4.4 Registrar pagamento

1. Gateway de pagamento confirma transação via webhook — A CONFIRMAR.
2. Sistema atualiza fatura e assinatura.
3. Sistema cria registro em `transactions`.

### 4.5 Bloqueio por inadimplência

1. Sistema detecta assinatura expirada ou não paga.
2. Sistema bloqueia envio de mensagens para o tenant.
3. Retorna HTTP 402 ao tentar usar bot.

---

## 5. Critérios de Aceite

- [ ] Criar planos com limites de uso.
- [ ] Associar assinatura a tenant e plano.
- [ ] Bloquear uso quando assinatura expirar.
- [ ] Gerar fatura automaticamente.
- [ ] Registrar transação de pagamento.
- [ ] Heartbeat de billing executa conforme cronograma.
- [ ] Respeitar limites de bots/agentes/mensagens por plano.
- [ ] Revenda gerencia planos de subclientes.

---

## 6. Impacto Técnico

### Backend
- `node-version/src/controllers/billingController.js`
- `node-version/src/services/billing/`
- `node-version/src/models/sql/models.js` (`Plan`, `Subscription`, `Invoice`, `Transaction`)
- `node-version/server.js` (heartbeat)

### Banco
- PostgreSQL: `plans`, `subscriptions`, `invoices`, `transactions`.

---

## 7. Testes Necessários

| Tipo | Teste | Prioridade |
|---|---|---|
| Integração | Criar plano | Alta |
| Integração | Criar assinatura | Alta |
| Integração | Bloquear bot quando assinatura expirada | Alta |
| Integração | Gerar fatura | Média |
| Integração | Registrar transação | Média |
| Integração | Heartbeat executa e notifica | Média |
| Integração | Respeitar limite de mensagens | Média |

---

## 8. Riscos

| Risco | Impacto | Mitigação |
|---|---|---|
| Gateway de pagamento não identificado | Alto | Confirmar integração e testar webhooks |
| Fatura não gerada no prazo | Médio | Monitorar heartbeat e logs |
| Bloqueio incorreto de tenant ativo | Alto | Validar regras de suspensão |

---

## 9. Dependências

- PostgreSQL operacional.
- Gateway de pagamento configurado (A CONFIRMAR).
- Scheduler/cron para heartbeat.

---

## 10. A CONFIRMAR / PENDENTE

- Gateway de pagamento real (Stripe, MercadoPago, Asaas, etc.).
- Webhook de confirmação de pagamento.
- Envio de notificações por email.
- Geração de PDF de fatura.
- Cobrança recorrente automática.
