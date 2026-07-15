# Guia para explicar a simulacao do SAP-1

## Resultado esperado

O programa soma `1 + 12` e envia o resultado para o registrador de saida.

- Decimal: `13`
- Binario em `LED7...LED0`: `00001101`
- LEDs acesos: `LED3`, `LED2` e `LED0`

## Programa gravado na RAM

| Endereco | Dado | Instrucao ou valor | Explicacao |
|---|---:|---|---|
| `0` | `0E` | `LDA E` | Carrega no acumulador o valor do endereco `E`. |
| `1` | `1F` | `ADD F` | Soma ao acumulador o valor do endereco `F`. |
| `2` | `E0` | `OUT` | Copia o acumulador para o registrador de saida. |
| `3` | `F0` | `HLT` | Encerra o programa. |
| `E` | `01` | operando | Primeiro valor: `1`. |
| `F` | `0C` | operando | Segundo valor: `12`. |

De `0 ns` a `600 ns`, o waveform seleciona esses seis enderecos e grava os dados. Os grupos `ENDERECO` e `DADOS` estao em hexadecimal para que aparecam diretamente como `0`, `1`, `2`, `3`, `E`, `F` e `0E`, `1F`, `E0`, `F0`, `01`, `0C`.

## Inicio da execucao

1. Em `600 ns`, `run_prog` passa para `1`.
2. `Limpar_Iniciar` gera o pulso de inicializacao entre `600 ns` e `620 ns`.
3. Em `645 ns`, o clock comeca a movimentar o sequenciador.
4. O SAP-1 busca e executa `LDA E`, `ADD F`, `OUT` e `HLT`.
5. O grupo `VALOR_A` passa a mostrar `1`, identificando o primeiro operando sem repetir o resultado no final.
6. O grupo `REG_B` passa de `0` para `12`: a instrucao `ADD F` carregou o segundo operando no registrador B.
7. O somador calcula `1 + 12` e `T6` grava internamente o resultado no acumulador.
8. O grupo `SAIDA` passa de `0` para `13`: a instrucao `OUT` copiou o resultado para os LEDs.
9. Nas linhas individuais, `LED3`, `LED2` e `LED0` ficam em nivel alto: `00001101`.

## Leitura direta do calculo no waveform

Com o zoom entre aproximadamente `700 ns` e `1000 ns`, a explicacao pode ser feita nesta ordem:

| Instante aproximado | VALOR_A | REG_B | SAIDA | O que aconteceu |
|---:|---:|---:|---:|---|
| `600 ns` | `1` | `0` | `0` | O primeiro operando fica identificado para a explicacao. |
| `730 ns` | `1` | `0` | `0` | `LDA E` carregou o primeiro numero internamente. |
| `850 ns` | `1` | `12` | `0` | `ADD F` carregou o segundo numero. |
| `950 ns` | `1` | `12` | `13` | `OUT` apresentou o resultado nos LEDs. |

Os tres grupos estao em decimal sem sinal e recolhidos, deixando os nomes e os numeros inteiros visiveis. `VALOR_A` e um marcador visual do primeiro operando e nao participa do processamento. `REG_B` representa os quatro bits reais `RB3...RB0 = 1100`.

## O que significam T1 a T6

As linhas `T1` a `T6` nao representam o resultado da soma. Elas formam um sequenciador *one-hot*: somente uma etapa fica ativa por vez.

| Etapa ativa | Vetor agrupado | Funcao principal |
|---|---|---|
| `T1` | `000001` | Coloca o contador de programa no registrador de endereco. |
| `T2` | `000010` | Incrementa o contador de programa. |
| `T3` | `000100` | Busca na RAM a instrucao atual. |
| `T4` | `001000` | Decodifica a instrucao e prepara seu operando. |
| `T5` | `010000` | Carrega o dado necessario no registrador apropriado. |
| `T6` | `100000` | Conclui a operacao; no `ADD`, grava a soma no acumulador. |

Os numeros `0001`, `0010`, `0100` e `1000` que apareciam antes eram apenas a versao agrupada e cortada desse sequenciador. A linha vetorial foi removida: agora T1, T2, T3, T4, T5 e T6 aparecem sempre em linhas individuais, tornando visivel qual etapa esta ativa sem depender do texto binario dentro de um pulso estreito.

## Como apresentar no Quartus

1. Abra `Waveform.vwf`, e nao um arquivo antigo com data no nome.
2. Execute a simulacao funcional.
3. Mostre primeiro `0 ns` a `600 ns` para explicar a gravacao da RAM.
4. Depois use zoom aproximadamente entre `600 ns` e `1000 ns` e aponte a sequencia `VALOR_A = 1`, `REG_B = 12`, `SAIDA = 13`.
5. Relacione cada mudanca com T1 a T6.
6. Por fim, mostre `SAIDA = 13` e as linhas `LED3 = 1`, `LED2 = 1`, `LED0 = 1`.

O somador CI 7483 e os demais componentes do SAP-1 estao implementados somente por diagramas de bloco (`.bdf`). Nao existe arquivo Verilog como fonte do circuito.
