# Teste de Congestionamento (Três UEs na Mesma Slice)

## 1. Objetivo

Esta etapa documenta a execução de testes de tráfego UDP e medições de latência para três equipamentos de usuário (UEs) conectados simultaneamente à mesma fatia lógica da rede 5G.

O cenário utiliza três UEs previamente configurados com SST 1 e DNN `internet`. Cada dispositivo gera tráfego com uma taxa-alvo diferente, permitindo observar o comportamento das sessões sob cargas distintas.

Os testes utilizam:

* **iperf3:** geração de tráfego UDP e medição de vazão, jitter e perda de datagramas;
* **nr-binder e ping:** medição do tempo de ida e volta (RTT) dos pacotes ICMP;
* **Arquivos de saída:** armazenamento dos resultados para análise posterior.

O objetivo é caracterizar o comportamento do cenário de três UEs na mesma slice. A existência de congestionamento deverá ser avaliada a partir dos resultados, e não presumida apenas pela configuração dos dispositivos.

---

## 2. Preparação do Ambiente

Antes de iniciar os testes, confirme que:

* O Open5GS está em execução.
* O gNodeB está conectado ao AMF.
* Os três UEs estão registrados e possuem sessões PDU ativas.
* Os endereços IP dos UEs foram identificados.
* O servidor de testes está acessível pelo endereço `10.45.0.1`.
* Os executáveis `iperf3` e `build/nr-binder` estão disponíveis.

Neste experimento, os endereços observados foram:

| UE   | Interface TUN | Endereço IPv4 |
| ---- | ------------- | ------------- |
| UE 1 | `uesimtun0`   | `10.45.0.9`   |
| UE 2 | `uesimtun1`   | `10.45.0.10`  |
| UE 3 | `uesimtun2`   | `10.45.0.11`  |

Confirme esses valores antes de executar os comandos, pois os endereços podem mudar entre execuções.

Acesse a pasta do UERANSIM:

```bash
cd ~/UERANSIM
```

Os comandos abaixo pressupõem que os testes e os arquivos de saída serão executados nesse diretório.

---

## 3. Inicialização dos Servidores iperf3

No terminal da máquina que hospeda o servidor acessível pelo endereço `10.45.0.1`, execute:

```bash
iperf3 -s -B 10.45.0.1 -p 5201 &
iperf3 -s -B 10.45.0.1 -p 5202 &
iperf3 -s -B 10.45.0.1 -p 5203 &
```

Cada processo disponibiliza uma porta de controle para um dos clientes:

| UE   | Porta | Protocolo do teste |
| ---- | ----: | ------------------ |
| UE 1 |  5201 | UDP                |
| UE 2 |  5202 | UDP                |
| UE 3 |  5203 | UDP                |

**Importante:** execute esses comandos na máquina que possui o endereço local `10.45.0.1`. A opção `-B` não cria esse endereço nem encaminha automaticamente os pacotes para ele.

A opção `&` permite iniciar cada processo em segundo plano. Os três servidores precisam permanecer ativos durante os testes.

Para verificar se estão escutando nas portas esperadas, execute:

```bash
sudo ss -lntp | grep -E ':5201|:5202|:5203'
```

Se necessário, confira também as mensagens de inicialização dos servidores e a disponibilidade das portas UDP.

---

## 4. Execução dos Testes UDP Simultâneos

No terminal da máquina que executa os UEs, dentro da pasta `UERANSIM`, execute:

```bash
iperf3 -c 10.45.0.1 -p 5201 -B 10.45.0.9 -u -b 1000M -t 30 > resultado_ue1_mesmo_slice.txt 2>&1 &
iperf3 -c 10.45.0.1 -p 5202 -B 10.45.0.10 -u -b 20M -t 30 > resultado_ue2_mesmo_slice.txt 2>&1 &
iperf3 -c 10.45.0.1 -p 5203 -B 10.45.0.11 -u -b 1M -t 30 > resultado_ue3_mesmo_slice.txt 2>&1 &
wait
```

### Parâmetros utilizados

| Parâmetro      | Função                                                       |
| -------------- | ------------------------------------------------------------ |
| `-c 10.45.0.1` | Define o endereço do servidor                                |
| `-p`           | Seleciona a porta do servidor                                |
| `-B`           | Associa o cliente ao endereço IP local especificado          |
| `-u`           | Seleciona UDP                                                |
| `-b`           | Define a taxa-alvo de transmissão                            |
| `-t 30`        | Define a duração de cada teste em 30 segundos                |
| `>`            | Redireciona a saída para um arquivo                          |
| `2>&1`         | Redireciona também as mensagens de erro                      |
| `&`            | Executa cada cliente em segundo plano                        |
| `wait`         | Aguarda a conclusão dos processos iniciados em segundo plano |

### Taxas-alvo configuradas

| UE   |   Taxa-alvo | Duração |
| ---- | ----------: | ------: |
| UE 1 | 1000 Mbit/s |    30 s |
| UE 2 |   20 Mbit/s |    30 s |
| UE 3 |    1 Mbit/s |    30 s |

A taxa de `1000M` representa uma solicitação de envio de até 1 Gbit/s pelo cliente UDP; não significa que essa vazão será efetivamente alcançada.

A taxa agregada solicitada é de aproximadamente 1021 Mbit/s. Se a capacidade disponível for inferior à carga oferecida, poderão ocorrer perdas de pacotes ou redução da vazão recebida. O teste deve ser dimensionado considerando também a capacidade do computador, do plano de usuário e do servidor.

---

## 5. Medição de Latência com nr-binder e ping

A medição de latência é executada separadamente dos testes de vazão. Não é necessário iniciar os servidores `iperf3` para realizar o `ping`.

Com os UEs conectados e as sessões PDU ativas, execute na pasta `UERANSIM`:

```bash
./build/nr-binder 10.45.0.9 ping -c 30 10.45.0.1 > latencia_ue1_mesmo_slice.txt 2>&1 &
./build/nr-binder 10.45.0.10 ping -c 30 10.45.0.1 > latencia_ue2_mesmo_slice.txt 2>&1 &
./build/nr-binder 10.45.0.11 ping -c 30 10.45.0.1 > latencia_ue3_mesmo_slice.txt 2>&1 &
wait
```

O `nr-binder` permite associar a aplicação ao endereço IP da sessão PDU correspondente. O recurso depende de a configuração automática de roteamento do UERANSIM estar habilitada.

Cada comando envia 30 solicitações ICMP ao destino `10.45.0.1`. A duração total pode variar em função do intervalo entre pacotes e de eventuais timeouts.

### O que será medido?

O `ping` apresenta o RTT (*Round-Trip Time*), que representa o tempo entre o envio de uma solicitação e a recepção da resposta correspondente.

Ao contrário da vazão, o RTT não é uma medição unidirecional. Ele inclui os atrasos dos dois sentidos do percurso e o processamento nos equipamentos envolvidos.

Os arquivos serão nomeados da seguinte forma:

* `latencia_ue1_mesmo_slice.txt`
* `latencia_ue2_mesmo_slice.txt`
* `latencia_ue3_mesmo_slice.txt`

Para consultar as estatísticas finais:

```bash
tail -n 4 latencia_ue1_mesmo_slice.txt
tail -n 4 latencia_ue2_mesmo_slice.txt
tail -n 4 latencia_ue3_mesmo_slice.txt
```

A linha final do `ping` normalmente apresenta os valores mínimo, médio, máximo e desvio médio do RTT, além das estatísticas de pacotes transmitidos e recebidos.

**Nota metodológica:** para avaliar o efeito da carga sobre a latência, o ideal é comparar medições realizadas antes, durante e depois da transmissão UDP. Medições realizadas somente antes do teste não demonstram, por si só, como a latência se comportou sob carga.

---

## 6. Métricas Coletadas

### 6.1. Vazão (Bitrate)

A vazão representa a taxa de dados efetivamente transmitida ou recebida, conforme a linha do relatório do `iperf3`.

Para o teste UDP, a taxa configurada em `-b` é a taxa-alvo do emissor. A vazão recebida pode ser menor devido a limitações de capacidade, processamento ou perdas.

### 6.2. Jitter

O jitter apresentado pelo `iperf3` representa a variação temporal entre a chegada dos datagramas UDP.

Valores maiores podem indicar maior variabilidade no espaçamento de chegada dos pacotes. A interpretação deve considerar a carga oferecida e as condições de execução.

### 6.3. Perda de pacotes

A coluna `Lost/Total Datagrams` apresenta o número de datagramas perdidos em relação ao total esperado no relatório.

A porcentagem de perda deve ser registrada para cada UE, permitindo comparar o comportamento dos dispositivos submetidos a taxas-alvo diferentes.

### 6.4. Latência (RTT)

A latência é obtida separadamente com `ping`, sendo expressa em milissegundos.

O RTT medido pelo ICMP não equivale necessariamente ao atraso de uma aplicação real nem ao atraso unidirecional da rede.

---

## 7. Consulta dos Resultados

Após a conclusão dos testes, confirme se os arquivos foram gerados:

```bash
ls -lh resultado_ue*_mesmo_slice.txt latencia_ue*_mesmo_slice.txt
```

Para consultar os relatórios UDP:

```bash
cat resultado_ue1_mesmo_slice.txt
cat resultado_ue2_mesmo_slice.txt
cat resultado_ue3_mesmo_slice.txt
```

Para consultar os relatórios de latência:

```bash
tail -n 4 latencia_ue1_mesmo_slice.txt
tail -n 4 latencia_ue2_mesmo_slice.txt
tail -n 4 latencia_ue3_mesmo_slice.txt
```

Os arquivos devem ser preservados para a análise comparativa posterior.

Se algum relatório apresentar erro de conexão, ausência de respostas ou perda total de pacotes, o resultado deve ser tratado como uma execução com falha, e não como uma medição válida de desempenho.

---

## 8. Critérios de Validação

* [ ] Os três servidores `iperf3` estão acessíveis nas portas configuradas.
* [ ] Os três clientes UDP iniciaram utilizando os endereços IP corretos.
* [ ] Os testes foram executados durante 30 segundos.
* [ ] Os relatórios de vazão, jitter e perda foram gerados.
* [ ] Os três testes de latência foram concluídos.
* [ ] Os relatórios de RTT foram salvos.
* [ ] As taxas configuradas e os resultados efetivamente obtidos foram registrados.
* [ ] Eventuais erros, timeouts e perdas foram identificados.

---

## 9. Limitações e Interpretação

Este experimento caracteriza o comportamento de três UEs configurados para utilizar a mesma slice eMBB (SST 1), submetidos simultaneamente a diferentes taxas-alvo de tráfego UDP.

**É importante distinguir o compartilhamento da mesma slice da ocorrência efetiva de congestionamento.** A configuração dos três UEs na mesma fatia lógica não comprova, por si só, a saturação dos recursos de rede, a ausência de isolamento entre fatias ou a existência de contenção entre os dispositivos. Essas conclusões dependem dos resultados obtidos e de sua comparação com cenários experimentais adicionais.

Além disso, os UEs utilizam taxas-alvo distintas. Portanto, diferenças de vazão, jitter e perda de pacotes devem ser interpretadas considerando a carga solicitada individualmente a cada dispositivo, e não apenas sua participação na mesma slice.

**Limitação relacionada à latência:** os testes de conectividade realizados até o momento identificaram dificuldades na comunicação ICMP entre as interfaces dos UEs e o endereço `10.45.0.1` da interface `ogstun`. Como os componentes do laboratório são executados no mesmo host, as rotas locais e as regras de roteamento do Linux podem interferir no caminho percorrido pelos pacotes. Assim, as medições de latência só deverão ser consideradas válidas após a confirmação da conectividade e do percurso efetivo do tráfego.

Por fim, os resultados deste cenário devem ser considerados uma caracterização inicial do ambiente. A avaliação do impacto do compartilhamento de recursos e do isolamento proporcionado pelas Network Slices exigirá a comparação com outros cenários, especialmente aquele em que os UEs utilizam slices distintas, mantendo-se as condições de teste tão consistentes quanto possível.

