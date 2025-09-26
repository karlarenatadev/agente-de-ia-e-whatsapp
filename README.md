# agente-de-ia-e-whatsapp

# 🤖 Chatbot de Suporte com Gemini, n8n e Google Sheets

Este projeto demonstra a criação de um **chatbot de suporte** divertido e inteligente, orquestrado pelo **n8n** (uma ferramenta de automação no-code/low-code), utilizando o modelo **Google Gemini** para as interações e registrando automaticamente todas as conversas em uma planilha do **Google Sheets**.

## ✨ Visão Geral do Projeto

O objetivo principal é oferecer uma experiência de suporte automatizada e humanizada, enquanto se mantém um registro organizado de todas as interações.

* **Agente de IA:** Usa o **Google Gemini** para processamento de linguagem natural e geração de respostas.
* **Workflow:** Gerenciado pelo **n8n**, que conecta a interface de chat, a IA e o banco de dados (Google Sheets).
* **Persistência:** O histórico da conversa é mantido com o **Memory Buffer** do n8n, e as mensagens de entrada são salvas no **Google Sheets**.
* **Personalidade:** O agente é configurado com uma `system message` para ser **educado, engraçado e usar emojis** (conforme visto na captura de tela: "Olá, Karla! Que bom te conhecer! Eu sou seu super agente de suporte, pronto para te ajudar com o que precisar! Como posso iluminar seu dia hoje?").

## 🛠️ Arquitetura do Workflow (n8n)

O fluxo de trabalho é linear e segue os seguintes passos:

1.  **`When chat message received` (Gatilho de Chat):**
    * **Função:** Inicia o workflow a cada nova mensagem do usuário na interface do chat.

2.  **`Edit Fields` (Preparação de Dados):**
    * **Função:** Extrai e nomeia as variáveis essenciais para uso posterior.
    * **Variáveis:**
        * `id_conversa`: Captura o ID da sessão (`{{ $json.sessionId }}`).
        * `Mensagem`: Captura o texto da mensagem do usuário (`{{ $json.chatInput }}`).

3.  **`Append row in sheet` (Google Sheets):**
    * **Função:** Registra a mensagem do usuário no Google Sheets.
    * **Configuração:** Utiliza a operação `append` e mapeia as variáveis criadas: `id_conversa` e `Mensagem`.
    * **Finalidade:** Garante a persistência do histórico da conversa em um formato de fácil acesso para análise.

4.  **`AI Agent` (Processamento de IA):**
    * **Função:** É o core da inteligência, orquestrando o modelo de linguagem, a memória e as ferramentas.
    * **Prompt de Entrada:** A mensagem do usuário (`{{ $('Edit Fields').item.json.Mensagem }}`) é passada para o agente.
    * **System Message (Personalidade):** "Você é um super agente de suporte, seja educado, engraçado e utilize emojis em suas respostas para deixar a conversa mais humanizada."
    * **Conexões:** O Agente está conectado a 4 componentes:
        * **`Google Gemini Chat Model`** (Modelo de Linguagem).
        * **`Simple Memory`** (Memória da Conversa).
        * **`Calculator`** (Ferramenta para cálculos matemáticos).
        * **`Wikipedia`** (Ferramenta para buscar informações externas, como no exemplo de "Augusto Cury").

5.  **`No Operation, do nothing` (Finalização):**
    * **Função:** Um nó final que não executa nenhuma ação, mas garante que o fluxo se complete após o Agente de IA enviar a resposta ao chat.

## 🔗 Componentes de IA e Ferramentas

| Componente | Tipo | Função no Workflow |
| :--- | :--- | :--- |
| **Google Gemini Chat Model** | Language Model | Geração da resposta da IA. |
| **Simple Memory** | Memory | Armazena o histórico da sessão (`id_conversa` é a chave) para que o agente se lembre de interações anteriores. |
| **Calculator** | Tool | Permite que o agente resolva problemas matemáticos. |
| **Wikipedia** | Tool | Permite que o agente acesse informações factuais e de conhecimento geral (Ex: "Quem é Augusto Cury?"). |

## 🖼️ Demonstração da Interação

A captura de tela da conversa mostra a funcionalidade em ação:

| Usuário (Karla) | Agente (Gemini) |
| :--- | :--- |
| `Me chamo Karla` | *Resposta inicial personalizada usando a system message.* |
| `Que é Augusto Cury?` | *O agente usa a ferramenta **Wikipedia** para buscar a informação e retornar uma resposta detalhada e contextualizada.* |

---

## 🚀 Como Executar Este Projeto

Para replicar este workflow, você precisará:

1.  Uma conta **n8n** (self-hosted ou cloud).
2.  Uma chave de API para o **Google Gemini** (Configurada no nó `Google Gemini Chat Model`).
3.  Uma conta **Google Sheets** com credenciais configuradas no n8n.
4.  Importar o arquivo `curso-no-code.json` no seu ambiente n8n.

Lembre-se de substituir os IDs do Google Sheets e as credenciais pelos seus próprios.
