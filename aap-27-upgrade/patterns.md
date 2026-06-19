# Patterns — AAP 2.7 Upgrade

## Padrão 1: Upgrade Seguro 2.6 → 2.7 (Containerizado)

```
Semana -2: Preparação
  1. aap-detect-direct-component-access → identificar todos os acessos diretos
  2. Inventariar tokens, scripts, CaC
  3. Criar contas no Gateway para usuários que não têm
  4. Comunicar equipes sobre mudanças

Semana -1: Migração de Auth
  1. Reconfigurar LDAP/SAML no Gateway (em paralelo com Controller)
  2. Testar com usuários piloto
  3. Documentar novos endpoints e tokens

Dia do Upgrade:
  1. Backup: ansible-playbook backup.yml
  2. Upgrade: ansible-playbook install.yml (com novo installer 2.7)
  3. Validar: ping ao Gateway, testar cada componente
  4. Recriar tokens: comunicar usuários/CI-CD para buscar novos tokens

Semana +1: Limpeza
  1. Atualizar scripts/pipelines para gateway URLs
  2. Atualizar CaC: ansible.platform + gateway hostname
  3. Monitorar 401s no log do Gateway
```

## Padrão 2: Migração de CaC Monolítico para ansible.platform

```yaml
# Pré-2.7 (ansible.controller)
- ansible.controller.job_template:
    controller_host: "https://controller.empresa.org"
    controller_oauthtoken: "{{ controller_token }}"
    # ... demais campos idênticos

# Pós-2.7 (ansible.platform)
- ansible.platform.job_template:
    controller_host: "https://aap.empresa.org"     # Gateway
    controller_oauthtoken: "{{ gateway_token }}"    # Token do Gateway
    # ... demais campos idênticos (mesmos nomes de parâmetros)

# O switch é basicamente:
# 1. Mudar namespace: ansible.controller → ansible.platform
# 2. Mudar host: controller.empresa.org → aap.empresa.org (Gateway)
# 3. Usar token criado no Gateway (não no Controller)
```

## Padrão 3: Zero-Downtime Upgrade no OCP

```yaml
# Antes do upgrade, configurar no AutomationController CR:
spec:
  # Suporte a reconexão após upgrade de nó
  ee_extra_env: |
    - name: RECEPTOR_KUBE_SUPPORT_RECONNECT
      value: enabled
  # Dar tempo para jobs terminarem
  termination_grace_period_seconds: 3600
  # PDB protege jobs durante drain de nós
  # (criar PodDisruptionBudget separadamente)
```
