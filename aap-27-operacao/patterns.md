# Patterns — AAP 2.7 Operação

## Padrão 1: Planejar Capacity para Alta Carga

```
Exemplo: 1000 hosts, 50 jobs simultâneos, playbooks simples

Calcular capacidade por nó execution (16 GB RAM, 8 CPUs):
  mem: (16384-2048)/100 = 143 forks
  cpu: 8*4 = 32 forks ← limitante

Para 50 jobs × forks=5 = 250 forks necessários
  → 250/32 = ~8 nós de execução

Controller nodes: 1 fork de controle por job
  → 50 jobs → 50 forks control
  → 1 nó controller = 32 control forks
  → 2 nós controller para folga
```

## Padrão 2: Observabilidade Completa

```
Camada 1 — Métricas de jobs (gratuito):
  Automation Analytics → console.redhat.com
  Job Explorer, ROI Calculator

Camada 2 — Dashboard local (sem internet):
  Automation Dashboard → 3 instâncias AAP
  Métricas de utilização, jobs, sucesso/falha

Camada 3 — Infraestrutura (Prometheus/Grafana):
  OCP: ServiceMonitor → coleta /metrics do AAP
  RHEL: node_exporter + Prometheus externo

Camada 4 — Billing:
  metrics-utility → relatórios mensais
  Enviar ao time de gestão/financeiro
```

## Padrão 3: Resposta a Incidente de Performance

```
Sintoma: Jobs lentos, fila crescendo

1. Verificar capacity:
   curl .../api/controller/v2/instances/ | jq '.[].capacity'
   → capacity = 0? → execution nodes sem recursos

2. Verificar jobs em pending:
   curl .../api/controller/v2/jobs/?status=pending | jq '.count'

3. Identificar bottleneck:
   - CPU alto? → scale horizontalmente (mais exec nodes)
   - Memory? → scale verticalmente ou adicionar exec nodes
   - Network? → verificar Mesh/Receptor (porta 27199)
   - Database? → verificar PostgreSQL connections

4. Ação de curto prazo:
   - Adicionar execution node ao instance group
   - Reduzir max_forks temporariamente
   - Cancelar jobs presos

5. Coleta para RCA:
   ansible-playbook log_gathering -e case_number=12345
```
