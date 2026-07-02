# Especificação: Autenticação e Identidade

**Código:** SPEC-AUTH  
**Versão:** 1.0  
**Status:** MVP / Ativo  
**Atualizado em:** 2026-07-02

---

## 1. Objetivo

Garantir que usuários e administradores acessem a plataforma de forma segura, com identidade verificada, controle de acesso por perfil e isolamento de dados por tenant. Suportar hierarquia de revendas e clientes finais.

---

## 2. Contexto

A plataforma é multi-tenant. Cada empresa (cliente final) opera em seu próprio ambiente isolado. Revendas podem criar e gerenciar subclientes. Administradores da plataforma gerenciam configurações globais. A autenticação é o ponto de entrada para todos os fluxos e deve ser robusta e escalável.

### Usuários envolvidos

- Cliente final (admin do tenant).
- Atendente/agente.
- Revenda.
- Admin master da plataforma.

---

## 3. Regras de Negócio

| ID | Regra | Origem |
|---|---|---|
| RN-AUTH-001 | O email deve ser único em toda a plataforma. | `ESCOPO.md` / `models.sql` |
| RN-AUTH-002 | A senha deve ser armazenada como hash bcrypt. | Segurança |
| RN-AUTH-003 | Cada usuário criado gera automaticamente um `tenant_id`. | Multi-tenancy |
| RN-AUTH-004 | Tokens JWT devem conter `sub` (user id) e `tenant_id`. | `authMiddleware.js` |
| RN-AUTH-005 | Rotas `/api/v1/*` exigem JWT de usuário. | `authMiddleware.js` |
| RN-AUTH-006 | Rotas `/api/v1/sadmin/*` exigem JWT de admin com role adequada. | `authMiddleware.js` |
| RN-AUTH-007 | Superadmin pode acessar dados de qualquer tenant. | RBAC |
| RN-AUTH-008 | Revendas só acessam seus próprios sub-tenants. | `resellerMiddleware.js` |
| RN-AUTH-009 | Contas inativas (`is_active = false`) não podem fazer login. | `authController.js` |
| RN-AUTH-010 | Limite de chats simultâneos por atendente (`max_concurrent_chats`). | `users` |

---

## 4. Fluxo Esperado

### 4.1 Cadastro de cliente final

1. Usuário envia email, senha e nome.
2. Sistema valida email único.
3. Sistema cria usuário e tenant.
4. Sistema emite JWT.
5. Usuário é redirecionado para onboarding.

### 4.2 Login

1. Usuário envia email e senha.
2. Sistema valida credenciais.
3. Sistema verifica se conta está ativa.
4. Sistema emite JWT.
5. Frontend armazena token e carrega contexto do tenant.

### 4.3 Login de admin master

1. Admin envia email e senha em rota separada.
2. Sistema valida credenciais e role.
3. Sistema emite JWT de admin.
4. Admin acessa painel administrativo.

### 4.4 Revenda criando subcliente

1. Revenda autenticada solicita criação de sub-tenant.
2. Sistema verifica limite de sub-tenants do plano da revenda.
3. Sistema cria usuário e tenant para o subcliente.
4. Subcliente vinculado à revenda (`reseller_id`).

---

## 5. Critérios de Aceite

- [ ] Cadastro com email duplicado retorna erro 400.
- [ ] Login com credenciais válidas retorna JWT.
- [ ] Login com senha inválida retorna erro 401.
- [ ] Conta inativa não consegue fazer login.
- [ ] Token expirado é rejeitado nas rotas protegidas.
- [ ] Usuário comum não acessa rotas de superadmin.
- [ ] Revenda não acessa sub-tenants de outra revenda.
- [ ] Superadmin consegue listar todos os tenants.
- [ ] JWT de usuário contém `tenant_id` corretamente.

---

## 6. Impacto Técnico

### Backend
- `node-version/src/controllers/authController.js`
- `node-version/src/controllers/adminController.js`
- `node-version/src/controllers/resellerController.js`
- `node-version/src/middlewares/authMiddleware.js`
- `node-version/src/middlewares/resellerMiddleware.js`
- `node-version/src/middlewares/tenancyMiddleware.js`
- `node-version/src/models/sql/models.js` (User, AdminUser, Reseller, ResellerSubTenant)

### Frontend
- `chatbot/src/Controller/AuthController.php`
- Sessão PHP armazena token e tenant_id.

### Banco
- PostgreSQL: tabelas `users`, `admin_users`, `resellers`, `reseller_sub_tenants`.

---

## 7. Testes Necessários

| Tipo | Teste | Prioridade |
|---|---|---|
| Unitário | Hash de senha com bcrypt | Alta |
| Unitário | Geração e validação de JWT | Alta |
| Integração | Cadastro com sucesso | Alta |
| Integração | Cadastro com email duplicado | Alta |
| Integração | Login com sucesso | Alta |
| Integração | Login com conta inativa | Média |
| Integração | Acesso a rota protegida sem token | Alta |
| Integração | Acesso a rota de admin com token de usuário | Alta |
| Integração | Revenda acessa sub-tenant próprio | Média |
| Integração | Revenda tenta acessar sub-tenant de outra revenda | Média |

---

## 8. Riscos

| Risco | Impacto | Mitigação |
|---|---|---|
| Secrets hardcoded | Alto | Remover fallbacks em `authMiddleware.js`, usar `.env` |
| Tokens sem expiração curta | Médio | Definir TTL e refresh token |
| Falta de MFA | Médio | Adicionar MFA em fase futura |
| Revenda extrapolar limite de sub-tenants | Médio | Validar limite no controller |

---

## 9. Dependências

- Banco PostgreSQL operacional.
- Variáveis de ambiente `SECRET_KEY` e `ADMIN_SECRET_KEY` configuradas.
- Modelos Sequelize sincronizados.

---

## 10. A CONFIRMAR / PENDENTE

- Recuperação de senha por email.
- MFA.
- Política de senha (tamanho mínimo, complexidade).
- Refresh token.
