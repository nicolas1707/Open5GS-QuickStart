# Inicialização e Conexão de Três UEs na Slice eMBB

## 1. Objetivo

Esta etapa documenta a inicialização do gNodeB e de três equipamentos de usuário (*User Equipment* — UEs) simulados pelo UERANSIM, previamente configurados para utilizar a mesma fatia lógica da rede 5G.

O objetivo é colocar os três dispositivos online simultaneamente no Core 5G implementado com Open5GS, verificando o registro dos UEs, o estabelecimento das sessões PDU e a criação das respectivas interfaces de rede virtuais.

Este procedimento prepara o ambiente para o teste de congestionamento, no qual os três UEs poderão gerar tráfego concorrente utilizando a mesma configuração de fatia eMBB.

**Escopo desta etapa:** inicialização dos processos, validação das conexões e identificação dos endereços IP atribuídos aos UEs.

A geração de tráfego concorrente e a medição de desempenho TCP e UDP serão realizadas posteriormente.

---

## 2. Cenário Experimental

O cenário utiliza três UEs independentes, cada um com seu próprio IMSI e arquivo de configuração YAML, todos associados à mesma fatia.

| Parâmetro      | UE 1              | UE 2              | UE 3              |
| -------------- | ----------------- | ----------------- | ----------------- |
| Arquivo YAML   | `ue1-sst1.yaml`   | `ue2-sst1.yaml`   | `ue3-sst1.yaml`   |
| IMSI           | `999700000000001` | `999700000000002` | `999700000000003` |
| SST            | 1                 | 1                 | 1                 |
| DNN            | `internet`        | `internet`        | `internet`        |
| Tipo de sessão | IPv4              | IPv4              | IPv4              |

Os três assinantes devem estar previamente cadastrados na WebUI do Open5GS, e os arquivos YAML devem estar configurados de acordo com os perfis correspondentes.

### Arquitetura do teste

```text
                  Open5GS 5G Core
                         |
                        AMF
                         |
                    Interface N2
                       (NGAP)
                         |
                       gNodeB
                         |
                  Interface N3 (GTP-U)
                         |
                 UPF / Plano de Usuário
                         |
          +--------------+--------------+
          |              |              |
         UE 1           UE 2           UE 3
          |              |              |
      uesimtun0      uesimtun1      uesimtun2
          |              |              |
      SST 1          SST 1          SST 1
      internet       internet       internet
```

O diagrama representa o cenário lógico pretendido. As interfaces TUN são criadas no sistema operacional que executa os processos de UE, após o estabelecimento das respectivas sessões PDU.

---

## 3. Preparação dos Terminais

Serão utilizados quatro terminais para a inicialização e um quinto terminal para a verificação das interfaces.

| Terminal   | Função                                   |
| ---------- | ---------------------------------------- |
| Terminal 1 | Inicialização do gNodeB                  |
| Terminal 2 | Inicialização do UE 1                    |
| Terminal 3 | Inicialização do UE 2                    |
| Terminal 4 | Inicialização do UE 3                    |
| Terminal 5 | Verificação das interfaces TUN e dos IPs |

Em todos os terminais destinados à execução do UERANSIM, acesse o diretório do projeto:

```bash
cd ~/UERANSIM
```

**Explicação:** o comando altera o diretório de trabalho para a pasta do UERANSIM. Caso o projeto esteja instalado em outro caminho, utilize o diretório correspondente.

Os comandos seguintes pressupõem que os executáveis estejam disponíveis em `build/` e os arquivos de configuração em `config/`.

---

## 4. Inicialização do gNodeB

No Terminal 1, execute:

```bash
./build/nr-gnb -c config/open5gs-gnb.yaml
```

### Funcionamento do comando

* `./build/nr-gnb`: executa o simulador da estação rádio base 5G NR.
* `-c`: indica que será utilizado um arquivo de configuração.
* `config/open5gs-gnb.yaml`: define os parâmetros do gNodeB, incluindo PLMN, TAC, endereços IP e informações do AMF.

O gNodeB estabelece a conexão de sinalização com o AMF do Open5GS pela interface N2, utilizando NGAP sobre SCTP.

### Validação esperada

Aguarde a mensagem:

```text
NG Setup procedure is successful
```

Essa mensagem indica que o procedimento de configuração da conexão NG entre o gNodeB e o Core 5G foi concluído com sucesso.

Mantenha esse terminal aberto enquanto os três UEs estiverem em execução.

---

## 5. Inicialização dos Três UEs

Após confirmar a conexão do gNodeB com o AMF, inicie cada UE em um terminal separado.

### 5.1. UE 1 — Terminal 2

```bash
sudo ./build/nr-ue -c config/ue1-sst1.yaml
```

Esse comando inicia o primeiro UE utilizando o arquivo que contém o IMSI `999700000000001` e a configuração de sessão SST 1 / DNN `internet`.

### 5.2. UE 2 — Terminal 3

```bash
sudo ./build/nr-ue -c config/ue2-sst1.yaml
```

O segundo processo inicia o UE identificado pelo IMSI `999700000000002`, com a mesma configuração de fatia e DNN do primeiro dispositivo.

### 5.3. UE 3 — Terminal 4

```bash
sudo ./build/nr-ue -c config/ue3-sst1.yaml
```

O terceiro processo inicia o UE identificado pelo IMSI `999700000000003`, também configurado para solicitar uma sessão PDU IPv4 na fatia SST 1.

### Por que utilizar três terminais?

Cada comando inicia um processo independente do UERANSIM. Isso permite acompanhar separadamente os registros, as mensagens de sinalização, o estabelecimento das sessões e eventuais erros de cada UE.

Embora os três dispositivos utilizem a mesma fatia lógica, eles possuem identidades de assinante distintas e estabelecem sessões PDU individuais.

---

## 6. Validação do Registro e das Sessões PDU

Observe as mensagens exibidas nos terminais dos três UEs.

Após a conexão bem-sucedida, cada dispositivo deverá apresentar mensagens que indiquem o registro na rede e o estabelecimento da sessão PDU.

Uma mensagem de referência é:

```text
PDU Session establishment is successful
```

Dependendo da versão do UERANSIM, a saída também pode identificar a sessão, a interface TUN e o endereço IP atribuído.

### Interpretação técnica

O procedimento envolve diferentes etapas:

1. O UE inicia o acesso à rede simulada.
2. O gNodeB encaminha a sinalização para o AMF.
3. O Core 5G executa os procedimentos de registro e autenticação do assinante.
4. O UE solicita a sessão PDU configurada.
5. O SMF processa a solicitação e coordena a configuração da sessão e do plano de usuário.
6. O UPF participa do encaminhamento dos pacotes de dados.
7. Após o estabelecimento bem-sucedido da sessão, o UERANSIM cria a interface TUN correspondente.

A mensagem de sucesso deve ser verificada individualmente nos três terminais. O sucesso de um UE não comprova que os outros dois também tenham estabelecido suas sessões.

---

## 7. Verificação das Interfaces TUN

Com os três processos em execução, abra o Terminal 5 e execute:

```bash
ip addr | grep uesimtun
```

### O que o comando faz?

* `ip addr`: exibe os endereços e as interfaces de rede conhecidos pelo kernel Linux.
* `|`: encaminha a saída do primeiro comando para o seguinte.
* `grep uesimtun`: filtra as linhas que contêm o nome das interfaces TUN utilizadas pelo UERANSIM.

As interfaces TUN permitem que o tráfego IP dos UEs seja encaminhado pelo plano de usuário da rede simulada.

### Resultado observado no laboratório

Durante a execução deste cenário, foram identificadas as seguintes associações:

| Interface TUN | UE associado | Endereço IPv4 observado | Slice | DNN        |
| ------------- | ------------ | ----------------------- | ----- | ---------- |
| `uesimtun0`   | UE 1         | `10.45.0.9`             | SST 1 | `internet` |
| `uesimtun1`   | UE 2         | `10.45.0.10`            | SST 1 | `internet` |
| `uesimtun2`   | UE 3         | `10.45.0.11`            | SST 1 | `internet` |

**Importante:** esses endereços correspondem aos resultados observados nesta execução do laboratório. A ordem de criação das interfaces e os endereços atribuídos podem variar em outras execuções. Por isso, confirme os valores reais antes de utilizá-los nos testes de tráfego.

A saída filtrada pode não mostrar todas as informações necessárias para identificar cada endereço IP. Para consultar cada interface individualmente, execute:

```bash
ip addr show uesimtun0
ip addr show uesimtun1
ip addr show uesimtun2
```

Esses comandos exibem os detalhes de cada interface, incluindo seu estado e o endereço IP configurado.

---

## 8. Relação com o Teste de Congestionamento

Com os três UEs registrados e suas sessões PDU estabelecidas, o ambiente fica preparado para investigar o comportamento de tráfego concorrente na fatia eMBB.

Os três dispositivos utilizam a mesma combinação de SST e DNN, mas possuem sessões e endereços IP individuais.

Isso permite executar testes nos quais diferentes UEs geram tráfego simultaneamente, observando o desempenho agregado e individual.

**Atenção:** colocar três UEs na mesma fatia não provoca congestionamento automaticamente. O congestionamento precisa ser induzido por geração de tráfego e avaliado a partir de métricas como vazão, latência, perda de pacotes e, quando aplicável, jitter.

Da mesma forma, compartilhar o SST e o DNN não garante, por si só, que exista um limite de banda ou uma política de QoS específica. Esses comportamentos dependem das configurações e dos recursos efetivamente aplicados à rede.

---

## 9. Critérios de Sucesso

* [ ] O gNodeB foi inicializado sem erros críticos.
* [ ] A mensagem `NG Setup procedure is successful` foi observada.
* [ ] O UE 1 foi registrado e estabeleceu sua sessão PDU.
* [ ] O UE 2 foi registrado e estabeleceu sua sessão PDU.
* [ ] O UE 3 foi registrado e estabeleceu sua sessão PDU.
* [ ] As interfaces `uesimtun0`, `uesimtun1` e `uesimtun2` foram identificadas.
* [ ] Os endereços IP reais de cada interface foram registrados.
* [ ] Os três UEs estão configurados para utilizar SST 1 e DNN `internet`.

A conclusão desses critérios valida a preparação do cenário de três UEs na mesma fatia. Ela ainda não representa um resultado de desempenho nem comprova a existência de congestionamento.

---

## 10. Próxima Etapa

Após validar a inicialização e a conectividade dos três UEs, a próxima etapa será executar testes de tráfego concorrente.

O objetivo será gerar carga de dados através das sessões PDU e medir o comportamento da rede quando os três dispositivos utilizarem simultaneamente a fatia eMBB.

Os resultados deverão ser registrados para permitir comparações posteriores com cenários em que os UEs utilizam fatias distintas.


