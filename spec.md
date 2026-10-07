# Spec — Zona Azul Digital

## 1. Contexto

API para uma operadora de estacionamento rotativo abrir e encerrar bilhetes por placa, cancelar bilhetes,
listar bilhetes ativos, consultar o histórico de uma placa e emitir relatório diário.

## 2. Contrato REST

| Método | Rota | Sucesso | Erros |
| --- | --- | --- | --- |
| POST | `/bilhetes` | 201 | 422 `placa_invalida`, 422 `entrada_invalida`, 409 `bilhete_em_aberto` |
| POST | `/bilhetes/{id}/encerramento` | 200 | 404 `bilhete_nao_encontrado`, 409 `bilhete_ja_encerrado` |
| POST | `/bilhetes/{id}/cancelamento` | 200 | 404 `bilhete_nao_encontrado`, 409 `bilhete_nao_aberto` |
| GET | `/bilhetes/ativos` | 200 (array) | — |
| GET | `/bilhetes?placa=ABC1D23` | 200 (array) | 422 `placa_invalida` |
| GET | `/relatorios/diario?data=AAAA-MM-DD` | 200 | 422 `data_invalida` |

### Tabela de erros

| Situação | Status | Body |
| --- | --- | --- |
| Placa ausente ou inválida (inclui JSON malformado ou body ausente em `POST /bilhetes`) | 422 | `{"erro": "placa_invalida"}` |
| entrada presente, mas fora de ISO-8601 com fuso | 422 | `{"erro": "entrada_invalida"}` |
| data ausente ou fora de AAAA-MM-DD | 422 | `{"erro": "data_invalida"}` |
| Bilhete inexistente (inclui {id} não numérico) | 404 | `{"erro": "bilhete_nao_encontrado"}` |
| Encerrar bilhete que não está aberto | 409 | `{"erro": "bilhete_ja_encerrado"}` |
| Cancelar bilhete que não está aberto | 409 | `{"erro": "bilhete_nao_aberto"}` |
| Abrir bilhete para placa que já tem bilhete aberto | 409 | `{"erro": "bilhete_em_aberto"}` |

### Formato do bilhete

| Campo | Tipo | Presente quando |
| --- | --- | --- |
| id | inteiro, sequencial a partir de 1 | sempre |
| placa | string | sempre |
| entrada | UTC−03:00 | sempre |
| status | aberto, encerrado ou cancelado | sempre |
| saida | UTC−03:00  | somente encerrado |
| minutos | inteiro | somente encerrado |
| valor_centavos | inteiro | somente encerrado |

Campos que não se aplicam são omitidos do JSON.

## 3. Regras de negócio

- RN01. **Placa válida:** exatamente 7 caracteres, somente A-Z maiúsculas e `0-9`. Letras minúsculas tornam a placa inválida.
- RN02. **Entrada opcional:** quando entrada é enviada, o bilhete abre naquele instante; sem ela, abre no instante atual.
  A entrada deve ter data, hora e fuso.
- RN03. **Tolerância:** se minutos ≤ 15, valor_centavos = 0.
- RN04. **Uma vaga por placa:** só pode existir um bilhete aberto por placa. Depois de encerrar ou cancelar, a placa pode abrir de novo.
- RN05. **Ordenação:** listas são ordenadas da entrada mais recente para a mais antiga.
- RN06. **Relatório diário:** considera os bilhetes encerrado cuja saida, no fuso -03:00, cai na data pedida.
  total_bilhetes = quantidade desses bilhetes; faturamento_centavos = soma dos seus valor_centavos;
  tempo_medio_minutos = média dos seus minuto.

## 4. Casos de uso e critérios de aceite

### UC1 — Abrir bilhete

Body: `{"placa": "ABC1D23"}` ou `{"placa": "ABC1D23", "entrada": "2026-10-12T08:30:00-03:00"}`.

- CA1.1 Placa válida → 201 com `{"id", "placa", "entrada", "status": "aberto"}`.
- CA1.2 entrada enviada como 2026-10-12T11:30:00Z → resposta com `"entrada": "2026-10-12T08:30:00-03:00"`.
- CA1.3 Placa ausente, com 6 ou 8 caracteres, minúscula ou com símbolo → 422 placa_invalida.
- CA1.4 entrada sem fuso ou em outro formato → 422 entrada_invalida.

### UC2 — Encerrar bilhete

- CA2.1 Bilhete aberto → 200 com id, placa, entrada, saida (instante atual), minutos, valor_centavos e status: "encerrado".
- CA2.2 Bilhete aberto há 95 minutos → minutos: 95 e valor_centavos: 900 (4 frações).
- CA2.3 Bilhete já encerrado ou cancelado → 409 bilhete_ja_encerrado. Inexistente → 404.

### UC3 — Listar ativos

- CA3.1 Retorna 200 com somente os bilhetes aberto, conforme RN08. Bilhete nao pode zer vazio.

### UC4 — Relatório diário

- CA4.1 Retorna 200 com `{"data", "total_bilhetes", "faturamento_centavos", "tempo_medio_minutos"}`, conforme RN09.
- CA4.2 data ausente, em outro formato (05/10/2026) ou inexistente (2026-02-30) → 422 data_invalida.

### UC5 — Cancelar bilhete

- CA5.1 Bilhete aberto → 200 com id, placa, entrada e status: "cancelado", sem saida, minutos e valor_centavos.
- CA5.2 Bilhete encerrado ou já cancelado → 409 bilhete_nao_aberto. Inexistente → 404.

### UC6 — Histórico por placa 

- CA6.1 Retorna 200 com todos os bilhetes da placa, de qualquer status, conforme RN08.
- CA6.2 Placa válida que nunca estacionou → 200 com vazio.
- CA6.3 Parâmetro placa ausente ou inválido → 422 placa_invalida.

### UC7 — Tolerância gratuita

- CA7.1 Bilhete encerrado com 15 minutos → valor_centavos: 0.
- CA7.2 Bilhete encerrado com 16 minutos → valor_centavos: 225.

### UC8 — Uma vaga por placa

- CA8.1 Segundo POST /bilhetes com placa que tem bilhete aberto → 409 bilhete_em_aberto.
- CA8.2 Após encerrar ou cancelar, a mesma placa abre novo bilhete com 201.
- CA8.3 Placa inválida nunca retorna 409: a validação (422) acontece antes.