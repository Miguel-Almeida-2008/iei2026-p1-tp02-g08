
# Problema a solucionar

Foram-nos apresentados vários problemas para resolver, do qual escolhemos o seguinte:

#### O Solucionador “Funciona na Minha Máquina”: Ambiente de Desenvolvimento em Contentor

Este projeto consiste em criar um ambiente de desenvolvimento hermético e reprodutível baseado em contentores para uma linguagem à escolha (ex: Python, C/C++, Rust ou Node.js), eliminando conflitos de dependências locais.
Assim, seriam necessários vários passos para chegar ao resultado pretendido:
	Criar um Dockerfile com o devido compilador, adicionar suporte ao utilizador não-root (para evitar problemas de autorizações);
	Utilizar o docker compose e volumes, para que toda a compilação, execução e testes ocorram dentro do contentor;
	Incluir uma pequena aplicação que realize testes no código e devolva o feedback, tudo em um único comando;
	E, opcionalmente, utilizar o "dev containers" para a integração automática com o visual studio code



Miguel:
# Docker:
## compose.yml

O **compose.yml** é o ficheiro responsável por ditar que imagem o docker irá usar, ou caso seja usado um Docker file, onde ele está localizado, os volumes do docker, o nome do container e as enviroment variables que irão estar no docker.

No contentor criado preferimos usar a imagem nginx:stable-alpine3.24-perl, a qual será automaticamente transferida com o comando ``docker compose up --build`` neste caso uma versão especifica para garantir que o software funcionará em qualquer desktop da mesma forma. O volume definido permite passar todos os ficheiros do projeto para dentro do docker e caso algum ficheiro seja alterado, tanto dentro do docker como fora, as mudanças seram aplicadas ambas dentro e fora do docker. Isto permite ao utilizador alterar o código do programa a qualquer momento, sendo o mesmo atualizado tanto dentro quanto fora do docker.
O campo environment irá buscar as variaveis ao ficheiro ``.env``, neste caso a variavel ``GUI_DISPLAY``. Esta variavel é responsavel por dizer qual irá ser o display usado pelo container para mostrar o conteodo de apps. Isto é necessãrio caso o programa de python tenha GUI, sendo necessário mudar a variavel segundo o sistema operativo onde será executado.

## Dockerfile

O **Dockerfile** é o ficheiro responsavel por instalar dependencias para o programa que queremos executar, tanto do proprio python quanto do proprio sistema. Para instalar as dependencias do sistema optamos por criar o ficheiro ``alpine-packages.txt`` no qual estão escritos os pacotes a instalar um em cada linha. No **Dockerfile** é copiado o ficheiro de texto para dentro da pasta ``/tmp/``. Isto é necessário pois quando é executado o **Dockerfile** os volumes ainda não foram criados logo o ficheiro de texto ainda não existe dentro do docker. Uma exceção a esta regra é a instrução ``CMD``, pois ela é executada após o docker ser criado. Após copiar o ficheiro para o container, é executado o seguinte comando: 
- ``RUN apk add python3 ruff py3-pip git ttf-dejavu $(tr '\n' ' ' < /tmp/alpine-packages.txt)``
Este comando ira inicialmente instalar algumas dependencias necessárias tanto para a execução do programa quanto para a execução de linting (ruff).
O mesmo ira acontecer com o ficheiro ``requirements.txt``, o qual servirá para instalar as dependencias de python necessárias tanto para a execução do programa (pelo ficheiro de texto) quanto para a execução de testes (pytest, pelo comando em si).
Antes de instalar as dependencias irá ser adicionado o caminho onde são instaladas as dependencias de python com o pip para a variavel ``PATH`` para que o sistema reconheça a venv onde irão estar a dependencias execute qualquer programa de python com essas bibliotecas.
No final será executado o ficheiro start.sh o qual, após o docker iniciar, correrá o comando ``sleep infinity`` para que o docker continue a correr no background.
## Devcontainers

Para um melhor uso do nosso projeto adicionamos também a funcionalidade dos **Devcontainers**. Os **Devcontainers** são uma ferramenta que permite interagir e trabalhar em tempo real dentro do container no **VSCode**, só sendo perciso instalar um plugin no mesmo. Com isto, o utilizador poderá abrir o container pelo **VSCode** e mudar e executar o seu programa de python sem qualquer preocupação sobre se irá correr no seu desktop, pois ele irá correr no container sempre com as mesmas dependencias, independentemente de onde esteja.

## Relatorios de Linting e Testes

Os **relatorios de linting** e os **testes** seram executados pelo programa ``runTests.sh`` e serão guardados no ficheiro ``lintingReports.txt``. Os memsos podem ser realizados com um unico comando:
- ``sh runTests.sh``
Os **testes** serão realizados segundo o ficheiro ``tests.py``, inserido ou modificado pelo utilizador, localizado em ``/app/src/app/`` dentro do container e em ``/src/app/`` fora do container. Os **relatorios de linting** serão realizados com o ``ruff`` segundo o ficheiro ``ruff.sh``, localizado em ``/app/scripts/`` dentro do container e em ``/scripts/`` fora do container.

## .gitignore

O **. gitignore** é o ficheiro responsável por ditar quais os ficheiros e pastas que não irão para o repositório. Neste caso só está listado o ficheiro ``lintingReports.txt`` pois é o unico que não é listado automaticamente no seu proprio ``.gitignore``, como por exemplo a pasta ``.ruff_cache``.

## .gitattributes

O **.gitattributes** é o ficheiro responsável por ditar o tipo de formatação de final de linha em todos os ficheiros. isto é necessário pois os ficheiros têm que estar na formatação **LF** para serem executados devidamente dentro do container.