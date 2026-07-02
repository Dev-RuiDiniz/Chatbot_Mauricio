# Especificações do Sistema (SDD)

Este diretório contém as especificações técnicas e funcionais do **SaaS Chatbot**, elaboradas seguindo **Specification-Driven Development (SDD)**.

Cada especificação descreve:

- Objetivo da funcionalidade.
- Contexto e usuários envolvidos.
- Regras de negócio.
- Fluxo esperado.
- Critérios de aceite.
- Impacto técnico.
- Testes necessários.
- Riscos.
- Dependências.

## Índice de especificações

| Arquivo | Módulo | Descrição |
|---|---|---|
| `spec-auth.md` | Autenticação e Identidade | Cadastro, login, JWT, roles, multi-tenancy, revendas |
| `spec-whatsapp.md` | WhatsApp Gateway | Conexão, envio/recebimento, QR code, status de mensagens |
| `spec-chat.md` | Chat / Inbox | Caixa de entrada, histórico, WebSocket, transferência |
| `spec-contacts.md` | Contatos e Tags | CRUD, importação, segmentação, sincronização |
| `spec-flow.md` | Flow Engine | Automação de fluxos, gatilhos, transbordo humano |
| `spec-ai.md` | Inteligência Artificial | Configuração de IA, RAG, respostas automáticas |
| `spec-billing.md` | Billing e Planos | Planos, assinaturas, faturas, transações |
| `spec-campaigns.md` | Campanhas | Disparo em massa, agendamento, delays, métricas |
| `spec-admin.md` | Admin e Revendas | Painel administrativo, auditoria, white-label |
| `spec-frontend.md` | Frontend PHP | Interface web, consumo da API, views |

## Como usar

Antes de implementar ou alterar qualquer funcionalidade:

1. Leia a especificação correspondente.
2. Valide se o requisito ainda está alinhado com o escopo (`docs/governanca/ESCOPO.md`).
3. Atualize a especificação se necessário.
4. Crie ou ajuste os testes antes de codificar (TDD).
5. Após implementar, atualize `docs/governanca/CONTEXTO.md` e `docs/governanca/RELATORIO.md`.
