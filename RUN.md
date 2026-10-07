# RUN — Arquivo mestre

Ponto de entrada do projeto **Zona Azul Digital**. Os demais arquivos são a documentação do sistema;
este arquivo descreve como transformá-la no sistema completo.

## 1. Papel

Você é um desenvolvedor backend sênior em Java e Spring Boot. Sua tarefa é implementar a API
documentada neste repositório seguindo Spec-Driven Development (SDD) e Test-Driven Development (TDD).
Não há ninguém disponível para responder perguntas: tudo o que é necessário está nos arquivos abaixo.

## 2. Documentação do projeto

Leia todos os arquivos, nesta ordem, antes de gerar qualquer código:

| Ordem | Arquivo | Conteúdo |
| --- | --- | --- |
| 1 | `constitution.md` | Regras fixas: tecnologias, porta, formato de dinheiro, datas e erros |
| 2 | `spec.md` | Contrato REST, regras de negócio e critérios de aceite (UC1–UC8) |
| 3 | `plan.md` | Estrutura do projeto e decisões técnicas |
| 4 | `tests.md` | Cenários de teste com casos de borda |
| 5 | `tasks.md` | Ordem de execução |

## 3. Prioridade em caso de conflito

**Contrato REST da `spec.md`** > restante da `spec.md` > `tests.md` > `constitution.md` > `plan.md` > `tasks.md`.

## 4. Execução

1. Leia os cinco arquivos da seção 2.
2. Execute as tarefas de `tasks.md` na ordem.
3. Em cada caso de uso, escreva primeiro os testes de `tests.md` e depois o código.
4. Se for possível executar comandos, rode `mvn test` dentro de `app/` e corrija até todos os testes passarem.
5. Em caso de ambiguidade, adote a interpretação mais simples coerente com a `spec.md` e registre-a
   no `README.md` gerado, na seção "Decisões assumidas".

## 5. Definição de Pronto

- As 6 rotas da `spec.md` existem com método, caminho, campos JSON e status codes idênticos
- Todos os erros respondem com o status e o corpo `{"erro": "<codigo>"}` da tabela de erros
- A API escuta na porta **8002** sem exigir variável de ambiente
- Existe um método `@Test` para cada cenário de `tests.md`
- `Containerfile`, `Dockerfile`, `README.md`, `pom.xml` e `.gitignore` existem em `app/`