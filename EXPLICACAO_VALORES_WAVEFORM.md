# Explicacao de VALOR_A, REG_B e do resultado 1 + 12

## Resumo do que aparece no waveform

Durante a execucao, as tres linhas principais devem ser lidas assim:

| Linha | Valor mostrado | Significado |
|---|---:|---|
| `VALOR_A` | `1` | Primeiro operando da soma. |
| `REG_B` | `12` | Segundo operando, carregado no registrador B. |
| `SAIDA` | `13` | Resultado final enviado aos LEDs. |

Portanto, a leitura visual e:

`VALOR_A = 1` + `REG_B = 12` -> `SAIDA = 13`

## O que e VALOR_A

`VALOR_A` e um marcador criado para deixar o primeiro operando permanentemente visivel no waveform.

Ele e formado pelos sinais:

| Bit | Valor |
|---|---:|
| `VA3` | `0` |
| `VA2` | `0` |
| `VA1` | `0` |
| `VA0` | `1` |

Juntos, eles formam:

`VA3...VA0 = 0001 = 1`

No arquivo de waveform, `VALOR_A` passa a valer `1` quando a execucao comeca, em aproximadamente `600 ns`, e permanece em `1` ate o final.

Esse marcador nao realiza o calculo e nao altera o processador. Ele existe somente para evitar que a linha do acumulador mostre primeiro `1` e depois `13`, o que repetiria o resultado que ja aparece em `SAIDA`.

O valor `1` utilizado de verdade pelo SAP-1 esta armazenado no endereco `E` da RAM:

| Endereco | Conteudo |
|---|---:|
| `E` | `01` |

## O que e REG_B

`REG_B` representa o registrador B real do SAP-1. Esse registrador guarda o segundo operando antes da soma.

Ele e mostrado por quatro sinais reais de observacao:

| Bit | Valor depois do carregamento |
|---|---:|
| `RB3` | `1` |
| `RB2` | `1` |
| `RB1` | `0` |
| `RB0` | `0` |

Juntos, eles formam:

`RB3...RB0 = 1100 = 12`

O valor `12` esta armazenado no endereco `F` da RAM:

| Endereco | Conteudo |
|---|---:|
| `F` | `0C` |

Quando a instrucao `ADD F` e executada, o SAP-1 le o endereco `F` e carrega `00001100` no registrador B. No waveform, `REG_B` muda de `0` para `12` por volta de `850 ns`.

Os sinais `RB3...RB0` foram adicionados apenas como saidas de observacao. Eles mostram o conteudo real do registrador B sem modificar o caminho do calculo.

## Como o SAP-1 chega ao resultado

O programa gravado na RAM e:

| Endereco | Dado | Instrucao |
|---|---:|---|
| `0` | `0E` | `LDA E` |
| `1` | `1F` | `ADD F` |
| `2` | `E0` | `OUT` |
| `3` | `F0` | `HLT` |
| `E` | `01` | Primeiro operando: `1` |
| `F` | `0C` | Segundo operando: `12` |

### 1. Inicializacao

Antes da execucao, os registradores estao zerados:

- acumulador interno: `0000`;
- registrador B: `0000`;
- saida: `00000000`.

Em aproximadamente `600 ns`, `run_prog` e ativado e `VALOR_A` passa a mostrar o primeiro operando `1`.

### 2. Execucao de LDA E

A instrucao `LDA E` significa carregar no acumulador o conteudo do endereco `E`.

1. O SAP-1 encontra `0E` no endereco `0`.
2. O opcode `0` identifica a instrucao `LDA`.
3. O operando `E` seleciona o endereco `E` da RAM.
4. A RAM entrega `01`.
5. O acumulador interno recebe `0001`, que vale `1`.

Isso acontece por volta de `730 ns`.

### 3. Execucao de ADD F

A instrucao `ADD F` significa somar ao acumulador o conteudo do endereco `F`.

1. O SAP-1 encontra `1F` no endereco `1`.
2. O opcode `1` identifica a instrucao `ADD`.
3. O operando `F` seleciona o endereco `F` da RAM.
4. A RAM entrega `0C`, que corresponde a `12`.
5. O registrador B recebe `1100`.
6. O waveform passa a mostrar `REG_B = 12`.

O carregamento do registrador B acontece por volta de `850 ns`.

### 4. Soma no CI 7483

O somador CI 7483 recebe:

- acumulador: `0001`, que vale `1`;
- registrador B: `1100`, que vale `12`.

A soma binaria e:

```text
  0001
+ 1100
------
  1101
```

`1101` em decimal e `13`.

Na etapa `T6` da instrucao `ADD`, o resultado do CI 7483 e gravado no acumulador interno. Isso ocorre por volta de `870 ns`.

### 5. Execucao de OUT

A instrucao `OUT` copia o resultado do acumulador para o registrador de saida:

`00001101 = 13`

No waveform, o grupo `SAIDA` passa a mostrar `13` por volta de `950 ns`.

Os LEDs ligados sao:

- `LED3 = 1`;
- `LED2 = 1`;
- `LED0 = 1`.

Assim:

`LED7...LED0 = 00001101 = 13`

## Sequencia para explicar ao professor

Use o zoom entre aproximadamente `600 ns` e `1000 ns` e explique:

1. `VALOR_A = 1`: este e o primeiro operando, armazenado na RAM no endereco `E`.
2. `REG_B = 12`: a instrucao `ADD F` buscou o segundo operando no endereco `F`.
3. O CI 7483 soma internamente `0001 + 1100 = 1101`.
4. `SAIDA = 13`: a instrucao `OUT` enviou `1101` para o registrador de saida e para os LEDs.

Uma frase curta para a apresentacao:

> O primeiro operando e 1, o ADD carrega 12 no registrador B, o CI 7483 soma os dois valores e o OUT apresenta 13 nos LEDs.

Todo o circuito utilizado no processamento continua implementado somente por diagramas de bloco `.bdf`.
