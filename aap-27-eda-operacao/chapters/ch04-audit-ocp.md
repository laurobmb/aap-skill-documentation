# Ch04 — Rule Audit e Webhooks em OCP

## Rule Audit

Rule Audit registra cada vez que um evento correspondeu a uma condição de uma regra e disparou uma ação.

```
Automation Decisions → Rule Audit

Listagem:
  Cada linha = uma execução de regra
  Heading = nome da regra que foi executada
```

## Ver Detalhes do Audit

```
Rule Audit → [regra selecionada]

Aba Details:
  - Quando foi criada
  - Quando foi disparada pela última vez
  - Rulebook Activation correspondente

Aba Events:
  - Payload do evento que disparou a ação
  - Source type (webhook, kafka, etc.)
  - Timestamp do evento

Aba Actions:
  - Ações executadas por essa regra
  - Link para Automation Execution (Controller) para ver o output do job
```

## Acesso à API do EDA

```
# Documentação disponível via platform gateway:
https://<gateway-host>/api/eda/v1/docs

# Nota sobre organizações/teams:
# Mudanças via API EDA levam até 15 minutos para propagar ao resto da plataforma.
# Para evitar erros, usar a API do platform gateway ou a UI unificada.
```

## Expor Webhook EDA em OCP (Route)

```yaml
# Rulebook com webhook source:
- name: Ouvir eventos do GLPI
  hosts: all
  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5000
  rules:
    - name: Processar abertura de chamado
      condition: event.payload.status == "new"
      action:
        run_job_template:
          name: "Criar VM vCenter"
          organization: "Infraestrutura"
```

```yaml
# Route OCP para expor a porta 5000 da activation:
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: eda-glpi-webhook
  namespace: aap
  labels:
    app: eda
    job-name: activation-job-<id>-5000
spec:
  host: eda-glpi.apps.ocp4.empresa.org
  to:
    kind: Service
    name: activation-job-<id>-5000
    weight: 100
  port:
    targetPort: 5000
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
```

```bash
# Aplicar a Route:
oc apply -f eda-glpi-route.yaml

# Testar enviando evento:
curl -H "Content-Type: application/json" \
     -X POST \
     https://eda-glpi.apps.ocp4.empresa.org \
     -d '{"status": "new", "id_chamado": 12345}'
```

## Testar Webhook Localmente (kubectl port-forward)

```bash
# Expor porta da activation localmente para testes:
kubectl port-forward svc/activation-job-<id>-5000 5000:5000

# Testar:
curl -H "Content-Type: application/json" \
     -X POST http://localhost:5000 \
     -d '{"evento": "teste"}'
```

## Sincronizar Conteúdo EDA com Redis

```
⚠️  Redis compartilhado com o Platform Gateway UI
   Sem Redis:
     - Não é possível criar/sync projetos
     - Não é possível criar/habilitar/desabilitar/reiniciar activations

# Verificar status do Redis:
podman ps | grep redis      # containerizado
oc get pods -n aap | grep redis  # OCP
```
