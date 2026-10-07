# Constitution — Regras persistentes do projeto

Regras válidas para todo o código da API Zona Azul Digital.

## 1. Tecnologias

- R1. API REST em Java 21 + Spring Boot 3.3.5, build com Maven. Não há frontend: somente a API.
- R2. Dependências: spring-boot-starter-web e spring-boot-starter-test. Sem Lombok e sem banco de dados.
- R3. Os dados ficam em memória durante a execução.
- R4. A API deve escutar na porta 8002 e subir sem nenhuma variável de ambiente obrigatória.

## 2. Contrato

- R5. Rotas, nomes de campos JSON, valores de status e códigos de erro seguem literalmente a spec.md.
  Os campos usam snake_case em português (valor_centavos, total_bilhetes).
- R6. Dinheiro é sempre um número inteiro em centavos. A API nunca retorna ponto flutuante.

## 3. Erros

- R7. Todo erro tem o corpo `{"erro": "<codigo>"}`, com o código exato da tabela de erros da spec.md.
- R8. Erros de formato (422) são verificados antes de regras de negócio (409): um payload inválido
  nunca dispara conflito.
- R9. A API nunca responde 500 para entrada do cliente.

## 4. Comportamento

- R10. O sistema valida somente o que a spec.md determina; não há regras extras.

## 5. Entregáveis

- R11. Em app/: pom.xml, Containerfile e Dockerfile (mesmo conteúdo, com EXPOSE 8002 e CMD),
  README.md com instruções de execução e testes, e .gitignore ignorando target/.
- R12. Nenhum segredo, senha ou token no código.
- R13. Um método @Test para cada cenário de tests.md.