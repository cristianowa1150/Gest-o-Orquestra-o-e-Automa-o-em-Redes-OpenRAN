# Relatório Técnico: Avaliação da Plataforma Service Management Orchestrator (SMO) em Ambientes Open RAN

**Disciplina:** Gestão, Orquestração e Automação em Redes Open RAN  
**Objeto de Estudo:** OSC SMO Framework (O-RAN Software Community / ONAP SDNC & Non-RT RIC)  
**Data:** Setembro de 2026  

---

## 1. Introdução e Arquitetura da Plataforma

O Service Management Orchestrator (SMO) é o elemento central definido pela **O-RAN Alliance** (Working Group 1) responsável pelo gerenciamento, orquestração e automação do ciclo de vida dos elementos de rede de acesso de rádio aberta e desagregada (Open RAN). 

Na implementação de referência mantida pela **O-RAN Software Community (OSC)** em conjunto com a iniciativa **ONAP (Open Network Automation Platform)**, o SMO adota uma arquitetura distribuída, orientada a microsserviços e implantada sobre infraestrutura de containers (Kubernetes). Os principais componentes e subsistemas incluem:

* **Non-Real-Time RAN Intelligent Controller (Non-RT RIC):** Módulo interno ao SMO encarregado da otimização de recursos de rádio em malha aberta (*open-loop control* com tempo de resposta $> 1.000\text{ ms}$). Hospeda **rApps** que analisam métricas de longo prazo e geram políticas de gerenciamento transmitidas ao Near-RT RIC via interface **A1**.
* **ONAP SDNC (Software Defined Network Controller - SDN-R):** Controlador baseado na plataforma OpenDaylight, munido da camada de abstração MD-SAL (Model-Driven Software Adaptation Layer). Atua como cliente NETCONF e endpoint de gerenciamento para a interface **O1**.
* **VES Collector & DMaaP (Data Movement as a Platform):** Mecanismo de alta performance para ingestão de eventos e telemetria. O VES Collector recebe mensagens HTTP POST em JSON formatadas segundo o padrão Virtual Event Streaming (VES) e as publica em tópicos dedicados no barramento Apache Kafka (DMaaP).
* **O2 Service Manager (O2 Adapter):** Camada de integração e abstração responsável por traduzir requisições de orquestração do SMO para provedores de infraestrutura de nuvem distribuída (**O-Cloud**).

---

## 2. Serviços e Funções Disponibilizados pelo SMO

A suíte de serviços do SMO provê o controle completo da operação de rede segundo o modelo FCAPS, estendido com inteligência artificial e automação:

### 2.1 Gerenciamento FCAPS
* **Fault Management (FM):** Recepção contínua e correlação de alarmes assíncronos transmitidos pelos nós O-CU, O-DU e Near-RT RIC.
* **Configuration Management (CM):** Leitura, auditoria e alteração do estado operacional das funções de rede por meio de transações NETCONF ACID (Atomicity, Consistency, Isolation, Durability) fundamentadas em modelos YANG.
* **Accounting & Performance Management (PM):** Ingestão de contadores de desempenho de hardware e de rádio (ex.: taxa de descarte de pacotes, utilização de PRBs). Os dados são roteados via DMaaP para bancos de dados de séries temporais (InfluxDB) e expostos em dashboards operacionais (Grafana).
* **Security Management:** Autenticação e autorização centralizadas via TLS mútuo (mTLS), controle de acesso baseado em papéis (RBAC) e gestão de certificados PKI via Keycloak e Traefik API Gateway.

### 2.2 Gerenciamento Inteligente de Políticas (Interface A1)
Criação e provimento de diretrizes operacionais de Radio Resource Management (RRM), como steering de tráfego (*Traffic Steering*), atenuação dinâmica de interferência e estratégias de economia de energia (*Energy Saving*).

---

## 3. Componentes O-RAN Suportados

O SMO provê suporte e gerenciamento unificado para os seguintes elementos padronizados pela O-RAN Alliance:

1. **O-CU-CP (Open Centralized Unit - Control Plane):** Gerenciamento de sinalização RRC e PDCP-C.
2. **O-CU-UP (Open Centralized Unit - User Plane):** Processamento do plano de usuário e agregação de dados.
3. **O-DU (Open Distributed Unit):** Controle em tempo real das camadas RLC, MAC e High-PHY.
4. **O-RU (Open Radio Unit):** Gerenciamento de parâmetros de radiofrequência e sincronização de Low-PHY via O-DU.
5. **Near-RT RIC (Near-Real-Time RAN Intelligent Controller):** Hospedagem de xApps para controle com restrição de latência ($10\text{ ms} - 1000\text{ ms}$).
6. **O-Cloud Infrastructure Nodes:** Hosts físicos e virtualizados de borda (*Edge Nodes*).

---

## 4. Implementação e Especificação das Interfaces O1 e O2

+-------------------+
                   |      O-RAN        |
                   |       SMO         |
                   +---------+---------+
                             |
          +------------------+------------------+
          |                                     |
          | Interface O1                        | Interface O2
  (NETCONF / VES HTTP)                     (O2ims / O2dms)
          |                                     |
          v                                     v

+---------------------+               +---------------------+
| O-DU / O-CU / xRIC  |               |       O-Cloud       |
| (Network Functions) |               |  (Infraestrutura)   |
+---------------------+               +---------------------+

### 4.1 Interface O1 (Gerenciamento de Elementos de Rede)
A interface O1 padroniza o canal de comunicação entre o SMO e as NFs gerenciadas:
* **Plano de Configuração (CM):** Utiliza o protocolo **NETCONF** (RFC 6241) rodando sobre SSH/TLS (porta 830). As operações (`get-config`, `edit-config`, `commit`) manipulam árvores de dados descritas na linguagem **YANG** (ex.: `o-ran-hardware.yang`, `o-ran-processing-element.yang`).
* **Plano de Eventos e Telemetria (FM/PM):** As NFs empacotam métricas e falhas em esquemas JSON padronizados (**VES Common Event Header**) e enviam requisições `HTTP POST` para o coletor VES do SMO na porta 8443.

### 4.2 Interface O2 (Gerenciamento de Infraestrutura O-Cloud)
Dividida em duas sub-interfaces para desacoplar a aplicação da infraestrutura física:
* **O2ims (Infrastructure Management Services):** Fornece endpoints REST/JSON para descoberta de inventário, consulta de topologia física, monitoramento de recursos de computação, armazenamento e aceleração de hardware (ex.: placas FPGA/GPU para aceleração de FEC).
* **O2dms (Deployment Management Services):** Gerencia a implantação das NFs na O-Cloud. Aceita comandos de orquestração baseados em manifestos do Kubernetes e repositórios de Helm Charts.

---

## 5. Mecanismos de Gerenciamento, Provisionamento e Orquestração

O fluxo de provisionamento no SMO segue o princípio de **Gerenciamento Declarativo**:
1. O operador de rede define o estado desejado da infraestrutura através de um modelo declarativo ou via API do SMO.
2. O **ONAP SDNC** valida a requisição contra a árvore de esquemas YANG carregada na memória do MD-SAL.
3. O SDNC mantém a separação rigorosa entre dois datastores:
   * **Config Datastore:** Contém as configurações desejadas enviadas pelos administradores ou rApps.
   * **Operational Datastore:** Reflete a configuração real lida diretamente dos equipamentos via requisições `get`.
4. Processos de auditoria e sincronização contínua (*Reconciliation Loops*) comparam ambos os datastores e aplicam correções automáticas caso seja identificado desvio de configuração (*configuration drift*).

---

## 6. Capacidades de Gerenciamento do O-Cloud

Através da interface O2, o SMO obtém pleno controle sobre os recursos de nuvem de borda:
* **Descoberta de Capacidades de Hardware:** Mapeamento automático de nós com suporte a instruções vetoriais, placas de aceleração Lookaside/Inline e conectividade SR-IOV.
* **Orquestração Multi-Cluster:** Capacidade de gerenciar múltiplos clusters Kubernetes geograficamente distribuídos a partir de um único painel de controle SMO.
* **Isolamento e Qualidade de Serviço (QoS):** Alocação de políticas de *CPU Pinning*, páginas de memória gigabyte (*HugePages*) e isolamento de rede para garantir que os contêineres do O-DU atinjam latências determinísticas de ordenamento de pacotes da interface *Fronthaul*.

---

## 7. Suporte ao Ciclo de Vida de Network Functions (LCM)

O gerenciamento do ciclo de vida das funções de rede (CNFs e VNFs) no SMO segue as diretrizes da arquitetura ETSI NFV MANO integradas aos workflows do ONAP:

[Onboarding] -> [Instanciação (O2dms)] -> [Config Day-0/1 (O1)] -> [Operação Day-2] -> [Terminação]

1. **Onboarding:** Validação e registro do pacote de software (pacotes CSAR/Helm Charts) contendo os descritores de implantação da NF e regras de dimensionamento (*scaling rules*).
2. **Instanciação:** O SMO dispara chamadas via O2dms para que o O-Cloud aloque o Namespace, Pods, Services e montagens de volumes necessários para a NF.
3. **Configuração Day-0 / Day-1:** Estabelecimento da conectividade básica via O1 NETCONF, injeção de endereços IP de transporte, parâmetros de segurança e chaves de criptografia.
4. **Configuração Day-2 (Operacional):** Alterações dinâmicas em tempo de execução, como reconfiguração de frequências de canal, potência de transmissão do rádio ou ampliação do número de células ativas.
5. **Terminação:** Encerramento gracioso dos processos da NF, desalocação de infraestrutura no O-Cloud via O2 e remoção de registros nos datastores operacionais.

---

## 8. Monitoramento, Telemetria e Gerenciamento de Falhas

### 8.1 Pipeline de Tratamento de Alertas
1. **Geração do Evento:** Uma anomalia na O-DU (ex.: saturação no uso de PRBs ou perda de sincronismo PTP) dispara um evento VES tipo `fault`.
2. **Ingestão no VES Collector:** O coletor recebe a carga JSON, valida o cabeçalho `commonEventHeader` e encaminha a mensagem para o tópico do Kafka `unauthenticated.SEC_FAULT_OUTPUT`.
3. **Processamento e Persistência:** O serviço **PM Mapper** processa os pacotes e persiste os dados na base **InfluxDB**.

### 8.2 Automação em Malha Fechada (Closed-Loop Automation)
O SMO não se limita ao monitoramento passivo. Ele implementa laços de automação baseados em eventos:
* Um **rApp** em execução no **Non-RT RIC** consome o fluxo de eventos de desempenho do Kafka.
* O algoritmo do rApp identifica uma condição de degradação de cobertura em determinado setor.
* O rApp gera uma nova política A1 ou dispara uma requisição de reconfiguração de tilt elétrico via NETCONF/O1 para o SDNC.
* O SDNC aplica a alteração no elemento de rede em tempo de execução, corrigindo a falha sem intervenção humana manual.

---

## 9. Referências Bibliográficas

* O-RAN Alliance. **O-RAN Architecture Description** (O-RAN.WG1.O-RAN-Architecture-v10.00).
* O-RAN Alliance. **O1 Interface specification for O-DU and O-CU** (O-RAN.WG10.O1-Interface).
* O-RAN Alliance. **O2 Interface General Aspects and Principles** (O-RAN.WG6.O2-GA&P).
* O-RAN Software Community (OSC). **SMO Architecture & OAM Project Documentation**. Disponível em: `<https://docs.o-ran-sc.org/>`.
* ONAP Documentation. **SDN Controller for Radio (SDN-R) Architecture Guide**.