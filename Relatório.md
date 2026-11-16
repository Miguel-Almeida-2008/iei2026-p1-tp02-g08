
# Problema a solucionar

Foram-nos apresentados vários problemas para resolver, do qual escolhemos o seguinte:

#### O Solucionador “Funciona na Minha Máquina”: Ambiente de Desenvolvimento em Contentor

Este projeto consiste em criar um ambiente de desenvolvimento hermético e reprodutível baseado em contentores para uma linguagem à escolha (ex: Python, C/C++, Rust ou Node.js), eliminando conflitos de dependências locais.
Assim, seriam necessários vários passos para chegar ao resultado pretendido:
	Criar um Dockerfile com o devido compilador, adicionar suporte ao utilizador não-root (para evitar problemas de autorizações);
	Utilizar o docker compose e volumes, para que toda a compilação, execução e testes ocorram dentro do contentor;
	Incluir uma pequena aplicação que realize testes no código e devolva o feedback, tudo em um único comando;
	E, opcionalmente, utilizar o "dev containers" para a integração automática com o visual studio code
