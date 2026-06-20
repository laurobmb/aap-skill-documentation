# Glossário — AAP 2.7 EDA Operação

| Termo | Definição |
|---|---|
| **Rulebook Activation** | Processo em background que executa um rulebook específico em uma Decision Environment, monitorando eventos continuamente |
| **Event Stream** | Endpoint seguro e autenticado exposto pelo platform gateway para receber eventos de sistemas externos |
| **Decision Environment (DE)** | Container image que executa ansible-rulebook. Similar às EEs para o Controller |
| **Rule Audit** | Log auditável de cada regra que foi disparada, incluindo o evento e a ação resultante |
| **Restart Policy** | Política que define se e quando uma activation reinicia após terminar |
| **Log Level** | Nível de detalhe dos logs: Error / Info / Debug |
| **Forward events** | Configuração do Event Stream que controla se eventos são encaminhados para activations (pode ser desabilitado para modo teste) |
| **HMAC** | Hash-based Message Authentication Code — método de verificação de integridade usando shared secret |
| **mTLS** | Mutual TLS — autenticação bidirecional onde tanto servidor quanto cliente apresentam certificados |
| **ECDSA** | Elliptic Curve Digital Signature Algorithm — assinatura com par de chaves pública/privada |
| **UUID estático** | Configuração de Event Stream para manter a URL do webhook consistente após recriação (para DR) |
| **tid (Log Tracking ID)** | Identificador que acompanha o ciclo de vida completo de uma activation, persiste entre reinícios |
| **rid (Request ID)** | Identificador de requisição HTTP, rastreável do gateway até o processamento no EDA |
| **aiid (Activation Instance ID)** | Identificador de uma instância específica de execução de uma activation |
| **Test mode** | Modo do Event Stream que recebe eventos mas NÃO os encaminha para activations (para debug) |
| **de-minimal** | Imagem base de Decision Environment com plugins EDA essenciais |
| **de-supported** | Imagem de DE com plugins EDA adicionais incluindo cloud sources (AWS, GCP, Azure) |
| **Source plugin** | Plugin Ansible que define de onde os eventos são coletados (webhook, kafka, Kafka, pg_listener, etc.) |
| **Rule Audit — Events** | Payload raw do evento que disparou a regra, com timestamp e source type |
| **SSL pass-through** | Configuração de load balancer onde o TLS não é terminado no LB, repassando para o backend (necessário para mTLS) |
