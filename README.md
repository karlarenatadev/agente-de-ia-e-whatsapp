# 📞 Automação de WhatsApp para Novos Leads com n8n e Gemini (Z-API)

Este projeto demonstra um fluxo de trabalho de automação avançada construído com **n8n** para processar mensagens recebidas no **WhatsApp** (via Z-API), utilizar o **Google Gemini** como um agente de suporte inteligente e salvar/atualizar informações de leads no **Google Sheets**.

## 🎯 Objetivo

Criar um sistema de atendimento e qualificação de leads via WhatsApp que:

1.  **Filtra** mensagens irrelevantes (grupos, newsletters, etc.).
2.  **Extrai** o número de telefone e nome do contato.
3.  **Registra/Atualiza** o lead no Google Sheets.
4.  Gera uma **resposta inteligente** em tempo real usando o **Gemini AI Agent**.
5.  **Envia** a resposta de volta ao usuário via API do WhatsApp (Z-API).

## 🛠️ Arquitetura do Workflow (n8n)

O fluxo é acionado por um webhook do WhatsApp (Z-API) e tem uma estrutura condicional para garantir que apenas mensagens de conversas individuais sejam processadas.

### 1. Início e Filtragem
| Nó | Tipo | Função |
| :--- | :--- | :--- |
| **`Webhook`** | Trigger | Recebe a notificação (payload) de novas mensagens do WhatsApp, geralmente configurado na plataforma Z-API. |
| **`If` (Se)** | Condicional | **Filtra** a mensagem. O fluxo só prossegue se a mensagem **NÃO** for de grupo (`isGroup: false`), newsletter (`isNewsletter: false`), broadcast (`broadcast: false`) ou enviada pela API (`fromApi: false`). Mensagens não qualificadas são descartadas pelo nó `No Operation, do nothing1`. |

### 2. Preparação de Dados
| Nó | Tipo | Variáveis Mapeadas | Função |
| :--- | :--- | :--- | :--- |
| **`Edit Fields`** | Set | **`id_conversa`** (do corpo do telefone), **`Mensagem`** (texto da mensagem), **`Nome`** (nome do chat). | Extrai os dados essenciais do payload do Webhook para facilitar o uso nos nós subsequentes. |

### 3. Persistência de Leads
| Nó | Tipo | Configuração | Função |
| :--- | :--- | :--- | :--- |
| **`Append or update row in sheet`** | Google Sheets | **Operação:** `appendOrUpdate`. **Coluna de Correspondência:** `WhatsApp` (`id_conversa`). | Utiliza o número de WhatsApp como chave única. Se o lead for novo, ele é **anexado**; se já existir (conversa recorrente), o nome é **atualizado**. |

### 4. Processamento da Inteligência Artificial
Este é o core do atendimento, onde o **Agente de IA** gera a resposta.

| Nó | Tipo | Configuração | Detalhe da Conexão |
| :--- | :--- | :--- | :--- |
| **`AI Agent`** | Langchain Agent | **System Message:** "Você é um super agente de suporte, seja educado, engraçado e utilize emojis em suas respostas para deixar a conversa mais humanizada." | Orquestra a resposta usando o Modelo de Linguagem, a Memória e as Ferramentas. |
| **`Google Gemini Chat Model`** | Language Model | Modelo de IA para gerar o texto da resposta. | Conectado ao `AI Agent`. |
| **`Simple Memory`** | Memory | Session Key: `id_conversa` (`phone`). | Permite que o agente se lembre do contexto da conversa com o lead. |
| **`Calculator` / `Wikipedia`** | Tools | Ferramentas opcionais para o agente realizar cálculos ou buscar informações externas. | Conectado ao `AI Agent`. |

### 5. Resposta ao Usuário
| Nó | Tipo | Configuração | Função |
| :--- | :--- | :--- | :--- |
| **`HTTP Request`** | Envio de API | **URL:** API de envio de texto do Z-API. **Body:** Mapeia `phone` com `id_conversa` e `message` com `{{ $json.output }}` (a resposta gerada pelo `AI Agent`). | Envia a resposta final do Gemini de volta para o lead no WhatsApp. |
| **`No Operation, do nothing`** | Finalizador | - | Garante que o fluxo de trabalho termine corretamente. |

## 🔑 Credenciais Necessárias

Para rodar este projeto, você precisará configurar as seguintes credenciais no n8n:

1.  **Google Sheets OAuth2 API:** Para ler e escrever na planilha de leads.
2.  **Google Gemini (PaLM) API:** Para o nó `Google Gemini Chat Model`.
3.  **Z-API Instance/Token:** Para configurar a URL do `HTTP Request` e o `client-token` no cabeçalho.
