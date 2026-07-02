# Especificação: Campanhas

**Código:** SPEC-CAMPAIGNS  
**Versão:** 1.0  
**Status:** MVP / Parcial  
**Atualizado em:** 2026-07-02

---

## 1. Objetivo

Permitir que clientes finais disparem mensagens em massa para contatos segmentados, com agendamento, delays aleatórios e controle de horário de envio, evitando bloqueios e respeitando limites do plano.

---

## 2. Contexto

Campanhas são usadas para comunicação ativa: promoções, lembretes, avisos. Cada campanha contém um template de mensagem e uma lista de contatos. O envio é processado por workers em fila RabbitMQ.

### Usuários envolvidos

- Cliente final (cria e executa campanhas).
- Atendente (não executa campanhas, apenas responde respostas).
- Sistema (processa envio em background).

---

## 3. Regras de Negócio

| ID | Regra | Origem |
|---|---|---|
| RN-CAMP-001 | Campanha pertence a um tenant. | `campaigns` (`tenant_id`) |
| RN-CAMP-002 | Campanha só pode ser disparada se houver instância WhatsApp conectada. | `botController.js` |
| RN-CAMP-003 | Campanha respeita limite de mensagens do plano. | Billing |
| RN-CAMP-004 | Envio usa delays aleatórios entre mensagens (`min_delay` e `max_delay`). | `campaigns` |
| RN-CAMP-005 | Envio respeita janela de sono (`sleep_start` e `sleep_end`). | `campaigns` |
| RN-CAMP-006 | Contatos em blacklist não devem receber campanha. | Contacts |
| RN-CAMP-007 | Contadores de envio, entrega, resposta e erro são atualizados. | `campaigns` |
| RN-CAMP-008 | Status de cada contato da campanha é rastreado. | `campaign_contacts` |

---

## 4. Fluxo Esperado

### 4.1 Criar campanha

1. Cliente final define nome, template, mídia opcional, agendamento e delays.
2. Cliente importa contatos para a campanha.
3. Sistema valida contatos e limites.
4. Sistema salva campanha e contatos.

### 4.2 Executar campanha

1. Sistema agenda worker ou executa imediatamente.
2. Worker consome contatos pendentes da fila.
3. Para cada contato, worker envia mensagem via WhatsApp Gateway.
4. Worker aguarda delay aleatório antes do próximo envio.
5. Worker respeita janela de sono.
6. Worker atualiza status do contato e contadores da campanha.

### 4.3 Acompanhar resultados

1. Cliente visualiza campanha.
2. Sistema exibe total de contatos, enviados, lidos, respondidos e erros.

---

## 5. Critérios de Aceite

- [ ] Criar campanha com template e contatos.
- [ ] Agendar campanha para horário futuro.
- [ ] Disparar campanha e enviar mensagens em massa.
- [ ] Respeitar delays aleatórios entre envios.
- [ ] Respeitar janela de sono.
- [ ] Não enviar para contatos em blacklist.
- [ ] Atualizar contadores de envio, entrega, resposta e erro.
- [ ] Bloquear campanha se limite de plano for excedido.

---

## 6. Impacto Técnico

### Backend
- `node-version/src/controllers/campaignsController.js`
- `node-version/src/services/campaignService.js`
- `node-version/src/workers/campaignWorker.js`
- `node-version/src/models/sql/models.js` (`Campaign`, `CampaignContact`)
- `node-version/src/services/whatsappCore.js`

### Infra
- RabbitMQ para fila de campanha.

### Banco
- PostgreSQL: `campaigns`, `campaign_contacts`.

---

## 7. Testes Necessários

| Tipo | Teste | Prioridade |
|---|---|---|
| Integração | Criar campanha | Alta |
| Integração | Importar contatos para campanha | Alta |
| Integração | Agendar campanha | Média |
| Integração | Disparar campanha e enviar mensagens | Alta |
| Integração | Respeitar delays e janela de sono | Média |
| Integração | Não enviar para blacklist | Média |
| Integração | Atualizar contadores | Média |

---

## 8. Riscos

| Risco | Impacto | Mitigação |
|---|---|---|
| Bloqueio do WhatsApp por envio massivo | Alto | Delays, janela de sono, limites de plano |
| Campanha com contatos inválidos | Médio | Validar números antes do envio |
| Estouro de limite de plano | Médio | Verificar antes de iniciar worker |
| Worker falhar no meio da campanha | Médio | Reprocessar contatos pendentes |

---

## 9. Dependências

- WhatsApp Gateway conectado.
- RabbitMQ disponível.
- Billing e planos configurados.
- Contatos e tags funcionando.

---

## 10. A CONFIRMAR / PENDENTE

- Segmentação avançada de contatos para campanha.
- A/B testing.
- Personalização de template com variáveis de contato.
- Relatório detalhado de campanha.
