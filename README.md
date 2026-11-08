# Laboratórios: agentes de IA

Laboratórios e artigos que acompanham uma série sobre LLMs, agentes de IA e engenharia de prompts e contexto. Cada pasta numerada tem um `artigo.md` com a explicação e um `main.py` com o código executável.

| Pasta | Assunto |
| --- | --- |
| `01_fundamentos_llm/` | Primeira chamada à API. O modelo deve admitir que não sabe a duração de um curso que não está no contexto. |
| `02_agentes_ia/` | Primeiro agente com function calling. O modelo pede a ferramenta `consultar_matricula`, o Python executa e devolve o resultado. |
| `03_prompts_contexto/` | Separação entre instruções, contexto e pergunta, com o catálogo da escola. |

## Pré-requisitos

- Python 3.10 ou superior.
- Uma chave de API da OpenAI, criada em [platform.openai.com/account/api-keys](https://platform.openai.com/account/api-keys).

## Instalação

Na raiz do projeto:

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
```

No Ubuntu e em outras distribuições com Python gerenciado pelo sistema (PEP 668), `pip install` fora de um ambiente virtual é recusado. Por isso o projeto usa `.venv/`. Use sempre `.venv/bin/python` e `.venv/bin/pip`, não `python` nem `pip` soltos.

## Configuração

Crie o arquivo `.env` na raiz, ao lado de `requirements.txt`:

```text
OPENAI_API_KEY=sua-chave-aqui
PERFIL_MODELO=fast
```

- `OPENAI_API_KEY` é a sua chave. O arquivo `.env` está no `.gitignore` e não deve ser versionado.
- `PERFIL_MODELO` é opcional. Se faltar, o código usa `fast`.

O perfil de modelo fica em `config/models.yaml`:

```yaml
fast:
  provider: openai
  model: gpt-5-mini
  reasoning_effort: low
```

Para usar outro modelo, crie outro perfil nesse arquivo e troque `PERFIL_MODELO`. O código não cita o nome do modelo.

## Execução

Rode cada laboratório a partir da raiz do projeto:

```bash
.venv/bin/python 01_fundamentos_llm/main.py
.venv/bin/python 02_agentes_ia/main.py
.venv/bin/python 03_prompts_contexto/main.py
```

A resposta do modelo pode variar entre execuções. O dado que o código fornece (como a duração do curso no catálogo) deve se repetir.

Se a chave estiver ausente ou errada, a API responde com erro de autenticação (`401`). No laboratório 03, um valor que começa com `COLOQUE_` é recusado antes da chamada, com uma mensagem própria.

## Experimentos

Cada artigo traz experimentos para alterar um detalhe por vez (instruções, contexto, ferramentas, histórico) e observar o efeito. Faça uma mudança por vez e volte ao código original antes de testar outra.

## Estrutura

```text
.
├── .env                      # sua chave (não versionar)
├── .gitignore
├── README.md
├── requirements.txt
├── config/
│   └── models.yaml           # perfis de modelo
├── 01_fundamentos_llm/
│   ├── artigo.md
│   └── main.py
├── 02_agentes_ia/
│   ├── artigo.md
│   └── main.py
└── 03_prompts_contexto/
    ├── artigo.md
    └── main.py
```
