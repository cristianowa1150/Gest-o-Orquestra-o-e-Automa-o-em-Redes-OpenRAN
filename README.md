# Opção 1: OSC-SMO para gestão e automação Open RAN

```text
.
├── index.html
├── README.md
├── docs/
│   └── relatorio_smo.md
└── config/
    ├── o1-netconf-device.xml
    └── ves-telemetry-event.json
```

Este repositório apresenta um estudo teórico-prático do **OSC-SMO**, a implementação de referência do Service Management Orchestrator (SMO) mantida pela **O-RAN Software Community (OSC)** e alinhada às especificações da **O-RAN Alliance**.

O recorte adotado é exclusivamente a **Opção 1 — OSC-SMO**. O SMO é analisado como a camada de gestão, orquestração e automação que integra o gerenciamento O1 dos elementos de rede, a administração O2 da O-Cloud e o Non-RT RIC com suas rApps.

---

## 🚀 Demonstração On-line (Console / Emulador SMO)

Você pode acessar a interface de controle e monitoramento simulado em tempo real: [Console de Simulação do SMO On-line](https://cristianowa1150.github.io/Gest-o-Orquestra-o-e-Automa-o-em-Redes-OpenRAN/index.html)

---

## 📂 Estrutura do Repositório

- `docs/relatorio_smo.md`: Relatório acadêmico/técnico sobre arquitetura, responsabilidades, interfaces O1/O2, FCAPS, Non-RT RIC, ciclo de vida e O-Cloud no OSC-SMO.
- `config/`: Exemplos práticos de arquivos de payload e mensagens de controle das interfaces O1:
  - `o1-netconf-device.xml`: Payload RPC NETCONF `<edit-config>` de uma alteração de configuração O1.
  - `ves-telemetry-event.json`: Evento JSON de falha e telemetria recebido pelo pipeline O1.
- `index.html`: Simulador visual do OSC-SMO, com ingestão de eventos, políticas do Non-RT RIC, O1 e O2.

### Campos dos arquivos de configuração

- No `o1-netconf-device.xml`, os comentários explicam a alteração de configuração enviada pelo SMO, o alvo O-RU, a largura de banda e a potência de transmissão.
- O `ves-telemetry-event.json` permanece em JSON puro. `commonEventHeader` identifica o evento, a origem, a prioridade e a severidade; `faultFields` descreve a falha, a condição do alarme e os valores de PRB observados.

---

## Escopo técnico da Opção 1

- **SMO:** arquitetura distribuída do OSC-SMO, serviços de OAM e automação orientada a eventos.
- **O1:** gestão de configuração com NETCONF/YANG e coleta de falhas e desempenho por VES.
- **O2:** descoberta e administração da O-Cloud por O2ims e implantação de funções de rede por O2dms.
- **Non-RT RIC:** rApps para análise de longo prazo, otimização e publicação de políticas A1 para o Near-RT RIC.
- **Ciclo de vida:** onboarding, instanciação, configuração Day-0/Day-1, operação Day-2, atualização e terminação.
- **Operação:** FCAPS, inventário, observabilidade, segurança por identidade e reconciliação do estado desejado.

## Tecnologias e padrões analisados

- **Arquiteturas:** O-RAN Alliance Architecture WG1/WG10, O-RAN Software Community e ETSI NFV MANO como referência de ciclo de vida.
- **Interfaces & Protocolos:**
  - **O1:** NETCONF/YANG (Configuração), HTTP REST/JSON VES (Virtual Event Streaming para FCAPS).
  - **O2:** REST APIs (O2ims para gerenciamento de infraestrutura e O2dms para orquestração de recursos).
  - **A1:** HTTP/REST (Interface entre Non-RT RIC e Near-RT RIC para publicação de políticas).
- **Gerenciamento de infraestrutura:** O-Cloud, Kubernetes multi-cluster e Helm Charts.

## Limites da simulação

O painel é um emulador didático: os eventos, métricas, sessões NETCONF e estados O2 são gerados no navegador. Ele demonstra os fluxos e responsabilidades do OSC-SMO, mas não estabelece sessões reais com O-DU, O-CU, O-RU, O-Cloud ou RIC.

---

## 👨‍💻 Autores

- Cristiano Silveira Silva
- Gilmar
- Josenildo
