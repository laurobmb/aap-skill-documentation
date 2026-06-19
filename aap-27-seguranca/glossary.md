# Glossário — AAP 2.7 Segurança

**Platform Gateway** — Componente central de proxy, roteamento e autenticação do AAP 2.7. Todo acesso externo passa por ele.

**Authenticator** — Instância de um plugin de autenticação (ex: "LDAP corporativo"). Pode haver múltiplos autenticadores do mesmo tipo.

**Authenticator Plugin** — Código que implementa um tipo de autenticação (LDAP, SAML, GitHub, etc.). Equivalente a coleções Ansible.

**Authenticator Map** — Regra que define permissões (allow, org, team, role, superuser) com base em triggers do IdP. Processada em ordem.

**Trigger** — Condição que dispara um authenticator map (Always, Never, Group, Attribute).

**ALLOW / SKIPPED / DENY** — Resultados possíveis de um authenticator map. SKIPPED + Revoke = DENY.

**PAT (Personal Access Token)** — Token de acesso pessoal criado pelo usuário. No 2.7, criado exclusivamente via Platform Gateway.

**OAuth2 Application** — Registro de uma aplicação externa (Jenkins, ServiceNow) que usa OAuth2 para acessar o AAP via tokens.

**Role** — Conjunto de permissões atribuível a usuários ou times em recursos específicos.

**Organization** — Unidade de isolamento de recursos no AAP. Cada org tem suas próprias credentials, inventories, projects, JTs.

**Team** — Grupo de usuários dentro de uma organização. Facilita atribuição de roles em bulk.

**External Secret Provider** — Integração com cofres de segredos externos (HashiCorp Vault, Azure Key Vault, CyberArk) para buscar senhas em runtime sem armazená-las no AAP.

**Credential Type** — Definição de schema de uma credencial. Tipos built-in + customizáveis via UI/CaC.

**no_log: true** — Diretiva Ansible que impede que inputs de tasks com senhas apareçam nos logs de jobs.
