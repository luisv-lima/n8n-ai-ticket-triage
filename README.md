# AI Workflow: Triagem Inteligente de Tickets de Suporte com n8n

Este repositório contém uma solução completa de automação de processos desenvolvida no **n8n**, projetada para otimizar a triagem, classificação, direcionamento e registro de chamados de suporte ao cliente utilizando **Inteligência Artificial (OpenAI)** e integrações com **Google Sheets** e **Slack**.

---

## 1. Problema e Processo Escolhido

Em equipes de atendimento e suporte técnico, o recebimento descentralizado de solicitações (via formulários ou e-mails) costuma gerar gargalos operacionais relevantes:
- **Atraso na triagem:** Mensagens críticas demoram a ser lidas e encaminhadas.
- **Triagem manual ineficiente:** Classificar manualmente a prioridade e a categoria de cada chamado consome tempo valioso da equipe.
- **Falta de padronização:** Ausência de histórico estruturado sobre o sentimento do cliente e resumos operacionais.

**A Solução:** Automatizar o ciclo de vida do ticket a partir do momento em que ele é enviado, utilizando IA para entender o contexto do chamado, classificar a urgência, notificar os responsáveis em tempo real e registrar todas as métricas em uma planilha centralizada.

<img width="731" height="347" alt="workflown8n" src="https://github.com/user-attachments/assets/72109104-ea77-4810-9254-a4d889cb8099" />

---

## 2. Arquitetura do Workflow e Sistemas Integrados

O workflow foi projetado com uma arquitetura orientada a eventos, garantindo execução rápida, escalabilidade e resiliência.

### Fluxo de Dados:
1. **Entrada (Webhook):** Ingestão dos dados brutos enviados pelo cliente (`nome`, `email`, `mensagem`).
2. **Validação:** Verificação condicional para assegurar que os campos essenciais foram preenchidos.
3. **Análise por IA:** O modelo da OpenAI analisa a mensagem e retorna uma estrutura JSON com categoria, prioridade, sentimento e resumo.
4. **Roteamento Condicional:** O nó `Switch` avalia a prioridade definida pela IA.
5. **Ações em Paralelo:**
   - **Prioridade Alta:** Envio imediato de alerta formatado no canal de urgência do **Slack** + Registro no **Google Sheets**.
   - **Prioridade Média/Baixa:** Registro direto no **Google Sheets**.

### Sistemas e APIs Integrados:
- **n8n Cloud:** Plataforma principal de orquestração de workflows.
- **OpenAI API (GPT):** Processamento de Linguagem Natural (NLP) e estruturação de dados em JSON.
- **Google Sheets API:** Armazenamento e histórico estruturado dos chamados.
- **Slack API:** Notificações instantâneas para a equipe de suporte.

<img width="710" height="238" alt="success" src="https://github.com/user-attachments/assets/98f422a7-b96f-460d-b142-201299380589" />

---

## 3. Função da Inteligência Artificial

A Inteligência Artificial atua como o motor de decisão do fluxo. Em vez de depender de regras engessadas baseadas em palavras-chave, o prompt instrui a IA a analisar o contexto emocional e operacional do texto e retornar **estritamente um objeto JSON** com os seguintes parâmetros:

- `categoria`: Técnico, Financeiro, Cancelamento ou Geral.
- `prioridade`: Alta, Média ou Baixa.
- `sentimento`: Positivo, Neutro ou Frustrado.
- `resumo`: Síntese do problema em uma única frase.

<img width="1041" height="1213" alt="image" src="https://github.com/user-attachments/assets/b3f338b6-22c0-4eba-be86-af6f5ac098e9" />

---

## 4. Principais Nós Utilizados

| Nó n8n | Função no Workflow |
| :--- | :--- |
| **Ticket Entrada** | Webhook HTTP (POST) para receber requisições de teste e produção. |
| **Email e Mensagem Preenchidos?** | Nó `IF` para validação de dados de entrada antes do envio à IA. |
| **Analisar Ticket (OpenAI)** | Nó LLM para análise sintática, sentimento e classificação estruturada. |
| **Parse e Mapear Campos** | Tratamento e extração das variáveis do JSON retornado pela IA. |
| **Prioridade Alta?** | Nó `Switch` para desvio condicional baseado no nível de urgência. |
| **Registrar Ticket Alta / Default** | Nó Google Sheets para escrita de novas linhas com mapeamento manual de colunas. |
| **Slack Suporte Urgente** | Nó Slack para postagem de alertas ricos no canal `#suporte-urgente`. |
| **Error Trigger & Notificar Erro** | Fluxo global de captura de falhas operacionais e alerta no Slack. |

---

## 5. Como Executar e Testar

### Pré-requisitos
1. Instância do **n8n** (Cloud ou Self-Hosted).
2. Credenciais configuradas no cofre do n8n:
   - Chave de API da **OpenAI**.
   - Conta autenticada do **Google Sheets**.
   - Conexão OAuth2 ou Bot do **Slack**.

### Passo a Passo

1. **Importar o Workflow:** Importe o arquivo JSON deste repositório para o seu painel do n8n.
2. **Configurar a Planilha:** Crie uma planilha no Google Sheets com os seguintes cabeçalhos na linha 1:
   `id_ticket` | `cliente_nome` | `cliente_email` | `analise_ia` | `prioridade` | `sentimento` | `resumo`
3. **Ativar o Webhook:** Abra o nó `Ticket Entrada` e copie a **Test URL**.
4. **Disparar uma Requisição:** Execute o comando abaixo no **PowerShell** ou **Terminal**:

```powershell
Invoke-RestMethod -Uri "SUA_URL_DO_WEBHOOK_AQUI" -Method Post -ContentType "application/json" -Body '{"nome": "Luís Lima", "email": "luis@exemplo.com", "mensagem": "A minha conta foi cobrada em duplicado e preciso do reembolso urgente!"}'
```

6. Evidências do Workflow Funcional
Notificação em Tempo Real no Slack
O chamado foi identificado como de prioridade Alta e sentimento Frustrado, acionando imediatamente o alerta no canal da equipe.

<img width="1197" height="1086" alt="slack" src="https://github.com/user-attachments/assets/13d5c444-5755-4d0d-8c2c-d96136c74fd3" />


Registro Persistido no Google Sheets
Todas as colunas foram preenchidas de forma alinhada e organizada com a análise feita pela IA.

<img width="988" height="400" alt="sheets" src="https://github.com/user-attachments/assets/3fc2bb65-d298-4a1d-a4b8-ed94a78c5adf" />
