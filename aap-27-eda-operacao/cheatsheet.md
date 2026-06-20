# Cheatsheet — AAP 2.7 EDA Operação

## Criar Activation (campos obrigatórios)

```
Name + Organization + Project + Rulebook + Decision Environment
```

## Restart Policy

```
Always     → reinicia sempre (máx 5x)
On failure → reinicia após 60s se falhou (máx 5x)
Never      → nunca reinicia
```

## Ciclo de Vida da Activation

```
Criar → Pending → Running
         ↓
       Failed → (restart se política permitir)
         ↓
       Stopped (desabilitado manualmente)
```

## Tipos de Event Stream

```
HMAC          → GitHub, secrets compartilhados
Basic Auth    → Datadog, Dynatrace
Token         → GitLab, ServiceNow
OAuth2        → Dynatrace (token opaco)
OAuth2 JWT    → Datadog (JWT)
ECDSA         → SendGrid, Twilio
mTLS          → PagerDuty (+ SSL pass-through no LB!)
```

## URL do Event Stream

```
https://<gateway>/api/eda/v1/external_event_stream/{uuid}/
```

## IDs de Log para Troubleshooting

```
rid  → rastrear requisição HTTP (header X-REQUEST-ID)
tid  → rastrear lifecycle da activation (History tab)
aiid → identificar instância específica de execução
```

## Diagnóstico Rápido

```
Pending?     → verificar CPU/memória/Redis
Restart loop? → Log level=Debug → ver Output → verificar rulebook YAML
Sem ações?   → Rule Audit → Events → verificar estrutura do payload vs condições
Sem eventos? → verificar Test mode / URL / credencial / rede / proxy
```

## Redis Required Para

```
Criar activation | Deletar activation | Habilitar | Desabilitar | Reiniciar
Criar projeto    | Sync de projeto
```
