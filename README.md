# Introducao

GitHub Actions são recursos disponibilizados pelo GitHub que possibilitam a automacao de procedimentos em relacao ao repositório.

Em repositórios públicos, pode ser usado de maneira gratuita, mas para reposiórios privados, apenas uma certa quantia mensal está disponível de maneira gratuita.

mais detalhes: https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions

Em GitHub Actions temos 3 blocos principais:
- Workflows
- Jobs
- Steps

Workflows
São vinculados ao repositório e contém um ou mais Jobs que podem ser iniciados através de eventos

Jobs
define o executador (ambiente de execucao), contém um ou mais steps. Pode ser executados em série ou em paralelo, podendo conter lógicas de condicionais.

Steps
executa um terminal shell script ou uma Action. Pode ser criada ou usar Steps de terceiros (já criados). São executados em ordem e também podem conter lógicas de condicionais.

# Criando um Workflow

Para criar um do zero, o GitHub utiliza arquivos no formato .yml, dentro da pasta .github>workflows

## Exemplo de um Workflow simples
(.github>workflows>first-action.yml)

Workflow simples que executa comandos linux no runner ubuntu-latest

```
# nome do Workflow
name: First Workflow - Hello World
# workflow_dispatch -> inicia manualmente
# https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows
# evento que disparará as jobs
on: workflow_dispatch 
jobs:
  # nome do job, pode ser qualquer coisa, não é uma palavra reservada
  first-job:
    # o Runner é disponibilizado por um serverless do github
    # github actions runners -> https://docs.github.com/pt/actions/using-github-hosted-runners/about-github-hosted-runners
    runs-on: ubuntu-latest
    steps:
      - name: Print greeting
        run: echo "Hello World"
      - name: Print goodbye
        run: echo "Done - bye!"

```

No exemplo acima, podemos ver na prática o que seria um Workflow com um Job e dois Steps.


# Eventos (disparadores de Workflows)

Dentro do objeto ˜on:˜ é escolhido o evento que será o disparador do Workflow. Segue abaixo exemplo de eventos que pode ser usado.

Repositório:
- push
- pull_request
- create (a branch ou tab foi criada)
- fork (repositorio foi forked)
- issues (issue foi criada)
- issue_comment (comentario na issue ou pr foi criado)
- watch (repositorio foi starred)
  etc

Outros:
- workflow_dispatch (ativada manualmente)
- repository_dispatch (REST API request ativa a workflow)
- schedule (agendar o workflow)
- workflow_call (ativada através de outras workflows)

mais detalhes: https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows


# Action

O que são Actions?
Uma Action é uma aplicacao que performa uma automacao frequentemente utilizada. Pode ser criada por conta própria ou utilizada por fontes oficiais de terceiros.

No GitHub temos uma aba de Marktplace onde pode ser encontradas várias Actions criadas por fontes oficiais. Isso facilita a criacao de Workflows, já que não é necessário recriar processos já criados pelo próprio time do GitHub.

## Action na prática

(.github>workflows>second-action-react-test.yml)

O Workflow abaixo é utilizado para rodar os testes da aplicacao encontrada no diretório second-action-react-demo

É um exemplo de utilizacao do dia-a-dia. Podemos criar um Workflow que valida se nenhum teste quebrou antes de enviar a atualizacao para producao ou até mesmo homologacao.

Pode ser notado o uso de Actions a partir do comando "uses", que indica o nome de uma Action encontrada no Marketplace

```
name: Second Workflow - Test React Demo
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      # primeiro é necessário baixar o nosso código do repositório
      # no serverless do github, onde o runner é executado
      - name: Get code
        # invés de utilizar runs-on e passar os comandos, também é possível
        # utilizar actions já prontas que podem ser encontradas no marketplace
        # do github, por exemplo -> https://github.com/marketplace/actions/checkout
        uses: actions/checkout@v3
        #Também é possível encontrar os programas pré-instalados no runner
        # https://docs.github.com/pt/actions/using-github-hosted-runners/about-github-hosted-runners
        # >
        # https://github.com/actions/runner-images/blob/main/images/linux/Ubuntu2204-Readme.md
        # já tem Node instalado, mas também seria possível utilizar uma action do marketplace
      - name: Install NodeJS
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        working-directory: ./second-action-react-demo
        run: npm ci
      - name: Run tests
        working-directory: ./second-action-react-demo
        run: npm test
```

# Rodando Jobs em paralelo

Para rodar os Jobs em paralelo é bem simples, basta apenas adicionar mais um objeto dentro de 'jobs'

Segue exemplo abaixo:

```
name: Third Workflow - Test and Deploy (parallel) React Demo
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      # primeiro é necessário baixar o nosso código do repositório
      # no serverless do github, onde o runner é executado
      - name: Get code
        # invés de utilizar runs-on e passar os comandos, também é possível
        # utilizar actions já prontas que podem ser encontradas no marketplace
        # do github, por exemplo -> https://github.com/marketplace/actions/checkout
        uses: actions/checkout@v3
        #Também é possível encontrar os programas pré-instalados no runner
        # https://docs.github.com/pt/actions/using-github-hosted-runners/about-github-hosted-runners
        # >
        # https://github.com/actions/runner-images/blob/main/images/linux/Ubuntu2204-Readme.md
        # já tem Node instalado, mas também seria possível utilizar uma action do marketplace
      - name: Install NodeJS
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        working-directory: ./second-action-react-demo
        run: npm ci
      - name: Run tests
        working-directory: ./second-action-react-demo
        run: npm test
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Install NodeJS
        uses: actions/setup-node@v3
        with:
          node-version: 18
      - name: Install dependencies
        working-directory: ./second-action-react-demo
        run: npm ci
      - name: Build project
        working-directory: ./second-action-react-demo
        run: npm run build
      - name: Deploy
        run: echo ˜Deploying ...˜
```

# Rodando Jobs em série

Para rodar uma Job em sequencia de uma outra Job é bem simples, basta apenas adicionar o atributo "needs" nas propriedades da Job e setar o identificador da Job que deseja executar anterior a que está sendo configurada

```
name: Forth Workflow - Test and Deploy (sequence) React Demo
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      # primeiro é necessário baixar o nosso código do repositório
      # no serverless do github, onde o runner é executado
      - name: Get code
        # invés de utilizar runs-on e passar os comandos, também é possível
        # utilizar actions já prontas que podem ser encontradas no marketplace
        # do github, por exemplo -> https://github.com/marketplace/actions/checkout
        uses: actions/checkout@v3
        #Também é possível encontrar os programas pré-instalados no runner
        # https://docs.github.com/pt/actions/using-github-hosted-runners/about-github-hosted-runners
        # >
        # https://github.com/actions/runner-images/blob/main/images/linux/Ubuntu2204-Readme.md
        # já tem Node instalado, mas também seria possível utilizar uma action do marketplace
      - name: Install NodeJS
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        working-directory: ./second-action-react-demo
        run: npm ci
      - name: Run tests
        working-directory: ./second-action-react-demo
        run: npm test
  deploy:
    # adiciona o atributo needs com o identificados do Job que seria o executado anterior a esse
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Install NodeJS
        uses: actions/setup-node@v3
        with:
          node-version: 18
      - name: Install dependencies
        working-directory: ./second-action-react-demo
        run: npm ci
      - name: Build project
        working-directory: ./second-action-react-demo
        run: npm run build
      - name: Deploy
        run: echo ˜Deploying ...˜
```

# Multiplos eventos

Também é possível definir mais de um evento que servem de gatilho para execucao do Workflow. Para isso, é necessário apenas adicionar colchetes e separar os eventos por vírgula.
Exemplo:
```
on: [push, workflow_dispatch]
```

# Controle de eventos

Vamos supor que não é desejavel executar algum Workflow específico na branch develop, e queremos que o evento seja disparado apenas para a branch principal, main

Para isso vamos entender dois novos conceitos: Activity Types e Filters


### Activity Types
Cada evento possui tipos de atividades, por exemplo:
pull_request -> opened, closed, edited etc

```
name:
on:
  # setando tipos no evento de pull request
  pull_request:
    types:
      - opened
      - closed
      - edited
  # adicionando um novo evento, mas com tipos padrões
  workflow_dispatch:
jobs:
  ...

```

Pode ser percebido que temos a oportunidade de controlar melhor nossos Workflows

Obs: cada evento tem activity types default caso nenhuma seja especificada

Mais detalhes: https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows

### Forked pull_requests
Quando um repositório é forked, caso as atividades padrões tenha sido setadas para o evento de pull_request, no repositório original o Workflow vai estar escutando o evento, porém uma aprovacao será necessária. Então é necessário ter cuidado com esse possível cenário.

### Filters
Podemos configurar a branch target que o evento será escutado.

Por padrão, os Workflows vão ser executados na branch principal.
Para especificar a branch, é necessário apenas adicionar o atributo "branches" dentro do objeto evento.

Segue exemplo abaixo:

```
name:
on:
  push:
    branches:
      - main
      - dev-* # dev-exemplo
      - feat/** # feat/ajuste-botao-home
      - feature/** # feature/ajuste-contrato
jobs:
  ...

```

Também é possível filtrar por path files
Segue exemplo abaixo:

```
name:
on:
  push:
    branches:
      - main
      - dev-* # dev-exemplo
      - feat/** # feat/ajuste-botao-home
      - feature/** # feature/ajuste-contrato
    # ignora evento quando a alteracao foi nos seguintes arquivos
    paths-ignore:
      - '.github/workflows/*'
    # vai ser ativado apenas se algum desses arquivos forem alterados
    paths:
      - 'src/main/**'
jobs:
  ...

```

# Skip Workflows

Caso a ultima mudanca que ocorreu no código foi irrelevante para o funcionamento da aplicacao, como por exemplo, um comentário, é possível subir as alteracoes sem disparar o fluxo de Workflows

Exemplo: git commit -m "docs: ajuste no readme [skip ci]"

o [skip ci] faz com que os Workflows não sejam disparados.

Mais detalhes: https://docs.github.com/en/actions/managing-workflow-runs/skipping-workflow-runs


# Artefatos

Ao executar um Workflow é possível obter um artefato gerado durante a execucao de um job de Build. Para isso o GitHub Actions possui duas Actions para facilitar a implementacao do Workflow com essa funcionalidade


### Upload Artefato

O primeiro passo seria fazer upload do artefato gerado.

Primeiro é necessário entender como um artefato é gerado após o comando build. Nesse caso foi criada uma pasta com o nome dist com os artefatos dentro.

Segue exemplo abaixo

```
    build:
        name: Build
        needs: test
        runs-on: ubuntu-latest
        steps:
          - name: Get code
            uses: actions/checkout@v3
          - name: Install dependencies
            working-directory: ./first-exercise
            run: npm ci
          - name: Build application
            working-directory: ./first-exercise
            run: npm run build
            # busca nome do arquivo JS na pasta dist
          - name: Upload artifact
            uses: actions/upload-artifact@v3
            with:
              name: dist-files
              path: ./first-exercise/dist
```

Para fazer o upload do artefato, foi implementado o step com o nome "Upload artifact" que usa uma Action pré pronta do Marketplace. Foi necessário apenas usar o objeto with com os atributos indicando o nome do artefato que será feito o upload e o path dele do repositório.


### Download Artefato

Para fazer download de um artefato é necessário ter feito o Upload antes (exemplo no tópico acima)

```
    deploy:
        name: Deploy
        needs: build
        runs-on: ubuntu-latest
        steps:
          # baixar artefato gerado no passo anterior
          - name: Get build artifacts
            uses: actions/download-artifact@v3
            with:
              name: dist-files
          - name: Output contents
            run: ls

```

Para realizar o download foi implementado o step com nome "Get build artifacts" que usa uma Action pré pronta do Marketplace. Foi necessário apenas usar o objeto with com o atributo name indicando o nome do artefato que foi feito o upload (nome de acordo com o tópico anterior)


# Gerando um Output

Um output pode ser utilizado em Jobs seguintes como parametro ou qualquer que seja o caso de uso.
Para gerar um output em um Job é necessário utilizar o objeto outputs dentro do escopo do Job.

Segue exemplo:


```
    build:
        name: Build
        needs: test
        runs-on: ubuntu-latest
        outputs:
          script-file: ${{ steps.publish.outputs.script-file }}
        steps:
        ...
        - name: Publish JS filename
          id: publish
          run: find ./first-exercise/dist/assets/*.js -type f -execdir echo 'script-file={}' >> $GITHUB_OUTPUT ';'
```

pode ser percebido que para acessar o texto que foi gerado como output, foi necessário
utilizar o caminho do texto pelo id que foi escolhido no step

id: publish

em run foi descrito um script que escreve o nome do arquivo .js no contexto do $GITHUB_OUTPUT

Para utilizar o output basta acessar utilizando o objeto needs e utilizando o caminho onde foi setado o objeto outputs

Segue exemplo:

```
          - name: Output filename
          # usando o objeto needs acessa o job que esse job depende (para output é a melhor opcao) e acessa o atributo script-file para acessar o filename
            run: echo "${{ needs.build.outputs.script-file }}"
```


# Utilizando Cache em Steps que se repetem

Para utilizar o recurso de cache, o GitHub já tem uma Action criada:
actions/cache@v3

É necessário criar o cache antes do recurso que será "cacheado" no exemplo abaixo foi criado um cache da pasta /.npm,
que é criada através do comando "npm ci" no step seguinte.

O cache é aplicado entre os Workflows/ Jobs e Steps. Para usá-lo é necessário o uso da Action.

Para manter o cache, é criado um hash do arquivo package-lock.json. Caso o arquivo mude
o hash também mudará, e se o hash mudar, um novo cache será criado.

O comando "npm ci" também possui um recurso de cache, onde as dependências são baixadas apenas
se a pasta /.npm não existir.

```
name: First Exercise - Output e Artefatos
on: push
jobs:
    lint:
        name: Lint
        runs-on: ubuntu-latest
        steps:
          - name: Get code
            uses: actions/checkout@v3
          - name: Cache dependencies
            uses: actions/cache@v3
            with:
              # se já existe a pasta .npm, ele verifica o hash do arquivo package-lock.json, se for o mesmo hash, utilizado cache
              # caso contrário, gera outro
              # https://github.com/actions/cache
              path: ˜/.npm 
              # hashFiles produz um hash para os arquivos, facilitando a identificacao caso haja alguma mudanca para atualizar o cache
              key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
          # Caso haja a pasta .npm, o comando npm ci não vai baixar novamente
          # o cache é usado para a pasta .npm permanecer caso nao haja mudancas no arquivo package-lock.json
          - name: Install dependencies
            working-directory: ./first-exercise
            run: npm ci
```

A lógica do cache será em cima da pasta /.npm, então caso seja necessário utilizar o cache em outros Jobs, será necessário utilizar o step
onde é usado a Action de cache.

Segue exemplo abaixo, onde o cache é utilizando no job "lint", "test" e "build":

```
name: First Exercise - Output e Artefatos
on: push
jobs:
    lint:
        name: Lint
        runs-on: ubuntu-latest
        steps:
          - name: Get code
            uses: actions/checkout@v3
          - name: Cache dependencies
            uses: actions/cache@v3
            with:
              # se já existe a pasta .npm, ele verifica o hash do arquivo package-lock.json, se for o mesmo hash, utilizado cache
              # caso contrário, gera outro
              # https://github.com/actions/cache
              path: ˜/.npm 
              # hashFiles produz um hash para os arquivos, facilitando a identificacao caso haja alguma mudanca para atualizar o cache
              key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
          # Caso haja a pasta .npm, o comando npm ci não vai baixar novamente
          # o cache é usado para a pasta .npm permanecer caso nao haja mudancas no arquivo package-lock.json
          - name: Install dependencies
            working-directory: ./first-exercise
            run: npm ci
          - name: Lint
            working-directory: ./first-exercise
            run: npm run lint
    test:
        name: Test
        needs: lint
        runs-on: ubuntu-latest
        steps:
          - name: Get code
            uses: actions/checkout@v3
          - name: Cache dependencies
            uses: actions/cache@v3
            with:
              path: ˜/.npm 
              # hashFiles produz um hash para os arquivos, facilitando a identificacao caso haja alguma mudanca para atualizar o cache
              key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
          - name: Install dependencies
            working-directory: ./first-exercise
            run: npm ci
          - name: Test
            working-directory: ./first-exercise
            run: npm run test
    build:
        name: Build
        needs: test
        runs-on: ubuntu-latest
        outputs:
          script-file: ${{ steps.publish.outputs.script-file }}
        steps:
          - name: Get code
            uses: actions/checkout@v3
          - name: Cache dependencies
            uses: actions/cache@v3
            with:
              path: ˜/.npm 
              # hashFiles produz um hash para os arquivos, facilitando a identificacao caso haja alguma mudanca para atualizar o cache
              key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
          - name: Install dependencies
            working-directory: ./first-exercise
            run: npm ci
          - name: Build application
            working-directory: ./first-exercise
            run: npm run build
            # busca nome do arquivo JS na pasta dist
          - name: Publish JS filename
            id: publish
            run: find ./first-exercise/dist/assets/*.js -type f -execdir echo 'script-file={}' >> $GITHUB_OUTPUT ';'
          - name: Upload artifact
            uses: actions/upload-artifact@v3
            with:
              name: dist-files
              path: ./first-exercise/dist
```

### Invalidando cache

No contexto acima, o cache é invalidado quando o arquivo package-lock.json é modificado.

# Variáveis de ambiente

Utilizar variáveis de ambiente em um sistema é muito comum, ainda mais quando temos mais de um ambiente
como por exemplo: dev, hom e prod.

Variáveis de ambiente são a melhor opção quando é necessário utilizar um não estático.

## env

É possível utilizar variáveis de ambiente direto no escopo do Workflow, Job ou Step (caso a mesma seja criada em um escopo mais específico, irá 
sobrescrever a do escopo anterior)

Segue exemplo abaixo - Workflow com variáveis de ambiente no escopo do Workflow e também do Job

escopo do Workflow pode ser resgatada da seguinte maneira -> $MONGODB_USERNAME
escopo do Job pode ser resgatada da seguinte maneira -> ${{ env.MONGODB_USERNAME }}

```
name: Third - Environment Variables
on:
  push:
    branches:
      - main
      - dev
# variáveis default https://docs.github.com/en/actions/learn-github-actions/variables#default-environment-variables
# variáveis que podem ser usadas em todos os jobs -> escopo do Workflow
env:
  # variáveis que são usadas em ./third-environment-variable/data/database.js
  MONGODB_DB_NAME: gha-demo
jobs:
  test:
    # variáveis que podem ser usadas em todos os steps -> escopo do Job
    # essa não é a melhor maneira de armazenar dados sensíveis como password
    # temos a possibilidade de armazenar no próprio GitHub
    # settings > security > secrets > actions
    env:
      MONGODB_CLUSTER_ADDRESS: estudo.srlzzrn.mongodb.net
      MONGODB_USERNAME: inrikys
      MONGODB_PASSWORD: zNnrllTBJfWIcZOB
      PORT: 8080
    environment: testing
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: npm-deps-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        working-directory: ./third-environment-variables
        run: npm ci
      - name: Run server
        working-directory: ./third-environment-variables
        # a variavel de ambiente em um runner linux pode ser acessa usando $ e o nome da variável
        run: npm start & npx wait-on http://127.0.0.1:$PORT
      - name: Run tests
        working-directory: ./third-environment-variables
        run: npm test
      - name: Output information
        run: |
          echo "MONGODB_USERNAME = ${{ env.MONGODB_USERNAME }}"
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        # MONGODB_USERNAME está no escopo do Job test
        # MONGODB_DB_NAME está no escopo do Workflow
        run: |
          echo "MONGODB_USERNAME: ${{ env.MONGODB_USERNAME}}"
          echo "MONGODB_DB_NAME: $MONGODB_DB_NAME"

```


## Secrets

Em caso de dados sensíveis, como password e usuário, é possível armazenar
o valor no próprio repositório, em configurações

![Settings do repositório](https://i.imgur.com/hCF7O6J.png)

Na aba Security, no lado esquerdo, clicar em Secrets and variables > Actions

![Secrets and variables](https://i.imgur.com/O2K15bv.png)

Basta apenas clicar em New repository secret, e adicionar a chave e valor
![Adicionar secrets ou variaveis](https://i.imgur.com/sFU0e3r.png)


Para utilizar os secrets criados, segue exemplo abaixo:

é possível acessar através de ${{ secrets.NOME_SECRET }}, porém ao tentar exibir no terminal, o GitHub irá anonimizar 
escondendo o valor

```
name: Third - Secrets
on:
  push:
    branches:
      - main
      - dev
env:
  # variáveis que são usadas em ./third-environment-variable/data/database.js
  # https://docs.github.com/pt/actions/security-guides/using-secrets-in-github-actions
  MONGODB_DB_NAME: ${{ secrets.MONGODB_DB_NAME }}
jobs:
  test:
    # settings do repositório > security > secrets > actions
    env:
      MONGODB_CLUSTER_ADDRESS: estudo.srlzzrn.mongodb.net
      MONGODB_USERNAME: ${{ secrets.MONGODB_USERNAME }}
      MONGODB_PASSWORD: ${{ secrets.MONGODB_PASSWORD }}
      PORT: 8080
    environment: testing
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: npm-deps-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        working-directory: ./third-environment-variables
        run: npm ci
      - name: Run server
        working-directory: ./third-environment-variables
        # a variavel de ambiente em um runner linux pode ser acessa usando $ e o nome da variável
        run: npm start & npx wait-on http://127.0.0.1:$PORT
      - name: Run tests
        working-directory: ./third-environment-variables
        run: npm test
      - name: Output information
        # GitHub deve esconder o valor dos secrets
        run: |
          echo "MONGODB_USERNAME = ${{ secrets.MONGODB_USERNAME }}"
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        # GitHub deve esconder o valor dos secrets
        run: |
          echo "MONGODB_USERNAME: ${{ env.MONGODB_USERNAME}}"
          echo "MONGODB_DB_NAME: $MONGODB_DB_NAME"
```

## Environment

Caso a aplicação tenha mais de um ambiente, o uso de Environment no repositório do GitHub é a melhor opção.

![Settings do repositório](https://i.imgur.com/hCF7O6J.png)

Clicar em Environments, na aba Code and automation

![Environments](https://i.imgur.com/THEnGlD.png)

Adicionar as Secrets

![Environments Secrets](https://i.imgur.com/I2BYLdC.png)

Também é possível escolher uma branch específica para a utilização das Secrets e variáveis do Environment

![Environment Conditions](https://i.imgur.com/JNV2TQq.png)



# Controlando fluxo de Jobs e Steps

## Condição IF

É possível controlar o fluxo com condição IF em um determinado trecho do script, entre Jobs ou Steps

No exemplo abaixo é utilizado uma condição IF após o comando de rodas os testes ser executado.
O IF utilizado no Step seguinte ao que executa os testes.

Para o IF funcionar corretamente em um contexto de falha, foi necessário acrescentar uma função especial do GitHub Actions,
a função failure(). Ela verifica se houve alguma falha nos Steps e Jobs anteriores. Caso ela não seja adicionada, quando 
um dos Steps ou Jobs anteriores falharem, o Workflow para de rodar, e com a função failure(), dá para escolher
qual Step será executado se algo falhar.

Dentro do IF também usamos o objeto de contexto "steps", para capturar o resultado do Step com id = run-tests.

```
name: Step if condition - Execution flow
on:
  push:
    branches:
      - main
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Cache dependencies
        id: cache
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        working-directory: ./forth-execution-flow
        run: npm ci
      - name: Test code
        # o uso do id permite que qualquer step seja referenciado
        id: run-tests
        working-directory: ./forth-execution-flow
        run: npm run test
        # Roda apenas se algum teste falhar
      - name: Upload test report
        if: ${{ failure() && steps.run-tests.outcome == 'failure' }}
        uses: actions/upload-artifact@v3
        with:
          name: test-report
          path: ./forth-execution-flow/test.json
```

tem 4 funções especiais
failure(), success(), always(), cancelled()
failure -> retorna true quando qualquer Step ou Job anterior falhar
success -> retorna true quando NENHUM Step ou Job anterior falhar
always -> sempre retorna true, caso sucesso ou falha
cancelled -> retorna true quando o Workflow for cancelado

https://docs.github.com/en/actions/learn-github-actions/expressions

Um outro exemplo de uso para a função failure():
Caso algum Job ou Step falhe, esse Job "report" será executado.
```
  report:
    needs: [primeiro-job-do-workflow, ultimo-job-do-workflow]
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - name: Output failure information
        run: |
          echo "Something went wrong"
          echo "${{ toJson(github) }}"
```

É importante ressaltar que no objeto needs é necessário ter o primeiro e o ultimo job do workflow, para não ser executado antes
que algum Job ou Step falhe.