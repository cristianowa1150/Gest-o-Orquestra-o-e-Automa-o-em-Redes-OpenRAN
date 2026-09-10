# Relatório Técnico: OSC-SMO em ambientes Open RAN

**Disciplina:** Gestão, Orquestração e Automação em Redes Open RAN  
**Objeto de estudo:** OSC-SMO, o Service Management Orchestrator da O-RAN Software Community  
**Data:** Setembro de 2026

## 1. Introdução e arquitetura

Este relatório trata exclusivamente da **Opção 1 — OSC-SMO**, o SMO da **O-RAN Software Community (OSC)** alinhado à arquitetura da **O-RAN Alliance**. O Service Management Orchestrator é a camada central de gestão, orquestração e automação do ciclo de vida dos elementos de uma rede Open RAN.

O OSC-SMO adota uma arquitetura distribuída, orientada a serviços e executada sobre infraestrutura de contêineres. Seu valor está em integrar inventário, configuração, observabilidade, políticas e implantação em um ponto coerente de operação, sem eliminar as responsabilidades dos elementos O-RAN.

Os principais componentes considerados são:

* **Serviços de gerenciamento O1:** adaptadores e controladores que atuam como clientes NETCONF/YANG para configuração e auditoria dos elementos de rede.
* **Non-RT RIC:** ambiente para rApps que analisam tendências, contexto e desempenho de longo prazo e geram políticas para o domínio RAN.
* **Pipeline de eventos e telemetria:** coleta, validação, correlação e distribuição de eventos VES, métricas e alarmes.
* **O2 Service Manager:** integração entre o SMO e os serviços de infraestrutura e implantação da O-Cloud.
* **Catálogo e inventário:** registro dos recursos, capacidades, versões, relacionamentos e estado operacional dos componentes gerenciados.

### 1.1 Escopo e responsabilidades

O OSC-SMO é responsável por manter uma visão operacional coerente da rede e da infraestrutura que a hospeda. Isso inclui:

* registrar inventário e capacidades de O-CU, O-DU, O-RU, RIC e recursos da O-Cloud;
* aplicar configuração e verificar conformidade por O1;
* coletar alarmes, métricas e eventos de desempenho;
* coordenar o ciclo de vida das funções de rede por O2;
* apoiar decisões de otimização por meio do Non-RT RIC e de rApps;
* preservar rastreabilidade, autorização e reconciliação do estado desejado.

O SMO não substitui o processamento de rádio do O-DU/O-CU, o processamento de RF do O-RU, o controle de baixa latência do Near-RT RIC ou a execução dos workloads na O-Cloud. Ele coordena esses domínios por interfaces padronizadas.

## 2. Serviços e funções

### 2.1 Gerenciamento FCAPS

* **Fault Management:** recebe e correlaciona alarmes assíncronos de O-CU, O-DU, O-RU e RIC.
* **Configuration Management:** lê, audita e altera o estado dos elementos por transações NETCONF fundamentadas em modelos YANG.
* **Accounting e Performance Management:** ingere contadores como utilização de PRBs, perda de pacotes, latência e disponibilidade.
* **Security Management:** aplica autenticação, autorização por papéis (RBAC), TLS mútuo, rotação de certificados e auditoria das operações.

### 2.2 Automação e políticas

O Non-RT RIC hospeda rApps para análise de longo prazo e otimização. Exemplos de políticas são *Traffic Steering*, economia de energia, balanceamento de carga e mitigação de interferência. A política pode ser publicada por A1 para o Near-RT RIC ou resultar em uma mudança O1, sempre respeitando validação, autorização e limites de operação.

## 3. Componentes O-RAN gerenciados

1. **O-CU-CP:** sinalização RRC e processamento de controle.
2. **O-CU-UP:** processamento do plano de usuário e agregação de dados.
3. **O-DU:** camadas RLC, MAC e High-PHY com requisitos de tempo mais restritos.
4. **O-RU:** rádio, conversão e parâmetros de RF sob coordenação do O-DU.
5. **Near-RT RIC:** execução de xApps e controle em intervalo aproximado de $10\text{ ms}$ a $1.000\text{ ms}$.
6. **Non-RT RIC:** execução de rApps, análise e políticas de maior horizonte dentro do SMO.
7. **O-Cloud:** recursos físicos e virtualizados que executam as funções de rede.

## 4. Interfaces O1, O2 e A1

```text
                         OSC-SMO
                    /       |       \
                  O1        O2        A1
                 /          |          \
          O-CU/O-DU/O-RU  O-Cloud  Near-RT RIC
                 \
                  eventos, métricas e políticas
```

### 4.1 Interface O1

A interface O1 conecta o SMO aos elementos de rede gerenciados:

* **Configuração:** NETCONF sobre SSH ou TLS, com operações como `get-config`, `edit-config` e `commit` sobre árvores YANG.
* **Telemetria e falhas:** mensagens HTTP/JSON no formato VES, contendo cabeçalho comum, origem, severidade e dados específicos do evento.
* **Auditoria:** comparação entre o estado desejado e o estado operacional, com reconciliação quando a política permitir.

O arquivo `config/o1-netconf-device.xml` demonstra uma alteração de largura de banda e potência de uma O-RU por `edit-config`. O arquivo `config/ves-telemetry-event.json` demonstra um alarme de congestionamento de PRBs.

### 4.2 Interface O2

A interface O2 desacopla as funções de rede da infraestrutura:

* **O2ims:** descoberta de inventário, topologia, capacidade de computação, armazenamento, rede e aceleradores.
* **O2dms:** implantação, atualização e remoção de funções usando descritores, manifestos e recursos da O-Cloud.

O2 permite operar múltiplos clusters e regiões mantendo uma visão de inventário e capacidade no SMO. A decisão de posicionamento considera recursos disponíveis, afinidade, latência, aceleração e requisitos de confiabilidade.

### 4.3 Interface A1

A1 transporta políticas entre o Non-RT RIC e o Near-RT RIC. No contexto deste trabalho, ela representa a saída de uma rApp após análise de métricas e contexto. O Near-RT RIC aplica a política no domínio de baixa latência e devolve informações para observabilidade.

## 5. Provisionamento e reconciliação

O fluxo do OSC-SMO segue o princípio de gerenciamento declarativo:

1. O operador define o estado desejado por catálogo ou API.
2. O serviço O1 valida o pedido contra os esquemas YANG e as políticas de segurança.
3. O SMO envia a mudança ao elemento e registra a transação.
4. O estado operacional é lido novamente por `get` e comparado com o desejado.
5. Um desvio de configuração é sinalizado e, caso autorizado, corrigido por reconciliação.

Essa separação evita confundir o que foi solicitado com o que realmente está ativo no equipamento. Toda automação precisa manter idempotência, rastreabilidade e uma resposta clara para sucesso ou falha.

## 6. O-Cloud e requisitos de operação

Pela O2, o SMO pode descobrir e coordenar:

* nós de borda, clusters e domínios de disponibilidade;
* CPU pinning, HugePages, SR-IOV e aceleradores FPGA/GPU;
* redes, volumes, namespaces, serviços e imagens de funções de rede;
* restrições de afinidade, isolamento e qualidade de serviço.

A O-Cloud deve oferecer capacidade suficiente para workloads de rádio e observar requisitos de latência, sincronismo, conectividade e recuperação. O SMO coordena a implantação, mas a garantia física desses requisitos depende da infraestrutura e dos elementos de rede.

## 7. Ciclo de vida das Network Functions

```text
[Onboarding] -> [Instanciação O2dms] -> [Configuração O1]
       -> [Operação Day-2] -> [Atualização] -> [Terminação]
```

1. **Onboarding:** validação do pacote, descritor, versão e requisitos da função.
2. **Instanciação:** alocação de namespace, pods, serviços, volumes e redes na O-Cloud.
3. **Day-0/Day-1:** conectividade, identidade, endereços e parâmetros iniciais via O1.
4. **Day-2:** ajustes dinâmicos, escalabilidade, manutenção e auditoria.
5. **Atualização:** substituição controlada da versão com verificação de saúde e rollback.
6. **Terminação:** encerramento gracioso, desalocação de recursos O2 e limpeza do inventário.

## 8. Telemetria, falhas e malha fechada

### 8.1 Pipeline de alertas

1. Uma anomalia na O-DU, como PRB acima do limite ou perda de sincronismo, gera um evento VES.
2. O coletor valida `commonEventHeader`, severidade, origem e campos da falha.
3. O evento é correlacionado com inventário, histórico e métricas de desempenho.
4. O SMO apresenta o diagnóstico e disponibiliza a informação para uma rApp.

### 8.2 Automação em malha fechada

Uma rApp do Non-RT RIC pode identificar degradação persistente, calcular uma ação e publicar uma política A1. Em outro cenário, pode solicitar uma mudança O1, como reequilíbrio de recursos. O resultado é confirmado por novos eventos e métricas. Falhas, falta de autorização ou ausência de capacidade devem interromper a automação e encaminhar o caso para análise humana.

## 9. Segurança, confiabilidade e observabilidade

O1 e O2 são superfícies de controle críticas. Uma implantação OSC-SMO deve aplicar autenticação forte, autorização mínima, proteção de credenciais, rotação de certificados e trilhas de auditoria. A ingestão de VES deve validar origem, versão, severidade e integridade antes de alimentar automações.

Mudanças devem passar por validação de esquema, janela de aplicação, confirmação do resultado e possibilidade de rollback. Métricas de saúde do SMO, filas de eventos, sessões O1, recursos O-Cloud e latência de políticas precisam ser acompanhadas separadamente.

## 10. Simulador e limites do protótipo

O `index.html` é um simulador didático executado no navegador. Ele gera métricas, eventos, estados O1/O2 e respostas de malha fechada localmente; não estabelece conexões reais com O-DU, O-CU, O-RU, RIC ou O-Cloud. Portanto, o painel demonstra o encadeamento conceitual do OSC-SMO, mas não mede desempenho de uma implantação de produção.

Os critérios de avaliação são: identificação dos papéis do SMO, uso coerente de O1/O2/A1, separação entre Non-RT e Near-RT RIC, descrição do ciclo de vida, tratamento de falhas e reconhecimento dos limites do protótipo.

## 11. Vantagens e desafios da escolha

O OSC-SMO é uma escolha bem alinhada ao tema por seguir a arquitetura da O-RAN Alliance e reunir O1, O2, O-Cloud e Non-RT RIC no mesmo estudo. Essa cobertura permite demonstrar tanto o gerenciamento de elementos quanto a orquestração da infraestrutura e a automação orientada por políticas.

Os principais desafios são a complexidade de uma implantação completa, a necessidade de integrar múltiplos componentes e o custo de operar Kubernetes, inventário, telemetria, segurança e RIC de forma consistente. Por isso, este trabalho implementa uma simulação focada nos fluxos essenciais, sem alegar que todos os serviços de produção foram reproduzidos.

## 12. Referências bibliográficas

* O-RAN Alliance. **O-RAN Architecture Description** (O-RAN.WG1.O-RAN-Architecture-v10.00).
* O-RAN Alliance. **O1 Interface specification for O-DU and O-CU** (O-RAN.WG10.O1-Interface).
* O-RAN Alliance. **O2 Interface General Aspects and Principles** (O-RAN.WG6.O2-GA&P).
* O-RAN Software Community (OSC). **SMO Architecture & OAM Project Documentation**. Disponível em: <https://docs.o-ran-sc.org/>.
