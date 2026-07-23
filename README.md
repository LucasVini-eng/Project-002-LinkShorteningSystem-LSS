# 🔗 Link Shortening System (LSS)

O **Link Shortening System (LSS)** é um serviço interativo desenvolvido em Python para encurtamento, validação e análise de segurança prévia de URLs. O objetivo principal da solução é oferecer uma camada de proteção ao usuário final, analisando a URL contra padrões comuns de **engenharia social e phishing** antes de processar o encurtamento.

---

## 🛠️ Tecnologias, IDEs & Infraestrutura

O projeto foi construído e implantado utilizando padrões modernos do ecossistema Python:

* **IDEs de Desenvolvimento:**
  * **PyCharm:** Utilizado para desenvolvimento backend, análise estática de código, refatoração segura e gerenciamento de ambientes virtuais (`venv`).
  * **Visual Studio Code (VS Code):** Utilizado para ajustes rápidos de estilo (CSS), edição de Markdown, gerenciamento do repositório Git e automações leves.
* **Framework Web:** **Streamlit** (gerenciamento de interface reativa e estados de sessão).
* **Provedor de Encurtamento:** `pyshorteners` integrado à API do **TinyURL**.
* **Deploy & Hospedagem:** **Streamlit Community Cloud** — implantação contínua (CD) conectada ao repositório GitHub para entrega rápida da aplicação em ambiente de produção na nuvem.

---

## 🏗️ 1. Desenho de Arquitetura & Visão Geral

O sistema utiliza o **Streamlit** como camada de apresentação e controlador de estado de sessão, desacoplando a lógica de validação de segurança e integração com provedores externos de encurtamento.

<img width="520" height="320" alt="image" src="https://github.com/user-attachments/assets/8be8b708-eca4-4d87-b71a-8d023049fafa" />

---

## 📋 2. Requisitos do Sistema

### Requisitos Funcionais (FR)
* **FR-01 - Validação Sintática:** O sistema deve verificar se a URL inserida possui formato sintático válido (com suporte a schemas HTTP/HTTPS).
* **FR-02 - Inspeção de Segurança:** O sistema deve analisar a URL em busca de elementos suspeitos associados a malwares ou engenharia social antes da geração do link.
* **FR-03 - Encurtamento de URL:** O sistema deve gerar um link encurtado apenas para URLs aprovadas nas etapas de validação e segurança.
* **FR-04 - Gestão de Estado:** O sistema deve invalidar a checagem de segurança anterior caso o usuário modifique a URL no campo de entrada.

### Requisitos Não-Funcionais (NFR)
* **NFR-01 - Desempenho / Latência:** O tempo de checagem local de heurísticas deve ser near-instantâneo ($< 50\text{ ms}$).
* **NFR-02 - Usabilidade:** Interface intuitiva com feedback visual claro (*Toasts/Alerts*) quanto ao status de segurança do link.
* **NFR-03 - Disponibilidade:** Hospedagem em nuvem via **Streamlit Cloud** garantindo alta disponibilidade sem custos fixos de servidor.

---

## 📡 3. Interfaces & Endpoints (Mapeamento de Fluxo)

Como o sistema opera no modelo **Server-Side Rendered (SSR)** via WebSockets no Streamlit, as interações do usuário acionam eventos no fluxo da aplicação em vez de requisições HTTP REST clássicas.

| Ação do Usuário | Componente Interno | Entrada | Saída / Efeito |
| :--- | :--- | :--- | :--- |
| **Digitação de URL** | `st.text_input` | `long_url` (string) | Atualização de estado `st.session_state` e reset de verificação. |
| **Clique "Check URL"** | `validators.url` + `check_safety()` | `long_url` | Validação sintática e checagem heurística de ameaças. |
| **Clique "Shorten URL"** | `pyshorteners.Shortener().tinyurl` | `long_url` (verificada) | Chamada REST ao provedor `TinyURL` e retorno do link curto. |

### Integração de API Externa (Provedor)
* **Endpoint de Terceiros:** `http://tinyurl.com/api-create.php?url={long_url}`
* **Método:** `GET`
* **Payload:** URL original serializada via query parameter.

## 🛡️ 4. Validação de Segurança e Prevenção de Engenharia Social
O motor de segurança do LSS atua em **três camadas de análise heurística** para mitigar riscos de phishing, distribuição de binários maliciosos e ataques baseados em engenharia social.<br>
<img width="620" height="420" alt="image" src="https://github.com/user-attachments/assets/ca013650-ea37-41eb-9c34-758a4f6500f4" /><br>

# Detalhamento das Camadas de Defesa

1. **Protocolo de Transport Seguro (`HTTPS`):**
   * Verificação via `urllib.parse.urlparse`. URLs sob o esquema `http://` não criptografado são rejeitadas para evitar ataques *Man-in-the-Middle (MitM)* ou interceptação de credenciais.

2. **Engenharia Social & Gatilhos Psicológicos (Keywords Check):**
   * Ataques de phishing frequentemente utilizam termos chamativos no domínio ou caminho para induzir cliques impulsivos. O sistema filtra termos mapeados como gatilhos de alto risco:
     * *Iscas Financeiras / Ofertas:* `gratis`, `ganhe-dinheiro`, `brinde`, `desconto`, `oferta`.
     * *Urgência / Ação Forçada:* `urgente`, `cadastro`.
     * *Apostas / Jogos:* `bet`.

3. **Prevenção contra Download Direto de Malwares:**
   * A análise verifica a extensão no caminho da URL (`path`). Requisições apontando diretamente para executáveis (`.exe`, `.bat`, `.cmd`, `.msi`) são bloqueadas para mitigar ataques *Drive-by Download*.

---

## 🔄 5. Transformação de URL Longa para Curta

O ciclo de vida da transformação de um link no sistema segue o padrão **Check-Then-Act**: <br>
[ Long URL ] ➔ [ Input Verification ] ➔ [ State: Verified = True ] ➔ [ API Request ] ➔ [ Short URL ]

### Algoritmo de Transformação Interno
1. **Sanitização:** A URL inserida é tratada e convertida para caracteres minúsculos na verificação heurística para evitar mutações de bypass (ex: `.EXE` ou `GrAtIs`).
2. **Controle de Estado Finito:** O botão de geração de link curto permanece oculto/desabilitado até que o `st.session_state.verified_url` seja avaliado como `True`.
3. **Resolução de Alias:** A biblioteca `pyshorteners` envia a URL validada para a infraestrutura do TinyURL, que aplica um algoritmo de *hashing* de base 62 no banco de dados deles, retornando o alias encurtado no formato `https://tinyurl.com/{hash}`.

---

## 🚀 Como Executar o Projeto

### Rodando Localmente (PyCharm / VS Code)

1. **Clone o repositório:**
   ```bash
   git clone [https://github.com/seu-usuario/link-shortener-lss.git](https://github.com/seu-usuario/link-shortener-lss.git)
   cd link-shortener-lss










