# Ch04 — Portas e Protocolos de Rede

## Portas Principais (Container e Operator)

| Porta | Proto | Serviço | Origem → Destino | Descrição |
|---|---|---|---|---|
| 80/443 | TCP | HTTP/HTTPS | EDA → Hub | Pull de Decision Environments |
| 80/443 | TCP | HTTP/HTTPS | Controller → Hub | Pull de Execution Environments |
| 80/443 | TCP | HTTP/HTTPS | Usuário/browser → Gateway | Acesso externo à plataforma |
| 5432 | TCP | PostgreSQL | Todos os componentes → DB | Acesso ao banco de dados |
| 6379 | TCP | Redis | Controller → Redis | Jobs e data storage do Controller |
| 6379 | TCP | Redis | EDA → Redis | Jobs e data storage do EDA |
| 6379 | TCP | Redis | Gateway → Redis | Data storage do Gateway |
| 16379 | TCP | Redis | Redis → Redis | Cluster bus (Redis HA) |
| 27199 | TCP | Receptor | Controller → Hop/Exec node | Mesh: distribuição de jobs |
| 27199 | TCP | Receptor | Hop node → Exec node | Mesh: comunicação bidirecional |
| 8080/8443 | TCP | HTTP/HTTPS | Gateway → Controller | NGINX interno do Controller |
| 8081/8444 | TCP | HTTP/HTTPS | Gateway → Hub | NGINX interno do Hub |
| 8082/8445 | TCP | HTTP/HTTPS | Gateway → EDA | NGINX interno do EDA |
| 8083/8446 | TCP | HTTP/HTTPS | Gateway → Gateway | NGINX interno do Gateway |

## Customizar Portas via Inventory Variables

```ini
# Portas NGINX dos componentes (configuráveis)
controller_nginx_http_port=8080
controller_nginx_https_port=8443
hub_nginx_http_port=8081
hub_nginx_https_port=8444
eda_nginx_http_port=8082
eda_nginx_https_port=8445
gateway_nginx_http_port=8083
gateway_nginx_https_port=8446
```

> Ao mudar portas, verificar conflitos consultando a referência completa de variáveis de inventário.

## Arquitetura de Rede: Acesso Externo via Platform Gateway

**Todo o tráfego externo passa pelo Platform Gateway** — acesso direto ao controller, hub ou EDA está bloqueado no AAP 2.7:

```
Usuário/CI/API externa
        ↓
  Platform Gateway (80/443)
        ├─→ Automation Controller  (8080/8443 interno)
        ├─→ Private Automation Hub (8081/8444 interno)
        └─→ Event-Driven Ansible   (8082/8445 interno)
```

> RPM-based (deprecated 2.6): acesso direto era possível. No 2.7, somente via Gateway.

## Automation Mesh — Portas

| Porta | Direção | Uso |
|---|---|---|
| 27199/TCP | Controller → Hop/Exec | Mesh direto (sem hop node) |
| 27199/TCP | Hop → Exec | Mesh via hop node |

Ambas as direções são usadas (comunicação bidirecional via Receptor).

## Firewalls — Regras Essenciais

```bash
# Exemplo com firewalld (RHEL 9/10)

# Gateway (acesso externo)
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp

# PostgreSQL (se externo)
firewall-cmd --permanent --add-port=5432/tcp

# Redis (entre componentes)
firewall-cmd --permanent --add-port=6379/tcp
firewall-cmd --permanent --add-port=16379/tcp  # Redis HA cluster bus

# Automation Mesh
firewall-cmd --permanent --add-port=27199/tcp

firewall-cmd --reload
```
