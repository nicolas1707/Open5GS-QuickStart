# Configuração dos UEs no UERANSIM

## 1. Objetivo

Esta etapa documenta a preparação dos arquivos de configuração YAML dos três equipamentos de usuário (*User Equipment* — UEs) no UERANSIM, dando continuidade ao provisionamento realizado anteriormente na WebUI do Open5GS.

O objetivo é criar três arquivos independentes, um para cada UE, garantindo que cada dispositivo utilize sua própria identidade de assinante e solicite uma única sessão PDU IPv4 associada à mesma fatia lógica da rede: **eMBB (SST 1), com DNN `internet`**.

Nesta etapa, serão realizadas somente a criação e a edição dos arquivos de configuração. A inicialização dos UEs e a validação das conexões serão documentadas separadamente.

---

## 2. Fundamentação Teórica

### 2.1. Arquivos YAML do UERANSIM

O UERANSIM utiliza arquivos YAML para definir os parâmetros de cada UE, incluindo identidade, credenciais de autenticação, configuração de fatias e sessões PDU solicitadas.

O parâmetro `supi` identifica o assinante que será utilizado pelo UE. Já o bloco `sessions` define as sessões PDU que o simulador tentará estabelecer após o registro bem-sucedido no Core 5G. <Cite refs={["turn871863search0"]}/>

Neste experimento, cada processo do UERANSIM utilizará um arquivo próprio. Dessa forma, os três dispositivos poderão ser inicializados individualmente, cada um com seu IMSI correspondente.

### 2.2. Configuração da fatia eMBB

Os três arquivos serão configurados com os seguintes parâmetros:

| Parâmetro                | Valor      |
| ------------------------ | ---------- |
| SST                      | `1`        |
| DNN                      | `internet` |
| Tipo de sessão PDU       | `IPv4`     |
| Número de sessões por UE | `1`        |

O campo `configured-nssai` indicará a fatia configurada para o UE, enquanto `default-nssai` definirá a fatia padrão utilizada no contexto de registro. O bloco `sessions` especificará a sessão PDU que será solicitada.

**Nota técnica:** a configuração do UE deve ser compatível com as fatias autorizadas pelo Open5GS e suportadas pelo gNodeB. A configuração dos arquivos, isoladamente, não comprova o estabelecimento das sessões. <Cite refs={["turn871863search0"]}/>

---

## 3. Criação dos Arquivos YAML

### 3.1. Acessar o diretório do UERANSIM

Abra um terminal e acesse o diretório em que o UERANSIM foi instalado.

Caso o projeto esteja localizado no diretório pessoal do usuário, execute:

```bash
cd ~/UERANSIM
```

O comando `cd` altera o diretório de trabalho do terminal para a pasta do projeto. Se o UERANSIM estiver instalado em outro local, ajuste o caminho conforme o ambiente utilizado.

### 3.2. Criar os três arquivos de configuração

Execute os seguintes comandos:

```bash
cp config/open5gs-ue.yaml config/ue1-sst1.yaml
cp config/open5gs-ue.yaml config/ue2-sst1.yaml
cp config/open5gs-ue.yaml config/ue3-sst1.yaml
```

**Explicação dos comandos:**

* `cp`: copia um arquivo para outro caminho.
* `config/open5gs-ue.yaml`: arquivo-base utilizado como referência.
* `config/ue1-sst1.yaml`, `config/ue2-sst1.yaml` e `config/ue3-sst1.yaml`: arquivos independentes que serão utilizados pelos três UEs.

A cópia do arquivo-base preserva os parâmetros comuns do ambiente, como MCC, MNC, endereço do gNodeB, algoritmos de segurança e demais configurações já estabelecidas.

Cada arquivo deverá ter seu `supi` e seus blocos de configuração de fatias e sessões revisados individualmente.

### 3.3. Verificar a criação dos arquivos

Execute:

```bash
ls config/
```

O comando `ls` lista os arquivos e diretórios presentes na pasta `config`.

Confirme que os seguintes arquivos estão presentes:

```text
ue1-sst1.yaml
ue2-sst1.yaml
ue3-sst1.yaml
```

---

## 4. Configuração do UE 1

### 4.1. Abrir o arquivo

Execute:

```bash
nano config/ue1-sst1.yaml
```

O editor `nano` permite modificar o arquivo diretamente pelo terminal.

### 4.2. Configurar a identidade do assinante

No início do arquivo, ajuste o parâmetro `supi` para:

```yaml
supi: 'imsi-999700000000001'
```

Esse valor identifica o UE 1 e deve corresponder ao assinante cadastrado na WebUI do Open5GS.

Mantenha os parâmetros `mcc` e `mnc` compatíveis com o ambiente:

```yaml
mcc: '999'
mnc: '70'
```

Confirme também que os parâmetros de autenticação `key`, `op` e `opType` correspondem aos valores configurados no perfil do assinante.

### 4.3. Configurar a fatia e a sessão PDU

Localize os blocos `configured-nssai`, `default-nssai` e `sessions`.

Remova as entradas referentes aos SST 2 e SST 3, assim como as sessões associadas aos DNNs `emergency` e `iot`.

A configuração final dessa seção deverá ser:

```yaml
# Fatia configurada para o UE
configured-nssai:
  - sst: 1

# Fatia padrão para o registro
default-nssai:
  - sst: 1

# Sessão PDU inicial
sessions:
  - type: 'IPv4'
    apn: 'internet'
    slice:
      sst: 1
```

O bloco `sessions` contém apenas uma sessão PDU IPv4, associada à fatia SST 1 e ao DNN `internet`.

Salve o arquivo no `nano` utilizando `Ctrl + O`, confirme com `Enter` e saia com `Ctrl + X`.

---

## 5. Configuração do UE 2

### 5.1. Abrir o arquivo

```bash
nano config/ue2-sst1.yaml
```

### 5.2. Ajustar a identidade

Altere o parâmetro `supi` para:

```yaml
supi: 'imsi-999700000000002'
```

Mantenha os parâmetros `mcc`, `mnc`, `key`, `op` e `opType` compatíveis com o assinante correspondente cadastrado no Open5GS.

### 5.3. Configurar a fatia e a sessão

Assim como no UE 1, mantenha somente o SST 1 em `configured-nssai` e `default-nssai`.

O bloco `sessions` deverá ser:

```yaml
configured-nssai:
  - sst: 1

default-nssai:
  - sst: 1

sessions:
  - type: 'IPv4'
    apn: 'internet'
    slice:
      sst: 1
```

Salve as alterações e saia do editor.

---

## 6. Configuração do UE 3

### 6.1. Abrir o arquivo

```bash
nano config/ue3-sst1.yaml
```

### 6.2. Ajustar a identidade

Altere o parâmetro `supi` para:

```yaml
supi: 'imsi-999700000000003'
```

Mantenha os demais parâmetros de identidade e autenticação compatíveis com o terceiro assinante cadastrado no Open5GS.

### 6.3. Configurar a fatia e a sessão

Mantenha somente a configuração da fatia SST 1 e uma sessão PDU IPv4 associada ao DNN `internet`:

```yaml
configured-nssai:
  - sst: 1

default-nssai:
  - sst: 1

sessions:
  - type: 'IPv4'
    apn: 'internet'
    slice:
      sst: 1
```

Salve as alterações e saia do editor.

---

## 7. Verificação dos Arquivos

Após editar os três arquivos, execute:

```bash
grep -nE "supi:|mcc:|mnc:|configured-nssai:|default-nssai:|sessions:|apn:|sst:" \
config/ue1-sst1.yaml config/ue2-sst1.yaml config/ue3-sst1.yaml
```

O comando `grep` procura os parâmetros indicados e mostra os números das linhas em que aparecem. Isso facilita conferir as identidades e os valores de SST e DNN sem precisar abrir os três arquivos novamente.

Verifique se:

* O UE 1 utiliza o IMSI `999700000000001`.
* O UE 2 utiliza o IMSI `999700000000002`.
* O UE 3 utiliza o IMSI `999700000000003`.
* Os três arquivos apresentam somente o SST 1 nos blocos de fatias.
* Cada arquivo contém somente uma sessão PDU associada ao DNN `internet`.
* Os parâmetros de autenticação correspondem aos respectivos perfis cadastrados no Open5GS.

A saída do comando deve ser analisada para confirmar que não restaram configurações antigas de SST 2, SST 3, `emergency` ou `iot` nos blocos relevantes.

---

## 8. Configuração Final Esperada

| Arquivo         | SUPI                   | SST | DNN        | Sessões PDU configuradas |
| --------------- | ---------------------- | --: | ---------- | -----------------------: |
| `ue1-sst1.yaml` | `imsi-999700000000001` |   1 | `internet` |                        1 |
| `ue2-sst1.yaml` | `imsi-999700000000002` |   1 | `internet` |                        1 |
| `ue3-sst1.yaml` | `imsi-999700000000003` |   1 | `internet` |                        1 |

Representação simplificada:

```text
                   UERANSIM
                      |
          +-----------+-----------+
          |           |           |
         UE 1        UE 2        UE 3
          |           |           |
       IMSI ...001  IMSI ...002  IMSI ...003
          |           |           |
          +-----------+-----------+
                      |
                SST 1 / eMBB
                DNN: internet
```

Os três arquivos ficam preparados para solicitar sessões PDU na mesma configuração de fatia. O estabelecimento efetivo das sessões ainda depende da inicialização dos processos, da autenticação e da aceitação das solicitações pelo Core 5G.

---

## 9. Critérios de Conclusão

* [ ] Os três arquivos YAML foram criados no diretório `config`.
* [ ] Cada arquivo possui o IMSI correspondente ao assinante cadastrado no Open5GS.
* [ ] Os parâmetros de autenticação estão consistentes com os perfis dos assinantes.
* [ ] `configured-nssai` contém somente o SST 1.
* [ ] `default-nssai` contém somente o SST 1.
* [ ] `sessions` contém somente uma sessão IPv4 para o DNN `internet`.
* [ ] Não existem sessões residuais para os DNNs `emergency` ou `iot` nesses arquivos.
* [ ] A verificação dos arquivos foi concluída.

---

## 10. Próxima Etapa

Após concluir e conferir os arquivos YAML, a próxima etapa será iniciar os três processos de UE no UERANSIM, utilizando um arquivo de configuração para cada dispositivo.

Durante essa execução, serão verificados o registro dos UEs no Core 5G, o estabelecimento das sessões PDU e a criação das interfaces TUN correspondentes.

Somente depois de validar a conectividade dos três dispositivos será iniciado o teste de tráfego concorrente para investigar o comportamento da fatia eMBB compartilhada.

