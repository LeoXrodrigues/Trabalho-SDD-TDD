# Tasks — Decomposição em tarefas

Tarefas em ordem de execução, com os cenários de tests.md que cada uma atende.

## Fase 1 — Base

- TK01 — Projeto Maven em app/ com pom.xml, application.properties (server.port=8002) e classe principal.
- TK02 — Classe Tarifa com os parâmetros e bean Clock no fuso -03:00.
- TK03 — ApiException e GlobalExceptionHandler com corpo {"erro": ...}.
- TK04 — Modelo Bilhete, enum StatusBilhete e BilheteRepository em memória.
- TK05 — Classe ApitestTest.java com MockMvc e limpeza do repositório antes de cada teste.

## Fase 2 — Casos de uso

- TK06: CalculadoraValor, com minutos completos, tolerância, frações e teto. Cenários T01 a T07.
- TK07: UC1 e UC8, abrir bilhete com validação de placa e entrada e bloqueio de placa já aberta. Cenários T08 a T14.
- TK08: UC2 e UC5, encerrar e cancelar bilhete. Cenários T15 a T18.
- TK09: UC3 e UC6, listar ativos e histórico por placa, com a ordenação da RN08. Cenários T19 a T21.
- TK10: UC4, relatório diário. Cenários T22 a T24.

## Fase 3 — Entrega

- TK11 — Containerfile e Dockerfile, README.md com execução local, Podman, Docker e testes, e .gitignore.
- TK12 — mvn test com todos os testes passando e conferência das 6 rotas contra a spec.md.
