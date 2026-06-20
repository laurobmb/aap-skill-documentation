# Ch04 — Container Registry do Hub (Execution Environments)

## O Hub como Registry Interno

O Private Automation Hub funciona como um registry interno de EEs para a organização:

```
Fluxo padrão:
  registry.redhat.io ──pull──► máquina local ──push──► hub.empresa.org/ee-infra:latest
                                                               │
                                                               └──pull──► AAP Controller (jobs)
```

## Pull de EE do registry.redhat.io

```bash
# 1. Login no registry Red Hat
podman login registry.redhat.io

# 2. Pull da EE desejada
podman pull registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9:latest
podman pull registry.redhat.io/ansible-automation-platform-25/ee-minimal-rhel9:latest

# 3. Verificar imagem local
podman images
```

## Tag para o Hub Privado

```bash
# Tag para seu Hub privado (antes de fazer push)
podman tag registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9:latest \
  hub.empresa.org/ee-supported-rhel9:2.7

# Ou com nome customizado
podman tag registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9:latest \
  hub.empresa.org/infra/ee-rhel9-satellite:latest
```

## Push de EE para o Hub

```bash
# Login no Hub privado
podman login hub.empresa.org

# Push
podman push hub.empresa.org/ee-supported-rhel9:2.7
```

## Gerenciar Tags via UI

```
Automation Content → Execution Environments → [EE]
  → Images tab → More Actions (⋮) → Manage tags
  → Adicionar nova tag no campo de texto → Add
  → Remover tag existente: clicar "x" na tag
```

## Sincronizar EE de Registry Externo para o Hub

```
# Via UI:
Automation Content → Execution Environments → Create execution environment

Campos:
  Name:          ee-supported-rhel9    (nome local no Hub)
  Upstream name: ee-supported-rhel9   (nome no registry remoto)
  Registry:      registry.redhat.io   (adicionar antes em Registries)
  Tags:          latest, 2.7          (limitar tags para economizar disco)
  → Create

→ More Actions (⋮) → Sync
```

> ⚠️ Sincronizar EEs com muitas tags consome muito disco. Especificar tags de inclusão/exclusão.

## Credencial de Registry (para registros autenticados)

```
# Para registries que requerem autenticação (registry.redhat.io, Quay privado):
Automation Execution → Infrastructure → Credentials → Create credential

Tipo:           Container Registry
Authentication URL: registry.redhat.io  (ou URL do registry)
Username:       sua-conta-RHN
Password/Token: senha ou token de service account
☑ Verify SSL

→ Associar essa credencial ao EE no Hub ou no Controller
```

## Configurar Permissões de Time no Container Registry

```
Automation Content → Execution Environments → [EE] → Team Access

Papéis disponíveis:
  Create new containers            → criar novos containers
  Change container namespace permissions → alterar permissões do repositório
  Change container                 → alterar informações do container
  Change execution environment tags → modificar tags
  Push to existing container       → fazer push para container existente
```

## Adicionar README ao Repositório de Containers

```
Automation Content → Execution Environments → [EE] → Detail tab → Add

Inserir texto em Markdown descrevendo:
- Propósito da EE
- Coleções incluídas
- Versão do Python/Ansible core
- Como usar (exemplo de navigator.yml ou Job Template)

→ Save
```

## Endereços do Registry Quay (atualização 2025)

```
# Liberar no firewall (outbound TCP 80/443):
cdn.quay.io
cdn01.quay.io → cdn06.quay.io   (novos endpoints adicionados em abril 2025)
registry.redhat.io
registry.access.redhat.com
```

## Referenciar EE do Hub no Controller

```yaml
# No Job Template via CaC:
ansible.platform.execution_environment:
  name: "EE Supported RHEL9"
  image: "hub.empresa.org/ee-supported-rhel9:2.7"
  credential: "Hub Registry Credential"
  controller_host: "https://aap.empresa.org"
  controller_oauthtoken: "{{ gateway_token }}"
```
