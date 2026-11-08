# Fundamentos de LLMs

Antes de construir agentes, entender a peça que gera texto: tokens, contexto, temperatura, alucinação e o primeiro contato com a API.

## Introdução

Você provavelmente já utilizou o ChatGPT ou o Gemini para responder perguntas, escrever um texto, resumir um documento ou até gerar código.

Mas você já se perguntou o que acontece por trás da tela quando digita uma pergunta e recebe uma resposta?

Como o sistema consegue interpretar o que você escreveu? De onde vêm as respostas? Por que ele às vezes inventa informações? E como transformar essa tecnologia em uma aplicação que consulta bancos de dados, responde clientes ou executa tarefas automaticamente?

A peça central para responder a essas perguntas chama-se LLM (Large Language Model) ou, em bom português, grande modelo de linguagem. A proposta deste artigo é explicar os fundamentos desse componente e executá-los no laboratório que está neste repositório.

## O que é um LLM

Se alguém perguntasse a você: complete a frase "Água mole em pedra dura…", você provavelmente responderia: "Tanto bate até que fura." A pergunta que fica é: de onde você tirou essa resposta? E automaticamente você responde: "ué, saiu da minha cabeça".

Mas você já se perguntou como conseguiu fazer isso?

Porque aprendeu, ao longo da vida, padrões de linguagem, ou seja, quais palavras costumam aparecer juntas e como elas se relacionam em diferentes situações.

Agora imagine um programa que passou por um treinamento envolvendo uma quantidade gigantesca de textos e aprendeu esses padrões de linguagem.

Esse é o princípio que ajuda a explicar o que é um LLM.

Um LLM é um modelo de inteligência artificial treinado para reconhecer padrões na linguagem e produzir respostas a partir das informações que recebe.

Resumindo em uma frase: você fornece um texto a um modelo e ele produz uma continuação que faz sentido dentro daquele contexto.

Antes de conversar com você, o modelo passa por uma etapa de treinamento. Então podemos dizer que o modelo é uma representação de conhecimento criada em uma etapa anterior ao uso, o treinamento. Durante o treinamento, o algoritmo processa grandes quantidades de texto e ajusta os parâmetros de uma rede neural, geralmente bastante complexa, com milhões, bilhões ou até trilhões de parâmetros. Com isso, ele aprende:

- a relação entre os termos;
- a estrutura das frases;
- muitos dos padrões presentes no conteúdo em que foi treinado.

Esse aprendizado permite que o mesmo modelo execute tarefas bem diferentes: responder perguntas, resumir documentos e até obedecer instruções que nunca recebeu antes naquele formato exato.

Uma comparação honesta: um LLM é como um cérebro de linguagem. Ele completa texto a partir do que já viu. Ele não é um cadastro, nem um banco de dados da sua escola.

## Contexto: como ele utiliza o que aprendeu?

Considere a frase: "O aluno quer aprender…"

Ela pode continuar de várias maneiras:

- O aluno quer aprender inglês.
- O aluno quer aprender matemática.
- O aluno quer aprender programação.

Qual continuação está correta?

Todas podem estar corretas. A diferença está no contexto.

Numa conversa em uma escola de idiomas, "inglês" seria a opção mais provável. Se o assunto fossem cursos de tecnologia, "programação" faria mais sentido. O modelo de linguagem usa esse contexto para calcular quais continuações são mais prováveis.

Essa é uma das ideias fundamentais dos modelos de linguagem: o contexto influencia a resposta.

## O modelo não sabe tudo

Imagine que um aluno pergunte: "Qual é o valor da minha mensalidade?"

O modelo pode explicar o que é uma mensalidade, mas isso não significa que ele conhece o valor específico devido por aquele aluno.

Para responder corretamente, seria necessário consultar o sistema financeiro da escola.

Aqui aparece uma distinção importante:

Conhecer o significado de uma informação não é o mesmo que ter acesso à informação real.

Um LLM utilizado isoladamente não consulta automaticamente o banco de dados da escola.

Mais adiante, veremos como conectar ferramentas que permitem às aplicações consultar essas informações.

Dois pontos importantes já aparecem aqui:

1. O modelo sozinho não consulta nada. Se ele estiver atendendo alunos de uma escola, não vai verificar automaticamente a matrícula de ninguém no banco de dados. Isso é feito por outros componentes acoplados ao modelo.
2. Gerar não é verificar. O modelo aprende durante o treinamento, mas a geração de uma resposta não verifica por si só se cada afirmação é verdadeira. Uma resposta bem escrita ainda pode conter erros.

## Tokens: a unidade básica do modelo

Agora precisamos entender como um computador trabalha com o texto enviado ao modelo. O computador não entende palavras, entende números. Por isso, antes de chegar ao modelo, todo texto passa por um componente chamado tokenizador, que o quebra em pedaços e transforma cada pedaço em um identificador numérico.

Esses pedaços são os tokens. Um token pode ser:

- uma palavra inteira;
- um pedaço de palavra;
- um sinal de pontuação;
- um espaço, que também faz parte da representação.

```text
Texto:       "Eu quero aprender"
                    │
                    ▼
            ┌──────────────┐
            │ Tokenizador  │
            └──────────────┘
                    │
                    ▼
Tokens:      ["Eu", " quero", " aprender"]
IDs:         [ 6789,   43122,     99871 ]      (valores ilustrativos)
```

Cada modelo tem seu próprio vocabulário e suas próprias regras de tokenização. O mesmo texto pode gerar quantidades diferentes de tokens dependendo do modelo usado.

### Uma analogia simples

Imagine que você queira transportar um livro, mas a transportadora só consiga trabalhar com pacotes numerados.

Antes do transporte, será necessário dividir o conteúdo do livro em partes e atribuir um número a cada uma delas.

Algo parecido acontece com o texto enviado a um modelo de linguagem.

O texto é dividido em unidades menores chamadas tokens, que são convertidas em identificadores numéricos.

Esse processo recebe o nome de tokenização.

### Por que tokens importam quando você constrói um agente

Porque tudo é token. As instruções, as mensagens anteriores, os documentos anexados e a resposta gerada consomem tokens. E isso tem três consequências práticas:

**Custo.** Em serviços cobrados por uso, tanto os tokens enviados ao modelo (entrada) quanto os gerados na resposta (saída) entram no cálculo. Os preços podem ser diferentes.

**Tempo.** Respostas mais longas geralmente demoram mais, pois o modelo gera o conteúdo token por token.

**Limites.** Os modelos possuem limites de contexto e de geração de respostas, medidos em tokens. A quantidade de palavras ou caracteres não corresponde necessariamente à quantidade de tokens.

## Como o texto é gerado, token a token

Do ponto de vista técnico, o caminho de uma entrada até a saída é este:

```text
"Eu quero aprender"
        │
        ▼
┌────────────────────┐
│ 1. Tokenização     │  texto ──► IDs numéricos
└────────────────────┘
        │
        ▼
┌────────────────────┐
│ 2. Embeddings      │  cada ID ──► vetor numérico
└────────────────────┘
        │
        ▼
┌────────────────────┐
│ 3. Rede neural     │  Transformer já treinado
│    (Transformer)   │
└────────────────────┘
        │
        ▼
┌────────────────────────────────────┐
│ 4. Distribuição de probabilidades  │
│    inglês      ████████████  0.41  │
│    matemática  ██████        0.22  │
│    IA          █████         0.19  │
│    Python      ████          0.12  │
│    ...                             │
└────────────────────────────────────┘
        │
        ▼
┌────────────────────┐
│ 5. Escolha do      │  greedy ou amostragem
│    próximo token   │
└────────────────────┘
        │
        ▼
"Eu quero aprender inglês" ──► volta para o passo 1
```

Os passos, em detalhe:

1. **Tokenização.** O texto vira uma sequência de IDs.
2. **Embeddings.** Cada ID é convertido em um vetor numérico, a forma como a rede neural de fato "enxerga" o token.
3. **Rede neural.** Os vetores passam por uma rede já treinada. A arquitetura mais popular hoje é o Transformer. Como a rede está treinada, a saída varia conforme a entrada.
4. **Distribuição de probabilidades.** A saída não é uma palavra, mas uma lista de tokens candidatos, cada um com sua probabilidade. O contexto pesa aqui: numa conversa sobre idiomas, "inglês" tende a ficar no topo.
5. **Escolha.** Há duas estratégias principais:
   - **Greedy:** escolhe sempre o token de maior probabilidade.
   - **Amostragem:** sorteia um token respeitando as probabilidades. O mais provável tem mais chance, mas não é garantido.

O token escolhido é anexado à entrada, e o processo se repete: agora "Eu quero aprender inglês" passa pela rede e gera o próximo token. Assim a resposta é construída aos poucos, até o modelo decidir parar ou atingir o limite de saída.

Consequência importante: por causa da amostragem, a mesma entrada não necessariamente gera a mesma saída. Essa imprevisibilidade é uma característica central dos modelos generativos, e você vai precisar lidar com ela ao construir agentes.

## Janela de contexto: a memória tem limite

Imagine que você esteja conversando com um atendente.

No início da conversa, ele recebe poucas informações.

Depois de algum tempo, vocês já falaram sobre cursos, pagamentos, matrícula, horários e reembolsos.

Para responder à próxima pergunta, o atendente precisa considerar as informações relevantes que foram discutidas.

Com um LLM acontece algo parecido.

O conjunto de informações disponibilizado ao modelo para produzir uma resposta é chamado de contexto.

Um LLM não tem memória infinita. Existe um limite de informação que ele consegue considerar em uma única chamada, e esse limite é medido em tokens. É a janela de contexto.

Tudo o que o modelo precisa saber para responder disputa o mesmo espaço:

```text
┌──────────────────── Janela de contexto (em tokens) ────────────────────┐
│                                                                         │
│  Instruções         "Você é o atendente virtual da escola..."           │
│  Histórico          mensagens anteriores da conversa                    │
│  Dados adicionados  documentos relevantes                               │
│  Ferramentas        resultados de consultas a sistemas externos         │
│  Mensagem atual     "Quero saber o valor da minha mensalidade."         │
│  ─────────────────────────────────────────────────────────────────────  │
│  Espaço reservado para a resposta                                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

Dependendo do serviço, há limites específicos para entrada e para saída, além do limite total. Não basta encher a janela de documentos e esperar uma resposta longa: a resposta também precisa caber.

No laboratório, esse teto de saída aparece como `max_output_tokens: 1200` dentro de `conectar()`.

### O modelo é stateless

O modelo não mantém estado entre chamadas. Em um chat com muitas mensagens, a aplicação reenvia o histórico a cada nova chamada. A conversa parece contínua para o usuário, mas para o modelo cada chamada começa do zero, com apenas o que foi enviado nela.

```text
Chamada 1:  [instruções] + [msg 1]
Chamada 2:  [instruções] + [msg 1] + [resp 1] + [msg 2]
Chamada 3:  [instruções] + [msg 1] + [resp 1] + [msg 2] + [resp 2] + [msg 3]
                                                     ...o volume só cresce
```

Isso tem consequências diretas:

- Quem decide o que entra no contexto é a aplicação, não o modelo.
- Se o limite for ultrapassado, o serviço pode retornar um erro. Algumas aplicações cortam mensagens antigas ou resumem o histórico antes de enviar, mas isso depende da implementação. Não presuma que todo modelo faz isso sozinho.
- Uma janela maior permite incluir mais coisa, mas o custo e o tempo continuam proporcionais aos tokens processados.
- Janela maior não garante atenção. Incluir uma informação não significa que o modelo vai dar a ela o peso necessário.

A lição para quem constrói agentes: o contexto é um recurso escasso e precisa ser administrado. Não se manda tudo para o modelo de forma indiscriminada; seleciona-se o que é relevante para cada tarefa.

## Temperatura e variabilidade

Imagine que você peça a duas pessoas para escrever uma história sobre uma viagem espacial.

Uma delas segue uma estrutura bastante previsível.

A outra inventa personagens, situações inesperadas e acontecimentos surpreendentes.

Nos modelos de linguagem, existe um parâmetro que pode ajudar a controlar a variabilidade das respostas: a temperatura.

Em termos simples, ela influencia o processo de escolha dos tokens.

A temperatura é o parâmetro que controla o quanto a escolha do próximo token se afasta do mais provável.

Importante: a temperatura controla a variabilidade das respostas, não sua precisão. Uma temperatura baixa não garante informações corretas, e nem todos os modelos permitem ajustar esse parâmetro.

Em agentes que alimentam outros programas, a regra geral é preferir temperaturas baixas. Variedade é bem-vinda em um texto criativo, não em um campo que outro sistema vai ler.

Observação prática deste laboratório: o perfil `fast` em `config/models.yaml` aponta para `gpt-5-mini` e controla o comportamento com `reasoning_effort: low`. Modelos de raciocínio como esse costumam rejeitar `temperature`. Por isso `01_fundamentos_llm/main.py` não envia temperatura. Ele envia `reasoning.effort`. Consulte a documentação do modelo que estiver no perfil antes de acrescentar parâmetros.

## Capacidades, limitações e alucinações

### O que o LLM faz bem

- Interpretar linguagem natural: entender o que foi escrito, mesmo de forma informal ou ambígua.
- Resumir, estruturar e reformular texto: ele foi treinado com texto e conhece suas estruturas.
- Seguir instruções complexas.

### Onde ele tropeça

- **Conhecimento congelado.** O LLM não é só um gerador de texto, é também um grande repositório de conhecimento obtido no treinamento. Mas esse conhecimento para na data de corte do treinamento.
- **Sem acesso externo por padrão.** Ele não acessa a internet nem o banco de dados. Dá para resolver isso com aplicações de agentes, mas não vem de fábrica.
- **Aritmética e lógica complexa são frágeis.** Para cálculos e regras determinísticas, um algoritmo em uma linguagem de programação é mais confiável.

### Alucinações

O problema mais traiçoeiro: o modelo gera texto plausível, mas incorreto. A resposta soa confiante e bem escrita, e é justamente isso que torna o erro perigoso. Em aplicações críticas, respostas exigem validação externa. Não dá para confiar cegamente.

## Respostas estruturadas

Até aqui falamos de texto, mas um agente frequentemente precisa entregar informação para outro programa, e programas lidam mal com texto livre.

Imagine que você quer classificar o assunto da mensagem de um aluno. Em texto livre, o modelo pode devolver um parágrafo inteiro de explicação. O programa que vai consumir isso prefere algo assim:

```json
{
  "categoria": "reembolso",
  "resumo": "Aluno pede reembolso da mensalidade de outubro após cancelar a matrícula."
}
```

Para isso se define um esquema, que diz:

- quais campos existem;
- quais são obrigatórios;
- quais tipos de valores são aceitos.

Por exemplo: `categoria` precisa ser uma de três opções (`suporte_tecnico`, `matricula` ou `reembolso`), `resumo` tem tamanho máximo e um eventual campo de valor só aceita números.

Mas atenção a duas camadas de validação:

Um JSON sintaticamente válido ainda pode descumprir o esquema, por exemplo, faltando um campo obrigatório. E um JSON que cumpre o esquema ainda pode estar errado no conteúdo. Dependendo do serviço, você precisará verificar a estrutura por conta própria; em todos os casos, precisará validar o formato e o significado do que o modelo gerou.

Este laboratório ainda devolve texto livre, em `resposta.output_text`. O esquema estruturado entra nos artigos seguintes. O que já vale aqui é a regra: texto bem escrito não é informação verificada.

## Afinal, qual é a diferença entre um LLM e um agente de IA?

Até aqui, estudamos um modelo que recebe informações e produz respostas.

Mas imagine que queremos algo mais. Um aluno informa:

"Quero saber se minha matrícula está confirmada."

Para responder, o sistema precisa consultar o cadastro da escola e, depois, pode precisar verificar o pagamento. Em seguida, deve informar o resultado ao aluno.

Nesse cenário, apenas gerar texto não é suficiente.

Precisamos de uma aplicação que coordene o modelo e os recursos necessários para executar a tarefa. Essa aplicação é o que chamamos de agente de inteligência artificial.

Um agente pode utilizar um LLM para interpretar solicitações, decidir quais ferramentas utilizar e trabalhar com os resultados dessas ferramentas, seguindo regras e limites definidos pela aplicação.

```text
Aluno faz uma pergunta
          |
          v
   Aplicação com IA
          |
          v
   Modelo de linguagem
          |
          v
 Consulta uma ferramenta
          |
          v
  Sistema de matrículas
          |
          v
    Retorna os dados
          |
          v
   Modelo prepara resposta
          |
          v
    Aluno recebe resposta
```

O modelo continua sendo importante, mas agora ele participa de um sistema maior.

Cada limitação vista aqui é a motivação de um componente que aparecerá ao longo da série:

| Limitação | O que a aplicação precisa fazer |
| --- | --- |
| O modelo não consulta nada sozinho | Acoplar ferramentas (cadastro, financeiro, documentos) |
| Gerar não é verificar | Validar formato e significado fora do modelo |
| A janela é finita e o modelo é stateless | A aplicação escolhe o que reenviar em cada chamada |
| Tokens custam tempo e dinheiro | Medir `usage` e não mandar contexto à toa |
| A saída varia | Instruções explícitas e, quando couber, temperatura baixa ou esforço de raciocínio baixo |
| Conhecimento de treinamento fica desatualizado | Trazer o dado real no contexto ou numa ferramenta |

Agora vamos ver o primeiro desses pontos na prática: chamar o modelo sem dado nenhum da escola e observar se ele inventa a resposta.

## Laboratório

### Objetivo

Fazer uma pergunta ao modelo sem fornecer o contexto necessário para respondê-la e observar se ele inventa a resposta.

A pergunta é: "Qual é a duração do curso de Python da escola?". Não há documento, banco de dados ou qualquer informação sobre a escola na chamada. Um modelo bem instruído deveria admitir que não sabe.

### Estrutura do projeto

O laboratório deste artigo já está no repositório, com estes arquivos:

```text
ai-agentic-em-laboratorios/
├── .env                         # chave da API (não versionar)
├── .gitignore                   # ignora .env, .venv e bytecode
├── requirements.txt             # openai, python-dotenv, pyyaml
├── config/
│   └── models.yaml              # perfil fast
└── 01_fundamentos_llm/
    ├── artigo.md                # este texto
    └── main.py                  # a chamada à API
```

O `main.py` fica um nível abaixo da raiz. Isso importa, porque o código localiza o `.env` e o `config/` subindo uma pasta a partir dele:

```python
RAIZ = Path(__file__).resolve().parents[1]
```

`parents[0]` é a pasta `01_fundamentos_llm`. `parents[1]` é a raiz `ai-agentic-em-laboratorios`.

A pasta `.venv/` é criada na raiz na hora de executar. Ela não entra no Git: o `.gitignore` já a exclui, junto com o `.env`.

### Pré-requisitos

- Python 3.10 ou superior. Nesta máquina o interpretador é o `python3` (o comando `python` não existe).
- Uma chave de API da OpenAI, criada em [platform.openai.com/account/api-keys](https://platform.openai.com/account/api-keys).
- Permissão para criar um ambiente virtual. O Python do Ubuntu é um ambiente gerenciado externamente (PEP 668): `pip install` solto no sistema é recusado.

### Passo a passo da implementação

Os arquivos abaixo são os do repositório. Se estiver recriando o laboratório do zero, crie-os nesta ordem, sempre a partir da raiz do projeto.

**1. Impedir que a chave e o ambiente virtual entrem no Git.** `.gitignore`:

```gitignore
.env
__pycache__/
*.py[cod]
.venv/
venv/
```

**2. Declarar as dependências.** `requirements.txt`:

```text
openai
python-dotenv
pyyaml
```

São exatamente três pacotes: o cliente da API, a leitura do `.env` e o leitor do YAML de perfis. O `tiktoken`, usado só no experimento 4, não está nesta lista.

**3. Guardar a chave fora do código.** `.env`, na raiz:

```text
OPENAI_API_KEY=sk-...sua-chave-aqui...
PERFIL_MODELO=fast
```

`PERFIL_MODELO` é opcional. Se a variável faltar ou estiver vazia no sentido de não existir no arquivo, `conectar()` usa `fast`. Troque `sk-...sua-chave-aqui...` pela chave real antes de executar. O valor de exemplo não autentica.

**4. Separar o modelo da lógica.** `config/models.yaml`:

```yaml
fast:
  provider: openai
  model: gpt-5-mini
  reasoning_effort: low
```

O perfil `fast` representa o modelo mais rápido e barato usado neste laboratório. Ajuste o nome do modelo para um disponível na sua conta. Você pode criar outros perfis (um mais robusto para tarefas complexas, por exemplo) e escolher entre eles pela variável `PERFIL_MODELO`. O código não cita `gpt-5-mini`: ele lê o perfil.

**5. Fazer a chamada.** `01_fundamentos_llm/main.py`:

```python
from pathlib import Path

import yaml
from dotenv import dotenv_values
from openai import OpenAI

RAIZ = Path(__file__).resolve().parents[1]


def conectar(nome=None):
    ambiente = dotenv_values(RAIZ / ".env")
    chave = ambiente.get("OPENAI_API_KEY", "")
    perfis = yaml.safe_load((RAIZ / "config/models.yaml").read_text(encoding="utf-8"))
    perfil = perfis[nome or ambiente.get("PERFIL_MODELO", "fast")]
    cliente = OpenAI(api_key=chave, timeout=30, max_retries=0)
    parametros = {
        "model": perfil["model"],
        "reasoning": {"effort": perfil["reasoning_effort"]},
        "max_output_tokens": 1200,
        "store": False,
    }
    return cliente, parametros


def main():
    pergunta = "Qual é a duração do curso de Python da escola?"
    cliente, parametros = conectar()
    resposta = cliente.responses.create(
        **parametros,
        input=pergunta,
        instructions="Responda em português. Não invente informações da escola.",
    )
    print("Saída:", resposta.output_text)


if __name__ == "__main__":
    main()
```

### A função `conectar`

Ela prepara tudo o que é necessário para chamar a API e devolve duas coisas: o cliente e os parâmetros da chamada.

Passo a passo:

1. `RAIZ = Path(__file__).resolve().parents[1]` calcula a raiz do projeto: a pasta acima de onde o `main.py` está.
2. `dotenv_values(RAIZ / ".env")` lê o arquivo de variáveis e devolve um dicionário. Dele saem a chave da API e, se existir, o perfil desejado. Essa leitura não usa variáveis já exportadas no shell: o que vale é o arquivo `.env` da raiz.
3. `yaml.safe_load(...)` carrega o `models.yaml` em `perfis`. A linha seguinte escolhe o perfil: o nome passado como argumento, ou o de `PERFIL_MODELO`, ou `fast` como padrão.
4. `OpenAI(api_key=chave, timeout=30, max_retries=0)` cria o cliente que fará as chamadas. A chamada desiste após 30 segundos e não tenta de novo em caso de falha, o que deixa os erros visíveis enquanto você aprende.
5. `parametros` é um dicionário com a configuração da chamada:

| Parâmetro | Valor | Significado |
| --- | --- | --- |
| `model` | do perfil | Qual modelo usar |
| `reasoning.effort` | `low` | Quanto o modelo raciocina antes de responder. Baixo é mais rápido e barato |
| `max_output_tokens` | `1200` | Teto de tokens na resposta, o espaço reservado na janela de contexto |
| `store` | `False` | Não armazena a resposta no provedor para consulta posterior |

### A função `main`

1. Define a pergunta, como se fosse um aluno perguntando.
2. Chama `conectar()` e recebe o cliente e os parâmetros.
3. Chama `cliente.responses.create(...)`, passando:
   - `**parametros`: os dois asteriscos desempacotam o dicionário, transformando cada par em um argumento nomeado. Equivale a escrever `model=...`, `reasoning=...`, `max_output_tokens=...`, `store=...`;
   - `input`: a pergunta do usuário;
   - `instructions`: as instruções para o modelo, "Responda em português. Não invente informações da escola."
4. Imprime `resposta.output_text`, apenas o texto gerado.

Repare no que não está na chamada: nenhum documento, nenhuma tabela de cursos, nenhuma consulta ao sistema da escola. O modelo não tem de onde tirar a duração do curso. É exatamente esse comportamento que queremos observar.

## Execução

Todos os comandos partem da raiz do projeto, `ai-agentic-em-laboratorios/`.

**1. Crie o ambiente virtual e instale as dependências dentro dele.**

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
```

**2. Coloque a chave real em `.env`.** O arquivo versionável não existe de propósito: só o `.env` local, já listado no `.gitignore`.

**3. Rode o laboratório com o Python do ambiente virtual.**

```bash
.venv/bin/python 01_fundamentos_llm/main.py
```

Não use `pip` nem `python` soltos neste ambiente. Os dois falham, e a falha não é do laboratório.

### O que acontece se você ignorar o ambiente virtual

`pip install -r requirements.txt` no Python do Ubuntu devolve:

```text
error: externally-managed-environment

× This environment is externally managed
╰─> To install Python packages system-wide, try apt install
    python3-xyz, where xyz is the package you are trying to
    install.

    If you wish to install a non-Debian-packaged Python package,
    create a virtual environment using python3 -m venv path/to/venv.
    Then use path/to/venv/bin/python and path/to/venv/bin/pip.
```

Em seguida, `python 01_fundamentos_llm/main.py` nem chega a abrir o script:

```text
Comando 'python' não encontrado, você quis dizer:
  comando 'python3' do deb python3
  comando 'python' do deb python-is-python3
```

O interpretador do projeto é `.venv/bin/python`, criado pelo `python3 -m venv .venv`.

### O que acontece se a chave ainda for o placeholder

Com `OPENAI_API_KEY=sk-...sua-chave-aqui...`, a chamada sai do seu computador e a API recusa:

```text
openai.AuthenticationError: Error code: 401 - Incorrect API key provided: sk-...su***********i.... You can find your API key at https://platform.openai.com/account/api-keys.
```

O trecho mascarado `sk-...su***********i....` é o placeholder, não uma chave válida. Troque o valor no `.env` e rode de novo. Não é preciso reinstalar o `.venv`.

## Resultado

Com a chave válida e o perfil `fast`, uma execução deste repositório produziu:

```text
Saída: Não tenho acesso a informações específicas sobre cursos de escolas individuais. Qual escola você quer saber?

Se preferir, posso:
- Ajudar a procurar a duração no site da escola se você me disser o nome ou enviar o link;
- Informar durações típicas de cursos de Python (ex.: cursos intensivos de 1–4 semanas, cursos introdutórios de 8–12 semanas, ou formações mais completas de 3–6 meses).
```

O modelo não inventou a duração do curso da escola. Não apareceu "40 horas" nem "3 meses" como se fosse um fato do cadastro. Ele disse que não tem a informação específica e pediu o nome ou o link.

Há um detalhe útil para o resto da série: no segundo item ele oferece faixas genéricas de cursos de Python. Isso não é o dado da escola. É conhecimento de treinamento, apresentado como típico. A instrução "Não invente informações da escola" segurou o fato que não estava no contexto. Ela não apagou o que o modelo já "sabe" sobre cursos em geral. Quando a resposta precisa ser só o dado da escola, o contexto tem de ser a única fonte permitida, como no experimento 3.

A saída varia entre execuções. Rode algumas vezes e compare.

## Experimentos

O laboratório fica mais interessante quando você mexe nele. Faça uma alteração por vez e volte ao `main.py` original entre um experimento e outro. O original é o arquivo citado acima.

### Experimento 1: remover a instrução contra invenção

Em `main()`, troque as instruções por apenas:

```python
        instructions="Responda em português.",
```

Rode algumas vezes:

```bash
.venv/bin/python 01_fundamentos_llm/main.py
```

O modelo continua admitindo que não sabe, ou passa a chutar uma duração típica de curso de Python? Modelos recentes alucinam cada vez menos, mas isso ainda acontece, e a instrução explícita ajuda a reduzir o risco.

Para provocar mais, experimente uma pergunta que pressupõe a informação:

```python
    pergunta = "Me lembre: quantas horas tem o curso de Python da escola mesmo?"
```

### Experimento 2: olhar a resposta completa

Troque a linha do `print` por:

```python
    print("Saída:", resposta)
```

Agora aparece o objeto completo que a API devolve, e não só o texto: identificador da resposta, modelo usado, status, parâmetros e a contagem de tokens. Uma chamada a um LLM via API retorna muito mais do que o texto, embora o texto seja o que normalmente interessa.

Para focar no que conversamos sobre custo, imprima só o uso:

```python
    print("Saída:", resposta.output_text)
    print("Uso:", resposta.usage)
```

Observe `input_tokens` e `output_tokens`. Repare também que, em modelos de raciocínio, parte dos tokens de saída corresponde ao raciocínio interno, que você paga mesmo sem ver no texto final.

Para comparar, altere `reasoning_effort` no perfil `fast` de `config/models.yaml`:

```yaml
fast:
  provider: openai
  model: gpt-5-mini
  reasoning_effort: medium
```

Rode de novo e compare `resposta.usage` com a execução em `low`. Ao terminar, volte o arquivo para `low`.

### Experimento 3: dar contexto ao modelo

Agora faça o oposto: forneça a informação que faltava, direto nas instruções.

```python
        instructions=(
            "Responda em português. Não invente informações da escola. "
            "Use apenas os dados abaixo.\n\n"
            "Cursos da Escola Exemplo:\n"
            "- Python para Iniciantes: 40 horas, 8 semanas\n"
            "- Python para Dados: 60 horas, 12 semanas\n"
        ),
```

O modelo agora tem de onde tirar a resposta. Repare que o curso "de Python" é ambíguo: ele escolhe um, lista os dois ou pergunta qual você quer? E veja em `resposta.usage` como os tokens de entrada aumentaram.

Esta é a versão mais rudimentar de uma ideia que vamos desenvolver na série: o conhecimento do agente vem do contexto que a aplicação monta, e não da memória do modelo.

### Experimento 4: contar tokens antes de enviar

O `tiktoken` não faz parte do `requirements.txt`. Instale-o no mesmo ambiente virtual:

```bash
.venv/bin/pip install tiktoken
```

E rode, num arquivo separado ou num interpretador interativo do venv (`.venv/bin/python`):

```python
import tiktoken

enc = tiktoken.get_encoding("o200k_base")
for texto in [
    "Eu quero aprender",
    "Qual é a duração do curso de Python da escola?",
    "I want to learn",
    "paralelepípedo",
]:
    ids = enc.encode(texto)
    pedacos = [enc.decode([i]) for i in ids]
    print(f"{len(ids):>2} tokens | {texto!r}")
    print("          ", pedacos)
```

Na primeira execução, o `tiktoken` baixa o arquivo de vocabulário, então é preciso estar conectado à internet. O `o200k_base` é o tokenizador dos modelos recentes da OpenAI; outros provedores usam vocabulários próprios.

Compare a quantidade de tokens de frases equivalentes em português e inglês, e veja como palavras longas ou menos comuns são quebradas em vários pedaços. É assim que um "texto curto" pode custar mais tokens do que parece.

## Resumo

- Um LLM produz a continuação mais plausível de um texto, token a token, a partir de padrões aprendidos no treinamento.
- Tokens são a moeda de tudo: custo, tempo de resposta e limites são medidos neles.
- A geração passa por tokenização, embeddings, Transformer e distribuição de probabilidades, e a escolha do próximo token pode ser determinística (greedy) ou sorteada (amostragem).
- A janela de contexto é limitada e o modelo é stateless: cabe à aplicação decidir o que enviar em cada chamada.
- Temperatura baixa para tarefas estruturadas, alta para tarefas criativas. Neste laboratório o perfil `fast` não usa temperatura: usa `reasoning_effort`.
- LLMs alucinam: geram texto plausível e incorreto com aparência de confiança. Aplicações críticas exigem validação externa.
- Respostas estruturadas precisam ser validadas no formato e no significado.
- No laboratório, a chamada sem dado da escola recusou inventar a duração do curso. A aplicação é que monta o contexto. A execução neste ambiente passa pelo `.venv`, e uma chave placeholder produz `401` antes de qualquer resposta.

No próximo artigo, começamos a dar ao modelo aquilo que ele não tem sozinho: os componentes que o transformam, aos poucos, em um agente.
