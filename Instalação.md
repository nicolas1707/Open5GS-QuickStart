# Instalação do Open5GS via Gerenciador de Pacotes

## Objetivo

Nesta etapa foi realizada a instalação do Open5GS utilizando o gerenciador de pacotes do Ubuntu 22.04 (Jammy Jellyfish).

Antes da instalação do Open5GS, foi necessário instalar e configurar o MongoDB, que é utilizado como banco de dados do núcleo 5G.

O MongoDB armazena:

- Assinantes (Subscribers)
- Dados de autenticação
- Configurações da rede
- Informações de slices
- Políticas e perfis dos usuários

---

# Instalação do MongoDB

## Atualização da lista de pacotes

```bash
sudo apt update
```

### O que este comando faz?

Atualiza a lista de pacotes disponíveis nos repositórios configurados no sistema.

### Impacto no processo

Garante que o Ubuntu conheça as versões mais recentes dos pacotes antes da instalação. A atualização deve ser feita sempre antes de instalar um novo software ou antes de atualizar o sistema

---

## Instalação do GnuPG

```bash
sudo apt install gnupg
```

### O que este comando faz?

Instala o GnuPG (GNU Privacy Guard), utilizado para validar assinaturas digitais de pacotes.

### Impacto no processo

Permite verificar a autenticidade dos pacotes disponibilizados pelo MongoDB.

---

## Importação da chave pública do MongoDB

```bash
curl -fsSL https://pgp.mongodb.com/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
```

### O que este comando faz?

Realiza o download da chave pública utilizada para assinar os pacotes do MongoDB e converte a chave para o formato utilizado pelo APT.

### Impacto no processo

Permite que o Ubuntu confirme que os pacotes realmente foram fornecidos pelo projeto MongoDB e não foram alterados por terceiros.

---

## Adição do repositório oficial do MongoDB

```bash
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```

### O que este comando faz?

Cria um arquivo contendo o endereço do repositório oficial do MongoDB para o Ubuntu 22.04.

### Impacto no processo

Permite que o gerenciador de pacotes encontre e instale o MongoDB diretamente da fonte oficial.

---

## Atualização da lista de pacotes

```bash
sudo apt update
```

### O que este comando faz?

Atualiza novamente os índices dos repositórios.

### Impacto no processo

Agora o sistema passa a enxergar os pacotes disponibilizados pelo repositório MongoDB recém-adicionado.

---

## Instalação do MongoDB

```bash
sudo apt install -y mongodb-org
```

### O que este comando faz?

Instala o servidor MongoDB e seus componentes.

### Impacto no processo

O MongoDB será utilizado pelo Open5GS para armazenar permanentemente os dados da rede e dos assinantes.

Sem ele, funções como UDM e UDR não conseguem persistir informações.

---

## Inicialização do MongoDB

```bash
sudo systemctl start mongod
```

### O que este comando faz?

Inicia o serviço MongoDB.

### Impacto no processo

Disponibiliza o banco de dados para acesso pelo Open5GS. O `systemctl` a ferramenta de linha de comando central utilizada para controlar o systemd, que é o sistema de inicialização e gerenciador de serviços padrão da maioria das distribuições Linux modernas. É ele quem gerencia o que roda em segundo plano.

---

## Inicialização automática durante o boot

```bash
sudo systemctl enable mongod
```

### O que este comando faz?

Configura o MongoDB para iniciar automaticamente quando o sistema operacional for iniciado.

### Impacto no processo

Garante que o banco de dados esteja disponível após reinicializações do servidor.

---

## Verificação do MongoDB

```bash
sudo systemctl status mongod
```

### O que este comando faz?

Exibe o estado atual do serviço MongoDB.

### Impacto no processo

Permite confirmar se o banco de dados está ativo e funcionando corretamente antes da instalação do Open5GS.

---

# Instalação do Open5GS

## Adição do repositório Open5GS

```bash
sudo add-apt-repository ppa:open5gs/latest
```

### O que este comando faz?

Adiciona ao sistema o repositório oficial mantido pelo projeto Open5GS.

### Impacto no processo

Permite instalar e atualizar o Open5GS utilizando o APT.

Sem esse repositório, o Ubuntu não consegue localizar os pacotes do Open5GS.

---

## Atualização da lista de pacotes

```bash
sudo apt update
```

### O que este comando faz?

Atualiza os índices dos repositórios após a adição do PPA do Open5GS.

### Impacto no processo

Permite que o sistema reconheça os pacotes Open5GS disponíveis para instalação.

---

## Instalação do Open5GS

```bash
sudo apt install open5gs
```

### O que este comando faz?

Instala todas as funções do núcleo Open5GS suportadas pelo pacote.

### Impacto no processo

Ao final da instalação estarão disponíveis todas as principais Network Functions (NFs) do 5G Core.

---

# Funções Instaladas

## Plano de Controle (Control Plane)

As funções abaixo são responsáveis pelas decisões da rede:

| Função | Responsabilidade |
|----------|----------|
| NRF | Registro e descoberta de serviços |
| AMF | Controle de acesso e mobilidade |
| AUSF | Autenticação dos usuários |
| UDM | Gerenciamento dos dados dos assinantes |
| UDR | Armazenamento persistente dos dados |
| SMF | Gerenciamento das sessões PDU |
| PCF | Definição de políticas e QoS |
| NSSF | Seleção de Network Slices |

---

## Plano de Usuário (User Plane)

As funções abaixo são responsáveis pelo transporte dos dados:

| Função | Responsabilidade |
|----------|----------|
| UPF | Encaminhamento do tráfego dos usuários |

---

# Estrutura Resultante

Após a instalação, o núcleo 5G possuirá a seguinte organização:

```text
                 CONTROL PLANE

 NRF    AMF    AUSF    UDM    UDR
  │       │       │      │      │
  └───────┴───────┴──────┴──────┘
                 │
                 ▼
               SMF
                 │
                 ▼

                 UPF

                 │
                 ▼
              Internet
```

---

# Verificação dos Serviços

```bash
systemctl | grep open5gs
```

### O que este comando faz?

Lista todos os serviços Open5GS registrados no systemd.

### Impacto no processo

Permite visualizar rapidamente quais funções foram instaladas e seu estado atual.

---

# Problemas de conexão pós instalação

Caso haja algum problema de conexão pós instalação, isso pode ser facilmente resolvível reiniciando o Open5GS.

Comando para desativar o Open5GS:

```bash
sudo systemctl stop open5gs-*
```

Comando para ativar o Open5GS:

```bash
sudo systemctl start open5gs-*
```

### O que este comando faz?

Lista todos os serviços Open5GS registrados no systemd.

### Impacto no processo

Permite visualizar rapidamente quais funções foram instaladas e seu estado atual.

---

# Resultado da Etapa

Ao final desta etapa temos:

- MongoDB instalado e operacional;
- Open5GS instalado via APT;
- Funções do 5G Core disponíveis como serviços systemd;
- Estrutura preparada para a instalação da WebUI;
- Ambiente pronto para a configuração do núcleo 5G SA.
