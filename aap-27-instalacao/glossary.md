# Glossário — AAP 2.7 Instalação

**Container topology** — Modelo de instalação do AAP em VMs/bare metal RHEL usando Podman. Substituiu o RPM-based no 2.7.

**Operator topology** — Modelo de instalação do AAP em OpenShift Container Platform usando Operators.

**Growth topology** — Topologia de menor footprint, sem redundância, para início ou POC. Container Growth usa 1 VM; Operator Growth usa SNO.

**Enterprise topology** — Topologia com redundância e alta disponibilidade para produção. Container Enterprise usa 11+ VMs.

**Platform Gateway** — Componente central de proxy e autenticação. Todo tráfego externo passa por ele no AAP 2.7.

**Redis standalone** — Modo Redis sem HA, adequado para growth topology.

**Redis sentinel** — Modo Redis com HA, requer 6 VMs adicionais para o cluster Redis.

**hub_seed_collections** — Variável que habilita o seed automático de coleções certificadas Red Hat no Hub. Requer 32 GB RAM e 45+ minutos.

**ICU (International Components for Unicode)** — Extensão obrigatória para PostgreSQL externo (15, 16 ou 17) no AAP.

**Receptor** — Tecnologia de comunicação bidirecional usada pelo Automation Mesh na porta 27199/TCP.

**SNO (Single Node OpenShift)** — Cluster OCP com um único nó, suportado para Operator Growth topology.

**registry.redhat.io** — Registry de containers Red Hat. Requer credenciais RHN ou service account para pull de imagens AAP.
