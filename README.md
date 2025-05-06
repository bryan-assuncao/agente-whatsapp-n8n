# N8N Workflow: Case 02 - Chatbot de Suporte via WhatsApp com IA

## 📝 Descrição

Este workflow do n8n automatiza um sistema de chatbot de suporte via WhatsApp. Ele recebe mensagens de usuários, registra informações do contato em uma planilha do Google Sheets, processa a mensagem do usuário utilizando um agente de Inteligência Artificial (IA) conectado ao modelo de linguagem da Groq, e envia uma resposta de volta ao usuário pelo WhatsApp. O agente de IA é instruído a ser um assistente de suporte educado e divertido, utilizando emojis para humanizar a conversa, e pode utilizar ferramentas como calculadora e Wikipedia para auxiliar nas respostas.

## ✨ Funcionalidades Principais

* **Recepção de Mensagens via Webhook:** Inicia com um gatilho (Webhook) que aguarda mensagens recebidas, provavelmente de uma integração com o WhatsApp através da Z-API.
* **Filtragem de Mensagens:** Verifica se a mensagem não é de grupo, newsletter, broadcast ou originada pela API, focando em conversas diretas com usuários.
* **Extração de Dados:** Coleta o número de telefone (ID da conversa), a mensagem e o nome do remetente.
* **Registro no Google Sheets:** Adiciona ou atualiza informações do contato (Nome e WhatsApp) em uma planilha específica do Google Sheets ("N8N - Planilha", aba "Case 2").
* **Processamento com Agente de IA (Langchain + Groq):**
    * Utiliza um agente de IA configurado com um prompt de sistema para atuar como um suporte amigável.
    * Conectado a um modelo de linguagem da Groq para geração de respostas.
    * Mantém um histórico da conversa (memória) para respostas contextuais.
    * Possui acesso a ferramentas como Calculadora e Wikipedia.
* **Envio de Resposta via HTTP Request:** Envia a resposta gerada pela IA de volta para o usuário no WhatsApp através de uma requisição HTTP para a Z-API.

## ⚙️ Como Funciona (Fluxo do Workflow)

1.  **Webhook (`Webhook`):** Aguarda uma requisição POST na URL `/receber-mensagem-zapi`. Espera-se que a Z-API envie os dados da mensagem do WhatsApp para este endpoint.
2.  **Condicional (`If`):** Verifica se a mensagem recebida é uma mensagem direta de um usuário (não é de grupo, newsletter, broadcast ou da API).
    * **Se NÃO for direta:** O fluxo termina (`No Operation, do nothing1`).
    * **Se FOR direta:** Prossegue para o próximo passo.
3.  **Extração/Edição de Campos (`Edit Fields`):**
    * Extrai `IdConversa` (telefone do usuário), `Mensagem` (texto da mensagem) e `Nome` (nome do chat do usuário) do corpo da requisição do webhook.
4.  **Google Sheets (`Google Sheets`):**
    * Conecta-se à planilha "N8N - Planilha", na aba "Case 2".
    * Adiciona uma nova linha com o `Whatsapp` (IdConversa) e `Nome` do usuário.
    * Se já existir uma entrada com o mesmo número de `Whatsapp`, atualiza o `Nome`.
5.  **Agente de IA (`AI Agent`):**
    * Recebe a `Mensagem` do usuário como entrada (prompt).
    * **Modelo de Linguagem (`Groq Chat Model`):** Utiliza o modelo da Groq para gerar a resposta.
    * **Memória (`Simple Memory`):** Armazena o histórico da conversa usando `IdConversa` como chave da sessão, permitindo que o agente lembre de interações anteriores na mesma conversa.
    * **Ferramentas (`Calculator`, `Wikipedia`):** O agente pode optar por usar a calculadora para cálculos ou a Wikipedia para buscar informações e formular uma resposta mais completa.
    * **Prompt de Sistema:** "Você é um agente de suporte, seja educado, engraçado e utilize emojis em suas respostas para deixar a conversa mais humanizada".
6.  **Envio de Resposta (`HTTP Request`):**
    * Envia uma requisição POST para a API da Z-API (`https://api.z-api.io/instances/SUA_INSTANCIA/token/SEU_TOKEN/send-text`).
    * No corpo da requisição, envia o `phone` (IdConversa) e `message` (a resposta gerada pelo `AI Agent`).
    * Inclui o `client-token` necessário para autenticação com a Z-API.

## 🚀 Pré-requisitos e Configuração

Antes de utilizar este workflow, você precisará configurar:

1.  **Credenciais do n8n:**
    * **Google Sheets:** Autentique sua conta Google Sheets no n8n para permitir que o workflow leia e escreva na planilha.
    * **Groq:** Adicione sua chave de API da Groq nas credenciais do n8n.
2.  **Webhook:**
    * Após ativar o workflow, a URL do webhook (Modo Produção) deverá ser configurada na sua plataforma Z-API para enviar os eventos de novas mensagens.
3.  **Google Sheets:**
    * Crie uma planilha no Google Drive com o nome "N8N - Planilha" (ou atualize o nó do Google Sheets com o ID da sua planilha).
    * Dentro desta planilha, crie (ou renomeie) uma aba para "Case 2".
    * Certifique-se de que a aba "Case 2" tenha pelo menos as colunas "Whatsapp" e "Nome".
4.  **HTTP Request (Z-API):**
    * No nó `HTTP Request` chamado "HTTP Request":
        * Atualize a URL com o **ID da sua instância** e **Token da Z-API**: `https://api.z-api.io/instances/SEU_ID_DE_INSTANCIA/token/SEU_TOKEN_ZAPI/send-text`.
        * No cabeçalho (Headers), atualize o `client-token` com o **Client Token da sua instância Z-API**.
        * **IMPORTANTE:** Por segurança, é altamente recomendável que você armazene estes tokens e IDs como variáveis de credencial no n8n, em vez de inseri-los diretamente no nó. O nó HTTP Request permite selecionar credenciais do tipo "Header Auth" ou configurar a URL e tokens dinamicamente usando expressões que buscam de credenciais mais seguras.

## 🛠️ Nós Principais Utilizados

* `n8n-nodes-base.webhook`: Para receber dados de entrada.
* `n8n-nodes-base.if`: Para controle de fluxo condicional.
* `n8n-nodes-base.set`: Para manipular e definir campos de dados.
* `n8n-nodes-base.googleSheets`: Para interagir com planilhas Google.
* `@n8n/n8n-nodes-langchain.agent`: Para criar um agente de IA conversacional.
* `@n8n/n8n-nodes-langchain.lmChatGroq`: Para usar os modelos de linguagem da Groq.
* `@n8n/n8n-nodes-langchain.memoryBufferWindow`: Para adicionar memória à conversa do agente.
* `@n8n/n8n-nodes-langchain.toolCalculator`: Ferramenta de cálculo para o agente.
* `@n8n/n8n-nodes-langchain.toolWikipedia`: Ferramenta de busca na Wikipedia para o agente.
* `n8n-nodes-base.httpRequest`: Para enviar a resposta via API da Z-API.
* `n8n-nodes-base.noOp`: Para finalizar um caminho do fluxo sem realizar ações.

## 💡 Como Usar

1.  Importe o arquivo `Case_02.json` para sua instância n8n.
2.  Configure todas as credenciais e URLs/IDs mencionados na seção "Pré-requisitos e Configuração".
3.  Ative o workflow.
4.  Configure o webhook gerado pelo nó `Webhook` (no modo de produção) na sua plataforma Z-API.
5.  Envie uma mensagem para o número de WhatsApp conectado à Z-API para testar.

## ⚠️ Observações Importantes

* **Segurança de Credenciais:** NUNCA exponha suas chaves de API, tokens ou outros dados sensíveis diretamente no arquivo JSON do workflow se for compartilhá-lo publicamente. Utilize o sistema de gerenciamento de credenciais do n8n. Este README assume que as credenciais (`googleSheetsOAuth2Api`, `groqApi`) e os tokens da Z-API estão configurados de forma segura na sua instância n8n e referenciados pelos nós.
* **Custos de API:** O uso de APIs como Groq e Z-API pode incorrer em custos dependendo do seu volume de uso. Monitore seu consumo.
* **Tratamento de Erros:** Para um ambiente de produção, considere adicionar tratamento de erros em cada etapa do workflow para lidar com falhas inesperadas (ex: API da Z-API indisponível, falha ao escrever na planilha).

---

Este projeto (`Case_02.json`) demonstra uma poderosa integração entre n8n, Google Sheets, IA (Groq e Langchain) e a Z-API para criar um chatbot funcional.
