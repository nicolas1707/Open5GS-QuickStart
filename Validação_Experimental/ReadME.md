# Validação Experimental

Esta seção reúne os experimentos realizados para validar o funcionamento de um ambiente **5G Standalone (5G SA)** implementado com **Open5GS** e **UERANSIM**.

Após a conclusão da configuração do ambiente, foram executados diversos testes experimentais com o objetivo de verificar o comportamento do Core 5G, do plano de usuário e das funcionalidades de **Network Slicing**, utilizando diferentes perfis de tráfego e cenários de comunicação.

Cada experimento documenta detalhadamente:

- objetivo do teste;
- metodologia empregada;
- comandos executados;
- resultados obtidos;
- evidências experimentais;
- análise técnica dos resultados.

---

# Etapas Documentadas

Os experimentos foram organizados de forma progressiva, iniciando pela validação da comunicação básica e evoluindo para cenários de desempenho e utilização simultânea de múltiplas fatias de rede.

| Etapa | Descrição |
|--------|-----------|
| **1** | Testes de conectividade e validação do ambiente |
| **2** | Testes de desempenho TCP |
| **3** | Testes de desempenho UDP |
| **4** | Validação da Slice eMBB |
| **5** | Validação da Slice URLLC |
| **6** | Validação da Slice mMTC |
| **7** | Validação de múltiplas Slices simultâneas |
| **8** | Análise comparativa dos resultados obtidos |

---

# Principais Testes Realizados

## Conectividade

- Validação da comunicação UE ↔ UPF
- Verificação da conectividade fim a fim
- Validação da atribuição de endereços IP
- Confirmação do encaminhamento de tráfego pelo User Plane

---

## Desempenho da Rede

Os testes de desempenho foram realizados utilizando o **iperf3**, permitindo avaliar diferentes características do plano de usuário.

Foram executados experimentos envolvendo:

- TCP
- UDP
- Vazão máxima
- Controle de banda
- Jitter
- Perda de pacotes
- Utilização de interfaces específicas através da opção `-B`

---

## Network Slicing

Os experimentos utilizam um cenário de **Network Slicing**, no qual um único equipamento de usuário estabelece simultaneamente três **PDU Sessions**, cada uma associada a uma fatia lógica distinta da rede.

| Slice | SST | DNN | Objetivo |
|--------|----:|-----|----------|
| eMBB | 1 | internet | Alta taxa de transmissão |
| URLLC | 2 | emergency | Baixa latência |
| mMTC | 3 | iot | Comunicação massiva de dispositivos |

Cada sessão utiliza uma interface TUN independente, permitindo o isolamento lógico do tráfego entre as diferentes fatias.

---

# Ferramentas Utilizadas

- Ubuntu 22.04 LTS
- Open5GS
- MongoDB
- Open5GS WebUI
- UERANSIM
- iperf3
- Linux Networking

---

# Ambiente Experimental

Todos os experimentos foram executados sobre um ambiente previamente configurado conforme a documentação disponível no repositório **Configuração do Ambiente**.

O ambiente utilizado possui:

- Core 5G Standalone operacional;
- gNodeB conectado ao AMF;
- UE registrado no Core;
- três PDU Sessions estabelecidas;
- três interfaces TUN (`uesimtun0`, `uesimtun1` e `uesimtun2`) ativas;
- três Network Slices configuradas.

---

# Objetivo da Documentação

Além da apresentação dos resultados experimentais, este repositório busca documentar o comportamento do ambiente diante de diferentes cenários de utilização, permitindo analisar:

- desempenho do plano de usuário;
- funcionamento das PDU Sessions;
- utilização das interfaces TUN;
- comportamento das diferentes fatias de rede;
- isolamento do tráfego entre Slices;
- impacto de diferentes perfis de comunicação sobre o Core 5G.

Cada experimento é acompanhado por evidências obtidas diretamente durante a execução do laboratório, incluindo capturas de tela, logs e resultados produzidos pelas ferramentas de teste.
