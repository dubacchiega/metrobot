# 🚇 MetrôBot SP 2.0 — Sistema Inteligente de Rotas (Grafos, Lógica & LLM)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/dubacchiega/metrobot/blob/main/MetroBot_2.ipynb)

## 👥 Integrantes do Grupo

- **Eduardo Rafael Bacchiega** — RA: `1749416`
- **Diego Rossetto do Nascimento** — RA: `1727950`
- **Walace Alves de Araújo** — RA: `1327400`
- **Marcos da Silva Teixeira** - RA: `2158303`

---
## 📝 Descrição

O **MetrôBot SP 2.0** é um sistema inteligente de navegação e recomendação de rotas para três linhas de metrô de São Paulo (Linhas 1-Azul, 2-Verde e 3-Vermelha). 

O projeto combina **Algoritmos de Busca em Grafos (BFS e DFS)**, **Motor de Inferência Lógica de Primeira Ordem (Forward Chaining)**, **Modelos de Linguagem (LLM via Groq / Ollama / RegEx Offline)** e uma **Interface Interativa em IPyWidgets**.

---

## 📌 Funcionalidades Principais

- **Modelagem da Malha em Grafos:** Suporte a 52 estações conectadas nas Linhas 1-Azul, 2-Verde e 3-Vermelha, mapeando trechos compartilhados e pontos de baldeação.
- **Algoritmos de Busca:**
  - **BFS (Breadth-First Search):** Encontra o caminho com o menor número de paradas.
  - **DFS (Depth-First Search):** Explora caminhos em profundidade para comparação de desempenho e quantidade de nós visitados.
- **Motor de Inferência Lógica (Forward Chaining):**
  - Mapeamento automático de pontos turísticos/locais para estações próximas.
  - Tratamento dinâmico de estações fechadas, elevadores em manutenção (acessibilidade) e paralisações de linhas inteiras.
  - Deduz automaticamente estações de integração e baldeação.
- **Processamento de Linguagem Natural (NLP / LLMs):**
  - **Intérprete:** Converte pedidos em linguagem natural para JSON estruturado.
  - **Narrador:** Gera explicações amigáveis e claras sobre o percurso e baldeações.
  - **Fallback Offline:** Módulo de contingência via Expressões Regulares (RegEx) e fuzzy matching para operar sem dependência de APIs externas.
- **Interface Gráfica Interativa:** Painel visual construído com `ipywidgets` e estilizado em HTML/CSS de alto contraste com as cores oficiais das linhas.

---

## 🏗️ Arquitetura do Sistema

```text
                         [Entrada do Usuário]
                   (Texto em Linguagem Natural)
                                │
                                ▼
                   ┌─────────────────────────┐
                   │   Módulo Intérprete     │
                   │ (Groq / Ollama / RegEx) │
                   └────────────┬────────────┘
                                │ JSON (Origem, Destino, Acessibilidade)
                                ▼
                   ┌─────────────────────────┐
                   │  Motor Lógico Dedutivo  │
                   │ (Encadeamento P/ Frente)│
                   └────────────┬────────────┘
                                │ Fatos Deduzidos (Origem, Destino, Alertas)
                                ▼
                   ┌─────────────────────────┐
                   │   Módulo de Busca       │
                   │      (BFS / DFS)        │
                   └────────────┬────────────┘
                                │ Caminho Mínimo & Baldeações
                                ▼
                   ┌─────────────────────────┐
                   │ Módulo Narrador & UI    │
                   │  (HTML / IPyWidgets)    │
                   └─────────────────────────┘
```

---

## 🗺️ Malha Metroviária Coberta

| Linha | Estação Inicial | Estação Final | Cor Oficial |
| :--- | :--- | :--- | :--- |
| **Linha 1-Azul** | Tucuruvi | Jabaquara | `Azul` |
| **Linha 2-Verde** | Vila Madalena | Vila Prudente | `Verde` |
| **Linha 3-Vermelha** | Palmeiras-Barra Funda | Corinthians-Itaquera | `Vermelha` |

### Estações de Integração Mapeadas:
- **Sé:** Linha 1-Azul ↔ Linha 3-Vermelha
- **Paraíso:** Linha 1-Azul ↔ Linha 2-Verde
- **Ana Rosa:** Linha 1-Azul ↔ Linha 2-Verde

---

## 🧠 Regras Lógicas de Primeira Ordem ($R_1$ a $R_7$)

O motor de inferência utiliza encadeamento para frente (*Forward Chaining*) com as seguintes regras formais:

- **R1 (Origem via Local):** `∀l ∀e (usuario_esta_em(l) ∧ proximo_de(l, e) → origem(e))`
- **R2 (Destino via Local):** `∀l ∀e (usuario_quer_ir(l) ∧ proximo_de(l, e) → destino(e))`
- **R3 (Bloqueio de Estação):** `∀e (fechada(e) → bloqueada(e))`
- **R4 (Acessibilidade):** `∀e (precisa_acessibilidade ∧ elevador_em_manutencao(e) → inacessivel(e))`
- **R5 (Alerta de Acessibilidade):** `∀p ∀e (papel(p, e) ∧ inacessivel(e) → alerta(p, e))`
- **R6 (Dedução de Integração):** `∀e ∀l1 ∀l2 (pertence(e, l1) ∧ pertence(e, l2) ∧ l1 ≠ l2 → integracao(e))`
- **R7 (Paralisação de Linha):** `∀e ∀l (pertence(e, l) ∧ linha_paralisada(l) → bloqueada(e))`

---

## ⚙️ Pré-requisitos e Instalação

### Bibliotecas Necessárias
```bash
pip install groq ollama ipywidgets python-dotenv
```

### Configuração da API Key (Opcional - Groq Cloud)
Caso utilize a API da Groq no Google Colab:
1. Abra a aba lateral de **Secrets (🔑)** no Colab.
2. Adicione a chave `GROQ_API_KEY` com o valor da sua API Key.

---

## 🧪 Casos de Teste Automatizados

O sistema inclui suíte de testes cobrindo rotas padrão e contingências diante de bloqueios:

| # | Origem → Destino | Condição / Bloqueios | Resultado Esperado |
| :-: | :--- | :--- | :--- |
| **1** | Tucuruvi → Corinthians-Itaquera | Operação Normal | 22 paradas, 1 baldeação (Sé) |
| **2** | Vila Madalena → Jabaquara | Operação Normal | 14 paradas, 1 baldeação (Paraíso/Ana Rosa) |
| **3** | Palmeiras-Barra Funda → Vila Prudente | Operação Normal | 16 paradas, 2 baldeações (Sé e Paraíso/Ana Rosa) |
| **4** | Tucuruvi → Brás | Estação Sé Fechada | Sem rota disponível |
| **5** | Vila Madalena → Jabaquara | Estação Paraíso Fechada | Sem rota disponível (Linha 2 cortada no trecho) |
| **6** | Vila Prudente → Jabaquara | Estação Paraíso Fechada | 13 paradas (Desvio automático via Ana Rosa ✅) |

---

## 🛠️ Tecnologias Utilizadas

- **Linguagem:** Python 3.10+
- **Estruturas de Dados:** Grafos (`collections.deque`)
- **NLP / LLM:** Groq SDK / Ollama / Expressões Regulares (RegEx)
- **Interface Gráfica:** `ipywidgets` + HTML5/CSS3 Dinâmico

---
