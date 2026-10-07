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

- D1 - Dados em memória e parâmetros fixos: ConcurrentHashMap (id para Bilhete), AtomicLong para o id e
  classe Tarifa com 450, 30, 7000 e 15.
  Justificativa: a correção sobe apenas o container da aplicação, sem banco e sem variável de ambiente.
- D2 - Dinheiro e média em inteiros: valores em long de centavos, e média arredondada para cima em 0,5
  com (soma × 2 + n) ÷ (2 × n).
  Justificativa: ponto flutuante acumula erro (0.1 + 0.2 ≠ 0.3), e inteiros eliminam essa classe de bug.
- D3 - Datas e relógio: bean java.time.Clock no fuso -03:00 para obter o "agora". Os DTOs tratam entrada e
  saida como String, lidas com OffsetDateTime.parse (erro vira 422 entrada_invalida) e devolvidas em -03:00,
  sem milissegundos.
  Justificativa: o Jackson converte datas para UTC por padrão, o que quebraria o fuso exigido.
- D4 - Minutos completos: Duration.between(entrada, saida).toMinutes(), com mínimo 0.
  Justificativa: um bilhete aberto há 30 minutos e alguns milissegundos deve contar 30 minutos, e não 31.
- D5 - Validação e erros: sem @Valid; placa conferida por ^[A-Z0-9]{7}$ e data por LocalDate.parse.
  ApiException(status, codigo) vira {"erro": codigo} em um @RestControllerAdvice. JSON malformado em
  POST /bilhetes vira 422 placa_invalida, e id não numérico vira 404 bilhete_nao_encontrado.
  Justificativa: cada erro precisa do seu código no corpo, e nenhuma entrada pode gerar 500.
- D6 - Formato do JSON: @JsonProperty para os nomes em snake_case (valor_centavos) e
  @JsonInclude(JsonInclude.Include.NON_NULL) para omitir campos vazios.
  Justificativa: os nomes do contrato não são camelCase, e o bilhete cancelado não pode trazer saida nem valor_centavos.
- D7 - Testes: ApitestTest.java com MockMvc e @SpringBootTest, repositório limpo antes de cada teste e campo
  entrada para simular tempo decorrido.
  Justificativa: o nome contém "test" minúsculo, padrão usado pelo corretor para encontrar os testes.
- D8 - Container: Containerfile e Dockerfile iguais, build com maven:3.9-eclipse-temurin-21, execução com
  eclipse-temurin:21-jre, jar app.jar, EXPOSE 8002 e CMD java -jar app.jar.
  Justificativa: funciona com Podman e Docker, e a aplicação compila dentro do próprio container.
