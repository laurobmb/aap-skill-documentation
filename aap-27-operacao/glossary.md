# Glossário — AAP 2.7 Operação

**mem_capacity** — Algoritmo de capacidade baseado em memória: `(RAM - 2GB) / 100MB_por_fork`.

**cpu_capacity** — Algoritmo de capacidade baseado em CPU: `CPUs × 4 forks_por_core`.

**fork** — Unidade de paralelismo do Ansible. Um fork representa uma conexão simultânea com um host gerenciado.

**Job Impact** — Impacto de um job na capacidade do nó: número de forks do job + 1 (processo pai).

**Instance Group** — Grupo de nós (control, execution, hop) que recebe jobs. Pode ter capacity policy.

**SYSTEM_TASK_FORKS_MEM** — Variável de configuração: MB de RAM por fork (padrão: 100).

**SYSTEM_TASK_FORKS_CPU** — Variável de configuração: forks por CPU core (padrão: 4).

**metrics-utility** — Ferramenta CLI para gerar relatórios de consumo e billing do AAP.

**must-gather** — Comando OCP (`oc adm must-gather`) para coletar diagnóstico completo do cluster.

**sos report** — Relatório de diagnóstico do RHEL (sosreport). O AAP tem plugin `aap_containerized`.

**log_gathering** — Playbook do installer AAP (`ansible.containerized_installer.log_gathering`) para coletar sos de todos os hosts.

**Automation Analytics** — Serviço SaaS da Red Hat em console.redhat.com para métricas de ROI e uso do AAP.

**Automation Dashboard** — Aplicação local (containerizada) para monitorar até 3 instâncias AAP sem depender de conectividade externa.

**AWX_CONTAINER_GROUP_POD_PENDING_TIMEOUT** — Tempo máximo que o Controller espera um pod do OCP entrar em Running antes de cancelar o job (padrão: 2 horas).

**PodDisruptionBudget** — Política OCP que protege pods durante operações de manutenção (upgrades, draining de nós).
