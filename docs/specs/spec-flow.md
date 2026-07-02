# Especificação: Flow Engine (Automação)

**Código:** SPEC-FLOW  
**Versão:** 1.0  
**Status:** MVP / Ativo  
**Atualizado em:** 2026-07-02

---

## 1. Objetivo

Permitir que clientes finais criem fluxos automáticos de conversação para responder perguntas frequentes, coletar informações e direcionar clientes para atendimento humano quando necessário.

---

## 2. Contexto

O Flow Engine interpreta fluxos definidos com nodes e edges. Cada fluxo pode ter palavras-chave de gatilho. Quando uma mensagem recebida casa com um gatilho, o sistema executa o fluxo mantendo o estado da sessão do contato no MongoDB.

### Usuários envolvidos

- Cliente final (cria e configura fluxos).
- Usuário final (interage com o bot via WhatsApp).
- Atendente (recebe conversa quando há transbordo).

---

## 3. Regras de Negócio

| ID | Regra | Origem |
|---|---|---|
| RN-FLOW-001 | Cada fluxo pertence a um tenant. | Multi-tenancy |
| RN-FLOW-002 | Um fluxo pode ter múltiplas palavras-chave de gatilho. | `Flow.js` |
| RN-FLOW-003 | Apenas um fluxo ativo por gatilho em um tenant. | Validação |
| RN-FLOW-004 | Sessão de fluxo é mantida por contato e tenant. | `flow_sessions` |
| RN-FLOW-005 | Fluxo pode transferir para atendimento humano (`is_human_support = true`). | `Flow.js` |
| RN-FLOW-006 | Fluxo inativo não deve ser executado. | `is_active` |
| RN-FLOW-007 | Resposta do bot é enviada via WhatsApp Gateway. | `flowWorker.js` |
| RN-FLOW-008 | Variáveis coletadas no fluxo podem ser usadas em nodes posteriores. | `SessionState.variables` |

---

## 4. Fluxo Esperado

### 4.1 Criar fluxo

1. Cliente final define nome, descrição, gatilhos, nodes e edges.
2. Sistema valida estrutura do fluxo.
3. Sistema salva fluxo na coleção `flows`.

### 4.2 Executar fluxo

1. Mensagem recebida do contato.
2. Sistema verifica se há gatilho ativo para aquela palavra-chave.
3. Sistema cria ou recupera sessão do contato em `flow_sessions`.
4. Sistema executa node atual e determina próximo node.
5. Sistema envia resposta ao contato.
6. Se node indicar transbordo humano, conversa vai para fila de atendimento.

### 4.3 Transbordo para humano

1. Fluxo define `is_human_support = true`.
2. Sistema atualiza sessão como concluída.
3. Sistema cria/alerta atendentes disponíveis via WebSocket.
4. Conversa aparece no Inbox.

---

## 5. Critérios de Aceite

- [ ] Criar fluxo com nodes e edges válidos.
- [ ] Definir palavras-chave de gatilho.
- [ ] Mensagem com gatilho dispara o fluxo correto.
- [ ] Resposta automática é enviada ao contato.
- [ ] Sessão de fluxo é mantida entre interações.
- [ ] Transbordo humano encaminha conversa para Inbox.
- [ ] Fluxo inativo não é executado.
- [ ] Apenas fluxos do tenant são visíveis.

---

## 6. Impacto Técnico

### Backend
- `node-version/src/models/nosql/Flow.js`
- `node-version/src/services/flow/`
- `node-version/src/workers/flowWorker.js`
- `node-version/src/controllers/flowController.js` (se existir)
- `node-version/src/services/whatsappCore.js`

### Banco
- MongoDB: `flows`, `flow_sessions`.

---

## 7. Testes Necessários

| Tipo | Teste | Prioridade |
|---|---|---|
| Unitário | Parse e validação de estrutura de fluxo | Alta |
| Integração | Criar fluxo com sucesso | Alta |
| Integração | Disparar fluxo por palavra-chave | Alta |
| Integração | Executar sequência de nodes | Alta |
| Integração | Transbordo para humano | Alta |
| Integração | Fluxo inativo não dispara | Média |
| Integração | Sessão expirada reinicia fluxo | Média |

---

## 8. Riscos

| Risco | Impacto | Mitigação |
|---|---|---|
| Loop infinito em fluxo mal formado | Alto | Limitar número de interações e validar ciclos |
| Fluxo complexo sem editor visual | Médio | Planejar editor visual futuro |
| Variáveis de sessão corrompidas | Médio | Validar estrutura de `variables` |

---

## 9. Dependências

- WhatsApp Gateway para envio de respostas.
- MongoDB para `flows` e `flow_sessions`.
- Chat / Inbox para transbordo humano.

---

## 10. A CONFIRMAR / PENDENTE

- Editor visual de fluxos no frontend.
- Variáveis e condições avançadas.
- Integração com APIs externas dentro de fluxos.
- Templates pré-definidos de fluxos.
