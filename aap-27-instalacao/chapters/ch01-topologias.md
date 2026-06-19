# Ch01 — Topologias e Modelos de Instalação AAP 2.7

## Breaking Change: RPM Removido no 2.7

> O instalador RPM foi **removido** no AAP 2.7 (foi deprecated no 2.5). Apenas dois modelos de instalação são suportados:

| Modelo | Infraestrutura | Gerenciado por |
|---|---|---|
| **Container** | VMs/bare metal com RHEL + Podman | Cliente |
| **Operator** | Red Hat OpenShift Container Platform | Cliente |

## Topologias Testadas pela Red Hat

### Container Growth — Início / Prova de Conceito

**1 única VM** com todos os componentes colocados:

| Requisito | Mínimo |
|---|---|
| RAM | 16 GB (32 GB se `hub_seed_collections=true`) |
| CPUs | 4 |
| Disco total | 60 GB |
| IOPS | 3000 |
| OS | RHEL 9.4+ ou RHEL 10+ |

**Grupos do inventory:**
```
automationgateway, automationcontroller, automationhub, automationeda, database
```
(todos apontando para o mesmo host)

---

### Container Enterprise — Produção com Redundância

**11 VMs** (cada VM: 16 GB RAM, 4 CPUs, 60 GB disco, 3000 IOPS):

| VMs | Função | Grupo |
|---|---|---|
| 2 | Platform gateway + Redis colocado | `automationgateway` |
| 2 | Automation controller | `automationcontroller` |
| 2 | Private automation hub + Redis colocado | `automationhub` |
| 2 | Event-Driven Ansible + Redis colocado | `automationeda` |
| 1 | Mesh hop node | `execution_nodes` |
| 2 | Mesh execution nodes | `execution_nodes` |
| 1 | Banco de dados externo | N/A (externo) |
| 1 | HAProxy (externo, opcional) | N/A |

> Para Redis HA: 6 VMs adicionais para Redis. Redis pode ser colocado em qualquer VM de componente **exceto** execution nodes e banco de dados. **Redis externo não é suportado** no containerizado.

---

### Operator Growth — OCP Pequeno Porte

**Single Node OpenShift (SNO):**
- 32 GB RAM, 16 CPUs, 128 GB disco, 3000 IOPS

> Apenas uma instância do Operator por namespace. Múltiplos deployments isolados requerem namespaces separados.

---

### Operator Enterprise — OCP Produção

**OpenShift Cluster: 3 primary nodes + 2 worker nodes**

Requisitos externos (obrigatórios no modelo suportado):
- PostgreSQL externo: 4 vCPUs, 16 GB RAM (mínimo)
- Redis externo ou gerenciado pelo Operator

---

## Versões Suportadas

| Componente | Versão |
|---|---|
| RHEL (Container) | 9.4+ ou 10+ |
| ansible-core | 2.14 (installer RHEL9) / 2.16 (operação) |
| PostgreSQL gerenciado | 15 |
| PostgreSQL externo | 15, 16 ou 17 (requer ICU) |
| CPU arch | x86_64, AArch64, s390x (IBM Z), ppc64le (IBM Power) |
| IP | IPv4, IPv6 (single-stack e dual-stack) |

> PostgreSQL 16 ou 17 externo: backup/restore via ferramentas nativas do PostgreSQL (não pelo installer AAP).

## Escolher a Topologia Certa

```
POC / avaliação / recurso limitado  →  Container Growth (1 VM)
Produção pequena sem alta disponibilidade  →  Container Growth (reforçado)
Produção enterprise com HA  →  Container Enterprise (11+ VMs)
Ambiente OpenShift já existente  →  Operator Growth ou Enterprise
OpenShift produção crítica  →  Operator Enterprise (PostgreSQL + Redis externos)
```
