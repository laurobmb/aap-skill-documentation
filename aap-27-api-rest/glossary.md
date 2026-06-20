# Glossário — AAP 2.7 API REST

| Termo | Definição |
|---|---|
| **Platform Gateway** | Ponto de entrada unificado da API no AAP 2.7. Todas as chamadas passam por `https://<gateway>/api/` |
| **OAuth2 PAT** | Personal Access Token OAuth2 — método recomendado para autenticação programática. Escopo configurável (read/write) |
| **Basic Authentication** | Autenticação com username:password codificados em Base64. Não recomendado para produção. Pode ser desabilitado |
| **Session Authentication** | Cookie de sessão para uso no browser. Timeout padrão de 1800s |
| **Named URL** | URL legível para identificar recursos por nome (sem PK numérico), usando padrão `<name>++<org>` |
| **NAMED_URL_FORMATS** | Endpoint `/api/controller/v2/settings/named-url/` listando formatos de Named URL por tipo de recurso |
| **Field lookup** | Sufixo no parâmetro de filtro para comparações: `__exact`, `__contains`, `__gte`, `__isnull`, etc. |
| **not__ prefix** | Prefixo de filtro para negação: `?not__status=successful` |
| **or__ prefix** | Prefixo para OR em filtros: `?or__status=failed&or__status=error` |
| **chain__ prefix** | Prefixo para aplicar filtros separadamente em cada objeto relacionado |
| **extra_vars** | Parâmetros extras passados ao job via API. Preservados no formato original (YAML ou JSON) |
| **Metrics Service API** | API separada (`/api/metrics/v1/`) para monitoramento de saúde, tarefas e feature flags |
| **csrftoken** | Token CSRF necessário para autenticação via session. Obtido via GET em `/api/gateway/v1/login/` |
| **Bearer token** | Formato do header de autorização OAuth2: `Authorization: Bearer <token>` |
| **Escopo (scope)** | Permissões do token OAuth2: `read` (somente leitura) ou `write` (leitura e escrita) |
| **Pagination** | API retorna resultados paginados. Controlar com `?limit=N&offset=M` |
| **/api/v1/health/** | Endpoint de saúde do metrics service — monitora DB, scheduler e componentes |
| **SSO** | Single Sign-On via LDAP, SAML, OIDC, GitHub, Azure — autenticação externa ao platform gateway |
