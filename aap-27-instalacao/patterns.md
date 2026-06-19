# Patterns — AAP 2.7 Instalação

## Padrão 1: Instalação Growth em VM Única (POC)

```
Situação: avaliação rápida, POC, ambiente de desenvolvimento
Solução: Container Growth — 1 VM, todos componentes colocados

Passos:
1. Provisionar RHEL 9.4+ com 16 GB RAM, 4 CPUs, 60 GB disco
2. Criar inventory com ansible_connection=local
3. Preencher senhas de admin e banco
4. Executar: ansible-playbook -i inventory install.yml
```

## Padrão 2: Instalação Enterprise com PostgreSQL Externo

```
Situação: produção com HA, múltiplas equipes, alta carga
Solução: Container Enterprise — 11 VMs + PostgreSQL externo

Decisão de banco:
- Usar PostgreSQL gerenciado apenas se 1 réplica e < 100 jobs concorrentes
- Para produção real: sempre PostgreSQL externo (mais controle de backup/restore)

Passos:
1. Provisionar 11 VMs (16 GB RAM cada)
2. Provisionar PostgreSQL 15 externo com ICU habilitado
3. Criar inventory com hosts separados por grupo
4. Definir <componente>_pg_host=db.empresa.org para todos
5. Executar installer
```

## Padrão 3: Múltiplos Ambientes no Mesmo Cluster OCP

```
Situação: empresa com ambientes dev, stage e prod no mesmo OCP
Solução: namespace-scoped deployment

oc new-project aap-dev
oc new-project aap-stage
oc new-project aap-prod

→ Instalar 1 Operator por namespace
→ Cada namespace é uma instância completamente isolada
→ PostgreSQL externo diferente para cada ambiente
```

## Padrão 4: Migração do RPM para Containerizado

```
Situação: cliente ainda no AAP 2.6 RPM, precisa ir para 2.7
Solução: RPM foi removido no 2.7 — migração obrigatória

Opções suportadas:
- RPM 2.6 → Container 2.7
- RPM 2.6 → Operator 2.7

Processo geral (ver skill aap-27-upgrade para detalhes):
1. Instalar novo ambiente Container/Operator 2.7
2. Exportar dados do RPM 2.6 (migration artifact)
3. Importar no novo ambiente
4. Validar e redirecionar tráfego
```
