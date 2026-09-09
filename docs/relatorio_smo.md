# Relatório Técnico: Avaliação de Plataformas SMO para Redes Open RAN

**Disciplina:** Arquitetura de Redes de Computadores / Open RAN  
**Objeto de Estudo:** OSC SMO (O-RAN Software Community) / ONAP Framework  
**Data:** Setembro/2026  

---

## 1. Arquitetura da Plataforma

O Service Management Orchestrator (SMO) na especificação da O-RAN Alliance é o núcleo responsável pelo gerenciamento, orquestração e automação do ciclo de vida das funções de rede RAN. No projeto de referência O-RAN Software Community (OSC), a arquitetura do SMO é modular e baseada em microsserviços integrados sobre Kubernetes, aproveitando módulos do ONAP (Open Network Automation Platform) e da O-RAN Alliance:

* **Non-Real-Time RIC (Non-RT RIC):** Módulo interno ao SMO responsável por políticas de gerenciamento de malha aberta (tempo de resposta > 1s) e pela execução de rApps.
* **ONAP SDNC (SDN Controller):** Componente baseado em OpenDaylight responsável pela terminação da interface O1 (NETCONF Client/MD-SAL) para configuração e gerenciamento de elementos.
* **VES Collector & DMaaP (Kafka):** Barramento de dados para recepção assíncrona de telemetria, eventos e alarmes provenientes dos nós da RAN.
* **O2 Service Manager:** Módulo encarregado da abstração de infraestrutura e orquestração de recursos em nuvem (O-Cloud).

---

## 2. Serviços e Funções Disponibilizados pelo SMO

O SMO fornece uma suíte de serviços categorizada segundo os requisitos operacionais da telecomunicação moderna:

* **Gerenciamento FCAPS:**
  * **Fault Management (FM):** Coleta e correlação de falhas via eventos VES (Virtual Event Streaming).
  * **Configuration Management (CM):** Provisionamento de parâmetros operacionais via NETCONF/YANG sobre SSH/TLS.
  * **Accounting & Performance Management (PM):** Ingestão de métricas e suporte a bilhetagem via InfluxDB/Grafana e pipelines Kafka.
  * **Security Management:** Autenticação centralizada e autorização baseada em papéis via Keycloak e Traefik Gateway.
* **Orquestração Inteligente (A1 Policy Management):** Definição e publicação de políticas de RRM (Radio Resource Management) repassadas ao Near-RT RIC.
* **Ciclo de Vida de Software (LCM):** Onboarding, deploy e atualização de funções de rede virtualizadas/containerizadas (VNF/CNF) e de rApps.

---

## 3. Componentes O-RAN Suportados

A arquitetura do OSC SMO estende suporte direto aos seguintes elementos desagregados da O-RAN:

* **O-CU-CP (Central Unit - Control Plane)**
* **O-CU-UP (Central Unit - User Plane)**
* **O-DU (Distributed Unit)**
* **Near-RT RIC (Near-Real-Time RAN Intelligent Controller)**
* **O-Cloud (Infraestrutura de Nuvem e Borda)**

---

## 4. Implementação das Interfaces O1 e O2

### Interface O1 (Gerenciamento de Elementos)
A interface O1 estabelece a conexão entre o SMO e as NFs (O-CU, O-DU, Near-RT RIC):
* **Plano de Configuração (CM):** Utiliza o protocolo **NETCONF** com suporte aos modelos de dados definidos em **YANG**. O SDNC atua como cliente NETCONF e as NFs atuam como servidores NETCONF.
* **Plano de Telemetria e Alertas (FM/PM):** Utiliza requisições **HTTP REST/JSON** formatadas segundo o padrão **VES (Virtual Event Streaming)**. Os eventos são transmitidos das NFs para o VES Collector do SMO.

### Interface O2 (Gerenciamento de Infraestrutura)
A interface O2 desacopla as funções de rede do hardware subjacente, ligando o SMO à O-Cloud:
* **O2ims (Infrastructure Management Services):** Prover e monitorar recursos físicos e virtualizados (hosts, pontes de rede, pools de CPU e aceleração) através de APIs RESTful.
* **O2dms (Deployment Management Services):** Gerenciar o ciclo de vida da infraestrutura da aplicação (por exemplo, orquestração via manifestos do Kubernetes / Helm Charts).

---

## 5. Mecanismos de Gerenciamento, Provisionamento e Orquestração

* **Declarativo via Modelos YANG:** As requisições de alteração de estado no SMO são validadas contra esquemas YANG antes de serem convertidas em operações `edit-config` na camada NETCONF.
* **Controle de Estado Interno:** O SDNC mantém datastores MD-SAL distintos:
  * **Config Datastore:** Armazena o estado desejado da rede.
  * **Operational Datastore:** Armazena o estado real informado pelas funções de rede.
  * *Mecanismos de Sincronização:* Auditorias periódicas executam comandos `get-config` para detectar desvios de configuração (*drift*).

---

## 6. Capacidades de Gerenciamento do O-Cloud

O SMO controla os recursos computacionais da nuvem por meio do adaptador O2:
* **Inventário Dinâmico:** Descoberta automática de nós de computação de borda (*Edge Nodes*) e recursos de hardware especializados (como aceleradores FH/FPGA/GPU).
* **Gerenciamento de Multi-Cluster:** Capacidade de provisionar workloads em múltiplos clusters Kubernetes distribuídos e isolados geograficamente.
* **Alocação Inteligente de Recursos:** Ajuste dinâmico de quotas de CPU, memória e isolamento SR-IOV para garantir exigências de latência da O-DU.

---

## 7. Suporte ao Ciclo de Vida de Network Functions (LCM)

O gerenciamento do ciclo de vida das NFs segue os padrões ETSI NFV MANO integrados ao ONAP:
1. **Onboarding:** Validação e registro de pacotes Helm/CSAR contendo a definição da CNF/VNF.
2. **Instanciação:** Alocação de infraestrutura via interface O2dms e *deploy* dos containers da NF na O-Cloud.
3. **Configuração Inicial (Day-0 / Day-1):** Injeção de credenciais, rotas de rede e identidades via NETCONF/O1.
4. **Reconfiguração Operacional (Day-2):** Alteração de parâmetros de rádio (ex: largura de banda de canal) em tempo de execução.
5. **Terminação:** Remoção segura de workloads e liberação de recursos na O-Cloud.

---

## 8. Monitoramento, Telemetria e Gerenciamento de Falhas

* **Pipeline de Ingestão de Dados:**
  * O evento VES chega via protocolo HTTP/JSON ao **VES Collector**.
  * O coletor publica a mensagem bruta no tópico correspondente do **Kafka (DMaaP)**.
* **Processamento de Alarmes e Análise:**
  * Os dados do Kafka são consumidos pelo **PM Mapper** e injetados na base de dados de séries temporais **InfluxDB**.
  * Painéis do **Grafana** fornecem visualização em tempo real de métricas de KPI (e.g., *PRB Usage*, *PRB Drop Rate*).
* **Ação Corretiva em Malha Fechada (Closed-Loop Automation):**
  * Quando um alarme crítico é detectado (ex: alta temperatura ou falha de link), regras de política ativam o rApp correspondente no Non-RT RIC para disparar uma ação de reconfiguração via NETCONF.

---

## Referências
* O-RAN Alliance. **O-RAN Architecture Description** (O-RAN.WG1.O-RAN-Architecture).
* O-RAN Software Community (OSC). **OAM & SMO Architecture Documentation**.
* ONAP Documentation. **SDN Controller for Radio (SDN-R)**.