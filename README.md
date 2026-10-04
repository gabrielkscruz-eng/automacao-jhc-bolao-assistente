# Assistente Virtual JHC Bolão

Assistente virtual para atendimento automatizado de clientes via WhatsApp, desenvolvido com **n8n**, **WAHA**, **Google Gemini**, **Google Sheets** e **Mercado Pago**.

O projeto utiliza uma arquitetura baseada em workflows independentes no n8n. O workflow principal interpreta as mensagens dos usuários, mantém o contexto da conversa e utiliza workflows auxiliares para consultar jogos, gerar pagamentos, verificar pagamentos, atualizar informações e encaminhar solicitações para atendimento humano.

---

# Visão geral

O sistema foi desenvolvido para automatizar o atendimento de clientes de um bolão, permitindo que o usuário interaja pelo WhatsApp sem a necessidade de um atendente humano para as operações mais comuns.

Entre as principais funções estão:

- Atendimento automatizado via WhatsApp
- Interpretação de mensagens utilizando Google Gemini
- Memória de conversação por usuário
- Cadastro e recuperação de informações do cliente
- Consulta de jogos ativos
- Consulta de cotas disponíveis
- Seleção de jogos e quantidade de cotas
- Cálculo automático dos valores
- Confirmação do pedido antes do pagamento
- Geração de link de pagamento
- Integração com Mercado Pago
- Recebimento de notificações de pagamento via webhook
- Validação de pagamentos aprovados
- Atualização das planilhas de controle
- Desativação automática de jogos expirados
- Encaminhamento de solicitações para atendimento humano

---

# Arquitetura

```text
                         ┌─────────────────────┐
                         │      WhatsApp       │
                         │        WAHA         │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │        n8n          │
                         │    AI Principal     │
                         └──────────┬──────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
              ▼                     ▼                     ▼
     ┌────────────────┐    ┌────────────────┐    ┌────────────────┐
     │ Google Gemini  │    │ Google Sheets  │    │   Workflows    │
     │      IA        │    │    Memória     │    │   auxiliares   │
     └────────────────┘    └────────────────┘    └───────┬────────┘
                                                         │
                              ┌──────────────────────────┼──────────────┐
                              │                          │              │
                              ▼                          ▼              ▼
                     ┌─────────────────┐       ┌────────────────┐ ┌──────────────┐
                     │ Jogos ativos    │       │ Pagamentos     │ │ Atendimento │
                     │ Ver_jogos_ativos│       │ Mercado Pago   │ │    humano    │
                     └─────────────────┘       └───────┬────────┘ └──────────────┘
                                                       │
                                                       ▼
                                               ┌─────────────────┐
                                               │    Verifica     │
                                               │    pagamento    │
                                               └─────────────────┘
```

O acesso externo aos serviços pode ser realizado através do **Cloudflare Tunnel**.

---

# Componentes

## n8n

O **n8n** é o núcleo de automação do projeto.

Ele é responsável por:

- Receber mensagens
- Executar a lógica dos workflows
- Chamar a inteligência artificial
- Consultar e atualizar planilhas
- Executar workflows auxiliares
- Receber webhooks
- Integrar serviços externos
- Controlar o fluxo de atendimento
- Processar pagamentos e suas confirmações

O projeto utiliza workflows separados para manter as responsabilidades organizadas.

---

# WAHA

O **WAHA (WhatsApp HTTP API)** é utilizado como camada de comunicação com o WhatsApp.

O fluxo geral é:

```text
WhatsApp
   │
   ▼
 WAHA
   │
   ▼
Webhook n8n
   │
   ▼
AI Principal
```

O WAHA recebe os eventos provenientes do WhatsApp e envia as informações para o n8n através de webhook.

O n8n processa a mensagem e, posteriormente, utiliza o WAHA para enviar a resposta ao usuário.

---

# Google Gemini

O workflow principal utiliza o Google Gemini como mecanismo de interpretação das mensagens.

O Gemini atua na interpretação e decisão da conversa.

A IA recebe o contexto da conversa e pode utilizar as ferramentas disponibilizadas pelo workflow para executar operações específicas.

A estrutura do workflow utiliza componentes como:

- Google Gemini Chat Model
- Memory Buffer
- Structured Output Parser
- AI Agent

A memória permite que a conversa mantenha contexto entre diferentes mensagens do mesmo usuário.

---

# Google Sheets

O Google Sheets funciona como uma camada de armazenamento e controle dos dados utilizados pelo sistema.

As planilhas são utilizadas para armazenar informações relacionadas a:

- Jogos
- Participantes
- Cotas
- Memória da IA
- Pagamentos
- Mensagens em espera
- Atendimento humano

Os identificadores das planilhas utilizados na instalação original não devem ser considerados parte da configuração pública do projeto.

Ao instalar o projeto em outro ambiente, as referências devem ser substituídas pelas próprias planilhas.

---

# Mercado Pago

O Mercado Pago é utilizado para processar os pagamentos realizados pelos clientes.

A integração é utilizada para:

- Criar cobranças
- Gerar links de pagamento
- Consultar pagamentos
- Receber notificações
- Validar pagamentos
- Confirmar o status da transação

As credenciais do Mercado Pago devem ser configuradas diretamente no n8n.

O token de acesso não deve ser armazenado diretamente nos arquivos JSON publicados no GitHub.

---

# Workflows

O projeto é composto pelos seguintes workflows:

```text
workflows/
│
├── AI principal.json
├── AI_gerar_link_pagamento.json
├── Verifica pagamento.json
├── Ver_jogos_ativos.json
├── Atualiza as tabelas.json
└── Chamar_jorge.json
```

Cada workflow possui uma responsabilidade específica dentro do sistema.

---

# 1. AI Principal

Arquivo:

```text
AI principal.json
```

É o workflow central do sistema.

Sua função é receber e interpretar as mensagens dos usuários e decidir qual operação deve ser executada.

## Principais responsabilidades

- Receber mensagens
- Identificar o usuário através da sessão
- Armazenar mensagens
- Recuperar memória
- Utilizar o Gemini
- Interpretar solicitações
- Consultar cadastro
- Consultar jogos
- Iniciar pedidos
- Solicitar confirmação
- Iniciar pagamentos
- Consultar jogos ativos
- Encaminhar solicitações para atendimento humano

A identificação da sessão utiliza o identificador do chat recebido pelo WhatsApp.

A memória utiliza uma chave de sessão, permitindo que diferentes usuários mantenham contextos separados.

---

# Memória da IA

A memória do assistente é armazenada no Google Sheets.

A estrutura utilizada permite registrar informações como:

```text
user_id
date
message
```

Esses dados permitem manter o contexto necessário para o funcionamento da conversa.

O workflow também utiliza uma estrutura de mensagens em espera para controlar o processamento das mensagens recebidas.

Essa lógica permite que mensagens de uma mesma sessão sejam processadas de maneira organizada em vez de tratar cada mensagem completamente isolada.

---

# 2. Ver Jogos Ativos

Arquivo:

```text
Ver_jogos_ativos.json
```

Este workflow consulta os jogos disponíveis e verifica quais jogos estão ativos para determinado usuário.

O workflow recebe informações como:

```text
Nome
numero
```

Primeiramente, consulta a tabela central de jogos.

Depois utiliza o `link_jogos` de cada jogo para acessar sua respectiva planilha e procurar o usuário através da coluna:

```text
CELULAR
```

O resultado pode separar informações como:

```text
nome_jogo
cotas_no_jogo
```

Dessa maneira, a IA consegue informar ao cliente quais jogos ativos possuem participação daquele usuário.

---

# 3. Atualiza as Tabelas

Arquivo:

```text
Atualiza as tabelas.json
```

Este workflow executa uma rotina automática para atualizar o status dos jogos.

A execução é programada para ocorrer em um horário definido na configuração do workflow.

O processo consulta as datas de expiração dos jogos e compara a data atual com:

```text
data_expiração
```

Quando um jogo expira, seu status é atualizado para:

```text
ativo = false
```

O processo percorre os jogos utilizando um loop.

## Fluxo

```text
Trigger
   │
   ▼
Ler tabela de jogos
   │
   ▼
Separar nome/data
   │
   ▼
Loop nos jogos
   │
   ▼
Verificar data de expiração
   │
   ├── Não expirou
   │       │
   │       └── Próximo jogo
   │
   └── Expirou
           │
           ▼
       ativo = false
```

Isso evita que jogos expirados continuem sendo oferecidos pelo assistente.

---

# 4. AI Gerar Link de Pagamento

Arquivo:

```text
AI_gerar_link_pagamento.json
```

Este workflow é responsável por preparar os dados necessários para um pedido de pagamento e realizar a integração com o Mercado Pago.

Ele recebe informações relacionadas ao cliente e aos jogos selecionados.

Entre os dados utilizados estão:

```text
nome
numero
jogos
id_chat
```

O campo `jogos` é recebido como uma estrutura contendo os jogos selecionados.

Cada jogo pode conter informações como:

```json
{
  "nome_jogo": "Nome do jogo",
  "quantidade_cotas": 1,
  "valor_unitario": 10,
  "subtotal": 10,
  "link_tabela": "LINK_DA_PLANILHA",
  "meio_pagamento": "pix"
}
```

## Cálculo

Para cada jogo é calculado:

```text
subtotal = valor_unitario × quantidade_cotas
```

Depois os subtotais são somados:

```text
total_geral = soma dos subtotais
```

Também é criado um identificador interno para o pedido.

Esse identificador permite relacionar posteriormente o pagamento ao pedido original.

---

# Fluxo de pagamento

O processo de pagamento ocorre da seguinte maneira:

```text
Cliente
   │
   ▼
AI Principal
   │
   ▼
Seleção dos jogos
   │
   ▼
Cálculo das cotas
   │
   ▼
Confirmação do pedido
   │
   ▼
AI Gerar Link de Pagamento
   │
   ▼
Mercado Pago
   │
   ▼
Link / cobrança
   │
   ▼
Cliente realiza pagamento
```

O sistema utiliza uma referência interna para relacionar o pagamento ao pedido original.

---

# 5. Verifica Pagamento

Arquivo:

```text
Verifica pagamento.json
```

Este workflow é responsável por receber as notificações do Mercado Pago e verificar se o pagamento realmente corresponde ao pedido criado.

O processo utiliza um webhook para receber a notificação.

Depois da notificação, o workflow consulta diretamente a API do Mercado Pago para obter os dados completos da transação.

---

# Validação do pagamento

O sistema não considera qualquer notificação recebida como pagamento válido.

O workflow realiza verificações sobre os dados da transação.

Entre os dados utilizados estão:

```text
external_reference
valor
currency_id
transaction_amount_refunded
status
status interno do pedido
```

A validação relaciona o pagamento recebido ao pedido criado anteriormente.

De maneira geral, são verificadas condições como:

```text
external_reference == id_interno
valor == valor esperado
currency_id == BRL
reembolso == 0
status Mercado Pago == approved
status interno == Pendente
```

Somente depois que as condições necessárias são atendidas o pagamento pode ser considerado aprovado.

Isso evita que um pedido seja confirmado com:

- Valor incorreto
- Referência incorreta
- Pagamento reembolsado
- Status diferente de aprovado
- Pedido já processado anteriormente

---

# 6. Chamar Jorge

Arquivo:

```text
Chamar_jorge.json
```

Este workflow é utilizado quando uma solicitação precisa ser encaminhada para atendimento humano.

Ele recebe informações relacionadas ao cliente e à conversa.

Entre os dados utilizados estão:

```text
id_chat
nome
```

Depois, o usuário é adicionado a uma lista de espera armazenada em uma planilha.

A estrutura pode conter:

```text
id_chat
nome
```

Isso permite separar as solicitações que podem ser tratadas automaticamente das que precisam de intervenção humana.

---

# Fluxo completo do atendimento

O fluxo principal do sistema pode ser representado da seguinte maneira:

```text
                     ┌───────────────┐
                     │    Cliente    │
                     └───────┬───────┘
                             │
                             ▼
                     ┌───────────────┐
                     │   WhatsApp    │
                     │     WAHA      │
                     └───────┬───────┘
                             │
                             ▼
                     ┌───────────────┐
                     │ AI Principal  │
                     └───────┬───────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
        Consultar       Ver jogos       Falar com
        informações      ativos           Jorge
              │              │              │
              │              │              ▼
              │              │       Lista de espera
              │              │
              └──────┬───────┘
                     │
                     ▼
               Selecionar jogos
                     │
                     ▼
                Calcular pedido
                     │
                     ▼
               Confirmar pedido
                     │
                     ▼
              Gerar pagamento
                     │
                     ▼
                Mercado Pago
                     │
                     ▼
                  Pagamento
                     │
                     ▼
              Webhook pagamento
                     │
                     ▼
              Verifica pagamento
                     │
                     ▼
              Pedido aprovado
```

---

# Fluxo de compra

O processo de compra segue uma sequência controlada.

## 1. Cliente solicita participação

O usuário informa quais jogos deseja participar.

## 2. IA consulta os dados

A IA utiliza as ferramentas disponíveis para obter os dados dos jogos.

## 3. Cliente informa as cotas

O cliente informa a quantidade de cotas desejadas para cada jogo.

Exemplo:

```text
1 com 2 cotas
2 com 1 cota
```

## 4. Sistema calcula os valores

Para cada jogo:

```text
subtotal = quantidade × valor da cota
```

Depois:

```text
total = soma dos subtotais
```

## 5. Cliente confirma

O pedido deve ser confirmado antes da criação do pagamento.

## 6. Sistema gera o pagamento

Após a confirmação, o workflow:

```text
AI_gerar_link_pagamento
```

é executado.

## 7. Cliente realiza o pagamento

O pagamento é processado pelo Mercado Pago.

## 8. Mercado Pago envia a notificação

O Mercado Pago envia uma notificação para o webhook configurado.

## 9. Sistema valida

O workflow:

```text
Verifica pagamento
```

consulta e valida os dados do pagamento.

## 10. Pedido é aprovado

Somente depois das validações necessárias o pedido pode ser considerado confirmado.

---

# Estrutura das planilhas

O sistema utiliza Google Sheets como camada de armazenamento.

## Tabela de jogos

Contém informações relacionadas aos jogos, incluindo campos como:

```text
nome_jogo
valor_cota
ativo
descricao
quant_max
quant_atual
link_jogos
data_expiração
hora_expiração
```

A tabela central é utilizada pelos workflows que precisam consultar os jogos disponíveis.

---

# Memória da IA

A memória da IA utiliza uma estrutura semelhante a:

```text
user_id
date
message
```

O objetivo é permitir que o assistente mantenha o contexto das conversas.

---

# Mensagens em espera

A estrutura de mensagens em espera é utilizada pelo fluxo principal para controlar mensagens pendentes e o processamento das conversas.

Isso permite organizar mensagens recebidas em sequência e evitar que cada mensagem seja tratada como uma conversa completamente nova.

---

# Pagamentos pendentes

A estrutura de pagamentos pode armazenar informações relacionadas ao pedido e à transação.

Entre os dados utilizados estão:

```text
id_interno
id_pagamento
id_chat
nome
numero
jogos
valor
status
dia_criado
dia_expira
```

O objetivo é permitir que o pagamento seja relacionado ao pedido original.

---

# Lista de espera

Utilizada pelo workflow:

```text
Chamar_jorge
```

Estrutura:

```text
nome
id_chat
```

Essa tabela permite identificar os usuários que precisam de atendimento humano.

---

# Docker

A infraestrutura foi projetada para funcionar com Docker Compose.

O ambiente contém três serviços principais:

```text
n8n-bolao
waha-bolao
cloudflared-bolao
```

---

# n8n

Responsável pela execução dos workflows.

O serviço utiliza:

```yaml
image: n8nio/n8n:latest
container_name: n8n-bolao
```

A porta interna utilizada pelo n8n é:

```text
5678
```

A porta externa é definida através da variável:

```text
N8N_PORT_EXTERNAL
```

Os dados persistentes do n8n são armazenados em:

```text
./n8n_data:/home/node/.n8n
```

O timezone utilizado pelo projeto é:

```text
America/Sao_Paulo
```

---

# WAHA

Responsável pela comunicação com o WhatsApp.

O serviço utiliza:

```yaml
image: devlikeapro/waha:latest
container_name: waha-bolao
```

A porta interna utilizada pelo WAHA é:

```text
3000
```

A porta externa é definida através de:

```text
WAHA_PORT_EXTERNAL
```

Os dados persistentes são armazenados em:

```text
./waha_data
./waha_media
```

O WAHA está conectado à mesma rede Docker do n8n.

A comunicação interna utiliza o nome do serviço:

```text
n8n-bolao
```

Isso permite que os containers se comuniquem pela rede Docker.

---

# Cloudflared

O Cloudflared fornece o túnel utilizado para disponibilizar os serviços externamente.

O serviço utiliza:

```yaml
image: cloudflare/cloudflared:latest
container_name: cloudflared-bolao
```

O túnel é iniciado utilizando um token configurado através da variável:

```text
CLOUDFLARE_TUNNEL_TOKEN
```

O token não deve ser colocado diretamente no `docker-compose.yml`.

---

# Docker Compose

O projeto utiliza o seguinte modelo de infraestrutura:

```yaml
networks:
  automacao-jhcBolao:
    driver: bridge

services:
  n8n-bolao:
    image: n8nio/n8n:latest
    container_name: n8n-bolao
    hostname: n8n-bolao
    restart: unless-stopped

    networks:
      - automacao-jhcBolao

    mem_limit: 1500m
    cpus: 1.0

    environment:
      TZ: ${TZ}

      N8N_HOST: ${N8N_HOST}
      N8N_PROTOCOL: ${N8N_PROTOCOL}
      WEBHOOK_URL: ${WEBHOOK_URL}
      N8N_EDITOR_BASE_URL: ${N8N_EDITOR_BASE_URL}
      N8N_PORT: ${N8N_PORT}
      N8N_SECURE_COOKIE: ${N8N_SECURE_COOKIE}

      N8N_BASIC_AUTH_ACTIVE: ${N8N_BASIC_AUTH_ACTIVE}
      N8N_BASIC_AUTH_USER: ${N8N_BASIC_AUTH_USER}
      N8N_BASIC_AUTH_PASSWORD: ${N8N_BASIC_AUTH_PASSWORD}

      N8N_RUNNERS_ENABLED: ${N8N_RUNNERS_ENABLED}
      EXECUTIONS_PROCESS: ${EXECUTIONS_PROCESS}
      EXECUTIONS_DATA_PRUNE: ${EXECUTIONS_DATA_PRUNE}
      EXECUTIONS_DATA_MAX_AGE: ${EXECUTIONS_DATA_MAX_AGE}

      NODE_OPTIONS: ${N8N_NODE_OPTIONS}

    ports:
      - "${N8N_PORT_EXTERNAL}:5678"

    volumes:
      - ./n8n_data:/home/node/.n8n

  waha-bolao:
    image: devlikeapro/waha:latest
    container_name: waha-bolao
    hostname: waha-bolao
    restart: unless-stopped

    networks:
      - automacao-jhcBolao

    mem_limit: 2g
    cpus: 1.2
    shm_size: 512mb

    environment:
      TZ: ${TZ}

      WHATSAPP_DEFAULT_ENGINE: ${WHATSAPP_DEFAULT_ENGINE}

      WHATSAPP_HOOK_URL: ${WHATSAPP_HOOK_URL}
      WHATSAPP_HOOK_EVENTS: ${WHATSAPP_HOOK_EVENTS}

      WHATSAPP_DASHBOARD_PASSWORD: ${WAHA_DASHBOARD_PASSWORD}
      WHATSAPP_API_KEY: ${WAHA_API_KEY}

      NODE_OPTIONS: ${WAHA_NODE_OPTIONS}

    ports:
      - "${WAHA_PORT_EXTERNAL}:3000"

    volumes:
      - ./waha_data:/app/.sessions
      - ./waha_media:/app/.media

    depends_on:
      - n8n-bolao

  cloudflared-bolao:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared-bolao
    hostname: cloudflared-bolao
    restart: unless-stopped

    networks:
      - automacao-jhcBolao

    mem_limit: 256m
    cpus: 0.3

    command: tunnel --no-autoupdate run --token ${CLOUDFLARE_TUNNEL_TOKEN}
```

---

# Variáveis de ambiente

O projeto utiliza um arquivo `.env` para armazenar configurações específicas da instalação e informações sensíveis.

O repositório contém apenas:

```text
.env.example
```

O arquivo real:

```text
.env
```

deve ser criado na máquina onde o projeto será executado.

Exemplo:

```env
TZ=America/Sao_Paulo

N8N_HOST=seu-dominio.com
N8N_PROTOCOL=https

WEBHOOK_URL=https://seu-dominio.com/
N8N_EDITOR_BASE_URL=https://seu-dominio.com/

N8N_PORT=5678
N8N_PORT_EXTERNAL=5678

N8N_SECURE_COOKIE=true

N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=seu_usuario
N8N_BASIC_AUTH_PASSWORD=sua_senha

N8N_RUNNERS_ENABLED=true
EXECUTIONS_PROCESS=main

EXECUTIONS_DATA_PRUNE=true
EXECUTIONS_DATA_MAX_AGE=168

N8N_NODE_OPTIONS=--max-old-space-size=1024

WHATSAPP_DEFAULT_ENGINE=NOWEB

WHATSAPP_HOOK_URL=http://n8n-bolao:5678/webhook/SEU_WEBHOOK_ID/waha
WHATSAPP_HOOK_EVENTS=message

WAHA_DASHBOARD_PASSWORD=sua_senha_waha
WAHA_API_KEY=sua_api_key_waha

WAHA_NODE_OPTIONS=--max-old-space-size=1024

WAHA_PORT_EXTERNAL=3001

CLOUDFLARE_TUNNEL_TOKEN=seu_cloudflare_tunnel_token
```

**O `.env` não deve ser publicado no GitHub.**

---

# Instalação

## Requisitos

Para executar o projeto são necessários:

- Docker
- Docker Compose
- Conta Google
- Google Sheets
- Google Gemini
- Mercado Pago
- Cloudflare
- WAHA
- Domínio público para os webhooks

---

# 1. Baixar o projeto

O projeto pode ser baixado ou clonado do GitHub.

```bash
git clone SEU_REPOSITORIO
cd jhc-bolao-assistente
```

---

# 2. Criar o `.env`

Copie o arquivo:

```text
.env.example
```

para:

```text
.env
```

Depois preencha as variáveis de acordo com o ambiente.

---

# 3. Iniciar os containers

Execute:

```bash
docker compose up -d
```

Verifique os containers:

```bash
docker compose ps
```

Para acompanhar os logs:

```bash
docker compose logs -f
```

Para consultar apenas o n8n:

```bash
docker compose logs -f n8n-bolao
```

Para consultar o WAHA:

```bash
docker compose logs -f waha-bolao
```

Para parar os serviços:

```bash
docker compose down
```

---

# Configuração do n8n

Após iniciar o n8n, acesse o endereço configurado em:

```text
N8N_EDITOR_BASE_URL
```

Depois importe os workflows:

```text
AI principal.json
AI_gerar_link_pagamento.json
Verifica pagamento.json
Ver_jogos_ativos.json
Atualiza as tabelas.json
Chamar_jorge.json
```

Após importar os workflows, configure as credenciais utilizadas pelos nodes.

---

# Credenciais necessárias

## Google

Necessário para:

- Google Sheets
- Google Gemini

As credenciais devem ser configuradas dentro do n8n.

---

## Mercado Pago

Necessário para:

- Criação de pagamentos
- Consulta de pagamentos
- Validação de pagamentos
- Recebimento de notificações

O token deve ser armazenado como uma credencial segura no n8n.

---

## WAHA

Necessário para:

- Receber mensagens
- Enviar mensagens
- Comunicação entre WhatsApp e n8n

A chave e a senha devem ser configuradas através do ambiente.

---

## Cloudflare

Necessário para disponibilizar o serviço através de um túnel.

O token do Cloudflare deve ser armazenado na variável:

```text
CLOUDFLARE_TUNNEL_TOKEN
```

---

# Webhooks

O projeto utiliza webhooks para comunicação entre os serviços.

O Mercado Pago utiliza um endpoint para notificar alterações nos pagamentos.

Exemplo:

```text
/webhook/pagamento_bolao
```

O WAHA também utiliza um webhook para encaminhar eventos do WhatsApp ao n8n.

As URLs dos webhooks devem ser configuradas de acordo com o domínio utilizado na instalação.

---

# Segurança

O projeto trabalha com informações potencialmente sensíveis, incluindo:

- Dados de clientes
- Números de telefone
- Identificadores de chat
- Dados de pagamentos
- Tokens
- Chaves de API
- Credenciais externas
- Informações de webhook

Por isso, os arquivos publicados no GitHub devem ser previamente sanitizados.

## Nunca publicar

```text
.env
Tokens
API Keys
Senhas
Bearer Tokens
Credenciais OAuth
Tokens do Mercado Pago
Tokens do Cloudflare
Chaves do WAHA
Dados pessoais
Dados reais de pagamentos
Capturas reais de webhooks
```

Também devem ser removidos dos workflows quaisquer dados reais presentes em `pinData`, quando esses dados contiverem informações privadas.

---

# Dados persistentes

Os containers utilizam volumes locais para manter seus dados.

```text
./n8n_data
./waha_data
./waha_media
```

Esses diretórios pertencem à instalação local e não fazem parte dos arquivos necessários para disponibilizar o projeto no GitHub.

---

# Fluxo de manutenção dos jogos

Os jogos são armazenados em uma tabela central.

O sistema verifica automaticamente as datas de expiração.

O fluxo funciona de maneira semelhante a:

```text
Trigger
   │
   ▼
Atualiza as tabelas
   │
   ▼
Consulta jogos
   │
   ▼
Verifica data de expiração
   │
   ▼
Jogo expirou?
   │
   ├── NÃO ──► Continua
   │
   └── SIM
         │
         ▼
     ativo = false
```

Isso evita que jogos expirados continuem sendo oferecidos pelo assistente.

---

# Atendimento humano

Quando uma solicitação não deve ser tratada automaticamente, a IA pode utilizar:

```text
Chamar_jorge
```

O workflow registra o usuário em uma lista de espera.

O registro contém informações necessárias para identificar a conversa e permitir que o responsável realize o atendimento.

Dessa maneira, o sistema consegue separar:

```text
Atendimento automático
        │
        ├── Solicitações comuns
        ├── Consulta de jogos
        ├── Seleção de cotas
        └── Pagamentos
```

de:

```text
Atendimento humano
        │
        └── Solicitações que exigem intervenção
```

---

# Princípios da IA

O comportamento do assistente é baseado em algumas regras importantes.

A IA deve:

- Utilizar os dados retornados pelas ferramentas
- Evitar inventar informações
- Manter o contexto da conversa
- Consultar os jogos através das ferramentas disponíveis
- Confirmar pedidos antes de iniciar o pagamento
- Utilizar o workflow específico para geração de pagamento
- Encaminhar solicitações que precisam de atendimento humano
- Retornar estruturas compatíveis com o processamento posterior do n8n

A utilização de workflows auxiliares permite separar a interpretação da conversa das operações específicas do sistema.

---

# Tratamento de pagamentos

O sistema utiliza uma abordagem de validação do pagamento antes de considerar um pedido confirmado.

```text
Pedido criado
     │
     ▼
ID interno gerado
     │
     ▼
Pagamento criado
     │
     ▼
Mercado Pago
     │
     ▼
Webhook
     │
     ▼
Consulta API Mercado Pago
     │
     ▼
Compara dados
     │
     ├── Referência
     ├── Valor
     ├── Moeda
     ├── Reembolso
     ├── Status
     └── Status interno
     │
     ▼
Pagamento aprovado
```

A utilização da referência interna permite relacionar o pagamento recebido ao pedido criado pelo sistema.

---

# Estrutura do repositório

Como o projeto será disponibilizado manualmente no GitHub, a estrutura pública pode permanecer simples:

```text
jhc-bolao-assistente/
│
├── README.md
│
├── docker-compose.yml
├── .env.example
│
└── workflows/
    │
    ├── AI principal.json
    ├── AI_gerar_link_pagamento.json
    ├── Verifica pagamento.json
    ├── Ver_jogos_ativos.json
    ├── Atualiza as tabelas.json
    └── Chamar_jorge.json
```

Os diretórios de dados do Docker não precisam ser enviados.

---

# Tecnologias utilizadas

| Tecnologia | Função |
|---|---|
| n8n | Automação e orquestração |
| Docker | Containerização |
| Docker Compose | Gerenciamento dos serviços |
| WAHA | Integração com WhatsApp |
| Google Gemini | Inteligência artificial |
| Google Sheets | Armazenamento e controle |
| Mercado Pago | Processamento de pagamentos |
| Cloudflare Tunnel | Exposição externa |
| Webhooks | Comunicação entre serviços |

---

# Status do projeto

O projeto possui os principais componentes necessários para:

- Atendimento automatizado
- Gerenciamento de jogos
- Memória da IA
- Consulta de participantes
- Seleção de cotas
- Cálculo de valores
- Geração de pagamentos
- Validação de pagamentos
- Atualização automática dos jogos
- Encaminhamento para atendimento humano
- Integração com WhatsApp

Os workflows são independentes e podem ser modificados individualmente conforme a necessidade da operação.

---

# Observações

Os arquivos JSON exportados pelo n8n podem conter referências específicas da instalação original, como:

- Credenciais
- IDs de documentos
- URLs
- Webhooks
- Configurações específicas
- `pinData`
- Dados de teste

Antes de utilizar os workflows em outra instalação, revise:

1. Credenciais do n8n
2. Google Sheets
3. Google Gemini
4. Mercado Pago
5. WAHA
6. Webhooks
7. Domínio público
8. Cloudflare Tunnel
9. IDs das planilhas
10. Configurações específicas do atendimento

As credenciais da instalação original não devem ser reutilizadas ou publicadas.

---

# Licença

Este projeto pode ser disponibilizado sob a licença definida pelo proprietário do repositório.

Caso nenhuma licença seja adicionada, os direitos autorais permanecem com o autor do projeto e o conteúdo não deve ser automaticamente considerado software de uso livre.