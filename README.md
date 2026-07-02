# SaaS Chatbot — Atendimento Inteligente via WhatsApp

Centralize, automatize e escale o atendimento ao cliente da sua empresa — ou da sua carteira de clientes — com uma plataforma white-label completa de chatbots e atendimento humano via WhatsApp.

---

## O que é

O **SaaS Chatbot** é uma plataforma multilocatária (multi-tenant) que permite a empresas de qualquer porte operarem atendimento via WhatsApp com automação inteligente, filas humanas, histórico de conversas, segmentação de contatos e cobrança por planos de uso. O modelo também suporta **revendas white-label**, permitindo que parceiros comerciais ofereçam a solução com sua própria marca e gerenciem seus clientes finais.

---

## Para quem é

- **Empresas** que precisam atender clientes pelo WhatsApp sem depender de celulares individuais.
- **Agências e revendas** que querem oferecer chatbot como serviço com marca própria.
- **Atendentes** que precisam de uma interface unificada para responder conversas em tempo real.
- **Gestores** que precisam de controle sobre limites de uso, planos e faturamento.

---

## Principais benefícios

### 1. Atendimento centralizado no WhatsApp

Conecte um ou mais números de WhatsApp da empresa em uma única plataforma. Todas as conversas chegam a uma caixa de entrada organizada, com histórico completo e acesso em tempo real.

### 2. Automação com fluxos de conversação

Crie fluxos automáticos de atendimento — saudações, menus de opções, perguntas frequentes e coleta de dados — sem necessidade de programação. O sistema direciona o cliente para o atendimento humano apenas quando realmente necessário.

### 3. Atendimento humano em tempo real

Atendentes visualizam conversas abertas, respondem mensagens, transferem atendimentos e acompanham status de leitura e entrega. Tudo em uma interface web moderna e responsiva.

### 4. Inteligência artificial integrada

Respostas inteligentes com IA local via Ollama, com suporte a base de conhecimento (RAG) para respostas contextualizadas. Ideal para dúvidas recorrentes e automação avançada.

### 5. Segmentação de contatos e campanhas

Organize contatos com tags, importe listas em lote e dispare campanhas de mensagens com agendamento, delays automáticos e controle de horários.

### 6. Modelo de negócio flexível

Planos de uso com limites de bots, atendentes e mensagens, assinaturas por cliente e faturamento integrado. Revendas gerenciam subclientes e comissionamento de forma hierárquica.

### 7. White-label para revendas

Revendas operam com sua própria identidade visual, gerenciando clientes finais dentro da plataforma sem precisar desenvolver tecnologia própria.

---

## Funcionalidades principais

| Área | O que você consegue fazer |
|---|---|
| **Conexão WhatsApp** | Conectar números via QR Code, enviar/receber textos e mídias, acompanhar status de entrega e leitura. |
| **Chat / Inbox** | Visualizar conversas, responder em tempo real, transferir entre atendentes e manter histórico persistente. |
| **Automação de Fluxos** | Criar fluxos de bot com palavras-chave de gatilho e transbordo para humano. |
| **Contatos e Tags** | Cadastrar, importar, segmentar e sincronizar contatos do WhatsApp. |
| **Campanhas** | Disparar mensagens em massa com agendamento, delays e controle de janela de envio. |
| **Inteligência Artificial** | Configurar IA com base de conhecimento para respostas automáticas contextuais. |
| **Planos e Faturamento** | Definir planos, controlar assinaturas, gerar faturas e acompanhar transações. |
| **Revendas e White-label** | Criar hierarquia de revendas com subclientes e marca própria. |
| **Administração** | Gerenciar usuários, tenants, permissões, auditoria e configurações globais. |

---

## Como funciona

1. **Cadastro** — a empresa ou revenda cria sua conta na plataforma.
2. **Conexão** — conecta o número de WhatsApp da empresa escaneando um QR Code.
3. **Configuração** — define fluxos automáticos, base de conhecimento e atendentes.
4. **Operação** — clientes finais enviam mensagens e o sistema responde automaticamente ou encaminha para um atendente humano.
5. **Gestão** — gestores acompanham conversas, métricas, limites de plano e faturamento.

---

## Tecnologia e segurança

- Isolamento completo de dados entre clientes (multi-tenancy).
- Autenticação segura com JWT e controle de acesso por perfil (RBAC).
- Histórico de mensagens armazenado de forma escalável.
- Infraestrutura containerizada com Docker para fácil implantação.
- Logs de auditoria para ações administrativas.

---

## Comece agora

Para executar a plataforma localmente ou em seu ambiente, consulte a documentação técnica em `docs/technical/` e a governança do projeto em `AGENTS.md`.

- **Visão geral da arquitetura:** `docs/governanca/ARQUITETURA.md`
- **Modelo de dados:** `docs/governanca/BANCO_DADOS.md`
- **Escopo e regras de negócio:** `docs/governanca/ESCOPO.md`
- **Roadmap de desenvolvimento:** `docs/governanca/ROADMAP.md`
- **Contexto e decisões:** `docs/governanca/CONTEXTO.md`
- **Registro de trabalho:** `docs/governanca/RELATORIO.md`
- **Especificações técnicas:** `docs/specs/`

---

## Status do projeto

O projeto encontra-se em fase de consolidação do MVP, com backend ativo em Node.js, frontend em PHP e infraestrutura orquestrada via Docker Compose. Consulte `docs/governanca/ROADMAP.md` para próximas etapas.

---

**SaaS Chatbot** — seu atendimento, mais inteligente.