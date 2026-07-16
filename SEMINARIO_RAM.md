# Seminário: a função da memória RAM no projeto SAP-1

## Objetivo da apresentação

Esta apresentação explica o que é uma memória RAM, como ela foi construída exclusivamente com diagramas de bloco e qual é sua função durante a programação e a execução do SAP-1.

A ideia principal é:

> A RAM guarda as instruções e os dados do programa. O endereço determina qual posição será acessada, e os sinais de controle determinam se essa posição será escrita ou lida.

---

## 1. Introdução sugerida

### Fala para o seminário

“Para executar um programa, o processador precisa de um local onde as instruções e os dados permaneçam armazenados. No nosso SAP-1, esse local é a memória RAM.

A RAM do projeto possui 16 endereços. Cada endereço armazena uma palavra de oito bits. Por isso, sua organização pode ser representada como 16 por 8 bits.

Os quatro bits de endereço escolhem uma das 16 posições. Os oito bits de dados representam o conteúdo que será gravado ou lido. Durante a programação, os endereços e os dados vêm dos pinos externos. Durante a execução, o endereço passa a ser fornecido pelo registrador MAR, e o processador lê automaticamente as instruções e os operandos.”

---

## 2. O que é uma memória RAM

RAM é a sigla de *Random Access Memory*, ou memória de acesso aleatório.

“Acesso aleatório” não significa que os valores são escolhidos ao acaso. Significa que qualquer posição pode ser acessada diretamente por meio de seu endereço, sem precisar percorrer as posições anteriores.

A RAM realiza duas operações principais:

- escrita: armazena um dado em determinado endereço;
- leitura: recupera o dado armazenado naquele endereço.

### Exemplo simples

Se o endereço selecionado for:

```text
A3 A2 A1 A0 = 1110
```

então a posição acessada será:

```text
1110 em binário = E em hexadecimal = 14 em decimal
```

Se essa posição contiver `00000001`, a leitura produzirá o número decimal `1`.

---

## 3. Capacidade da RAM do projeto

A memória utiliza quatro bits de endereço:

```text
A3 A2 A1 A0
```

Com quatro bits, o número de combinações é:

```text
2⁴ = 16 endereços
```

Os endereços possíveis vão de:

```text
0000 até 1111
```

Em hexadecimal:

```text
0 até F
```

Cada endereço guarda oito bits, portanto a capacidade total é:

```text
16 palavras × 8 bits = 128 bits = 16 bytes
```

### Organização

| Característica | Valor |
|---|---:|
| Bits de endereço | 4 |
| Quantidade de endereços | 16 |
| Bits por palavra | 8 |
| Capacidade total | 128 bits |
| Capacidade em bytes | 16 bytes |

### Fala sugerida

“A RAM não possui 256 endereços apenas porque os dados têm oito bits. A quantidade de endereços é determinada pelas quatro entradas A0 até A3. Portanto, existem 16 posições, e cada posição guarda uma palavra completa de oito bits.”

---

## 4. Por que existem dois blocos RAM no SAP-1

O arquivo `RAM.bdf` implementa uma memória de:

```text
16 endereços × 4 bits
```

Como o SAP-1 trabalha com palavras de oito bits, são utilizadas duas instâncias:

| Instância | Função |
|---|---|
| `inst105` | armazena os quatro bits superiores, `D7...D4` |
| `inst106` | armazena os quatro bits inferiores, `D3...D0` |

Os dois blocos recebem o mesmo endereço e os mesmos controles. Dessa forma, ambos acessam simultaneamente a mesma posição.

```text
                 mesmo endereço A3...A0
                            │
               ┌────────────┴────────────┐
               │                         │
        RAM inst105               RAM inst106
        bits D7...D4              bits D3...D0
               │                         │
               └────────────┬────────────┘
                            │
                   palavra de 8 bits
```

Por exemplo, para armazenar `00001100`:

```text
RAM superior: 0000
RAM inferior: 1100
Palavra total: 00001100 = 12
```

---

## 5. Função dos sinais da RAM

Cada bloco `RAM.bdf` possui os seguintes sinais principais:

| Sinal | Tipo | Função |
|---|---|---|
| `A0...A3` | entrada | Selecionam um dos 16 endereços. |
| `D0...D3` | entrada | Fornecem os quatro bits que poderão ser escritos. |
| `_WE` | entrada | Controla escrita ou leitura; é ativo em nível baixo. |
| `_CS` | entrada | Habilita o caminho de saída da memória; é ativo em nível baixo. |
| `_O0..._O3` | saída | Fornecem os quatro bits lidos da posição selecionada. |
| `internal0...internal3` | saída auxiliar | Permitem observar internamente os bits selecionados no diagrama. |

O caractere `_` no início de `_WE` e `_CS` indica que esses controles são ativos em nível baixo.

### Controle `_WE`

| `_WE` | Operação |
|---:|---|
| `0` | escrita habilitada |
| `1` | leitura, sem alterar o conteúdo armazenado |

### Controle `_CS`

| `_CS` | Estado do caminho de saída |
|---:|---|
| `0` | memória selecionada para fornecer sua saída |
| `1` | saídas desconectadas, em alta impedância |

### Fala sugerida

“Os sinais possuem uma barra lógica representada pelo sublinhado. Isso significa que o valor zero ativa a função. Assim, WE igual a zero solicita escrita e CS igual a zero permite que a memória forneça o dado lido.”

---

## 6. Como um endereço seleciona uma posição

Dentro de `RAM.bdf`, as entradas `A0...A3` alimentam uma lógica decodificadora.

O decodificador é construído com:

- inversores `NOT`, que produzem os endereços complementados;
- portas `AND4`, que reconhecem cada combinação dos quatro bits.

Existem 16 combinações possíveis e apenas uma delas deve ficar ativa para cada endereço.

### Exemplos

Para selecionar o endereço `0`:

```text
A3 A2 A1 A0 = 0000
seleção = NOT A3 AND NOT A2 AND NOT A1 AND NOT A0
```

Para selecionar o endereço `E`:

```text
A3 A2 A1 A0 = 1110
seleção = A3 AND A2 AND A1 AND NOT A0
```

Para selecionar o endereço `F`:

```text
A3 A2 A1 A0 = 1111
seleção = A3 AND A2 AND A1 AND A0
```

### Ideia principal

> O endereço não é o dado. O endereço escolhe a posição; o dado é o conteúdo guardado nessa posição.

---

## 7. Como a célula de memória foi implementada

O arquivo `CelulaRam.bdf` implementa uma célula capaz de armazenar um bit.

Cada célula possui:

- uma entrada `D`;
- uma entrada de escrita `WE`;
- uma entrada `SELECT`;
- uma saída `Q`;
- um flip-flop D, responsável por guardar o bit;
- portas lógicas que controlam escrita e leitura;
- um buffer de três estados, que conecta a célula à saída apenas quando ela está selecionada.

### Escrita na célula

Quando a célula está selecionada e o sinal interno de escrita é ativado:

1. o bit presente em `D` chega ao flip-flop;
2. a combinação `WE AND SELECT` produz o evento de gravação;
3. o flip-flop armazena o bit;
4. o valor permanece armazenado após o pulso de escrita.

### Leitura da célula

Quando a célula está selecionada e a escrita não está ativa:

1. o flip-flop mantém o bit armazenado;
2. o buffer de três estados é habilitado;
3. o valor de `Q` chega à saída da memória.

As células não selecionadas mantêm seus buffers em alta impedância. Isso evita que várias células tentem controlar a mesma linha de saída.

### Quantidade de células

Cada arquivo `RAM.bdf` contém:

```text
16 endereços × 4 bits = 64 células de um bit
```

Como o SAP-1 utiliza duas instâncias:

```text
2 × 64 = 128 células de um bit
```

Isso corresponde aos 128 bits de capacidade total.

---

## 8. Como a RAM foi implementada

A memória foi construída exclusivamente com diagramas de bloco:

1. `CelulaRam.bdf` implementa o armazenamento de um bit;
2. `RAM.bdf` reúne 64 células e a lógica de seleção, formando `16 × 4 bits`;
3. `SAP1.bdf` utiliza dois blocos RAM, formando `16 × 8 bits`.

Os principais elementos internos são:

- flip-flops D;
- portas `NOT`;
- portas `AND`;
- decodificação com `AND4`;
- buffers de três estados.

Não existe uma implementação da RAM escrita manualmente em Verilog. Os arquivos de projeto utilizados para construir o circuito são diagramas `.bdf`.

### Fala sugerida

“A construção é hierárquica. Primeiro temos uma célula de um bit. Depois repetimos essa célula para formar 16 palavras de quatro bits. Finalmente, utilizamos dois blocos iguais em paralelo para obter palavras de oito bits. Toda essa estrutura está representada em diagramas de bloco.”

---

## 9. Modos de programação e execução

A RAM é utilizada em dois momentos diferentes:

### Modo de programação

```text
run_prog = 0
```

Nesse modo:

- o CI 74157 seleciona o endereço externo `A0...A3`;
- os pinos externos `D7...D0` fornecem o dado;
- `LeituraEscrita` controla diretamente `_WE`;
- um pulso em nível baixo grava o dado na posição escolhida.

### Modo de execução

```text
run_prog = 1
```

Nesse modo:

- o CI 74157 seleciona o endereço interno `MAR_A0...MAR_A3`;
- a unidade de controle utiliza `_CE` para permitir a leitura da RAM;
- `LeituraEscrita` permanece no estado de leitura;
- o dado lido pode ser encaminhado ao barramento do processador.

### Relação de `_CS` com os dois modos

No `SAP1.bdf`, `_CS` é produzido pela combinação de `run_prog` e `_CE`:

- durante a programação, `run_prog = 0` força `_CS = 0`, deixando a memória selecionada;
- durante a execução, `run_prog = 1` faz `_CS` acompanhar `_CE`;
- quando `_CE = 0`, a RAM pode colocar o dado no barramento.

### Caminho do endereço

```text
Endereço externo A0...A3 ─┐
                           ├─> CI 74157 ──> RAM inst105 e inst106
Endereço interno do MAR ───┘
                    seleção: run_prog
```

---

## 10. Como acontece uma escrita

Para gravar uma palavra, o circuito segue esta sequência:

1. coloca o endereço desejado em `A3...A0`;
2. coloca o valor desejado em `D7...D0`;
3. mantém `run_prog = 0`, selecionando o endereço externo;
4. aplica um pulso de escrita, levando `LeituraEscrita` e `_WE` a zero;
5. o decodificador ativa apenas a palavra correspondente ao endereço;
6. os oito flip-flops daquela palavra armazenam os oito bits;
7. `_WE` volta a um e o valor permanece guardado.

### Exemplo: gravação do número 12

```text
Endereço = 1111 = F
Dado     = 00001100 = 12
_WE      = pulso em 0
```

Depois da escrita:

```text
RAM[F] = 00001100
```

Os outros 15 endereços não são alterados porque somente a linha `SELECT` do endereço F está ativa.

---

## 11. Como acontece uma leitura

Para ler uma palavra:

1. o endereço desejado é aplicado em `A3...A0`;
2. o decodificador seleciona uma das 16 palavras;
3. `_WE` permanece em um, impedindo uma nova gravação;
4. `_CS` vai para zero quando a memória deve fornecer o dado;
5. os buffers de três estados da posição selecionada liberam o conteúdo;
6. os oito bits chegam ao barramento.

### Exemplo: leitura do número 1

```text
Endereço = 1110 = E
RAM[E]   = 00000001
```

Quando a leitura é habilitada:

```text
Saída da RAM = 00000001 = 1
```

A leitura não apaga nem altera o valor armazenado.

---

## 12. Conteúdo usado no programa `1 + 12 = 13`

Antes da execução, o waveform programa seis posições da RAM:

| Endereço | Conteúdo | Binário | Interpretação |
|---:|---:|---|---|
| `0` | `0E` | `00001110` | instrução `LDA E` |
| `1` | `1F` | `00011111` | instrução `ADD F` |
| `2` | `E0` | `11100000` | instrução `OUT` |
| `3` | `F0` | `11110000` | instrução `HLT` |
| `E` | `01` | `00000001` | primeiro operando: `1` |
| `F` | `0C` | `00001100` | segundo operando: `12` |

Os endereços `0...3` contêm instruções. Os endereços `E` e `F` contêm dados.

Isso mostra que a mesma RAM armazena tanto o programa quanto os operandos.

### Como interpretar uma instrução

Cada instrução possui oito bits divididos em dois grupos:

```text
bits 7...4: código da operação
bits 3...0: endereço do operando
```

Exemplo:

```text
0E = 0000 1110
     └──┘ └──┘
      LDA    E
```

Outro exemplo:

```text
1F = 0001 1111
     └──┘ └──┘
      ADD    F
```

Nas instruções `OUT` e `HLT`, o segundo grupo não é utilizado no cálculo e foi gravado como zero.

---

## 13. Participação da RAM no cálculo `1 + 12 = 13`

### Etapa 1 — busca de `LDA E`

O contador de programa indica o endereço `0`. Esse endereço passa pelo MAR e seleciona:

```text
RAM[0] = 0E = LDA E
```

A instrução é carregada no registrador de instrução.

### Etapa 2 — leitura do primeiro operando

O campo de endereço da instrução vale `E`. O MAR passa a selecionar:

```text
RAM[E] = 01 = 1
```

Esse valor é enviado ao barramento e armazenado no acumulador.

### Etapa 3 — busca de `ADD F`

O contador de programa avança para `1`. A RAM fornece:

```text
RAM[1] = 1F = ADD F
```

### Etapa 4 — leitura do segundo operando

O campo de endereço da nova instrução vale `F`. O MAR seleciona:

```text
RAM[F] = 0C = 12
```

Esse valor é enviado ao registrador B.

### Etapa 5 — soma

O CI 7483 recebe:

```text
Acumulador = 00000001 = 1
Registro B = 00001100 = 12
```

e produz:

```text
00001101 = 13
```

O acumulador armazena o resultado.

### Etapa 6 — busca de `OUT`

No endereço `2`, a RAM fornece:

```text
RAM[2] = E0 = OUT
```

A instrução transfere o conteúdo do acumulador para o registrador de saída, tornando `13` visível nos LEDs.

### Etapa 7 — busca de `HLT`

No endereço `3`, a RAM fornece:

```text
RAM[3] = F0 = HLT
```

Essa instrução interrompe o avanço automático do processador.

### Ideia principal

> A RAM não faz a soma. Ela fornece a instrução `ADD` e os operandos `1` e `12`; o CI 7483 realiza o cálculo.

---

## 14. Relação com as etapas `T1...T6`

As etapas temporais organizam quando a RAM será endereçada e quando seu conteúdo será utilizado.

### Busca da instrução

De forma simplificada:

1. o contador de programa fornece o endereço;
2. o MAR armazena esse endereço;
3. a unidade de controle ativa a leitura da RAM;
4. a instrução lida é armazenada no registrador de instrução.

### Execução de `LDA` ou `ADD`

1. o endereço presente na parte inferior da instrução é transferido ao MAR;
2. a RAM seleciona a posição indicada;
3. `_CE` habilita a leitura;
4. em `LDA`, o dado é armazenado no acumulador;
5. em `ADD`, o dado é armazenado no registrador B e depois o resultado do somador é gravado no acumulador.

A RAM responde combinacionalmente à leitura, mas os registradores recebem os valores em etapas controladas pelo clock.

---

## 15. Relação com o waveform

O waveform possui duas fases principais:

| Intervalo aproximado | Modo | Função da RAM |
|---:|---|---|
| `0...600 ns` | programação | recebe endereços, dados e pulsos de escrita externos |
| após `600 ns` | execução | fornece instruções e operandos de acordo com o MAR |

Durante a programação, os grupos de sinais percorrem:

```text
ENDEREÇO: 0, 1, 2, 3, E, F
DADO:     0E, 1F, E0, F0, 01, 0C
```

Durante a execução, a RAM participa dos eventos visíveis:

| Evento | Conteúdo fornecido pela RAM |
|---|---|
| carregamento do acumulador | `RAM[E] = 1` |
| carregamento do registrador B | `RAM[F] = 12` |
| busca de `OUT` | `RAM[2] = E0` |
| busca de `HLT` | `RAM[3] = F0` |

As linhas `VALOR_A`, `REG_B` e `SAIDA` tornam mais fácil explicar o efeito desses dados, mas não são todas saídas diretas da RAM:

- `VALOR_A = 1` é um marcador visual do primeiro operando;
- `REG_B = 12` mostra o valor que foi lido da RAM e armazenado no registrador B;
- `SAIDA = 13` mostra o resultado final depois da soma e da instrução `OUT`.

---

## 16. Diferença entre RAM e registradores

Tanto a RAM quanto os registradores armazenam bits, mas possuem funções diferentes.

| Característica | RAM | Registrador com CI 74173 |
|---|---|---|
| Quantidade de posições | 16 palavras | uma palavra por registrador funcional |
| Seleção | endereço `A0...A3` | sinais de carga e habilitação |
| Função | guardar programa e dados | guardar temporariamente um valor de trabalho |
| Exemplos | instruções, `1` e `12` | acumulador, registrador B e saída |
| Leitura | posição escolhida pelo endereço | conteúdo do registrador selecionado |

### Fala sugerida

“A RAM funciona como um conjunto de gavetas numeradas. O endereço escolhe a gaveta. Um registrador funciona como uma única gaveta com uma função específica, como acumulador ou registrador B.”

---

## 17. Roteiro de fala completo

“A RAM é a memória que armazena as instruções e os dados utilizados pelo SAP-1. O termo acesso aleatório significa que qualquer posição pode ser acessada diretamente por seu endereço.

No nosso projeto, a RAM possui quatro bits de endereço, A0 até A3. Quatro bits formam 16 combinações, por isso existem 16 posições, numeradas de 0 até F em hexadecimal. Cada posição guarda oito bits. A capacidade total é de 16 palavras de oito bits, ou 128 bits.

O arquivo RAM.bdf implementa apenas quatro bits por endereço. Por isso, usamos duas instâncias no SAP1.bdf. A instância inst105 armazena os quatro bits superiores e a instância inst106 armazena os quatro bits inferiores. Como as duas recebem o mesmo endereço, juntas formam uma palavra de oito bits.

Internamente, cada bloco RAM possui 64 células de um bit. Cada célula é construída no arquivo CelulaRam.bdf com um flip-flop D, portas de controle e um buffer de três estados. O flip-flop guarda o bit e o buffer permite que somente a célula selecionada controle a linha de saída.

As quatro entradas de endereço passam por uma lógica decodificadora. Essa lógica usa inversores e portas AND de quatro entradas para gerar 16 sinais de seleção. Para cada endereço, apenas uma palavra fica selecionada.

O sinal WE é ativo em nível baixo. Quando WE vale zero, o dado presente nas entradas é gravado no endereço selecionado. Quando WE vale um, o conteúdo é apenas lido e não é alterado. O sinal CS também é ativo em nível baixo e controla quando a memória pode fornecer sua saída.

O SAP-1 possui dois modos. Durante a programação, run_prog vale zero. O CI 74157 escolhe o endereço externo, os pinos D7 até D0 fornecem os dados e LeituraEscrita gera os pulsos de gravação.

Durante a execução, run_prog vale um. O endereço passa a vir do MAR e a unidade de controle ativa CE quando precisa que a RAM coloque uma instrução ou um dado no barramento.

No programa demonstrado, gravamos 0E no endereço 0, representando LDA E; 1F no endereço 1, representando ADD F; E0 no endereço 2, representando OUT; e F0 no endereço 3, representando HLT. Também gravamos o número 1 no endereço E e o número 12 no endereço F.

Na execução, a RAM primeiro fornece a instrução LDA E. Em seguida, fornece o valor 1, que é armazenado no acumulador. Depois fornece a instrução ADD F e o valor 12, que é armazenado no registrador B.

O CI 7483 soma 1 e 12 e produz 13. A RAM não realiza essa soma; ela apenas guardou e forneceu os valores necessários. Depois, a RAM fornece a instrução OUT, que envia 13 para os LEDs, e finalmente fornece HLT, que encerra o processamento.

Portanto, a RAM funciona como a memória de trabalho do programa: guarda o que deve ser executado e os dados que serão processados, enquanto os demais componentes buscam, interpretam e executam essas informações.”

---

## 18. Perguntas que podem ser feitas

### Quantos endereços existem na RAM?

Existem 16 endereços, porque quatro bits de endereço produzem `2⁴ = 16` combinações.

### Quantos bits existem em cada endereço?

Cada endereço armazena oito bits.

### Qual é a capacidade total?

São 16 palavras de oito bits, totalizando 128 bits ou 16 bytes.

### Por que são utilizados dois blocos RAM?

Porque cada `RAM.bdf` armazena quatro bits por endereço. Dois blocos em paralelo formam oito bits.

### O que significa acesso aleatório?

Significa que qualquer posição pode ser acessada diretamente pelo endereço, e não que o conteúdo seja aleatório.

### Qual é a função de `_WE`?

Controlar escrita e leitura. Como é ativo em nível baixo, zero solicita escrita e um mantém a memória em leitura.

### Qual é a função de `_CS`?

Selecionar a memória para fornecer sua saída. Em nível alto, o caminho de saída fica em alta impedância.

### O que acontece com as posições não selecionadas durante uma escrita?

Elas mantêm seus valores, pois somente uma linha `SELECT` é ativada pelo decodificador.

### A leitura apaga o dado?

Não. A leitura apenas apresenta uma cópia do conteúdo na saída.

### A RAM usa o clock geral do processador?

Não diretamente. A célula grava quando a combinação de seleção e pulso de escrita produz a transição no flip-flop. O clock geral organiza os registradores e as etapas da execução.

### A RAM faz a soma de 1 com 12?

Não. Ela armazena as instruções e fornece os operandos. A soma é realizada pelo CI 7483.

### Quem escolhe o endereço durante a programação?

Os pinos externos `A0...A3`, selecionados pelo CI 74157 quando `run_prog = 0`.

### Quem escolhe o endereço durante a execução?

O MAR, selecionado pelo CI 74157 quando `run_prog = 1`.

### Por que instruções e dados estão na mesma RAM?

Porque o SAP-1 utiliza o mesmo espaço de 16 endereços para guardar o programa e os operandos.

### A memória é permanente?

Não. Ela é construída com flip-flops e funciona como memória volátil. No waveform, o programa é gravado antes de iniciar a execução.

### A RAM foi implementada em Verilog?

Não. A célula, o bloco de quatro bits e a conexão dos dois blocos foram implementados em diagramas `.bdf`.

---

## 19. Conclusão sugerida

### Fala para encerrar

“Para concluir, a RAM é o componente que guarda o programa e os dados do SAP-1.

Ela possui 16 endereços de oito bits. Dois blocos de 16 por 4 bits funcionam em paralelo, e cada bloco é construído com células de um bit, decodificação de endereço e buffers de três estados.

Durante a programação, os endereços e os dados são externos. Durante a execução, o CI 74157 entrega o controle dos endereços ao MAR, e a unidade de controle solicita as leituras necessárias.

No exemplo de 1 mais 12, a RAM guarda as instruções LDA, ADD, OUT e HLT, além dos operandos 1 e 12. Ela fornece cada informação no momento correto para que o processador produza e apresente o resultado 13.”

---

## Resumo em uma frase

> A RAM do SAP-1 é uma memória de 16 palavras de oito bits que armazena as instruções e os dados e fornece cada conteúdo de acordo com o endereço selecionado.

A memória está implementada exclusivamente em diagramas de bloco:

- `CelulaRam.bdf`: célula de armazenamento de um bit;
- `RAM.bdf`: memória de 16 palavras de quatro bits;
- `SAP1.bdf`: duas instâncias em paralelo, formando a memória de 16 palavras de oito bits.
