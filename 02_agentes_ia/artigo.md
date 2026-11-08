# Agentes de IA

Quando gerar texto não basta: ferramentas, o ciclo do agente e o primeiro agente que consulta um sistema de verdade.

## Introdução

No artigo anterior, vimos que um LLM é muito bom em interpretar e produzir linguagem, mas que, sozinho, ele não consulta nada. Ele pode explicar o que é uma matrícula, mas não sabe se a sua matrícula está ativa.

E é justamente esse tipo de pergunta que um sistema de atendimento precisa responder.

Então como fazer o modelo sair do mundo das palavras e passar a consultar sistemas, buscar dados e executar tarefas? Quem decide o que deve ser consultado? Quem de fato executa a consulta? E como o resultado volta para virar uma resposta que o usuário entende?

A resposta para essas perguntas tem nome: agente de IA. A proposta deste artigo é explicar o que é um agente, como ele usa ferramentas e construir, no laboratório, o nosso primeiro agente.

## O que é um agente de IA

Imagine a secretaria de uma escola.

Um aluno chega ao balcão e pergunta: "Minha matrícula M001 está ativa?"

O atendente não sabe a resposta de cabeça. Mas ele sabe o que fazer: entende a pergunta, percebe que precisa consultar o sistema acadêmico, digita o código da matrícula, lê o resultado na tela e responde ao aluno com suas próprias palavras.

Repare que o atendente fez três coisas diferentes:

1. Interpretou o pedido.
2. Agiu, consultando um sistema.
3. Respondeu, transformando o dado da tela em uma frase.

Um agente de IA faz exatamente isso. O LLM cuida da interpretação e da resposta. A ação fica por conta das ferramentas. E uma aplicação coordena tudo.

De forma resumida:

Um agente de IA é uma aplicação que usa um LLM para interpretar solicitações, decidir quais ferramentas utilizar e trabalhar com os resultados dessas ferramentas, seguindo regras e limites definidos pela aplicação.

```text
┌──────────────────────────── Agente de IA ────────────────────────────┐
│                                                                       │
│   ┌─────────────┐     ┌─────────────┐     ┌──────────────────────┐   │
│   │ Instruções  │     │     LLM     │     │     Ferramentas      │   │
│   │ (regras e   │ ──► │ (interpreta │ ◄─► │ (consultar matrícula,│   │
│   │  limites)   │     │  e decide)  │     │  mensalidade, etc.)  │   │
│   └─────────────┘     └─────────────┘     └──────────────────────┘   │
│                              ▲                                        │
│                              │                                        │
│                     ┌────────────────┐                                │
│                     │   Histórico    │                                │
│                     │ (o que já      │                                │
│                     │  aconteceu)    │                                │
│                     └────────────────┘                                │
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘
```

A diferença em relação ao artigo anterior é sutil, mas fundamental. Antes, o modelo respondia. Agora, o modelo decide.

## Ferramentas: as mãos do agente

Uma ferramenta é qualquer recurso que a aplicação disponibiliza para o agente: uma função Python, uma consulta a um banco de dados, uma chamada a uma API, a leitura de um arquivo.

Na API da OpenAI, esse recurso aparece como function calling, ou chamada de função. No laboratório, a lista vai no parâmetro `tools` de `cliente.responses.create`.

### O modelo não vê a ferramenta, vê a descrição dela

Aqui está um ponto que costuma confundir quem está começando: o modelo não tem acesso ao código da ferramenta. Ele não sabe se ela consulta um PostgreSQL, uma planilha ou se devolve um valor fixo. Isso simplesmente não interessa a ele.

O que o modelo recebe é uma descrição:

- o nome da ferramenta;
- o que ela faz;
- quais parâmetros ela espera.

É com base nessas informações que ele decide se deve usar a ferramenta ou não.

```text
O que o modelo VÊ                        O que o modelo NÃO VÊ
┌───────────────────────────────────┐      ┌───────────────────────────────────┐
│ nome:      consultar_matricula    │      │ def consultar_matricula(codigo):  │
│ descrição: Consulta o status de   │      │     # SELECT status FROM ...      │
│            uma matrícula          │      │     # chamada a uma API interna   │
│ parâmetros: codigo (texto,        │      │     # regras de negócio           │
│             obrigatório)          │      │     ...                           │
└───────────────────────────────────┘      └───────────────────────────────────┘
          a vitrine                                  a cozinha
```

Pense em um cardápio de restaurante. Você escolhe o prato pelo nome e pela descrição, sem nunca entrar na cozinha. Se o cardápio diz apenas "Prato 7", você não tem como saber se é o que quer. Com o modelo é igual: a qualidade da descrição define a qualidade da decisão.

No código, a vitrine é o dicionário `FERRAMENTA`. A cozinha é a função `consultar_matricula`.

### A descrição é um contrato

Os parâmetros da ferramenta são descritos em JSON Schema, o mesmo tipo de esquema que vimos na seção de respostas estruturadas do artigo anterior. Ele diz quais campos existem, quais são obrigatórios e quais tipos de valores são aceitos.

Com a opção `strict` ligada, o modelo é obrigado a gerar argumentos que respeitam exatamente esse esquema: sem campos faltando, sem campos inventados.

No laboratório o esquema tem um único campo, `codigo`, do tipo texto, obrigatório, e `additionalProperties: False` recusa qualquer outro campo.

### O modelo não executa nada

Este é o ponto mais importante do artigo.

Quando o modelo decide usar uma ferramenta, ele não a executa. Ele devolve um pedido, algo como: "quero chamar `consultar_matricula` com `codigo = M001`".

Quem executa é a aplicação. No nosso caso, o próprio Python verifica se o modelo pediu uma ferramenta, chama a função, obtém o resultado e devolve esse resultado ao modelo em uma segunda chamada, para que ele construa a resposta final.

```text
Aluno: "Qual é a situação da minha matrícula M001?"
          │
          ▼
┌───────────────────────────┐
│ 1ª chamada ao modelo      │  recebe: instruções + pergunta + ferramentas
│                           │  devolve: "chame consultar_matricula(M001)"
└───────────────────────────┘
          │
          ▼
┌───────────────────────────┐
│ Python executa a função   │  consultar_matricula("M001")
│                           │  resultado: "ativa"
└───────────────────────────┘
          │
          ▼
┌───────────────────────────┐
│ 2ª chamada ao modelo      │  recebe: instruções + pergunta + pedido
│                           │          + resultado da ferramenta
│                           │  devolve: "A matrícula M001 está ativa."
└───────────────────────────┘
          │
          ▼
Aluno recebe a resposta
```

São duas chamadas ao modelo e, entre elas, uma chamada de função.

Essa separação não é um detalhe técnico. Ela é uma decisão de segurança e de controle: o modelo sugere, a aplicação executa. É a aplicação que decide se aquele pedido é permitido, valida os argumentos e controla o acesso aos sistemas.

## O ciclo do agente: pensar, agir, observar

O fluxo que acabamos de ver é a versão mais simples de um padrão que se repete em praticamente todo agente:

```text
          ┌──────────────────────────────────┐
          │                                  │
          ▼                                  │
   ┌─────────────┐                           │
   │   Pensar    │  o modelo analisa o       │
   │             │  contexto e decide        │
   └─────────────┘                           │
          │                                  │
          ├──── precisa de ferramenta? ──────┤
          │ não                         sim  │
          ▼                              │   │
   ┌─────────────┐               ┌─────────────┐
   │  Responder  │               │    Agir     │  a aplicação executa
   │             │               │             │  a ferramenta
   └─────────────┘               └─────────────┘
                                         │
                                         ▼
                                 ┌─────────────┐
                                 │  Observar   │  o resultado entra
                                 │             │  no histórico
                                 └─────────────┘
```

No laboratório base, o ciclo roda uma única vez: o modelo pede no máximo uma ferramenta e depois responde. `parallel_tool_calls=False` impede que ele peça várias ferramentas na mesma chamada. Em aplicações reais, o agente pode ter várias ferramentas à disposição e girar esse ciclo várias vezes. Por exemplo: consultar a matrícula, depois consultar o financeiro e só então responder. O experimento 5 generaliza o fluxo para esse caso.

Também pode acontecer o contrário: o modelo decidir não usar nenhuma ferramenta. Se o aluno disser apenas "bom dia", não há nada a consultar, e o modelo responde diretamente na primeira chamada.

## Histórico e papéis

Lembra que o modelo é stateless? Ele não guarda nada entre uma chamada e outra.

Isso significa que, na segunda chamada, o modelo não lembra que pediu uma ferramenta. Se a aplicação enviar apenas o resultado (`ativa`), ele não vai saber a que aquilo se refere.

Por isso a aplicação monta um histórico e o reenvia completo. Cada item do histórico tem um papel:

| Item | Quem produziu | Exemplo |
| --- | --- | --- |
| Mensagem com `role: user` | O usuário | "Qual é a situação da minha matrícula M001?" |
| `function_call` | O modelo, na 1ª chamada | pedido para chamar `consultar_matricula` com `{"codigo": "M001"}` |
| `function_call_output` | A aplicação, após executar a função | `"ativa"` |

O pedido de ferramenta e o resultado são ligados por um identificador, o `call_id`. É ele que diz ao modelo: "este resultado é a resposta daquele pedido". Quando há várias ferramentas e várias chamadas, é esse vínculo que evita confusão.

No código, a lista `historico` começa com a pergunta. Depois da primeira chamada, `historico.extend(resposta.output)` recoloca no contexto o que o modelo acabou de produzir, inclusive o pedido. O resultado entra em seguida, com o mesmo `call_id`.

## Cuidados ao dar ferramentas a um agente

Dar ferramentas a um modelo é dar a ele a capacidade de agir. Alguns cuidados aparecem desde o primeiro agente:

- **Argumentos não são confiáveis.** Eles foram gerados por um modelo a partir de um texto escrito por um usuário. Valide-os antes de usar, como faria com qualquer entrada externa. Neste laboratório, `json.loads` só garante que o texto é JSON. A função ainda trata código desconhecido devolvendo `"não encontrada"`.
- **Ferramentas com efeito colateral exigem mais controle.** Consultar uma matrícula é inofensivo. Cancelar uma matrícula ou emitir um reembolso não é. Ações irreversíveis pedem confirmação, permissões e registro.
- **Descrições ruins geram decisões ruins.** Nomes vagos e descrições genéricas fazem o modelo usar a ferramenta errada, ou não usar nenhuma. O experimento 4 mostra isso.
- **Cada volta do ciclo custa.** Cada chamada ao modelo consome tokens e tempo, e o histórico cresce a cada passo. Agentes que giram o ciclo várias vezes precisam de um limite de passos. No experimento 5 o limite é cinco.
- **Instruções importam.** "Não invente dados" e "se faltar o código, peça-o" continuam sendo a primeira linha de defesa contra alucinações.

Agora vamos colocar tudo isso em prática.

## Laboratório

### Objetivo

Construir um agente que responde à pergunta "Qual é a situação da minha matrícula M001?" usando uma ferramenta, e observar cada etapa do processo: a decisão do modelo, a execução da função pelo Python e a construção da resposta final.

A ferramenta será uma função Python simples que devolve um valor fixo. Para o modelo, como vimos, isso não faz diferença: ele só conhece a descrição.

### Estrutura do projeto

O projeto é o mesmo do artigo anterior. Este artigo acrescenta a pasta `02_agentes_ia/`:

```text
ai-agentic-em-laboratorios/
├── .env                         # chave da API (não versionar)
├── .gitignore
├── requirements.txt             # openai, python-dotenv, pyyaml
├── .venv/                       # criado na execução; não versionar
├── config/
│   └── models.yaml              # perfil fast
├── 01_fundamentos_llm/
│   ├── artigo.md
│   └── main.py
└── 02_agentes_ia/
    ├── artigo.md                # este texto
    └── main.py                  # o laboratório deste artigo
```

O `.env`, o `config/models.yaml` e as dependências continuam os mesmos. Não há pacote novo: function calling faz parte do cliente `openai` que o laboratório anterior já instalou.

`RAIZ = Path(__file__).resolve().parents[1]` continua apontando para a raiz, porque `main.py` está um nível abaixo, em `02_agentes_ia/`.

### Pré-requisitos

Os mesmos do artigo anterior:

- Python 3.10 ou superior. Nesta máquina o comando é `python3`.
- Chave real em `.env`. O placeholder `sk-...sua-chave-aqui...` produz `401` antes de qualquer decisão do modelo.
- Ambiente virtual `.venv` na raiz, com as dependências instaladas.

Se o ambiente ainda não existe:

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
```

`pip` e `python` soltos falham neste Ubuntu, pelo mesmo motivo do laboratório anterior: o Python do sistema é gerenciado externamente, e o comando `python` não está instalado.

### Passo a passo da implementação

O arquivo novo é só `02_agentes_ia/main.py`. O restante do projeto não muda.

```python
import json
from pathlib import Path

import yaml
from dotenv import dotenv_values
from openai import OpenAI

RAIZ = Path(__file__).resolve().parents[1]

FERRAMENTA = {
    "type": "function",
    "name": "consultar_matricula",
    "description": "Consulta o status de uma matrícula da escola.",
    "strict": True,
    "parameters": {
        "type": "object",
        "properties": {"codigo": {"type": "string"}},
        "required": ["codigo"],
        "additionalProperties": False,
    },
}


def conectar():
    ambiente = dotenv_values(RAIZ / ".env")
    perfis = yaml.safe_load((RAIZ / "config/models.yaml").read_text(encoding="utf-8"))
    perfil = perfis[ambiente.get("PERFIL_MODELO", "fast")]
    cliente = OpenAI(
        api_key=ambiente.get("OPENAI_API_KEY", ""), timeout=30, max_retries=0
    )
    parametros = {
        "model": perfil["model"],
        "reasoning": {"effort": perfil["reasoning_effort"]},
        "max_output_tokens": 1200,
        "store": False,
    }
    return cliente, parametros


def consultar_matricula(codigo):
    return {"M001": "ativa"}.get(codigo, "não encontrada")


def main():
    pergunta = "Qual é a situação da minha matrícula M001?"
    cliente, parametros = conectar()
    instrucoes = (
        "Responda em português. Para informar o status de uma matrícula, "
        "consulte a ferramenta. Se faltar o código, peça-o. Não invente dados."
    )
    historico = [{"role": "user", "content": pergunta}]
    resposta = cliente.responses.create(
        **parametros,
        instructions=instrucoes,
        input=historico,
        tools=[FERRAMENTA],
        parallel_tool_calls=False,
    )
    chamadas = [item for item in resposta.output if item.type == "function_call"]
    if chamadas:
        historico.extend(resposta.output)
        for chamada in chamadas:
            argumentos = json.loads(chamada.arguments)
            print("Ferramenta solicitada pelo modelo:", chamada.name)
            print("Argumentos:", argumentos)
            observacao = consultar_matricula(**argumentos)
            print("Resultado da ferramenta:", observacao)
            historico.append(
                {
                    "type": "function_call_output",
                    "call_id": chamada.call_id,
                    "output": observacao,
                }
            )
        resposta = cliente.responses.create(
            **parametros, instructions=instrucoes, input=historico
        )
    print("Resposta:", resposta.output_text)


if __name__ == "__main__":
    main()
```

Vamos por partes.

### A descrição da ferramenta: `FERRAMENTA`

Este bloco não é a ferramenta. É um dicionário Python com a descrição da ferramenta, aquilo que será entregue ao modelo para que ele possa decidir se vai usá-la.

| Campo | Valor | Significado |
| --- | --- | --- |
| `type` | `function` | É uma ferramenta do tipo função |
| `name` | `consultar_matricula` | Nome que o modelo cita quando pede para usá-la |
| `description` | Consulta o status de uma matrícula da escola. | O que a ferramenta faz. Junto com os parâmetros, é a base da decisão |
| `strict` | `True` | O modelo deve respeitar exatamente o esquema de parâmetros |
| `parameters` | JSON Schema | Um objeto com um único campo, `codigo`, do tipo texto, obrigatório, e nenhum campo adicional |

### A função `conectar`

É a mesma ideia do artigo anterior, um pouco mais compacta: lê o `.env`, carrega o perfil `PERFIL_MODELO` (ou `fast`) do `models.yaml` e devolve o cliente da OpenAI e os parâmetros da chamada.

Há uma diferença em relação a `01_fundamentos_llm/main.py`: esta versão não aceita o argumento `nome`. O perfil vem só do `.env`. O perfil `fast` continua sendo `gpt-5-mini` com `reasoning_effort: low`. O dicionário `parametros` é o mesmo: modelo, esforço de raciocínio, teto de 1200 tokens de saída e `store: False`.

### A ferramenta: `consultar_matricula`

```python
def consultar_matricula(codigo):
    return {"M001": "ativa"}.get(codigo, "não encontrada")
```

Esta, sim, é a ferramenta. Uma função Python que recebe o código da matrícula e devolve `"ativa"` para `M001` e `"não encontrada"` para qualquer outro código.

Em um sistema real, aqui estaria uma consulta ao banco de dados ou a uma API do sistema acadêmico. Mas, para o modelo, nada muda: ele nunca vê este código.

O nome da função é o mesmo declarado em `FERRAMENTA`. A aplicação, ao receber o pedido `consultar_matricula`, sabe qual função executar. No laboratório base essa ligação é direta: o código chama `consultar_matricula` pelo nome Python. No experimento 5, um dicionário `FUNCOES` faz a ponte pelo nome que o modelo devolve.

### O fluxo principal: `main`

**1. Pergunta, instruções e histórico.**

```python
historico = [{"role": "user", "content": pergunta}]
```

As instruções dizem ao modelo para consultar a ferramenta, pedir o código se ele estiver faltando e não inventar dados. As duas strings entre parênteses formam uma única string; a quebra de linha é só para facilitar a leitura.

O histórico começa com uma única mensagem, com o papel `user` e o conteúdo da pergunta.

**2. Primeira chamada ao modelo.**

```python
resposta = cliente.responses.create(
    **parametros,
    instructions=instrucoes,
    input=historico,
    tools=[FERRAMENTA],
    parallel_tool_calls=False,
)
```

A novidade em relação ao artigo anterior são dois parâmetros:

- `tools`: a lista de ferramentas disponíveis. Aqui, só uma.
- `parallel_tool_calls=False`: impede que o modelo peça várias ferramentas ao mesmo tempo. Ele pode pedir nenhuma ou uma.

**3. Verificar se o modelo pediu uma ferramenta.**

```python
chamadas = [item for item in resposta.output if item.type == "function_call"]
```

Como vimos no experimento 2 do artigo anterior, a resposta da API traz muito mais do que texto. O campo `output` é uma lista de itens. Procuramos os itens do tipo `function_call`. Se a lista estiver vazia, o modelo respondeu direto, sem ferramenta, e o `if` seguinte é pulado.

**4. Executar a ferramenta.**

```python
historico.extend(resposta.output)
```

Antes de tudo, o histórico recebe a saída da primeira chamada, incluindo o pedido de ferramenta. Na segunda chamada, o modelo precisa saber o que ele mesmo pediu.

Depois, para cada chamada (aqui será uma só):

- `json.loads(chamada.arguments)` converte os argumentos, que chegam como texto JSON, em um dicionário Python;
- os `print` mostram qual ferramenta o modelo pediu e com quais argumentos, para acompanharmos a decisão;
- `consultar_matricula(**argumentos)` é onde a função de fato é executada, pelo Python;
- o resultado entra no histórico como um item `function_call_output`, ligado ao pedido pelo `call_id`.

**5. Segunda chamada ao modelo.**

```python
resposta = cliente.responses.create(
    **parametros, instructions=instrucoes, input=historico
)
```

Agora o modelo recebe as instruções e o histórico completo: a pergunta, o pedido de ferramenta e o resultado. Com isso, ele constrói a resposta em linguagem natural. Esta segunda chamada não reenvia `tools`: o trabalho da ferramenta já foi feito.

**6. Resposta final.**

```python
print("Resposta:", resposta.output_text)
```

Se houve ferramenta, `resposta` é a da segunda chamada. Se não houve, é a da primeira. Nos dois casos, `output_text` traz o texto para o usuário.

## Execução

Na raiz do projeto, com o `.venv` já criado e a chave real no `.env`:

```bash
.venv/bin/python 02_agentes_ia/main.py
```

Não use `python 02_agentes_ia/main.py`. Neste ambiente o comando `python` não existe.

### Se a segunda chamada reclamar do raciocínio

Nesta execução, com `gpt-5-mini`, `reasoning_effort: low` e `store: False`, a segunda chamada aceitou o histórico e devolveu a frase final. O erro abaixo não apareceu.

Ele pode aparecer com outro modelo ou outra versão da API: devolver itens de raciocínio no histórico com `store: False` gera uma mensagem dizendo que o item não foi persistido. Nesse caso, peça à API o raciocínio criptografado, acrescentando esta chave ao dicionário `parametros` dentro de `conectar()`:

```python
"include": ["reasoning.encrypted_content"],
```

Assim o item de raciocínio volta no `output` de forma que a chamada seguinte consegue reenviá-lo sem ter sido gravado no provedor.

## Resultado

Uma execução deste repositório, com o perfil `fast`, produziu:

```text
Ferramenta solicitada pelo modelo: consultar_matricula
Argumentos: {'codigo': 'M001'}
Resultado da ferramenta: ativa
Resposta: A matrícula M001 está ativa.
```

Cada linha conta uma parte da história:

1. O modelo decidiu usar a ferramenta certa, a única disponível.
2. Ele extraiu da pergunta o argumento correto, `M001`, e o entregou no formato exigido pelo esquema.
3. O Python executou a função, que devolveu `ativa`.
4. Na segunda chamada, o modelo transformou o dado em uma frase para o aluno.

A frase desta execução foi "A matrícula M001 está ativa." Outra execução pode dizer "Sua matrícula M001 está ativa." O dado não muda. A redação muda, porque a segunda chamada continua sendo geração de texto.

Uma resposta simples e direta. Em um cenário real, o modelo poderia precisar combinar os resultados de várias ferramentas antes de responder. É o que o experimento 5 faz.

## Experimentos

Faça uma alteração por vez e volte ao `02_agentes_ia/main.py` original entre um experimento e outro. Rode sempre com:

```bash
.venv/bin/python 02_agentes_ia/main.py
```

### Experimento 1: pergunta sem o código

```python
pergunta = "Qual é a situação da minha matrícula?"
```

O modelo não tem o argumento obrigatório. Pelas instruções, ele deveria pedir o código em vez de chamar a ferramenta. Repare que, nesse caso, não aparecem as linhas "Ferramenta solicitada" e "Resultado da ferramenta": o modelo respondeu na primeira chamada.

Experimente também remover a frase "Se faltar o código, peça-o" das instruções. Ele passa a inventar um código?

### Experimento 2: matrícula inexistente

```python
pergunta = "Qual é a situação da matrícula M999?"
```

A ferramenta devolve `não encontrada`. Observe como o modelo comunica isso ao aluno. Ele apenas repassa o resultado ou sugere conferir o código?

### Experimento 3: pergunta que não precisa de ferramenta

```python
pergunta = "Bom dia! Vocês atendem aos sábados?"
```

Não há ferramenta para horário de atendimento. O modelo deve responder sem chamar `consultar_matricula` e, pelas instruções, sem inventar dados da escola. É o caso do ciclo que termina em "Responder" sem passar por "Agir".

### Experimento 4: uma descrição ruim

Troque o nome e a descrição da ferramenta por algo vago:

```python
"name": "funcao_auxiliar",
"description": "Função auxiliar.",
```

A aplicação executa `consultar_matricula` diretamente, então a execução continua funcionando se o modelo ainda pedir a ferramenta e mandar `codigo`. O que muda é a decisão: o modelo ainda percebe que deve usar essa ferramenta? É a prova prática de que o modelo escolhe pelo cardápio, não pela cozinha.

Se ele pedir a ferramenta com outro nome de argumento, `consultar_matricula(**argumentos)` quebra. Isso também é informação: o contrato entre a descrição e a função precisa coincidir, e hoje quem garante isso é o código, não um mapa de nomes.

### Experimento 5: transformar o fluxo em um ciclo

O laboratório roda o ciclo uma única vez. Este experimento generaliza para várias ferramentas e várias voltas, com um limite de passos.

Acrescente uma segunda ferramenta, logo abaixo de `consultar_matricula`:

```python
FERRAMENTA_MENSALIDADE = {
    "type": "function",
    "name": "consultar_mensalidade",
    "description": "Consulta o valor e a situação da mensalidade de uma matrícula.",
    "strict": True,
    "parameters": {
        "type": "object",
        "properties": {"codigo": {"type": "string"}},
        "required": ["codigo"],
        "additionalProperties": False,
    },
}


def consultar_mensalidade(codigo):
    return {"M001": "R$ 450,00, em aberto"}.get(codigo, "não encontrada")


FUNCOES = {
    "consultar_matricula": consultar_matricula,
    "consultar_mensalidade": consultar_mensalidade,
}
```

E substitua a função `main` por esta versão:

```python
def main():
    pergunta = "Minha matrícula M001 está ativa? E quanto devo de mensalidade?"
    cliente, parametros = conectar()
    instrucoes = (
        "Responda em português. Use as ferramentas para consultar matrícula e "
        "mensalidade. Se faltar o código, peça-o. Não invente dados."
    )
    historico = [{"role": "user", "content": pergunta}]
    for passo in range(1, 6):
        resposta = cliente.responses.create(
            **parametros,
            instructions=instrucoes,
            input=historico,
            tools=[FERRAMENTA, FERRAMENTA_MENSALIDADE],
            parallel_tool_calls=False,
        )
        chamadas = [item for item in resposta.output if item.type == "function_call"]
        if not chamadas:
            break
        historico.extend(resposta.output)
        for chamada in chamadas:
            argumentos = json.loads(chamada.arguments)
            observacao = FUNCOES[chamada.name](**argumentos)
            print(f"Passo {passo}: {chamada.name}({argumentos}) -> {observacao}")
            historico.append(
                {
                    "type": "function_call_output",
                    "call_id": chamada.call_id,
                    "output": observacao,
                }
            )
    print("Resposta:", resposta.output_text)
```

O que mudou:

- as ferramentas disponíveis agora são duas;
- o dicionário `FUNCOES` liga o nome que o modelo pede à função que o Python executa;
- o fluxo virou um laço: enquanto o modelo pedir ferramentas, a aplicação executa e devolve o resultado; quando ele parar de pedir, temos a resposta final;
- o laço tem um limite de cinco passos, para que um agente confuso não gire indefinidamente consumindo tokens.

Como `parallel_tool_calls` continua desligado, o modelo precisa de duas voltas para consultar as duas ferramentas, e uma terceira chamada para responder. Nesta execução o ciclo produziu:

```text
Passo 1: consultar_matricula({'codigo': 'M001'}) -> ativa
Passo 2: consultar_mensalidade({'codigo': 'M001'}) -> R$ 450,00, em aberto
Resposta: Sua matrícula M001 está ativa.
A mensalidade atual é R$ 450,00 e está em aberto.

Deseja que eu gere/acesse boleto, verifique vencimento ou opções de pagamento/parcelamento?
```

Os dois dados vieram das ferramentas. A última frase não veio de ferramenta nenhuma: o modelo ofereceu boleto, vencimento e parcelamento, coisas que a aplicação não sabe fazer. É o mesmo fenômeno do artigo anterior. A instrução segurou a invenção do status e do valor, e o modelo ainda completou a conversa com uma oferta plausível. Quando a resposta precisa parar no dado consultado, a instrução tem de dizer isso.

Este laço, com poucas linhas, é o coração de praticamente todo framework de agentes. Pensar, agir, observar, repetir.

Há uma limitação no código acima: se o limite de cinco passos for atingido enquanto o modelo ainda pede ferramentas, o `break` não acontece, a última `resposta` ainda é um pedido de ferramenta e `output_text` vem vazio. Uma aplicação real precisa tratar esse caso, por exemplo, avisando o usuário de que não foi possível concluir a tarefa. Ao terminar o experimento, volte o `main.py` para a versão de uma ferramenta.

## Resumo

- Um agente de IA é uma aplicação que usa um LLM para interpretar pedidos, decidir quais ferramentas usar e trabalhar com os resultados, dentro de regras e limites definidos pela aplicação.
- Ferramentas são os recursos que o agente pode acionar: funções, bancos de dados, APIs.
- O modelo não vê o código da ferramenta, apenas o nome, a descrição e os parâmetros. A qualidade da descrição define a qualidade da decisão.
- O modelo não executa a ferramenta: ele pede, a aplicação executa e devolve o resultado.
- No fluxo deste laboratório há duas chamadas ao modelo e, entre elas, a execução de `consultar_matricula`. Se não precisar de ferramenta, o modelo responde já na primeira.
- Como o modelo é stateless, a aplicação mantém o histórico: pergunta, pedido de ferramenta e resultado, ligados pelo `call_id`.
- Argumentos gerados pelo modelo precisam de validação, e ações com efeito colateral exigem controle extra.
- O padrão pensar, agir, observar, repetido em laço com limite de passos, é a base dos agentes que vamos construir daqui para frente.
- Neste ambiente a execução é `.venv/bin/python 02_agentes_ia/main.py`. A chamada com `M001` devolveu `ativa` e a frase "A matrícula M001 está ativa."
