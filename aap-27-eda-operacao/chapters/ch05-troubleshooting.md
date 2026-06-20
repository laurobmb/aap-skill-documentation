# Ch05 — Troubleshooting EDA

## IDs de Rastreamento nos Logs

| ID | Abreviação | Propósito | Onde encontrar |
|---|---|---|---|
| `X-REQUEST-ID` | `rid` | Rastreia requisição HTTP do gateway até o EDA | Headers HTTP de resposta + logs EDA |
| `Log Tracking ID` | `tid` | Rastreia o ciclo de vida completo de uma activation (persiste em reinícios) | History tab da activation + todos os logs |
| `Activation Instance ID` | `aiid` | Identifica logs de uma instância específica de execução | Logs da activation |

**Formato nos logs:**
```
[rid: <UUID>] [tid: <UUID>] [aiid: <ID>] aap_eda.tasks.orchestrator Processing request...
```

## Filtrar Logs por ID de Rastreamento

```bash
# 1. Obter tid da UI:
Automation Decisions → Rulebook Activations → [activation] → History tab
→ selecionar instância → copiar Log Tracking ID

# 2. Buscar nos logs do sistema (containerizado):
grep "<tid>" /var/log/ansible-automation-platform/eda/*.log

# 3. Para sosreport/mustgather — coleta automática de todos os logs EDA:
# Containerizado: sos report (inclui logs de /var/log/ansible-automation-platform/eda/)
# OCP: must-gather (coleta pods do namespace aap)
```

## Activation Presa em Status PENDING

```
Sintomas: Activation criada mas não sai de "Pending"

Diagnóstico:
1. Verificar se há outras activations rodando e se os limites (CPU/memória) foram atingidos
   → Se sim: terminar uma ou mais activations em execução

2. Verificar se os workers EDA, Redis e activation worker estão rodando:
   # Containerizado:
   podman ps | grep -E "eda|redis"

   # OCP:
   oc get pods -n aap | grep eda

3. Verificar logs internos:
   # Containers a verificar: worker, scheduler, API, nginx
   podman logs <eda-worker-container>
   oc logs -n aap <eda-worker-pod>

4. Se os logs não mostram a causa → abrir chamado no Red Hat Support
```

## Activation em Restart Loop

```
Sintomas: Activation reinicia constantemente (status oscila entre Running/Failed)

Diagnóstico passo a passo:
1. Ir para Automation Decisions → Rulebook Activations → [activation] → History tab
2. Selecionar uma instância que falhou → ler o Output

3. Verificar Restart policy:
   - "On failure" = confirma que há um erro causando falha
   - Examinar o YAML do rulebook para erros de sintaxe/indentação

4. Mudar Log level para Debug:
   Desabilitar activation → Edit → Log level: Debug → habilitar → verificar Output

5. Causas comuns:
   - Erro de sintaxe no rulebook
   - Source plugin não disponível na DE usada
   - Credencial incorreta ou expirada
   - Conexão com o sistema externo falha (Kafka, DB, API)
```

## Activation Rodando mas Não Dispara Ações

```
Sintomas: Activation em status Running, recebe eventos mas nenhuma ação ocorre

Diagnóstico:
1. Verificar as condições (when) do rulebook:
   - As condições batem exatamente com o payload recebido?
   - Verificar estrutura do payload no Rule Audit → Events tab

2. Verificar sintaxe e indentação do YAML do rulebook
   - Um erro de indentação impede a avaliação das condições

3. Verificar a ação configurada:
   - run_job_template: nome e organização corretos?
   - O Job Template existe e está acessível pela credencial AAP?

4. Verificar mapeamento de event stream:
   - Se usando event streams, confirmar que o stream correto está associado à activation
   - Mismatch de source → activation recebe dados mas não processa
```

## Event Stream não Envia Eventos

```
Sintomas: Event stream configurado mas eventos não chegam à activation

Diagnóstico:
1. Verificar se o event stream NÃO está em modo Teste:
   Event Streams → [stream] → ☑ Forward events to rulebook activation?

2. Verificar o sistema de origem (GitHub, GLPI, etc.):
   - Está enviando para a URL correta?
   - URL: https://<gateway>/api/eda/v1/external_event_stream/{uuid}/

3. Verificar conectividade de rede:
   - A rede permite conexão ao platform gateway?
   - O proxy do platform gateway está rodando?

4. Verificar credencial do event stream:
   - Secret/token correto no sistema de origem?
   - Credencial foi alterada no EDA mas não atualizada no sistema externo?

5. Verificar autenticação:
   - HMAC: X-Hub-Signature header presente e correto?
   - Token: header correto (Authorization ou X-Gitlab-Token)?
   - mTLS: certificado do cliente válido e CA presente no servidor?

6. Verificar se o rulebook reage aos eventos:
   - A source definida no rulebook é compatível com o tipo do event stream?
```

## Problemas com mTLS + Load Balancer

```
Sintoma: Conexões mTLS falham passando pelo load balancer

Solução:
⚠️  Habilitar SSL pass-through (L4 routing) no load balancer.
O SSL termination e validação do certificado cliente DEVE ocorrer no platform gateway proxy.
NÃO terminar SSL no load balancer para event streams mTLS.
```
