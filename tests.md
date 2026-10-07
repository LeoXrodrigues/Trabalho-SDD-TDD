# Tests — Cenários de teste (TDD)

Cada cenário corresponde a um método @Test em ApitestTest.java, nomeado tNN_descricao.
O repositório é limpo antes de cada teste. Para simular tempo decorrido, o bilhete é aberto com
entrada = agora − N minutos e encerrado em seguida.

## 1. Cálculo de valor — casos de borda

Parâmetros: tarifa 450, fração 30 minutos (225 centavos), teto 7000, tolerância 15 minutos.

- T01: 15 minutos - valor_centavos 0 (borda: tolerância exata)
- T02: 16 minutos - valor_centavos 225 (borda: tolerância +1, cobra desde o primeiro minuto)
- T03: 30 minutos - valor_centavos 225 (borda: fração exata)
- T04: 31 minutos - valor_centavos 450 (borda: adjacência +1 minuto)
- T05: 95 minutos - valor_centavos 900 (exemplo do contrato, 4 frações)
- T06: 930 minutos - valor_centavos 6975 (borda: logo abaixo do teto)
- T07: 931 minutos - valor_centavos 7000 (borda: teto)

## 2. Abrir bilhete (UC1 e UC8)

- T08: POST /bilhetes com placa ABC1D23 - 201, status aberto e entrada terminando em -03:00 (feliz)
- T09: entrada 2026-10-12T11:30:00Z - 201 e entrada 2026-10-12T08:30:00-03:00 (borda: conversão de fuso)
- T10: placa abc1d23, placa ABC1D2 e placa ausente - 422 placa_invalida (borda: formato)
- T11: entrada 2026-10-12T08:30:00, sem fuso - 422 entrada_invalida (borda: formato)
- T12: abrir ABC1D23 duas vezes - a segunda responde 409 bilhete_em_aberto (conflito mínimo)
- T13: placa com bilhete aberto e entrada inválida - 422 entrada_invalida, e não 409 (precedência)
- T14: abrir de novo a mesma placa após encerrar ou cancelar - 201 (placa liberada)

## 3. Encerrar e cancelar (UC2 e UC5)

- T15: encerrar id 999 - 404 bilhete_nao_encontrado (borda: inexistente)
- T16: encerrar o mesmo bilhete duas vezes - a segunda responde 409 bilhete_ja_encerrado (borda)
- T17: cancelar bilhete aberto - 200, status cancelado, sem saida e sem valor_centavos (feliz)
- T18: cancelar bilhete encerrado - 409 bilhete_nao_aberto (borda)

## 4. Listagens (UC3 e UC6)

- T19: GET /bilhetes/ativos com abertos, um encerrado e um cancelado - somente os abertos, do mais recente ao mais antigo (feliz)
- T20: GET /bilhetes?placa=ABC1D23 - todos os bilhetes da placa, de qualquer status; placa que nunca estacionou retorna lista vazia (borda)
- T21: GET /bilhetes?placa=abc - 422 placa_invalida (borda)

## 5. Relatório diário (UC4)

- T22: encerrados hoje com 30 e 31 minutos - total 2, faturamento 675 e média 31, porque 30,5 arredonda para cima (borda: arredondamento)
- T23: hoje, um encerrado, um aberto e um cancelado - total 1, só o encerrado conta (borda)
- T24: data=05/10/2026 - 422 data_invalida (borda: formato)