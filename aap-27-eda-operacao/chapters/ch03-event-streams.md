# Ch03 — Event Streams: Tipos e Configuração

## O que são Event Streams?

Event Streams são endpoints seguros e autenticados para receber eventos de sistemas externos. Permitem:
- Um único endpoint para múltiplos rulebooks
- Autenticação robusta (HMAC, mTLS, OAuth2)
- Modo de teste sem disparar automação
- Persistência de URL mesmo após recriação (UUID estático)

## Tipos de Event Stream

| Tipo | Autenticação | Exemplos de uso |
|---|---|---|
| **HMAC** | Shared secret + hash | GitHub webhooks |
| **Basic Authentication** | Username + password | Datadog, Dynatrace |
| **Token Authentication** | Token no header (Authorization ou custom) | GitLab (`X-Gitlab-Token`), ServiceNow |
| **OAuth2** | Client credentials (token opaco) | Dynatrace |
| **OAuth2 with JWT** | Client credentials (JWT) | Datadog |
| **ECDSA** | Par de chaves pública/privada | SendGrid, Twilio |
| **mTLS** | TLS mútuo (Certificate ou Subject) | PagerDuty |

**Tipos especializados pré-configurados:**
- **GitHub** → specialização de HMAC (campos pré-preenchidos)
- **GitLab** → specialização de Token
- **ServiceNow** → especialização de Token
- **Dynatrace** → especialização de Basic ou OAuth2

## Criar Credencial de Event Stream

```
Automation Decisions → Infrastructure → Credentials → Create credential

Campos:
  Name:            cred-github-webhook
  Organization:    Infraestrutura
  Credential type: GitHub Event Stream

Type Details (para GitHub/HMAC):
  HMAC Secret: <segredo-compartilhado-configurado-no-github>

→ Create credential

# Importante: após criar o event stream, a credencial NÃO pode ser deletada
# enquanto o event stream existir.
```

## Criar Event Stream

```
Automation Decisions → Event Streams → Create event stream

Campos:
  Name:             stream-github-push
  Organization:     Infraestrutura
  Event stream type: GitHub Event Stream  (ou outro tipo)
  Credentials:      cred-github-webhook   (exatamente 1 credencial)
  Headers:          X-GitHub-Event,X-GitHub-Delivery  (ou * para todos exceto sensíveis)
  ☑ Forward events to rulebook activation  (desmarcar para modo teste)

→ Create event stream

# A URL gerada é usada para configurar o webhook no sistema externo:
# https://aap.empresa.org/api/eda/v1/external_event_stream/{uuid}/
```

## Headers HTTP — Configuração Segura

```
# Incluir TODOS os headers (exceto sensíveis):
Headers: *
# Excluídos automaticamente: X-Envoy*, X-Trusted-Proxy*, X-Forwarded-For*, X-Real-Id*
# Redactado: Authorization header → "Authorization: Redacted"

# Incluir headers específicos:
Headers: Host,X-GitHub-Event,X-GitHub-Delivery,X-Hub-Signature-256

# Campo vazio = NENHUM header incluído no payload (não recomendado)
```

> Se a automação depende de headers específicos no payload → definir explicitamente.

## UUID Estático (Disaster Recovery)

```
# Para garantir que a URL do webhook não mude após recriar o event stream:
# (Usar apenas em cenários de DR onde o sistema externo não pode ser reconfigurado)

# Via API:
curl -X POST https://aap.empresa.org/api/eda/v1/event_streams/ \
  -H "Authorization: Bearer <token>" \
  -d '{"name": "stream-pagerduty", "uuid": "seu-uuid-estatico-aqui", ...}'

# ⚠️  Segurança: UUIDs estáticos tornam URLs previsíveis.
# Usar apenas com autenticação robusta (HMAC, mTLS) + restrições de rede.
```

## Modo Teste (Diagnosticar sem Disparar Automação)

```
# Desabilitar forwarding para testar a conexão sem triggar rulebooks:
Event Stream → Edit → ☐ Forward events to rulebook activation → Save

# Verificar se eventos chegam:
Event Streams → [stream] → Events received (contador atualizado em tempo real)
                          → Last event received (timestamp)

# Após validar: reabilitar ☑ Forward events to rulebook activation
```

## Associar Event Stream a uma Activation

```
# Na criação da Rulebook Activation ou ao editá-la:
Rulebook Activations → Create/Edit
  → Sources section: Replace source with event stream
  → selecionar o event stream

# ⚠️  Se o projeto foi atualizado enquanto o stream estava attached:
# Re-attach o event stream APÓS reiniciar a activation.
```

## mTLS + Load Balancer

```
# Se usando event stream mTLS atrás de load balancer:
⚠️  OBRIGATÓRIO: habilitar SSL pass-through (L4 routing) no load balancer.
O SSL termination para mTLS deve ocorrer no platform gateway proxy, não no LB.
```
