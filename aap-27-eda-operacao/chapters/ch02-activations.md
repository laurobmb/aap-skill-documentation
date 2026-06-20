# Ch02 — Rulebook Activations: Ciclo de Vida

## Criar Rulebook Activation

```
Automation Decisions → Rulebook Activations → Create rulebook activation

Campos Obrigatórios:
  Name:                 activation-glpi-webhook
  Organization:         Infraestrutura
  Project:              glpi-automacoes
  Rulebook:             rulebooks/glpi-vm-creation.yml  (lista do projeto selecionado)
  Decision Environment: de-supported-rhel9

Campos Opcionais:
  Credentials:    (Vault, AAP platform, custom) — máx 1 credencial do tipo AAP
  Restart policy: Always | Never | On failure
  Log level:      Error | Info | Debug
  Service name:   (nome K8s para inbound connections se o activation expõe porta)
  Variables:      (JSON/YAML equivalente a --vars do ansible-rulebook)
  ☐ Skip audit events  (não registrar no Rule Audit)
  ☑ Rulebook activation enabled  (habilitar imediatamente ao criar)

→ Create rulebook activation
```

## Restart Policy

| Política | Comportamento |
|---|---|
| **Always** | Reinicia imediatamente ao terminar (sucesso ou falha). Máx 5 reinícios |
| **Never** | Nunca reinicia quando o container termina |
| **On failure** | Reinicia após 60s se o container falhou. Máx 5 reinícios |

## Log Level

| Nível | O que registra |
|---|---|
| **Error** | Apenas mensagens de erro |
| **Info** | Sucesso/falha, ações disparadas, nomes de actions e erros |
| **Debug** | Tudo do Info + dados de debug (produção = verboso demais) |

## Habilitar / Desabilitar

```
# Via UI:
Automation Decisions → Rulebook Activations
  → toggle na linha da activation → confirmar

# Impacto:
Desabilitar: para o processamento de eventos imediatamente
Habilitar:   reinicia o processamento
```

> **Operações que requerem Redis disponível:** criar, deletar, habilitar, desabilitar, reiniciar activations.

## Reiniciar Activation

```
# Requisito: activation deve estar habilitada E restart policy = Always

Automation Decisions → Rulebook Activations
  → More Actions (⋮) → Restart rulebook activation → confirmar

# Quando reiniciar:
  - Após atualizar o projeto (novo conteúdo do rulebook)
  - Após re-attach de event stream
  - Para recuperação de falha
```

## Editar Activation

```
# OBRIGATÓRIO: desabilitar a activation antes de editar

1. Toggle activation → OFF → confirmar desabilitação
2. Clicar ícone Edit na activation
3. Editar os campos desejados
4. Toggle "Rulebook activation enabled" → ON (para já habilitar ao salvar)
5. Save rulebook activation
```

## Duplicar Activation

```
# Útil para criar activation similar com configurações parecidas

More Actions (⋮) → Duplicate rulebook activation

# Activation duplicada:
  - Criada como DESABILITADA
  - Nome: "<nome_original> @ HH:MM:SS"
  - ⚠️  Editar o nome antes de habilitar para evitar duplicação de jobs

# A activation original continua rodando normalmente após duplicação.
```

## Deletar Activation

```
More Actions (⋮) → Delete rulebook activation → confirmar

# Nota: não é possível deletar um projeto se ainda há activations usando-o.
```

## Status das Activations

| Status | Significado |
|---|---|
| **Running** | Rodando em background, processando eventos |
| **Pending** | Aguardando recursos (ver troubleshooting ch05) |
| **Failed** | Falhou — verificar History tab |
| **Stopped** | Desabilitada ou terminada sem reinício |
| **Completed** | Terminou normalmente (restart policy=Never) |

## Ver Output de uma Activation

```
Automation Decisions → Rulebook Activations → [activation]
  → History tab → selecionar instância de execução
  → Output (logs de execução)

# Para eventos que dispararam ações:
Automation Decisions → Rule Audit
```
