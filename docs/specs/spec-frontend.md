# Especificação: Frontend PHP

**Código:** SPEC-FRONTEND  
**Versão:** 1.0  
**Status:** MVP / Parcial  
**Atualizado em:** 2026-07-02

---

## 1. Objetivo

Fornecer uma interface web para atendentes e administradores de clientes finais acessarem a plataforma, consumindo a API Node.js de forma segura e responsiva.

---

## 2. Contexto

O frontend é escrito em PHP 8 com padrão MVC próprio. Ele consome a API Node.js através de um cliente customizado (`OmniChannelApiClient`) e renderiza views server-side. A autenticação é feita via API Node, e a sessão PHP armazena o token JWT.

### Usuários envolvidos

- Atendente.
- Cliente final (admin do tenant).
- Revenda (administra sub-tenants via API).

---

## 3. Regras de Negócio

| ID | Regra | Origem |
|---|---|---|
| RN-FRONT-001 | Frontend não armazena lógica de negócio; é apenas consumidor da API. | Arquitetura |
| RN-FRONT-002 | Sessão PHP mantém token JWT e tenant_id. | `HomeController.php` |
| RN-FRONT-003 | Rotas protegidas redirecionam para login se não autenticado. | `Router.php` |
| RN-FRONT-004 | Frontend exibe erros da API de forma amigável. | `ApiException` |
| RN-FRONT-005 | Frontend sincroniza contatos e status do bot na home. | `HomeController.php` |
| RN-FRONT-006 | Frontend disponibiliza endpoints JSON proxy (`ApiOmniController`). | `ApiOmniController.php` |
| RN-FRONT-007 | Frontend deve ser compatível com Apache e mod_rewrite. | `Dockerfile` |

---

## 4. Fluxo Esperado

### 4.1 Login no frontend

1. Usuário acessa tela de login.
2. Frontend envia credenciais para API Node.
3. API retorna JWT.
4. Frontend armazena JWT em sessão PHP.
5. Frontend redireciona para home.

### 4.2 Home do atendente

1. Atendente autenticado acessa home.
2. Frontend busca contatos e status do bot na API.
3. Frontend renderiza lista de contatos e status da conexão WhatsApp.
4. Frontend exibe erros caso API indisponível.

### 4.3 Proxy JSON para API

1. Frontend recebe requisição em `/api/omni/*`.
2. Valida sessão.
3. Encaminha para API Node com token JWT.
4. Retorna resposta JSON para o cliente JavaScript.

---

## 5. Critérios de Aceite

- [ ] Tela de login funciona e redireciona autenticado.
- [ ] Home exibe contatos e status do bot.
- [ ] Sessão mantém token JWT corretamente.
- [ ] Rotas protegidas redirecionam para login.
- [ ] Proxy JSON encaminha requisições para API Node.
- [ ] Erros da API são exibidos de forma amigável.
- [ ] Frontend roda em Apache com mod_rewrite.

---

## 6. Impacto Técnico

### Frontend
- `chatbot/src/Controller/AuthController.php`
- `chatbot/src/Controller/HomeController.php`
- `chatbot/src/Controller/ApiOmniController.php`
- `chatbot/src/Service/OmniChannelApiClient.php`
- `chatbot/src/Service/ApiException.php`
- `chatbot/src/Router.php`
- `chatbot/src/Bootstrap.php`
- `chatbot/views/`
- `chatbot/public/router.php`

### Banco local
- MySQL: tabela `users` (uso mínimo/legado).

---

## 7. Testes Necessários

| Tipo | Teste | Prioridade |
|---|---|---|
| Integração | Login redireciona para home | Alta |
| Integração | Acesso a home sem sessão redireciona para login | Alta |
| Integração | Proxy JSON retorna dados da API | Média |
| Integração | Exibição de erro amigável da API | Média |
| Integração | Logout limpa sessão | Média |

---

## 8. Riscos

| Risco | Impacto | Mitigação |
|---|---|---|
| Sessão PHP expirar antes do JWT | Médio | Sincronizar expirações ou renovar token |
| Frontend depender muito da API Node | Médio | Tratar indisponibilidade da API com mensagens claras |
| Falta de testes automatizados no PHP | Alto | Configurar PHPUnit |

---

## 9. Dependências

- API Node.js operacional.
- MySQL local para tabela `users`.
- Apache com mod_rewrite.

---

## 10. A CONFIRMAR / PENDENTE

- Interface completa de chat/Inbox.
- Dashboards e relatórios no frontend.
- Editor visual de fluxos.
- Telas de administração de revendas.
