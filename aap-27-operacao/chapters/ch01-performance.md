# Ch01 — Performance e Scaling

## Tipos de Scaling

| Tipo | Quando usar | Limitação |
|---|---|---|
| **Vertical** | Bottleneck em recurso único, fácil de implementar | Limite físico da VM, não elimina SPOF |
| **Horizontal** | Alta disponibilidade, crescimento de carga | Mais complexo, requer cuidado com Redis/Postgres |

## Capacidade de Jobs — Algoritmo

O Controller usa a menor entre duas medidas:

### Capacidade por Memória (mem_capacity)

```
forks = (RAM_MB - 2048) / 100

Exemplo: 16 GB RAM
  (16384 - 2048) / 100 = ~143 forks
```

- 2 GB reservados para serviços internos
- ~100 MB por fork (padrão `SYSTEM_TASK_FORKS_MEM`)

### Capacidade por CPU (cpu_capacity)

```
forks = CPUs * 4

Exemplo: 8 CPUs
  8 * 4 = 32 forks
```

- 4 forks por core (padrão `SYSTEM_TASK_FORKS_CPU`)

> O Controller usa **a menor** entre mem_capacity e cpu_capacity.

### Impacto por Job

```
forks reais usados = forks_do_job + 1  (processo pai Ansible)
Exemplo: job com forks=5 contra 5 hosts = impacto de 6 na capacidade
```

Forks padrão do Ansible: **5**. Para grandes inventários, ajustar no Job Template.

## Scaling Horizontal do EDA (AAP 2.7)

```ini
# inventory file — múltiplos nós EDA
[automationeda]
eda1.exemplo.org
eda2.exemplo.org

[all:vars]
eda_max_running_activations=12   # por nó (padrão: 12)
# Total capacity = 2 nós × 12 = 24 activations simultâneas
```

### Sizing do EDA por Workload

| Workload | Recomendação |
|---|---|
| < 100 activations | 1 nó EDA (Growth topology) |
| 100–500 activations | 2 nós EDA dedicados |
| > 500 activations | 3+ nós + load balancer |

## Tuning do Controller

### Forks por tipo de workload

| Workload | Ajuste recomendado |
|---|---|
| Muitos hosts, tarefas simples | Aumentar forks (mais paralelismo) |
| Tarefas intensivas em CPU | Reduzir forks (menos por core) |
| Playbooks com gather_facts | Reduzir forks (gather_facts = alto impacto) |

```ini
# No inventory ou Job Template
# Sobrescrever padrão de capacidade por memória
controller_extra_settings:
  - setting: SYSTEM_TASK_FORKS_MEM
    value: "100"          # MB por fork (padrão: 100)
  - setting: SYSTEM_TASK_FORKS_CPU
    value: "4"            # forks por core (padrão: 4)
```

### controller_percent_memory_capacity

```ini
# Reservar fração da memória para o controller (resto para forks)
controller_percent_memory_capacity=0.5   # 50% para jobs
```

## Escalar para Enterprise Topology

Quando a Growth topology não é suficiente:
1. CPU > 80% sustained → adicionar execution nodes
2. Latência de API > 500ms → adicionar control/gateway nodes
3. Memória > 85% → vertical scale ou adicionar nós
4. Redis saturado → Redis HA (sentinel mode)

```
Growth topology        → Enterprise topology
1 VM tudo junto        → VMs separadas por função
                          gateway(2) + controller(2) + hub(2) + eda(2) + exec(2+)
```

## KPIs para Monitorar

| Indicador | Alerta | Ação |
|---|---|---|
| CPU usage | > 80% sustained | Escalar horizontalmente |
| API latency | > 500ms | Adicionar nós de gateway ou controller |
| Job queue depth | > 50 jobs pending | Adicionar execution nodes |
| EDA activation failures | > 5% | Verificar DE + adicionar nós EDA |
| Redis memory | > 80% | Aumentar Redis ou configurar HA |
