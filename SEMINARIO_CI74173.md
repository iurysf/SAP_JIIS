# Seminário: a função do CI 74173 no projeto SAP-1

## Objetivo da apresentação

Esta apresentação explica o que é o CI 74173, como ele armazena dados, como controla sua ligação com o barramento e por que ele é essencial para o funcionamento do SAP-1.

Ao final, a ideia principal deve ficar clara:

> O CI 74173 é a memória temporária do processador. Ele recebe quatro bits, guarda esses bits no momento determinado pelo clock e somente coloca o valor no barramento quando a unidade de controle autoriza.

---

## 1. Introdução sugerida

### Fala para o seminário

“Neste projeto nós construímos um processador SAP-1 utilizando diagramas de blocos. Para um processador funcionar, não basta apenas realizar operações matemáticas. Ele também precisa guardar temporariamente instruções, operandos e resultados.

O componente responsável por grande parte desse armazenamento é o CI 74173. Ele funciona como um registrador de quatro bits. Isso significa que ele consegue guardar quatro valores binários ao mesmo tempo, formando números entre zero e quinze.

No nosso SAP-1, vários CI 74173 são utilizados. Quando precisamos armazenar oito bits, usamos dois componentes em conjunto: um para os quatro bits menos significativos e outro para os quatro bits mais significativos.”

---

## 2. O que é o CI 74173

O CI 74173 é um registrador de quatro bits com:

- quatro entradas de dados;
- quatro saídas de dados;
- entrada de clock;
- controle de gravação;
- controle de saída;
- entrada de limpeza;
- saídas de três estados.

Internamente, ele utiliza quatro flip-flops do tipo D. Cada flip-flop armazena um bit.

| Flip-flop | Entrada | Saída | Bit armazenado |
|---|---|---|---|
| 0 | `D0` | `Q0` | bit 0 |
| 1 | `D1` | `Q1` | bit 1 |
| 2 | `D2` | `Q2` | bit 2 |
| 3 | `D3` | `Q3` | bit 3 |

Por exemplo, se as entradas forem:

`D3...D0 = 1100`

o registrador poderá armazenar:

`Q3...Q0 = 1100 = 12`

### Fala sugerida

“Podemos imaginar o CI 74173 como uma pequena caixa de memória. Nas entradas D nós apresentamos um valor. Quando os sinais de controle permitem e acontece a borda do clock, o componente fecha a caixa e guarda aquele valor. Depois disso, o dado continua armazenado mesmo que as entradas mudem.”

---

## 3. Função de cada entrada e saída

No diagrama `CI74173.bdf`, o componente possui os seguintes sinais:

| Sinal | Tipo | Função no componente |
|---|---|---|
| `D0` a `D3` | entrada | Recebem os quatro bits que podem ser armazenados. |
| `Q0` a `Q3` | saída | Apresentam os quatro bits que estão armazenados. |
| `CP` | entrada | Clock que determina o momento da gravação. |
| `E1` e `E2` | entrada | Habilitam a gravação de um novo valor. |
| `OE1` e `OE2` | entrada | Habilitam as saídas de três estados. |
| `MR` | entrada | Limpa o registrador, fazendo os bits armazenados voltarem para zero. |

### Polaridade dos controles neste projeto

Os sinais `E1`, `E2`, `OE1` e `OE2` passam por inversores no diagrama. Portanto, são controles ativos em nível baixo.

- `E1 = 0` e `E2 = 0`: a gravação fica habilitada.
- `OE1 = 0` e `OE2 = 0`: as saídas ficam habilitadas.
- `MR = 1`: o conteúdo é apagado.

### Resumo de funcionamento

| MR | E1 | E2 | Evento de clock | Resultado |
|---:|---:|---:|---|---|
| `1` | X | X | não importa | O registrador é zerado. |
| `0` | `0` | `0` | borda de subida | O valor de `D3...D0` é armazenado. |
| `0` | `1` | X | borda de subida | O valor anterior é mantido. |
| `0` | X | `1` | borda de subida | O valor anterior é mantido. |

O símbolo `X` significa que aquele sinal pode ser zero ou um sem alterar o resultado descrito.

---

## 4. Por que o clock é importante

O clock impede que o registrador altere seu conteúdo a qualquer momento.

Enquanto as entradas `D0...D3` podem mudar continuamente, a gravação só acontece quando:

1. `E1` e `E2` estão habilitados;
2. ocorre uma borda de subida em `CP`.

Depois da gravação, o valor permanece armazenado até que:

- outro valor seja gravado; ou
- `MR` limpe o registrador.

### Fala sugerida

“A função do clock é organizar o tempo do processador. Os dados podem estar passando pelo barramento, mas o CI 74173 não grava imediatamente. Ele espera a autorização da unidade de controle e a borda correta do clock. Isso garante que todos os componentes trabalhem de forma sincronizada.”

---

## 5. O que são as saídas de três estados

As saídas do CI 74173 podem assumir três condições:

- nível lógico `0`;
- nível lógico `1`;
- alta impedância, representada por `Z`.

Alta impedância significa que o componente fica eletricamente desconectado do barramento. Ele continua guardando o valor internamente, mas não tenta controlar as linhas compartilhadas.

| OE1 | OE2 | Comportamento das saídas |
|---:|---:|---|
| `0` | `0` | `Q0...Q3` colocam o valor armazenado no barramento. |
| `1` | X | Saídas em alta impedância. |
| X | `1` | Saídas em alta impedância. |

### Por que isso é necessário

O SAP-1 possui um barramento compartilhado. Vários componentes podem enviar dados por esse mesmo caminho, mas somente um deve falar por vez.

Se dois registradores colocassem valores diferentes no barramento simultaneamente, haveria conflito. Os controles `OE1` e `OE2` evitam esse problema.

### Fala sugerida

“O barramento pode ser comparado a um único microfone utilizado por várias pessoas. Todos podem ter informações para transmitir, mas apenas uma pessoa deve usar o microfone por vez. Os sinais OE determinam quando o CI 74173 pode falar no barramento. Quando não está autorizado, ele entra em alta impedância e deixa o caminho livre.”

---

## 6. Como o CI 74173 foi construído no projeto

O componente foi implementado no arquivo `CI74173.bdf`, somente com diagrama de blocos.

Os principais elementos internos são:

- quatro flip-flops D, um para cada bit;
- portas lógicas que controlam a gravação;
- caminhos de realimentação para manter o valor anterior;
- inversores para os controles ativos em nível baixo;
- buffers `TRI` para produzir as saídas de três estados;
- lógica de limpeza conectada ao sinal `MR`.

### Funcionamento interno simplificado

Para cada bit, o circuito decide entre dois valores:

1. o novo bit presente na entrada `D`;
2. o bit que já estava armazenado na saída do flip-flop.

Quando a gravação está habilitada, o novo dado é selecionado. Quando a gravação está desabilitada, a saída é realimentada e o valor anterior é mantido.

Na borda de subida do clock, o flip-flop registra o valor selecionado.

### Fala sugerida

“Apesar de utilizarmos o nome CI 74173, ele não aparece como código Verilog. O funcionamento foi montado graficamente. Os quatro flip-flops guardam os bits, as portas lógicas escolhem entre manter ou atualizar o conteúdo, e os buffers TRI controlam a comunicação com o barramento.”

---

## 7. Onde o CI 74173 aparece no SAP-1

O CI 74173 é reutilizado em vários pontos do processador. Como cada componente armazena quatro bits, registradores de oito bits são construídos com dois componentes.

### Acumulador

No acumulador, os componentes `inst6` e `inst7` formam um registrador de oito bits:

- `inst6`: parte mais significativa;
- `inst7`: parte menos significativa.

O acumulador guarda:

- o valor carregado pela instrução `LDA`;
- o resultado produzido pelo somador.

No exemplo do projeto:

1. recebe `00000001`, que vale `1`;
2. depois recebe `00001101`, que vale `13`.

### Registrador B

Os componentes `inst8` e `inst9` formam o registrador B:

- `inst8`: parte mais significativa;
- `inst9`: parte menos significativa.

O registrador B guarda o segundo operando utilizado pelo somador.

No programa atual, ele recebe:

`00001100 = 12`

No waveform, esse conteúdo aparece no grupo:

`REG_B = 12`

Os sinais `RB3...RB0` correspondem aos quatro bits menos significativos reais armazenados pelo CI 74173 `inst9`.

### Registrador de saída

Os componentes `inst97` e `inst98` formam o registrador de saída de oito bits.

Quando a instrução `OUT` é executada, esse registrador recebe o conteúdo do acumulador e mantém o valor disponível para os LEDs.

No exemplo:

`LED7...LED0 = 00001101 = 13`

### Outros usos

Outras instâncias do CI 74173 também são utilizadas como armazenamento temporário em partes internas do SAP-1. Em todos os casos, o princípio é o mesmo:

1. receber bits;
2. aguardar a habilitação;
3. gravar no clock;
4. manter o valor;
5. liberar a saída somente quando autorizado.

---

## 8. Participação no cálculo 1 + 12 = 13

O CI 74173 não realiza a soma. A soma é realizada pelo CI 7483.

A função do CI 74173 é guardar os valores antes e depois da operação.

### Etapa 1 — armazenamento do primeiro operando

A instrução `LDA E` lê da RAM:

`01 = 1`

O acumulador, construído com CI 74173, armazena esse valor.

### Etapa 2 — armazenamento do segundo operando

A instrução `ADD F` lê da RAM:

`0C = 12`

O registrador B, também construído com CI 74173, armazena esse valor.

No waveform:

`REG_B = 12`

### Etapa 3 — operação matemática

O CI 7483 recebe simultaneamente:

```text
Acumulador: 0001 = 1
Registro B: 1100 = 12
```

O somador produz:

```text
  0001
+ 1100
------
  1101 = 13
```

### Etapa 4 — armazenamento do resultado

Na etapa de controle adequada, o acumulador recebe:

`1101 = 13`

O CI 74173 mantém esse resultado estável mesmo depois que a saída do somador muda.

### Etapa 5 — apresentação nos LEDs

A instrução `OUT` transfere o resultado para o registrador de saída:

`00001101 = 13`

O registrador de saída mantém os LEDs acesos:

- `LED3 = 1`;
- `LED2 = 1`;
- `LED0 = 1`.

### Ideia principal

> O CI 7483 calcula, mas o CI 74173 guarda. Sem os registradores, os operandos e o resultado não permaneceriam estáveis durante as etapas do processador.

---

## 9. Relação com o waveform

As linhas principais do waveform são:

| Linha | Relação com o CI 74173 |
|---|---|
| `VALOR_A = 1` | É um marcador visual do primeiro operando; não é o conteúdo direto de um CI 74173. |
| `REG_B = 12` | Mostra o conteúdo real armazenado no CI 74173 do registrador B. |
| `SAIDA = 13` | Mostra o valor mantido pelos CI 74173 do registrador de saída. |
| `T1...T6` | Mostram as etapas que determinam quando os registradores carregam ou liberam dados. |

### Sequência observada

| Momento aproximado | Evento |
|---:|---|
| `600 ns` | `VALOR_A` identifica o primeiro operando `1`. |
| `730 ns` | O acumulador armazena internamente o valor `1`. |
| `850 ns` | O registrador B armazena `12`. |
| `870 ns` | O acumulador armazena o resultado `13`. |
| `950 ns` | O registrador de saída armazena e apresenta `13`. |

---

## 10. Roteiro de fala completo

“O CI 74173 é um registrador de quatro bits. Sua função é armazenar temporariamente informações binárias dentro do SAP-1.

Ele possui quatro entradas, chamadas D0 até D3, e quatro saídas, chamadas Q0 até Q3. Cada par de entrada e saída está associado a um flip-flop D. Assim, cada flip-flop guarda um bit e o conjunto guarda quatro bits.

A gravação não acontece o tempo todo. Primeiro, as entradas E1 e E2 devem estar habilitadas. Como esses sinais são ativos em nível baixo no nosso circuito, os dois devem estar em zero. Depois, na borda de subida do clock CP, o valor das entradas D é armazenado.

Se a gravação não estiver habilitada, o CI 74173 mantém o valor anterior. Isso é importante porque o processador precisa que os dados permaneçam estáveis durante várias etapas.

O componente também possui OE1 e OE2, que controlam as saídas. Quando os dois estão em zero, o valor armazenado pode ser colocado no barramento. Caso contrário, as saídas ficam em alta impedância. Dessa forma, somente um componente controla o barramento por vez.

O sinal MR é utilizado para limpar o registrador. Quando ele é ativado, as quatro saídas voltam para zero.

No nosso SAP-1, usamos vários CI 74173. Dois formam o acumulador de oito bits, dois formam o registrador B e dois formam o registrador de saída.

Na operação mostrada no waveform, a instrução LDA carrega o número 1 no acumulador. Depois, a instrução ADD carrega o número 12 no registrador B. O CI 7483 soma esses dois valores e produz 13. Em seguida, o acumulador guarda o resultado e a instrução OUT transfere esse 13 para o registrador de saída.

Portanto, o CI 74173 não é o componente que soma. Sua função é guardar o 1, guardar o 12, guardar o resultado 13 e controlar quando cada valor pode entrar no barramento. Sem ele, o processador não conseguiria manter os dados organizados entre uma etapa e outra.”

---

## 11. Perguntas que podem ser feitas

### O CI 74173 realiza a soma?

Não. A soma é realizada pelo CI 7483. O CI 74173 armazena os operandos e o resultado.

### Por que são utilizados dois CI 74173 em alguns registradores?

Porque cada CI 74173 armazena quatro bits. Dois componentes formam um registrador de oito bits.

### O que acontece quando E1 ou E2 está desabilitado?

O registrador não aceita um novo dado e mantém o conteúdo que já estava armazenado.

### Qual é a função de OE1 e OE2?

Eles determinam quando o valor armazenado pode ser colocado no barramento.

### O dado é perdido quando as saídas ficam em alta impedância?

Não. Alta impedância apenas desconecta as saídas do barramento. O conteúdo continua armazenado nos flip-flops.

### Para que serve MR?

Serve para zerar o conteúdo do registrador durante a inicialização ou limpeza do processador.

### Qual é a diferença entre registrador B e acumulador?

O acumulador guarda o primeiro operando e depois recebe o resultado. O registrador B guarda o segundo operando que será enviado ao somador.

### Por que o valor continua aparecendo depois do pulso de clock?

Porque os flip-flops mantêm o estado gravado até a próxima gravação ou até a limpeza.

---

## 12. Conclusão sugerida

### Fala para encerrar

“Para concluir, o CI 74173 é fundamental porque fornece memória temporária e controle de barramento ao SAP-1.

Ele sincroniza a gravação com o clock, mantém os valores estáveis, permite a limpeza dos registradores e evita conflitos no barramento usando saídas de três estados.

No nosso exemplo, ele participa armazenando o valor 1 no acumulador, o valor 12 no registrador B e o resultado 13 no registrador de saída. O CI 7483 faz a soma, mas são os CI 74173 que garantem que cada valor seja preservado e utilizado no momento correto.”

---

## Resumo em uma frase

> O CI 74173 é um registrador de quatro bits que armazena, mantém e libera dados no momento determinado pelos sinais de controle do SAP-1.

O componente e o processador estão implementados exclusivamente em diagramas de bloco:

- `CI74173.bdf`: implementação interna do registrador;
- `SAP1.bdf`: utilização das instâncias do CI 74173 no processador.
