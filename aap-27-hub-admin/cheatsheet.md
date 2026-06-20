# Cheatsheet — AAP 2.7 Hub Admin

## Autenticação (mudança 2.7)

```
✗ API direta ao Hub não é mais suportada
✓ Toda autenticação via Platform Gateway
✓ Offline token para sync com console.redhat.com (expira em 30d de inatividade)
```

## Fluxo de Aprovação de Coleções

```
upload → staging repo → Collection Approvals ("Needs review")
       → 👍 Aprovado → Published repo
       → 👎 Rejeitado → Rejected repo
       → Auto-approve habilitado → promovido automaticamente para approved repos
```

## Tipos de Repositório

```
pipeline=staging  → qualquer upload → approval dashboard (oculto da busca)
pipeline=approved → somente via promoção do staging
sem pipeline      → publicação direta (custom, público ou privado)
```

## Push de EE para Hub

```bash
podman login registry.redhat.io
podman pull registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9:latest
podman tag  registry.redhat.io/.../ee-supported-rhel9:latest hub.empresa.org/ee-rhel9:2.7
podman login hub.empresa.org
podman push hub.empresa.org/ee-rhel9:2.7
```

## Rollback de Repositório

```
Automation Content → Repositories → [repo] → Versions → Revert to this version
```

## Verificação de Assinatura

```bash
gpg --import --no-default-keyring --keyring ~/.ansible/pubring.kbx chave.asc
ansible-galaxy collection install ns.col --keyring ~/.ansible/pubring.kbx
```

## Namespace — Permissões Mínimas para Devs

```
Papel: Namespace Publisher → upload de coleções ao namespace
Papel: Namespace Editor    → editar metadados
```

## Limites de Tamanho

```
console.redhat.com / Ansible Galaxy:  máx 20MB por coleção
Private automation hub:               evitar >200MB (usar EE para ambientes completos)
```
