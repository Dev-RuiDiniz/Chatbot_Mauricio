# Especificação: Admin e Revendas

**Código:** SPEC-ADMIN  
**Versão:** 1.0  
**Status:** MVP / Ativo  
**Atualizado em:** 2026-07-02

---

## 1. Objetivo

Permitir que administradores da plataforma gerenciem tenants, revendas, planos, faturas e auditoria, e que revendas operem white-label gerenciando seus subclientes.

---

## 2. Contexto

O painel administrativo é separado do painel do cliente final. Admins têm roles distintas (`superadmin`, `support`, `finance`, `readonly`). Revendas têm uma hierarquia própria com limites de sub-tenants e comissionamento.

### Usuários envolvidos

- Admin master (`superadmin`).
- Admin support, finance, readonly.
- Revenda.
- Subcliente (cliente final vinculado a revenda).

---

## 3. Regras de Negócio

| ID | Regra | Origem |
|---|---|---|
| RN-ADMIN-001 | Apenas admins podem acessar rotas `/api/v1/sadmin/*`. | `authMiddleware.js` |
| RN-ADMIN-002 | Roles definem permissões: superadmin, support, finance, readonly. | `AdminUser` |
| RN-ADMIN-003 | Ações administrativas são registradas em `audit_logs`. | `AuditLog` |
| RN-ADMIN-004 | Revenda é um tenant com permissões adicionais. | `Reseller` |
| RN-ADMIN-005 | Revenda pode criar sub-tenants até o limite do plano. | `Reseller` (`max_sub_tenants`) |
| RN-ADMIN-006 | Sub-tenant é vinculado a uma revenda. | `ResellerSubTenant` |
| RN-ADMIN-007 | Superadmin pode bloquear/desbloquear tenants. | `adminController.js` |
| RN-ADMIN-008 | Admin finance pode gerenciar faturas e transações. | RBAC |
| RN-ADMIN-009 | Admin readonly pode visualizar mas não alterar. | RBAC |
| RN-ADMIN-010 | Audit logs são imutáveis. | `AuditLog` (sem `updated_at`) |

---

## 4. Fluxo Esperado

### 4.1 Criar admin

1. Superadmin acessa painel administrativo.
2. Define email, nome, senha e role.
3. Sistema salva admin user com hash de senha.

### 4.2 Criar revenda

1. Superadmin ou admin autorizado cria revenda.
2. Define tenant, nome, plano, limite de sub-tenants e comissão.
3. Sistema cria `Reseller` e vincula a um tenant existente.

### 4.3 Revenda cria subcliente

1. Revenda autenticada solicita criação de sub-tenant.
2. Sistema verifica limite de `max_sub_tenants`.
3. Sistema cria usuário e tenant para subcliente.
4. Sistema vincula sub-tenant à revenda.

### 4.4 Auditar ação administrativa

1. Admin executa ação em tenant, revenda ou usuário.
2. Sistema captura ação, entidade, detalhes, IP e user agent.
3. Sistema salva em `audit_logs`.

---

## 5. Critérios de Aceite

- [ ] Superadmin acessa rotas administrativas.
- [ ] Roles restringem ações no painel admin.
- [ ] Criar revenda com sucesso.
- [ ] Revenda cria sub-tenant dentro do limite.
- [ ] Bloqueio de tenant funciona.
- [ ] Ações administrativas são auditadas.
- [ ] Admin readonly não altera dados.
- [ ] Finance admin gerencia faturas.

---

## 6. Impacto Técnico

### Backend
- `node-version/src/controllers/adminController.js`
- `node-version/src/controllers/resellerController.js`
- `node-version/src/middlewares/authMiddleware.js`
- `node-version/src/middlewares/resellerMiddleware.js`
- `node-version/src/models/sql/models.js` (`AdminUser`, `AuditLog`, `Reseller`, `ResellerSubTenant`)

### Banco
- PostgreSQL: `admin_users`, `audit_logs`, `resellers`, `reseller_sub_tenants`.

---

## 7. Testes Necessários

| Tipo | Teste | Prioridade |
|---|---|---|
| Integração | Criar admin com role | Alta |
| Integração | Acesso a rota de superadmin bloqueado para admin readonly | Alta |
| Integração | Criar revenda | Alta |
| Integração | Revenda criar sub-tenant | Alta |
| Integração | Revenda excede limite de sub-tenants | Média |
| Integração | Bloquear tenant | Média |
| Integração | Auditar ação administrativa | Alta |

---

## 8. Riscos

| Risco | Impacto | Mitigação |
|---|---|---|
| Escalada de privilégios | Alto | Validar roles em todos os endpoints |
| Revenda extrapolar limite | Médio | Verificar `max_sub_tenants` antes de criar |
| Audit log não registrar ação | Médio | Middleware de auditoria em rotas administrativas |

---

## 9. Dependências

- Autenticação e autorização funcionando.
- Billing e planos configurados.
- PostgreSQL operacional.

---

## 10. A CONFIRMAR / PENDENTE

- Dashboards financeiros detalhados.
- Comissionamento automático para revendas.
- Relatório de auditoria com filtros.
- Configuração de marca white-label no frontend.
