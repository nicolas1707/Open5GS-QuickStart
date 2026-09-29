# 8. Inicialização e Validação do Ambiente

## Objetivo

Após a configuração do **Open5GS**, do **gNodeB** e do **UE**, esta etapa consiste em iniciar todos os componentes da rede e verificar se o processo completo de registro (Registration) ocorre corretamente.

Ao final desta etapa, espera-se que:

- o gNodeB estabeleça a interface **N2** com o AMF;
- o UE realize o procedimento de autenticação **5G-AKA**;
- sejam criadas as **PDU Sessions** configuradas;
- o UE receba um endereço IP fornecido pelo núcleo 5G.

Esta é a primeira etapa em que todos os componentes da arquitetura passam a interagir simultaneamente. 

---

# Ordem de Inicialização

Os componentes devem ser iniciados na seguinte ordem:

```text
1. Open5GS
        │
        ▼
2. gNodeB (UERANSIM)
        │
        ▼
3. UE (UERANSIM)
```

Essa sequência garante que:

- o AMF esteja aguardando conexões NGAP;
- o gNodeB consiga realizar o procedimento **NG Setup**;
- o UE encontre uma célula disponível para iniciar o registro.

---

# 1. Verificação dos Serviços do Open5GS

Antes de iniciar o UERANSIM, confirme que os principais serviços do Open5GS estão ativos.

```bash
systemctl status open5gs-amfd
```

### O que este comando faz?

Verifica o estado do **AMF (Access and Mobility Management Function)**.

### Impacto no processo

O AMF é responsável pelo registro do UE.

Caso ele não esteja em execução, nenhuma conexão poderá ser estabelecida.

---

Também é recomendado verificar o SMF e o UPF.

```bash
systemctl status open5gs-smfd
```

```bash
systemctl status open5gs-upfd
```

### O que estes comandos fazem?

Verificam se:

- o **SMF** está preparado para criar PDU Sessions;
- o **UPF** está preparado para encaminhar o tráfego do usuário.

---

# 2. Inicialização do gNodeB

Abra um terminal.

Entre na pasta do UERANSIM.

```bash
cd ~/UERANSIM
```

Em seguida execute:

```bash
./build/nr-gnb -c config/open5gs-gnb.yaml
```

### O que este comando faz?

Inicializa o simulador do gNodeB utilizando o arquivo de configuração criado anteriormente.

Durante a inicialização o gNodeB:

- lê o arquivo YAML;
- abre uma conexão SCTP;
- conecta-se ao AMF;
- executa o procedimento **NG Setup**.

### Impacto no processo

Após esse momento, o núcleo 5G reconhece que existe uma estação rádio disponível para atender UEs.

---

# 3. Validação do NG Setup

Durante a inicialização do gNodeB, espera-se observar mensagens semelhantes a:

```text
Trying to establish SCTP connection...
```

```text
SCTP connection established
```

```text
NG Setup procedure is successful
```

### O que essas mensagens significam?

| Mensagem | Significado |
|----------|-------------|
| Trying to establish SCTP connection | O gNodeB está tentando conectar-se ao AMF. |
| SCTP connection established | A conexão N2 foi estabelecida com sucesso. |
| NG Setup procedure is successful | O AMF aceitou o registro do gNodeB. |

Caso essas mensagens apareçam, a interface **N2** encontra-se operacional.

---

# 4. Inicialização do UE

Abra um segundo terminal.

Acesse novamente a pasta do UERANSIM.

```bash
cd ~/UERANSIM
```

Execute:

```bash
sudo ./build/nr-ue -c config/open5gs-ue.yaml
```

### O que este comando faz?

Inicializa o simulador do UE utilizando o perfil configurado anteriormente.

Durante sua execução o UE:

- procura um gNodeB disponível;
- seleciona uma célula compatível;
- inicia o procedimento de registro;
- executa a autenticação 5G-AKA;
- solicita a criação das PDU Sessions configuradas.

### Impacto no processo

Este é o momento em que o assinante cadastrado na WebUI efetivamente tenta utilizar a rede 5G.

---

# 5. Validação do Registro

Durante a execução do UE, algumas mensagens importantes devem aparecer.

## Seleção da rede

```text
Selected plmn[999/70]
```

Indica que o UE encontrou a rede configurada.

---

## Seleção da célula

```text
Selected cell plmn[999/70] tac[1]
```

Indica que o gNodeB possui o mesmo:

- MCC;
- MNC;
- TAC.

---

## Registro

```text
Initial Registration is successful
```

Indica que:

- AMF;
- AUSF;
- UDM;
- UDR

executaram corretamente o procedimento de registro.

---

## Segurança

```text
Security Mode Complete
```

Indica que a autenticação 5G-AKA foi concluída.

---

## PDU Session

```text
PDU Session establishment is successful
```

Esta mensagem aparecerá três vezes. Indica que o SMF autorizou a criação simultânea das três sessões de dados solicitadas pelo UE (internet, emergency e iot).



---

# 6. Verificação da Interface TUN

Após a criação da primeira PDU Session, o Linux cria automaticamente uma interface virtual.

Verifique utilizando:

```bash
ip addr
```

ou

```bash
ip link
```

### O que este comando faz?

Lista todas as interfaces de rede presentes no sistema.

### Impacto no processo

Devido à configuração paralela de fatias, devem existir três interfaces semelhantes a:

```text
uesimtun0
uesimtun1
uesimtun2
```

Estas interfaces representam as ligações IP do UE com o núcleo 5G, separadas por APN/Slice.

---

# 7. Verificação do Endereço IP

Verifique os endereços atribuídos a todas as interfaces de túnel:

```bash
ip addr | grep uesimtun
```

### O que este comando faz?

Mostra o endereço IP atribuído ao UE.

### Impacto no processo

Espera-se que cada interface receba um IP fixo atribuído pelo UPF:

```text
uesimtun0 (eMBB): 10.45.0.5

uesimtun1 (URLLC): 10.45.0.6

uesimtun2 (mMTC): 10.45.0.7
```

Esse endereço foi atribuído pelo **SMF**, sendo utilizado posteriormente pelo **UPF** para encaminhar o tráfego da sessão PDU.

---

# Fluxo Completo da Inicialização

```text
                Open5GS
                   │
                   ▼
          AMF escutando NGAP
                   │
                   ▼
         gNodeB inicia SCTP
                   │
                   ▼
          NG Setup Successful
                   │
                   ▼
           UE encontra célula
                   │
                   ▼
         Registration Request
                   │
                   ▼
             Autenticação
           (AUSF + UDM)
                   │
                   ▼
        Registration Accept
                   │
                   ▼
      PDU Session Establishment
                   │
                   ▼
      Interface uesimtun criada
                   │
                   ▼
         UE recebe endereço IP
```

---

# Critérios de Sucesso

A etapa é considerada concluída com sucesso quando todos os itens abaixo forem atendidos:

- Open5GS em execução;
- gNodeB conectado ao AMF (NG Setup Successful);
- UE registrando-se com sucesso;
- Autenticação 5G-AKA concluída;
- PDU Sessions estabelecidas (3 sessões);
- Interfaces `uesimtun0`, `uesimtun1` e `uesimtun2` criadas e ativas no kernel Linux
- Endereços IP estáticos (10.45.0.5, .6 e .7) atribuídos corretamente a cada túnel pelo UPF.

Com esses resultados, o ambiente estará completamente operacional e pronto para os testes de conectividade, captura de tráfego e validação das Network Slices.
