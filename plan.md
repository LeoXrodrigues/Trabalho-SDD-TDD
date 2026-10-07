# Plan — Arquitetura e decisões técnicas

## 1. Estrutura

Projeto Maven na pasta app/, pacote com.zonaazul.api, organizado em:

- controller : BilheteController e RelatorioController 
- service : BilheteService (regras dos UCs), CalculadoraValor (RN03–RN06) e RelatorioService 
- repository : BilheteRepository em memória 
- model : Bilhete e enum StatusBilhete 
- dto : records de entrada e saída 
- exception : ApiException e GlobalExceptionHandler 
- config : constantes da tarifa e o Clock 

## 2. Decisões técnicas

- D1. **Persistência em memória** com `ConcurrentHashMap<Long, Bilhete>` e `AtomicLong` para o id.
  Justificativa: a correção sobe apenas o container da aplicação, sem banco, e o enunciado não exige persistência.
- D2. **Parâmetros como constantes** em uma classe `Tarifa` (450, 30, 7000, 15).
  Justificativa: a variante é fixa e a API não pode exigir variável de ambiente.
- D3. **Dinheiro em `long` de centavos**, sem `double` nem `BigDecimal`.
  Justificativa: ponto flutuante acumula erro (0.1 + 0.2 ≠ 0.3), e centavos inteiros eliminam essa classe de bug.
- D4. **Relógio injetável:** um bean `java.time.Clock` no fuso `ZoneOffset.ofHours(-3)`, usado para "agora".
  Justificativa: os testes podem usar um relógio fixo, e todas as datas ficam no fuso -03:00.
- D5. **Datas tratadas como texto na fronteira:** os DTOs recebem e devolvem `entrada` e `saida` como `String`.
  A leitura usa `OffsetDateTime.parse` (erro → 422 `entrada_invalida`) e a escrita converte para -03:00,
  trunca em segundos e formata com `DateTimeFormatter.ISO_OFFSET_DATE_TIME`.
  Justificativa: o Jackson converte datas para UTC por padrão, o que quebraria o fuso exigido.
- D6. **Minutos completos:** `Duration.between(entrada, saida).toMinutes()`, com mínimo 0.
  Justificativa: um bilhete aberto há 30 minutos e alguns milissegundos deve contar 30 minutos (fração exata), e não 31.
- D7. **Validação manual**, sem `@Valid`: a placa é conferida pela expressão `^[A-Z0-9]{7}$`, e a data do relatório
  com `LocalDate.parse` (que rejeita `2026-02-30`).
  Justificativa: cada erro precisa de um código próprio no corpo (`placa_invalida`, `data_invalida`).
- D8. **Tratamento global de erros:** `ApiException(status, codigo)` é convertida em `{"erro": codigo}` por um
  `@RestControllerAdvice`. JSON malformado ou body ausente em `POST /bilhetes` vira 422 `placa_invalida`;
  `{id}` não numérico vira 404 `bilhete_nao_encontrado`.
  Justificativa: garante o corpo de erro do contrato em todas as rotas e evita 500.
- D9. **Campos opcionais omitidos:** `@JsonInclude(JsonInclude.Include.NON_NULL)` nos DTOs de saída.
  Justificativa: o bilhete cancelado não pode trazer `saida` nem `valor_centavos`.
- D10. **Nomes JSON em snake_case** com `@JsonProperty` (ex.: `valor_centavos`).
  Justificativa: os nomes do contrato não seguem o camelCase padrão do Java.
- D11. **Média arredondada para cima em 0,5** com aritmética inteira: `(soma × 2 + n) ÷ (2 × n)`.
  Justificativa: evita ponto flutuante e segue a regra do enunciado.
- D12. **Testes** em `ApitestTest.java` (MockMvc + `@SpringBootTest`), com o repositório limpo antes de cada teste
  e o gancho `entrada` para simular tempo decorrido.
  Justificativa: o nome contém "test" minúsculo, padrão usado pelo corretor para encontrar os testes.
- D13. **`Containerfile` e `Dockerfile` iguais**, em dois estágios: build com `maven:3.9-eclipse-temurin-21` e
  execução com `eclipse-temurin:21-jre`, jar com nome fixo `app.jar`, `EXPOSE 8002` e `CMD ["java", "-jar", "app.jar"]`.
  Justificativa: funciona tanto com Podman quanto com Docker, e a aplicação compila dentro do próprio container.