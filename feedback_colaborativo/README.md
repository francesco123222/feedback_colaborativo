## ⚙️ Como Executar o Projeto

1. **n8n:** Crie um novo fluxo no seu n8n e faça o "Import" utilizando o arquivo `.json` disponível nas pastas deste repositório.
2. **Webhook:** Copie a URL do nó de gatilho (Webhook) gerada pelo seu n8n e cole-a na variável `WEBHOOK_N8N_TEST_URLL` dentro do seu arquivo `.env` local antes de rodar as células do Notebook.
3. **Credenciais:** Siga o guia abaixo para configurar as chaves de acesso obrigatórias dentro dos nós do n8n.

## 🔐 Configuração das Credenciais

O projeto depende de integrações com os serviços do ecossistema Google. Você precisará acessar as seguintes plataformas para gerar suas chaves:

1. **Autenticação do Google Sheets:**
   * Acesse o [Google Cloud Console](https://console.cloud.google.com/).
   * Crie um projeto, ative a **Google Sheets API** e configure a *Tela de consentimento OAuth*.
   * Em *Credenciais*, crie um ID do cliente OAuth 2.0 para gerar seu **Client ID** e **Client Secret**, vinculando-os ao nó do Google Sheets no n8n.

2. **Chave de API do Modelo de IA:**
   * Acesse o [Google AI Studio](https://aistudio.google.com/).
   * Vá até a seção de chaves de API e clique em **Create API Key**.
   * Copie a chave gerada e insira-a na configuração de credencial do nó de IA (`Message a model`) do n8n.

3. **Envio de E-mails (Gmail):**
   * Acesse o painel de segurança da sua Conta Google.
   * Certifique-se de que a verificação de duas etapas está ativa e gere uma **Senha de App (App Password)** específica.
   * Utilize essa senha gerada nas configurações de credencial SMTP/Gmail do nó de e-mail do n8n.

## 🧠 Arquitetura do Workflow

O repositório disponibiliza **dois arquivos `.json`** para importação no n8n, divididos em três opções de execução:

### 📂 Arquivo 1: `webhook-feedbacks.json`
Este arquivo traz duas abordagens de fluxos que compartilham o recebimento de dados via Webhook:

### Opção 1: Webhook Simples (Fluxo de 2 nós)
Ideal para testes rápidos ou integrações diretas via API/Notebook:

1. **Webhook**: Captura os dados em tempo real, após a execução (*Pull in events from...*) de um dos nós iniciais.
2. **Send an Email Webhook**: Envia o relatório respectivo para o destinatário (É necessário configurar a credencial do SMTP).


### Opção 2: Webhook com Google Sheets (Fluxo de 4 nós)
Indicado para quem envia dados via Webhook, mas precisa registrar o histórico em planilha (**Enviar pelo Google Forms**):

1. **Webhook Sheets**: Captura os dados em tempo real, após a execução (*Pull in events from...*) de um dos nós iniciais.
2. **Split Out**: Ele pega os campos gerados pela IA (*payload*) e divide cada parte pelo corpo (*body*).
3. **Filter**: Valida se o registro já foi processado anteriormente (evitando loops ou envios duplicados).
4. **Send an Email Sheets**: Envia o relatório respectivo para o destinatário (É necessário configurar a credencial do SMTP).
5. **Update row in sheet**: Altera a coluna de controle da planilha original para garantir a rastreabilidade dos processos (Obrigatório configurar as credenciais pelo Google, seguir passos acima)

---

### 📂 Arquivo 2: `producao-sheets-trigger.json`
Este arquivo contém o fluxo robusto desenhado especificamente para rodar de forma autônoma na nuvem:

### Opção 3 (Use somente para Produção): Integração com Google Sheets Trigger
A solução mais robusta, automatizada e monitorada contra falhas:

1. **Gatilho:** Captura em tempo real novas linhas ou atualizações vindas de um formulário integrado ao **Google Sheets**.
2. **Filtro de Segurança:** Valida se o registro já foi processado anteriormente (evitando loops ou envios duplicados).
3. **Análise de IA (Gemini):** O nó `Message a model` envia o feedback bruto para o modelo de linguagem estruturar os dados.
4. **Tratamento de Payload (JavaScript):** Um nó de código transforma a resposta textual da IA em um objeto JSON tipado (com resumo, prioridade, sentimento, pontos críticos e recomendações de gestão).
5. **Notificação por E-mail:** Envia um relatório executivo estruturado para o gestor responsável.
6. **Atualização do Status:** Altera a coluna de controle da planilha original para garantir a rastreabilidade do processo.
7. **Tratamento de Erros:** Um nó global de `Error Trigger` intercepta qualquer falha em tempo de execução e dispara um e-mail de alerta técnico imediatamente.

### 🤖 Requisitos de IA

* **Modelos Utilizados:** `gemini-3.1-pro` ou `gemini-3.1-flash-lite` (configurados no nó do n8n com as respectivas chaves de API obtidas no [Google AI Studio](https://aistudio.google.com/)).
* **Formato de Entrada:** O prompt do sistema espera receber obrigatoriamente um objeto ou string contendo os seguintes campos estruturados:
  * **Nome**
  * **E-mail**
  * **Departamento**
  * **Feedback**

> 📝 **Nota de Configuração:** O usuário precisa criar um **Google Forms (Formulário Google)** contendo exatamente essas perguntas e vinculá-lo a uma planilha do Google Sheets. Essa planilha servirá como a base de dados que enviará os feedbacks para os fluxos do n8n.