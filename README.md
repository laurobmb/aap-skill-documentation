# Cursor Skills — Red Hat Ansible Automation Platform

Skills para o [Cursor IDE](https://cursor.sh) com conteúdo extraído da documentação oficial do **Red Hat Ansible Automation Platform**, organizados para consulta direta durante desenvolvimento, implantação e consultoria.

---

## O que são Cursor Skills?

Skills são arquivos `SKILL.md` que ensinam o agente de IA do Cursor a responder perguntas técnicas usando conteúdo real — sem alucinações e sem precisar abrir PDFs. O agente carrega apenas os capítulos relevantes para cada pergunta.

---

## Skills disponíveis — AAP 2.7 (branch `v2.7`)

> **AAP 2.7 — 16 PDFs oficiais → 11 skills** *(administer.pdf e develop.pdf são consolidados em múltiplas skills temáticas)*

| Skill | Conteúdo | Fonte |
|---|---|---|
| `aap-27-instalacao` | Topologias (Container/Operator, Growth/Enterprise), inventory file completo, Redis HA, PostgreSQL externo, portas e rede | plan + install |
| `aap-27-seguranca` | Autenticação centralizada (LDAP, SAML, OIDC, 14 providers), authenticator maps, RBAC (orgs/times/roles), credentials + external secrets, PATs via Gateway | secure + administer |
| `aap-27-desenvolvimento` | ansible-dev-tools, EEs com MCP server (GitHub/AWS), EDA rulebooks, CaC ansible.platform, coleções e Private Hub | develop + get-started |
| `aap-27-extensao` | **Ansible MCP Server standalone** (novo 2.7, porta 8448), Intelligent Assistant + BYOK (RAG customizado), Lightspeed Coding, Terraform/Vault | extend + integrate |
| `aap-27-operacao` | Performance tuning (capacity algorithm, forks, EDA scaling), OCP pod resources, metrics-utility, Automation Dashboard, troubleshooting | optimize + observe + troubleshoot |
| `aap-27-upgrade` | **Breaking changes 2.7** (RPM removido, auth direta removida), checklist pré-upgrade, detecção de acesso direto, upgrade containerizado/OCP, migração auth/CaC/APIs | upgrade + migrate + reference + whats-new |
| `aap-27-controller-uso` | **Job Templates** (todos os campos, surveys, job slicing, extra_vars, webhooks SCM), **Workflows** (visualizer, convergência, approval nodes, artifacts), **Projects** (SCM, branch override, git refspec), **Inventários dinâmicos** (VMware vCenter/ESXi, AWS EC2, Azure, Satellite, Terraform State, OpenShift Virtualization, GCE), **Notificações** (Slack, email, webhook, PagerDuty) | administer + develop |
| `aap-27-mesh-backup` | **Automation Mesh** (hybrid/control/execution/hop, topologias de design, single-hop, multi-hop, outbound-only), adição de nós em OCP via UI, Mesh Ingress (sem inbound), topology viewer, CA customizada, **Backup/Restore** (containerizado: backup.yml/restore.yml; OCP: AnsibleAutomationPlatformBackup CR; metrics service) | administer |
| `aap-27-hub-admin` | **Private Automation Hub** — autenticação 2.7 (offline token via Gateway), sync rh-certified (console.redhat.com) e community (Galaxy), namespaces + times + fluxo de aprovação de coleções, tipos de repositório (staging/approved/custom), container registry (pull/push/tag EEs, sync de registros externos, permissões de times), assinatura GPG de conteúdo | administer |
| `aap-27-eda-operacao` | **Event-Driven Ansible** — ciclo de vida de Rulebook Activations (criar, habilitar/desabilitar/reiniciar/duplicar, restart policy, log levels), **Event Streams** (6 tipos: HMAC/Basic/Token/OAuth2/ECDSA/mTLS; GitHub/GitLab/ServiceNow/Dynatrace especializados), headers HTTP seguros, UUID estático para DR, modo teste, Rule Audit, webhooks em OCP (Route), troubleshooting (IDs de rastreamento tid/aiid/rid, pending, restart loop) | administer |
| `aap-27-api-rest` | **API REST do Controller** — navegação (endpoint /api/controller/v2/), named URLs (acesso por nome sem PK), filtros avançados (field lookups, OR/AND/NOT/chain), autenticação OAuth2 PAT (recomendado), Basic, Session, SSO; exemplos práticos (lançar JT/WF, monitorar jobs, criar inventário), **Metrics Service API** (/api/metrics/v1/health/, /tasks/) | develop |

**Breaking changes obrigatórios documentados no 2.7:**
- Instalador RPM **removido** — apenas Container (RHEL+Podman) e Operator (OCP)
- Autenticação direta a controller/hub/EDA **removida** — tudo via Platform Gateway
- Tokens de componente (controller PATs) **removidos** — recriar no Gateway
- LDAP/SAML do Controller **removidos** — reconfigurar no Gateway

---

## Skills disponíveis — AAP 2.6 (branch `v2.6`)

| Skill | Conteúdo |
|---|---|
| `aap-26-instalacao` | Arquitetura, topologias, instalação containerizada e Operator/OpenShift, rede e portas, PostgreSQL |
| `aap-26-automacoes` | EDA Rulebooks, Decision Environments, Configuration as Code, Execution Environments, desenvolvimento |
| `aap-26-operacao` | Gestão de acesso, autenticação (15 métodos), hardening/STIG, troubleshooting, logs |
| `aap-26-mesh-performance` | Automation Mesh (nós hybrid/control/execution/hop), topologias, OCP Mesh, performance tuning, upgrade |
| `aap-26-integracoes` | Terraform + Vault (HashiCorp), Private Automation Hub (coleções e EEs), awx-manage, backup/restore OCP |
| `aap-26-controller-uso` | Job Templates, Workflows + Visualizer, Inventories dinâmicos (VMware/Satellite/AWS/Azure), Instance Groups, API REST e webhooks |
| `aap-26-analytics-migracao` | Migração entre deployment types, Automation Calculator (ROI), Savings Planner, Job Explorer |
| `aap-26-dashboard` | Automation Dashboard (app local para métricas de até 3 instâncias AAP) |
| `aap-26-dev-tools` | ansible-navigator, Ansible Lightspeed Coding Assistant, Automation Intelligent Assistant com MCP |
| `aap-26-self-service` | Self-Service Automation Portal (Helm/OCP, RBAC, troubleshooting) |
| `aap-26-developer-hub` | Ansible Plug-ins para Red Hat Developer Hub |

---

## Instalação

### Opção 1: Clone direto no ~/.cursor/skills/

```bash
# AAP 2.7
git clone -b v2.7 git@github.com:laurobmb/skills-cursor-aap-2.6.git /tmp/aap27-skills
cp -r /tmp/aap27-skills/aap-27-* ~/.cursor/skills/

# AAP 2.6
git clone -b v2.6 git@github.com:laurobmb/skills-cursor-aap-2.6.git /tmp/aap26-skills
cp -r /tmp/aap26-skills/aap-26-* ~/.cursor/skills/
```

### Opção 2: Usar as skills direto do workspace

```
Adicione os diretórios das skills ao seu workspace Cursor.
Exemplo: ~/.cursor/skills/aap-27-instalacao/SKILL.md
```

---

## Como usar no Cursor

Após copiar as skills para `~/.cursor/skills/`, elas ficam disponíveis automaticamente no Cursor IDE.

**Exemplos de uso:**

```
"Qual topologia usar para produção AAP 2.7 com alta disponibilidade?"
→ usa aap-27-instalacao

"Como configuro LDAP no AAP 2.7 com authenticator maps para organizações?"
→ usa aap-27-seguranca

"Como faço upgrade do AAP 2.6 para 2.7? Quais breaking changes?"
→ usa aap-27-upgrade

"Como conecto o Cursor ao AAP via MCP server?"
→ usa aap-27-extensao

"Quais forks o meu nó de 16GB RAM suporta?"
→ usa aap-27-operacao

"Como criar um EE com MCP server do GitHub?"
→ usa aap-27-desenvolvimento

"Como criar um Job Template com survey e job slicing no AAP 2.7?"
→ usa aap-27-controller-uso

"Como adicionar execution nodes remotos via hop node no AAP 2.7?"
→ usa aap-27-mesh-backup

"Como fazer backup antes do upgrade no ambiente containerizado?"
→ usa aap-27-mesh-backup

"Como criar inventário dinâmico do VMware vCenter ou Terraform State?"
→ usa aap-27-controller-uso

"Como configuro o Hub para sincronizar coleções certificadas Red Hat para 30 organizações?"
→ usa aap-27-hub-admin

"Como criar namespaces e times para publicar coleções internas com aprovação?"
→ usa aap-27-hub-admin

"Como fazer push de EE customizada para o registry do Hub?"
→ usa aap-27-hub-admin

"Como criar um event stream do GitHub (HMAC) para disparar automações no EDA?"
→ usa aap-27-eda-operacao

"Minha rulebook activation está presa em Pending ou reiniciando em loop. Como resolver?"
→ usa aap-27-eda-operacao

"Como integrar o GLPI com EDA via webhook em OCP (Route)?"
→ usa aap-27-eda-operacao

"Como lançar um Job Template via API REST com curl ou Python?"
→ usa aap-27-api-rest

"Como filtrar jobs por status, data e organização via API?"
→ usa aap-27-api-rest

"Como criar um token OAuth2 para uso em CI/CD?"
→ usa aap-27-api-rest
```

---

## Sobre o Ansible MCP Server no AAP 2.7

O AAP 2.7 introduz o **Ansible MCP Server** como componente standalone (Tech Preview) — diferente do MCP do chatbot do 2.6:

| | AAP 2.6 (Intelligent Assistant) | AAP 2.7 (MCP Server standalone) |
|---|---|---|
| Tipo | Chatbot interno na UI do AAP | Servidor MCP externo (porta 8448) |
| Consumidor | Usuários do AAP | AI agents: Cursor, Claude, ChatGPT |
| Requer LLM | Sim | Não |
| Status | GA (OCP) / TP (Container) | Tech Preview |

---

## Mapa de PDFs → Skills (AAP 2.7)

| PDF Oficial | Skill |
|---|---|
| `aap-plan-2-7-pdf.pdf` | aap-27-instalacao |
| `aap-install-2-7-pdf.pdf` | aap-27-instalacao |
| `aap-secure-2-7-pdf.pdf` | aap-27-seguranca |
| `aap-administer-2-7-pdf.pdf` | aap-27-seguranca + **aap-27-controller-uso** + **aap-27-mesh-backup** |
| `aap-develop-2-7-pdf.pdf` | aap-27-desenvolvimento + **aap-27-controller-uso** |
| `aap-get-started-2-7-pdf.pdf` | aap-27-desenvolvimento |
| `aap-extend-2-7-pdf.pdf` | aap-27-extensao |
| `aap-integrate-2-7-pdf.pdf` | aap-27-extensao |
| `aap-optimize-2-7-pdf.pdf` | aap-27-operacao |
| `aap-observe-2-7-pdf.pdf` | aap-27-operacao |
| `aap-troubleshoot-2-7-pdf.pdf` | aap-27-operacao |
| `aap-upgrade-2-7-pdf.pdf` | aap-27-upgrade |
| `aap-migrate-2-7-pdf.pdf` | aap-27-upgrade |
| `aap-reference-2-7-pdf.pdf` | aap-27-upgrade |
| `aap-whats-new-2-7-pdf.pdf` | aap-27-upgrade |
| `aap-discover-2-7-pdf.pdf` | aap-27-upgrade |

> O `aap-administer-2-7-pdf.pdf` (13MB, ~12.000 linhas) e `aap-develop-2-7-pdf.pdf` (23MB, ~20.000 linhas) contêm conteúdo suficiente para mais de uma skill — daí a separação em `aap-27-controller-uso`, `aap-27-mesh-backup`, `aap-27-hub-admin`, `aap-27-eda-operacao` e `aap-27-api-rest`.

| PDF Oficial | Skills mapeadas |
|---|---|
| `aap-administer-2-7-pdf.pdf` | aap-27-seguranca + **aap-27-controller-uso** + **aap-27-mesh-backup** + **aap-27-hub-admin** + **aap-27-eda-operacao** |
| `aap-develop-2-7-pdf.pdf` | aap-27-desenvolvimento + **aap-27-controller-uso** + **aap-27-api-rest** |

---

## Créditos

Skills geradas com o processo [book-to-skill](https://github.com/virgiliojr94/book-to-skill) — ferramenta para conversão de documentação técnica (PDFs) em skills estruturadas para agentes de IA.

Documentação fonte: [Red Hat Ansible Automation Platform 2.7](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.7) e [AAP 2.6](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6).
