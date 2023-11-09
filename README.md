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

``
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

``

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









