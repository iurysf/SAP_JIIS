# Seminário: a função do CI 7483 no projeto SAP-1

## Objetivo da apresentação

Esta apresentação explica o que é o CI 7483, como ele realiza uma soma binária de quatro bits e como duas instâncias desse componente formam a unidade aritmética de oito bits do SAP-1.

A ideia principal é:

> O CI 7483 recebe dois números binários e produz a soma. No projeto, dois CI 7483 trabalham juntos para calcular operações de oito bits, como `1 + 12 = 13`.

---

## 1. Introdução sugerida

### Fala para o seminário

“O SAP-1 precisa de um circuito capaz de realizar operações aritméticas. O componente responsável pelo cálculo é o CI 7483.

O CI 7483 é um somador binário completo de quatro bits. Ele recebe duas palavras binárias de quatro bits, chamadas A e B, recebe também um carry de entrada e gera uma soma de quatro bits e um carry de saída.

Como o nosso SAP-1 trabalha com dados de oito bits, utilizamos dois CI 7483. Um calcula os quatro bits menos significativos e o outro calcula os quatro bits mais significativos. O carry produzido pelo primeiro é enviado ao segundo, permitindo que os dois funcionem como um único somador de oito bits.”

---

## 2. O que é o CI 7483

O CI 7483 é um somador binário completo de quatro bits.

Ele realiza a operação:

`A + B + C0`

As duas parcelas A e B possuem quatro bits:

```text
A = A4 A3 A2 A1
B = B4 B3 B2 B1
```

O resultado é formado por:

```text
S = S4 S3 S2 S1
```

Além desses quatro bits, o componente possui:

- `C0`: carry de entrada;
- `C4`: carry de saída.

No CI 7483:

- `A1`, `B1` e `S1` correspondem ao bit menos significativo;
- `A4`, `B4` e `S4` correspondem ao bit mais significativo.

### Exemplo simples

```text
  A = 0011 = 3
  B = 0101 = 5
 C0 = 0
-------------
  S = 1000 = 8
 C4 = 0
```

### Fala sugerida

“O CI 7483 pode ser entendido como uma calculadora binária de quatro bits. Ele soma os quatro bits de A com os quatro bits de B e considera também um possível carry que veio de uma etapa anterior.”

---

## 3. Função de cada sinal

| Sinal | Tipo | Função |
|---|---|---|
| `A1...A4` | entrada | Formam o primeiro número de quatro bits. |
| `B1...B4` | entrada | Formam o segundo número de quatro bits. |
| `C0` | entrada | Carry de entrada, somado aos dois números. |
| `S1...S4` | saída | Formam os quatro bits menos significativos do resultado. |
| `C4` | saída | Carry final, usado quando o resultado ultrapassa quatro bits ou para alimentar outro somador. |

É importante observar que os números nos nomes `A1`, `A2`, `A3` e `A4` identificam posições binárias. Eles não representam os valores decimais que serão somados.

---

## 4. O que é o carry

O carry, também chamado de “vai um”, aparece quando a soma de uma coluna binária não cabe em apenas um bit.

As regras básicas são:

| A | B | Carry de entrada | Soma | Carry de saída |
|---:|---:|---:|---:|---:|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 1 | 0 |
| 1 | 0 | 0 | 1 | 0 |
| 1 | 1 | 0 | 0 | 1 |
| 0 | 0 | 1 | 1 | 0 |
| 0 | 1 | 1 | 0 | 1 |
| 1 | 0 | 1 | 0 | 1 |
| 1 | 1 | 1 | 1 | 1 |

Por exemplo:

```text
1 + 1 = 10 em binário
```

O bit `0` permanece na posição atual e o bit `1` é enviado como carry para a próxima posição.

### Fala sugerida

“O carry é o equivalente binário do ‘vai um’ que utilizamos na soma decimal. Sempre que uma coluna produz um valor maior que um, uma parte do resultado segue para a próxima coluna.”

---

## 5. Como os quatro bits são somados

O cálculo começa na posição menos significativa:

1. `A1`, `B1` e `C0` produzem `S1` e um carry interno;
2. esse carry participa do cálculo de `A2 + B2`;
3. o processo continua nas posições 3 e 4;
4. o carry produzido pela última posição aparece em `C4`.

Assim, cada posição depende não apenas dos bits A e B daquela coluna, mas também do carry que chega da coluna anterior.

### Exemplo com carry

```text
    1111 = 15
  + 0001 =  1
  ------
  1 0000 = 16
  ↑ └───┘
 C4   S
```

Nesse exemplo:

- `S4...S1 = 0000`;
- `C4 = 1`.

O carry é indispensável para representar corretamente o resultado `10000`.

---

## 6. O CI 7483 possui memória?

Não.

O CI 7483 é um circuito combinacional. Isso significa que:

- ele não possui flip-flops;
- ele não armazena os operandos;
- ele não armazena o resultado;
- ele não utiliza clock;
- suas saídas mudam de acordo com as entradas.

No SAP-1, os valores são mantidos por registradores construídos com o CI 74173. O CI 7483 apenas recebe os valores atuais desses registradores e calcula continuamente o resultado.

| Componente | Função | Possui memória? | Utiliza clock? |
|---|---|---:|---:|
| CI 74157 | Seleciona uma entre duas fontes | Não | Não |
| CI 74173 | Armazena quatro bits | Sim | Sim |
| CI 7483 | Soma duas palavras binárias | Não | Não |

### Ideia principal

> O CI 7483 calcula; o CI 74173 guarda.

---

## 7. Como o CI 7483 foi implementado

O componente foi construído no arquivo `ci7483.bdf`, exclusivamente com diagrama de blocos.

A implementação utiliza portas lógicas, incluindo:

- portas `XOR`, responsáveis pela parte principal dos bits de soma;
- portas `AND`, que identificam combinações capazes de gerar carry;
- portas `NAND` e `NOR`, que combinam e simplificam a lógica interna;
- inversores, utilizados onde é necessário complementar um sinal.

Essas portas formam a lógica combinacional que calcula simultaneamente os bits `S1...S4` e o carry `C4`.

Não existe arquivo Verilog implementando o CI. Tanto o componente quanto suas ligações no processador estão representados em diagramas de bloco.

### Fala sugerida

“O símbolo do CI 7483 esconde uma rede de portas lógicas montada no próprio arquivo BDF. As portas XOR participam do cálculo da soma e as combinações de AND, NAND e NOR determinam quando um carry deve ser produzido. Portanto, o resultado surge diretamente da lógica do circuito, sem código Verilog e sem armazenamento interno.”

---

## 8. Onde o CI 7483 aparece no SAP-1

No arquivo `SAP1.bdf`, existem duas instâncias do CI 7483:

| Instância | Parte calculada | Bits do resultado |
|---|---|---|
| `inst130` | parte baixa | quatro bits menos significativos |
| `inst129` | parte alta | quatro bits mais significativos |

Os dois componentes recebem como operandos:

1. o valor armazenado no acumulador;
2. o valor armazenado no registrador B.

O encadeamento do carry é:

```text
Acumulador[3:0] ─┐
                 ├─> CI 7483 inst130 ── C4 ──> C0 do CI 7483 inst129
Registro B[3:0] ─┘          │                         │
                            └─ soma[3:0]              └─ soma[7:4]

Acumulador[7:4] ──────────────────────────────┐
                                             ├─> CI 7483 inst129
Registro B[7:4] ─────────────────────────────┘
```

O `C4` de `inst130` está ligado ao `C0` de `inst129`. Por isso, um “vai um” gerado na metade inferior não é perdido: ele participa do cálculo da metade superior.

### Por que são necessários dois componentes?

Cada CI 7483 soma apenas quatro bits. Como o barramento de dados do projeto possui oito bits, dois componentes são necessários:

```text
4 bits inferiores + 4 bits superiores = operação de 8 bits
```

---

## 9. Relação com os sinais `Su` e `Eu`

O CI 7483 realiza a soma, mas a lógica ao redor dele permite que a unidade aritmética também faça subtração.

### Quando `Su = 0`: soma

- os bits do registrador B chegam ao somador sem inversão;
- o carry inicial `C0` do somador inferior vale zero;
- a operação é `A + B`.

### Quando `Su = 1`: subtração

- portas XOR externas invertem os bits do registrador B;
- o carry inicial recebe o valor um;
- a operação passa a ser `A + NOT B + 1`;
- essa expressão corresponde a `A - B` em complemento de dois.

```text
Su = 0  →  A + B
Su = 1  →  A + NOT B + 1  →  A - B
```

O sinal `Eu` controla buffers externos que permitem colocar o resultado da unidade aritmética no barramento.

É importante diferenciar:

- `C0` e `C4` são sinais do CI 7483;
- `Su` e `Eu` são sinais de controle do SAP-1, conectados à lógica ao redor do somador;
- o CI 7483 não possui pinos chamados `Su` ou `Eu`.

### Fala sugerida

“O CI 7483 continua sendo um somador. A subtração é obtida preparando a entrada B em complemento de dois. O sinal Su inverte B e acrescenta um pelo carry inicial. Já Eu determina quando o resultado calculado pode ser colocado no barramento.”

---

## 10. Participação no cálculo `1 + 12 = 13`

O programa executado é:

```text
LDA E   → carrega o valor 1 no acumulador
ADD F   → soma o valor 12
OUT     → envia o resultado ao registrador de saída
HLT     → encerra a execução
```

Os dados estão armazenados na RAM como:

```text
Endereço E: 00000001 = 1
Endereço F: 00001100 = 12
```

### Etapa 1 — primeiro operando

A instrução `LDA E` coloca no acumulador:

```text
Acumulador = 00000001 = 1
```

### Etapa 2 — segundo operando

Durante a instrução `ADD F`, o valor lido da RAM é armazenado no registrador B:

```text
Registro B = 00001100 = 12
```

### Etapa 3 — configuração para soma

Como a instrução é `ADD`:

```text
Su = 0
```

Consequentemente:

- B não é invertido;
- o carry inicial vale zero;
- a unidade aritmética calcula `A + B`.

### Etapa 4 — cálculo dos quatro bits inferiores

A instância `inst130` recebe:

```text
  A[3:0] = 0001
  B[3:0] = 1100
       C0 =    0
----------------
  S[3:0] = 1101
       C4 =    0
```

O cálculo por coluna é:

| Posição | Cálculo | Bit da soma | Carry para a próxima posição |
|---|---|---:|---:|
| bit 0 | `1 + 0 + 0` | 1 | 0 |
| bit 1 | `0 + 0 + 0` | 0 | 0 |
| bit 2 | `0 + 1 + 0` | 1 | 0 |
| bit 3 | `0 + 1 + 0` | 1 | 0 |

Assim, a metade inferior produz:

```text
1101 = 13
```

### Etapa 5 — cálculo dos quatro bits superiores

A instância `inst129` recebe:

```text
  A[7:4] = 0000
  B[7:4] = 0000
C0 recebido =    0
-----------------
  S[7:4] = 0000
```

### Etapa 6 — resultado completo

As duas metades são reunidas:

```text
Parte alta       Parte baixa
   0000             1101
          00001101 = 13
```

Ou, mostrando a operação completa:

```text
  00000001 = 1
+ 00001100 = 12
----------
  00001101 = 13
```

### Etapa 7 — armazenamento do resultado

O resultado do CI 7483 existe de forma combinacional enquanto as entradas estiverem presentes. Na etapa de controle apropriada, o acumulador grava `00001101` em seus CI 74173.

Depois, a instrução `OUT` transfere esse valor para o registrador de saída, que mantém o número `13` visível nos LEDs.

O CI 7483 não mantém os LEDs acesos diretamente. Ele calcula o valor; os registradores armazenam esse valor e a lógica de saída o apresenta.

---

## 11. Relação com o waveform

As linhas principais do waveform são:

| Linha | Relação com o cálculo |
|---|---|
| `VALOR_A = 1` | Marcador visual do primeiro operando usado na apresentação. Não é uma saída direta do CI 7483. |
| `REG_B = 12` | Conteúdo real do registrador B e segundo operando recebido pela unidade aritmética. |
| `SAIDA = 13` | Resultado que foi calculado, armazenado e depois enviado ao registrador de saída. |
| `T1...T6` | Etapas de controle que organizam carregamento dos operandos, gravação do resultado e saída. |

### Sequência observada

| Momento aproximado | Evento |
|---:|---|
| `600 ns` | O processador começa a execução e `VALOR_A` identifica o valor `1`. |
| `730 ns` | A instrução `LDA` faz o acumulador armazenar `1`. |
| `850 ns` | A instrução `ADD` faz o registrador B armazenar `12`. |
| `850–870 ns` | O CI 7483 apresenta combinacionalmente o resultado `13`. |
| `870 ns` | O acumulador armazena o resultado `13`. |
| `950 ns` | A instrução `OUT` faz o registrador de saída apresentar `13`. |

O período entre o carregamento do registrador B e a gravação do acumulador evidencia a função do somador: com A valendo `1` e B valendo `12`, sua saída passa a valer `13`.

---

## 12. Diferença entre calcular, armazenar e mostrar

No projeto, três ações diferentes são realizadas por componentes diferentes:

| Ação | Componente principal | Exemplo |
|---|---|---|
| Calcular | CI 7483 | produz `1 + 12 = 13` |
| Armazenar | CI 74173 | mantém `1`, `12` ou `13` |
| Mostrar | registrador de saída e LEDs | mantém `13` visível |

Essa separação explica por que o resultado não aparece instantaneamente nos LEDs assim que os operandos existem. Primeiro o CI 7483 calcula; depois o clock permite armazenar o resultado; por fim a instrução `OUT` o transfere para a saída.

---

## 13. Roteiro de fala completo

“O CI 7483 é um somador binário completo de quatro bits. Ele recebe dois números, A e B, cada um com quatro bits, e também recebe um carry de entrada chamado C0. Como saída, ele produz quatro bits de soma, S1 até S4, e um carry final chamado C4.

O carry funciona como o ‘vai um’ da soma decimal. Quando a soma de uma posição ultrapassa o valor que cabe em um bit, o carry segue para a próxima posição. Por isso, o componente consegue calcular corretamente uma soma envolvendo todas as quatro colunas.

O CI 7483 é combinacional. Ele não possui memória e não utiliza clock. Sempre que A, B ou C0 mudam, o resultado também muda de acordo com a nova operação. Quem mantém os operandos e o resultado no SAP-1 são os registradores construídos com o CI 74173.

No nosso projeto, um único CI 7483 não seria suficiente porque ele trabalha com apenas quatro bits e o SAP-1 possui um barramento de oito bits. Por isso, usamos dois componentes. A instância inst130 soma os quatro bits inferiores e a instância inst129 soma os quatro bits superiores.

O carry C4 do somador inferior está ligado ao C0 do somador superior. Essa ligação transforma os dois componentes em um somador de oito bits. Se a metade inferior gerar um ‘vai um’, ele será considerado na metade superior.

A unidade aritmética também consegue fazer subtração. Para somar, o sinal Su vale zero, os bits de B chegam sem inversão e o carry inicial vale zero. Para subtrair, Su vale um, a lógica externa inverte B e acrescenta um pelo carry inicial. Assim, o somador calcula A mais o complemento de dois de B, que equivale a A menos B.

No exemplo mostrado no waveform, a instrução LDA carrega o número 1 no acumulador. Depois, a instrução ADD carrega o número 12 no registrador B. Como Su vale zero, a operação selecionada é a soma.

O somador inferior calcula 0001 mais 1100 e produz 1101, sem carry final. O somador superior recebe 0000 mais 0000 e também recebe carry zero, produzindo 0000. Juntando as duas metades, obtemos 00001101, que corresponde ao número decimal 13.

Em seguida, o acumulador armazena esse resultado. A instrução OUT transfere o 13 para o registrador de saída e os LEDs passam a representá-lo.

Portanto, o CI 7483 é o componente que efetivamente realiza o cálculo. Ele não guarda o resultado e não controla sozinho os LEDs. Sua função é receber os operandos e gerar a resposta binária que os outros blocos do SAP-1 irão armazenar e utilizar.”

---

## 14. Perguntas que podem ser feitas

### O CI 7483 realiza soma de quantos bits?

Cada CI 7483 realiza uma soma de quatro bits.

### Por que o projeto utiliza dois CI 7483?

Porque o SAP-1 trabalha com dados de oito bits. Um componente calcula a metade inferior e o outro calcula a metade superior.

### Como os dois somadores são conectados?

O `C4` do somador inferior é ligado ao `C0` do somador superior. Assim, o carry atravessa a fronteira entre os dois grupos de quatro bits.

### Para que serve o `C0`?

Ele adiciona um carry de entrada à operação. No projeto, também é usado pela lógica de complemento de dois durante uma subtração.

### Para que serve o `C4`?

Ele indica o carry produzido depois da quarta posição. Pode alimentar outro somador, como acontece no SAP-1.

### O CI 7483 precisa de clock?

Não. Ele é combinacional e calcula continuamente a partir das entradas atuais.

### O CI 7483 armazena o resultado?

Não. O resultado é armazenado pelos registradores implementados com CI 74173.

### Como um somador consegue fazer subtração?

A lógica externa inverte B e acrescenta um, formando o complemento de dois. Dessa maneira, `A + NOT B + 1` equivale a `A - B`.

### O sinal `Su` é um pino do CI 7483?

Não. `Su` é um sinal de controle do SAP-1. Ele atua nas portas XOR ao redor do somador e no carry inicial.

### O sinal `Eu` é um pino do CI 7483?

Não. `Eu` habilita buffers externos que colocam o resultado da unidade aritmética no barramento.

### Por que o resultado só aparece depois nos LEDs?

Porque o CI 7483 primeiro calcula o valor. Depois, o resultado precisa ser gravado no acumulador e transferido pela instrução `OUT` ao registrador de saída.

### O que aconteceria se o carry entre os dois CIs não estivesse conectado?

As somas que gerassem carry nos quatro bits inferiores produziriam um resultado incorreto nos quatro bits superiores.

### Onde está a implementação do componente?

No arquivo `ci7483.bdf`, em diagrama de blocos. As duas instâncias e suas ligações estão em `SAP1.bdf`.

---

## 15. Conclusão sugerida

### Fala para encerrar

“Para concluir, o CI 7483 é o núcleo aritmético do SAP-1. Cada componente soma quatro bits, recebe um carry de entrada e produz um carry de saída.

No projeto, dois CI 7483 estão encadeados para formar uma unidade de oito bits. O primeiro calcula a metade inferior e envia seu carry ao segundo, que calcula a metade superior.

Na operação demonstrada, o acumulador fornece o valor 1, o registrador B fornece o valor 12 e os somadores produzem 00001101, equivalente a 13. Depois, os registradores armazenam e apresentam esse resultado.

Assim, podemos resumir a divisão de funções da seguinte maneira: o CI 74157 seleciona caminhos, o CI 74173 guarda dados e o CI 7483 realiza o cálculo.”

---

## Resumo em uma frase

> O CI 7483 é um somador binário de quatro bits que, usado em duas instâncias encadeadas no SAP-1, calcula operações de oito bits como `1 + 12 = 13`.

O componente e o processador estão implementados exclusivamente em diagramas de bloco:

- `ci7483.bdf`: implementação interna do somador;
- `SAP1.bdf`: utilização e conexão das duas instâncias no processador.
