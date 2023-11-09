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
Define o executador (ambiente de execucao), contém um ou mais steps. Pode ser executados em série ou em paralelo, podendo conter lógicas de condicionais.

Steps
Executa um terminal shell script ou uma Action. Pode ser criada ou usar Steps de terceiros (já criados). São executados em ordem e também podem conter lógicas de condicionais.

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
