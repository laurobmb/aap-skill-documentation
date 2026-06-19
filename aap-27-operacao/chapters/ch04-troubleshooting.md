# Ch04 — Troubleshooting

## Coleta de Diagnóstico

### OpenShift — must-gather

```bash
# Coletar diagnóstico completo do cluster
oc adm must-gather \
  --image=registry.redhat.io/ansible-automation-platform-26/aap-must-gather-rhel9 \
  --dest-dir /tmp/aap-diagnostics

# Para namespace específico
oc adm must-gather \
  --image=registry.redhat.io/ansible-automation-platform-26/aap-must-gather-rhel9 \
  --dest-dir /tmp/aap-diagnostics \
  -- /usr/bin/ns-gather aap

# Compactar para attach ao support case
tar cvaf aap-must-gather.tar.gz /tmp/aap-diagnostics/
```

### Containerizado (RHEL) — sos report

```bash
# Via playbook do installer (coleta de todos os hosts)
cd /opt/ansible-automation-platform/installer
ansible-playbook -i inventory ansible.containerized_installer.log_gathering

# Com opções extras
ansible-playbook -i inventory ansible.containerized_installer.log_gathering \
  -e 'target_sos_directory=/tmp/reports' \
  -e 'case_number=0123456' \
  -e 'clean=true' \
  -e 'upload=true'

# Direto com plugin aap_containerized
sudo sos report -e aap_containerized \
  -k aap_containerized.username=<usuário_instalação>
```

## Containers do AAP (Containerizado) — Referência

| Container | Função |
|---|---|
| `automationcontroller-task` | Execução de playbooks, tasks |
| `automationcontroller-web` | API REST do Controller (via Gateway) |
| `automationcontroller-rsyslog` | Logging centralizado |
| `automation-eda-api` | API do EDA |
| `automation-eda-daphne` | WebSocket EDA |
| `automation-eda-worker-*` | Workers de activações EDA |
| `automationgateway` | Proxy, auth, roteamento |
| `automationhub-api` | API do Hub (Galaxy NG) |
| `automationhub-worker` | Tarefas assíncronas do Hub |
| `automationhub-content` | Serviço de conteúdo (downloads) |
| `postgresql` | Banco de dados |
| `redis` | Cache |

```bash
# Listar todos os containers
podman ps --all --format "{{.Names}}"

# Ver logs de um container
podman logs automationcontroller-task
podman logs automationcontroller-task --follow --tail 100

# Executar comando dentro do container
podman exec -it automationcontroller-task /bin/bash
```

## Problemas Comuns

### HTTP 401 após upgrade 2.6 → 2.7

```
Causa: token de componente (controller) sendo usado — removido no 2.7
Solução: criar novo PAT via Gateway
  Gateway → Access Management → Users → Tokens → Create Token
```

### Job preso em "pending"

```bash
# Verificar capacity do instance group
curl -H "Authorization: Bearer <token>" \
  https://aap.exemplo.org/api/controller/v2/instance_groups/<id>/

# Verificar se há capacity disponível
# "capacity": 0 → sem capacity → verificar execution nodes

# Ver fila de jobs
curl -H "Authorization: Bearer <token>" \
  https://aap.exemplo.org/api/controller/v2/jobs/?status=pending
```

### Playbook não aparece no Job Template

```
Causa: projeto não sincronizado ou branch/arquivo errado
Solução:
  1. Forçar atualização do projeto: Projects → Sync
  2. Verificar SCM Branch/Tag
  3. Verificar se o arquivo .yml está na raiz do repo ou no caminho configurado
```

### Pod OCP preso em Pending

```bash
# Ver eventos do pod
oc describe pod <pod-name> -n aap

# Causa comum: ResourceQuota ou sem nós disponíveis
oc get resourcequota -n aap
oc get nodes

# Ver logs do operator
oc logs deployment/aap-operator-controller-manager -n aap-operator
```

### Mesh/Receptor não conecta

```bash
# Verificar status das instâncias
curl -H "Authorization: Bearer <token>" \
  https://aap.exemplo.org/api/controller/v2/instances/

# Verificar porta 27199 entre control e execution nodes
nc -zv <execution_node> 27199

# Ver logs do receptor
podman logs automationcontroller-task 2>&1 | grep -i receptor
```

## Debug Mode no UI

```
Settings → Automation Execution → Troubleshooting
  - Enable tmp dir cleanup: false (para inspecionar /tmp após job)
  - Debug Web Requests: true (profiling de requests lentas)
  - Release Receptor Work: false (manter pod após job para debug)
  - Keep receptor work on error: true (manter work se erro)
```
