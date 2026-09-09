```
.
├── index.html
├── README.md
├── docs/
│   └── relatorio_smo.md
└── config/
    ├── o1-netconf-device.xml
    └── ves-telemetry-event.json
```


# Gestão, Orquestração e Automação em Redes Open RAN: Estudo e Simulação do Service Management Orchestrator (SMO)

Este repositório contém o estudo teórico-prático, a especificação técnica e o ambiente de simulação e controle para a plataforma **Service Management Orchestrator (SMO)**, desenvolvida sob as especificações da **O-RAN Alliance** e implementada com base na arquitetura do **O-RAN Software Community (OSC)** e **ONAP**.

---

## 🚀 Demonstração On-line (Console / Emulador SMO)

Você pode acessar a interface de controle e monitoramento simulado em tempo real:
👉 <a href="https://cristianowa1150.github.io/Gest-o-Orquestra-o-e-Automa-o-em-Redes-OpenRAN/index.html" target="_blank" rel="noopener noreferrer">Console de Simulação do SMO On-line</a>

---

## 📂 Estrutura do Repositório

- `docs/relatorio_smo.md`: Relatório acadêmico/técnico completo abordando os tópicos exigidos na avaliação (Arquitetura, Interfaces O1/O2, FCAPS, Non-RT RIC, LCM de NFs e O-Cloud).
- `config/`: Exemplos práticos de arquivos de payload e mensagens de controle das interfaces O1:
  - `o1-netconf-device.xml`: Payload RPC NETCONF `<edit-config>` baseado em modelos YANG.
  - `ves-telemetry-event.json`: Evento JSON de telemetria/alarme para o coletor VES.
- `index.html`: Dashboard interativo simulando a ingestão de eventos VES, controle de políticas A1 e gerenciamento de infraestrutura O2.

---

## 🛠️ Tecnologias e Padrões Analisados

- **Arquiteturas:** O-RAN Alliance Architecture WG1/WG10, ETSI NFV MANO, ONAP Framework (SDNC, DMaaP).
- **Interfaces & Protocolos:**
  - **O1:** NETCONF/YANG (Configuração), HTTP REST/JSON VES (Virtual Event Streaming para FCAPS).
  - **O2:** REST APIs (O2ims para gerenciamento de infraestrutura e O2dms para orquestração de recursos).
  - **A1:** HTTP/REST (Interface entre Non-RT RIC e Near-RT RIC para publicação de políticas).
- **Gerenciamento de Infraestrutura:** O-Cloud, Kubernetes (Multi-cluster), Helm Charts.

---

## 👨‍💻 Autores

- Cristiano Silveira Silva
- Gilmar 
- Josenildo
- 
