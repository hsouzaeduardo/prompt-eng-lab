# prompt-eng-lab

Notebooks de laboratório do curso de Engenharia de Prompt / LLMs.

| Notebook | Tema | Precisa de chave de API? |
|---|---|---|
| [`Mod02.ipynb`](Mod02.ipynb) | Chat completion, streaming e geração de imagem (DALL·E 3) com Azure OpenAI | **Sim** (Azure OpenAI) |
| [`Mod03.ipynb`](Mod03.ipynb) | Agente RAG corporativo com LangChain + Chroma em memória | **Sim** (Azure OpenAI) |
| [`Mod04.ipynb`](Mod04.ipynb) | Otimização sistemática de prompts (métrica → dataset → baseline → iteração) | **Não** — roda com simulador |

> Comece pelo **Mod04** se você ainda não tem credenciais: ele roda 100% offline, com um LLM simulado.

---

## Opção 1 — Google Colab (recomendado para a aula)

Não precisa instalar nada na sua máquina. Basta uma conta Google.

### Abrir os notebooks

- [Abrir Mod02 no Colab](https://colab.research.google.com/github/hsouzaeduardo/prompt-eng-lab/blob/main/Mod02.ipynb)
- [Abrir Mod03 no Colab](https://colab.research.google.com/github/hsouzaeduardo/prompt-eng-lab/blob/main/Mod03.ipynb)
- [Abrir Mod04 no Colab](https://colab.research.google.com/github/hsouzaeduardo/prompt-eng-lab/blob/main/Mod04.ipynb)

Depois de abrir, clique em **Arquivo → Salvar uma cópia no Drive** para poder editar e manter suas alterações.

### Rodar

1. Execute a célula de `%pip install ...` (primeira célula de código). No Colab isso leva ~1 minuto.
2. Se o Colab pedir para **reiniciar a sessão** depois do install, aceite (Ambiente de execução → Reiniciar sessão) e continue da célula seguinte.
3. Execute as células **na ordem**, de cima para baixo (`Shift + Enter`).

### Credenciais no Colab (Mod02 e Mod03)

No Colab **não existe arquivo `.env`**. Use os **Secrets** do Colab (ícone 🔑 na barra lateral esquerda):

1. Crie dois secrets: `AZURE_OPENAI_ENDPOINT` e `AZURE_OPENAI_KEY`.
2. Ative o toggle **"Acesso ao notebook"** em cada um.
3. Adicione uma célula **antes** da célula que cria o client, com:

```python
import os
from google.colab import userdata

os.environ["AZURE_OPENAI_ENDPOINT"] = userdata.get("AZURE_OPENAI_ENDPOINT")
os.environ["AZURE_OPENAI_KEY"] = userdata.get("AZURE_OPENAI_KEY")
```

O `load_dotenv()` que já existe no notebook simplesmente não encontra `.env` e não faz nada — as variáveis acima já estarão no ambiente e `os.getenv(...)` vai funcionar normalmente.

> ⚠️ **Nunca** cole a chave direto em uma célula de código de um notebook que você vai versionar ou compartilhar.

### Cuidados no Colab

- A sessão é **efêmera**: ao desconectar, arquivos gerados (ex.: `output.png` do Mod02) e a base Chroma em memória do Mod03 são perdidos. É só rodar de novo.
- Sessões ociosas caem depois de ~90 minutos. Se der erro de variável indefinida, rode as células desde o começo.
- Para baixar um arquivo gerado: painel **Arquivos** (📁) → clique com o botão direito no arquivo → *Fazer download*.

---

## Opção 2 — Notebook local

### Pré-requisitos

- Python **3.10+** (`python --version`)
- VS Code com a extensão **Jupyter**, ou o Jupyter Lab

### Passo a passo

Clone o repositório:

```bash
git clone https://github.com/hsouzaeduardo/prompt-eng-lab.git
cd prompt-eng-lab
```

Crie e ative um ambiente virtual:

**Windows (PowerShell)**
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**macOS / Linux**
```bash
python3 -m venv .venv
source .venv/bin/activate
```

> Se o PowerShell bloquear a ativação, rode uma vez:
> `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`

Instale as dependências (ou deixe as células `%pip install` dos notebooks fazerem isso por você):

```bash
pip install --upgrade pip
pip install jupyterlab ipykernel python-dotenv

# Mod02
pip install openai requests

# Mod03
pip install langchain langchain-openai langchain-chroma langchain-text-splitters langgraph chromadb tiktoken

# Mod04
pip install pandas
```

Abra os notebooks:

```bash
jupyter lab        # ou: code .
```

No VS Code, clique em **Select Kernel** no canto superior direito do notebook e escolha o Python do `.venv`.

### Credenciais locais (Mod02 e Mod03)

Crie um arquivo `.env` na raiz do projeto (mesma pasta dos notebooks):

```env
AZURE_OPENAI_ENDPOINT=https://SEU-RECURSO.openai.azure.com/
AZURE_OPENAI_KEY=sua-chave-aqui
```

O `load_dotenv()` dos notebooks carrega esse arquivo automaticamente.

> ⚠️ O `.env` **não deve ser commitado**. Confira que ele está no `.gitignore`.

---

## Sobre os nomes dos modelos (Azure OpenAI)

Nos notebooks, os parâmetros `model=` / `azure_deployment=` referem-se ao **nome do deployment** no seu recurso Azure OpenAI — não ao nome do modelo. Se no seu portal os deployments tiverem outro nome, ajuste:

| Notebook | Onde | Valor padrão |
|---|---|---|
| Mod02 | chat e streaming | `gpt-4o` |
| Mod02 | imagem | `dall-e-3` |
| Mod03 | chat | `gpt-4o` |
| Mod03 | embeddings | `text-embedding-3-small` |

---

## Mod04 — trocando o simulador por um modelo real (opcional)

O Mod04 roda por padrão com `BACKEND = "simulador"`, sem nenhuma chave. Para usar um modelo de verdade, edite a célula de configuração do backend:

```python
BACKEND = "gemini"   # "simulador" | "gemini" | "anthropic" | "azure"
```

e forneça a chave correspondente via variável de ambiente:

| Backend | Variável | Pacote |
|---|---|---|
| `gemini` | `GEMINI_API_KEY` | `pip install google-genai` |
| `anthropic` | `ANTHROPIC_API_KEY` | `pip install anthropic` |
| `azure` | `AZURE_OPENAI_ENDPOINT` + `AZURE_OPENAI_KEY` | `pip install openai` |

No Colab, use os Secrets como mostrado acima. Rodar com modelo real **gera custo** e as métricas deixam de ser determinísticas (os resultados variam entre execuções) — é justamente esse o ponto do módulo.

---

## Problemas comuns

| Sintoma | Causa provável | Solução |
|---|---|---|
| `AssertionError: Faltou AZURE_OPENAI_ENDPOINT no .env` | Credenciais não carregadas | Local: confira o `.env` na mesma pasta. Colab: rode a célula dos Secrets antes. |
| `NameError: name 'client' is not defined` | Célula de configuração não foi executada | Rode as células na ordem, de cima para baixo. |
| `ModuleNotFoundError` | Kernel errado ou install não rodou | Confirme o kernel do `.venv`; rode a célula `%pip install`. |
| `401` / `Access denied` | Chave ou endpoint incorretos | Revalide no portal do Azure; o endpoint termina com `/`. |
| `DeploymentNotFound` (404) | Nome do deployment diferente | Ajuste `model=` / `azure_deployment=` (ver tabela acima). |
| `429 Rate limit` | Muitas chamadas / quota baixa | Aguarde alguns segundos e rode novamente. |
| Import falha logo após o `%pip install` no Colab | Sessão precisa reiniciar | Ambiente de execução → Reiniciar sessão, e continue. |
