# Seminário: a função do CI 74157 no projeto SAP-1

## Objetivo da apresentação

Esta apresentação explica o que é o CI 74157, como ele seleciona uma entre duas fontes de dados e qual é sua função específica no endereçamento da memória RAM do SAP-1.

A ideia principal é:

> O CI 74157 escolhe quem controla o endereço da RAM: os pinos externos usados durante a programação ou o registrador de endereço usado durante a execução automática.

---

## 1. Introdução sugerida

### Fala para o seminário

“O SAP-1 precisa utilizar a memória RAM em dois momentos diferentes.

No primeiro momento, antes da execução, nós precisamos escolher manualmente os endereços para gravar as instruções e os operandos. No segundo momento, durante a execução, o próprio processador precisa controlar os endereços da memória.

Como existem duas possíveis fontes para o mesmo endereço da RAM, é necessário um componente que escolha qual delas será utilizada. Esse componente é o CI 74157.

O CI 74157 é um multiplexador quádruplo de duas entradas. Ele possui quatro canais e, em cada canal, escolhe uma entre duas entradas. Dessa forma, ele consegue selecionar de uma só vez um endereço completo de quatro bits.”

---

## 2. O que é o CI 74157

O CI 74157 é um multiplexador quádruplo 2 para 1.

O termo “quádruplo” significa que existem quatro multiplexadores funcionando em paralelo. O termo “2 para 1” significa que cada canal recebe duas possíveis entradas e produz uma única saída.

No diagrama `ci74157.bdf`, os quatro canais são:

| Canal | Entrada 0 | Entrada 1 | Saída |
|---|---|---|---|
| A | `A0` | `A1` | `Za` |
| B | `B0` | `B1` | `Zb` |
| C | `C0` | `C1` | `Zc` |
| D | `D0` | `D1` | `Zd` |

Todos os canais compartilham:

- o sinal de seleção `S`;
- o sinal de habilitação `E`.

### Exemplo simples

Considere duas palavras de quatro bits:

```text
Entrada 0: 0011
Entrada 1: 1100
```

Se `S = 0`, a saída será:

`0011`

Se `S = 1`, a saída será:

`1100`

### Fala sugerida

“O CI 74157 pode ser comparado a uma chave seletora de quatro vias. Ele recebe duas palavras de quatro bits, mas somente uma delas é encaminhada para as saídas. O sinal S determina qual palavra atravessa o componente.”

---

## 3. Função de cada sinal

| Sinal | Tipo | Função |
|---|---|---|
| `A0`, `B0`, `C0`, `D0` | entrada | Formam a primeira palavra de quatro bits. |
| `A1`, `B1`, `C1`, `D1` | entrada | Formam a segunda palavra de quatro bits. |
| `Za`, `Zb`, `Zc`, `Zd` | saída | Formam a palavra selecionada. |
| `S` | entrada | Escolhe entre o conjunto 0 e o conjunto 1. |
| `E` | entrada | Habilita o funcionamento das quatro saídas. |

O sinal `E` é ativo em nível baixo:

- `E = 0`: o multiplexador funciona normalmente;
- `E = 1`: as quatro saídas são forçadas para zero.

---

## 4. Tabela de funcionamento

| E | S | Za | Zb | Zc | Zd |
|---:|---:|---|---|---|---|
| `1` | X | `0` | `0` | `0` | `0` |
| `0` | `0` | `A0` | `B0` | `C0` | `D0` |
| `0` | `1` | `A1` | `B1` | `C1` | `D1` |

O símbolo `X` significa que o valor de `S` não altera o resultado quando o componente está desabilitado.

### Equação lógica de um canal

Quando o componente está habilitado, cada saída pode ser representada por:

`Z = (entrada_0 AND NOT S) OR (entrada_1 AND S)`

Para o canal A:

`Za = (A0 AND NOT S) OR (A1 AND S)`

O mesmo raciocínio é utilizado nos canais B, C e D.

---

## 5. O CI 74157 possui memória?

Não.

O CI 74157 é um circuito combinacional. Isso significa que:

- ele não possui flip-flops;
- ele não armazena o valor anterior;
- ele não precisa de clock;
- a saída acompanha imediatamente a entrada selecionada.

Essa é uma diferença importante em relação ao CI 74173:

| Componente | Função | Possui memória? | Utiliza clock? |
|---|---|---:|---:|
| CI 74157 | Seleciona uma entre duas fontes | Não | Não |
| CI 74173 | Armazena quatro bits | Sim | Sim |

### Fala sugerida

“O CI 74157 não guarda informação. Se a entrada selecionada mudar, a saída também muda. A função dele é apenas decidir por qual caminho o dado deve passar. Já o CI 74173 é um registrador e consegue manter um valor armazenado.”

---

## 6. Como o CI 74157 foi implementado

O componente foi construído no arquivo `ci74157.bdf`, exclusivamente com diagrama de blocos.

A implementação utiliza:

- inversores `NOT`;
- portas `AND`;
- portas `OR`;
- uma lógica repetida para cada um dos quatro canais.

### Funcionamento interno de cada canal

1. O sinal `S` é utilizado diretamente em um caminho.
2. Uma porta `NOT` produz o valor `NOT S` para o outro caminho.
3. A entrada do conjunto 0 é combinada com `NOT S`.
4. A entrada do conjunto 1 é combinada com `S`.
5. Uma porta `OR` reúne os dois caminhos.
6. O sinal `E` permite ou bloqueia a saída.

Como `S` e `NOT S` nunca ficam ativos ao mesmo tempo, apenas uma entrada de cada par chega à saída.

### Fala sugerida

“Internamente, cada canal possui dois caminhos. Um caminho é habilitado quando S vale zero e o outro é habilitado quando S vale um. Depois, uma porta OR reúne esses caminhos. Esse circuito é repetido quatro vezes para selecionar os quatro bits ao mesmo tempo.”

---

## 7. Onde o CI 74157 aparece no SAP-1

No arquivo `SAP1.bdf`, o CI 74157 aparece como a instância:

`CI74157: inst42`

Ele está conectado antes das entradas de endereço da memória RAM.

Sua função é selecionar entre dois endereços de quatro bits:

1. endereço externo, definido pelos pinos `A0...A3`;
2. endereço interno, fornecido pelo registrador de endereço da memória, identificado pelos sinais `MAR_A0...MAR_A3`.

As quatro saídas do CI 74157 seguem para as quatro entradas de endereço da RAM.

### Caminho simplificado

```text
Endereço externo A0...A3 ─┐
                          ├─ CI 74157 ──> endereço da RAM
Endereço interno do MAR ──┘
```

O componente não altera os dados armazenados na RAM. Ele apenas determina qual endereço será acessado.

---

## 8. Função de run_prog

No projeto, o sinal de seleção `S` do CI 74157 está conectado a:

`run_prog`

O sinal de habilitação `E` está conectado ao `GND`, portanto:

`E = 0`

Isso deixa o multiplexador permanentemente habilitado.

Assim, a seleção fica:

| run_prog | Fonte selecionada | Modo do projeto |
|---:|---|---|
| `0` | endereço externo `A0...A3` | programação da RAM |
| `1` | endereço interno `MAR_A0...MAR_A3` | execução do programa |

### Fala sugerida

“O sinal run_prog é a chave que muda o controle da memória. Enquanto run_prog vale zero, nós escolhemos manualmente os endereços usados para gravar a RAM. Quando run_prog passa para um, o controle é entregue ao processador, e o endereço passa a vir do MAR.”

---

## 9. Funcionamento durante a programação da RAM

Entre `0 ns` e `600 ns`, o waveform mantém:

`run_prog = 0`

Nesse período, o CI 74157 seleciona o endereço externo.

O grupo `ENDERECO` percorre:

`0, 1, 2, 3, E, F`

Esses endereços são utilizados para gravar:

| Endereço | Dado | Função |
|---|---:|---|
| `0` | `0E` | instrução `LDA E` |
| `1` | `1F` | instrução `ADD F` |
| `2` | `E0` | instrução `OUT` |
| `3` | `F0` | instrução `HLT` |
| `E` | `01` | primeiro operando: `1` |
| `F` | `0C` | segundo operando: `12` |

### O papel do CI 74157 nessa etapa

O CI 74157 encaminha o endereço escolhido externamente para a RAM.

Por exemplo:

```text
A3...A0 = 1110
```

O multiplexador encaminha:

```text
Endereço da RAM = 1110 = E
```

Assim, o valor `01` pode ser gravado no endereço `E`.

---

## 10. Funcionamento durante a execução

Em aproximadamente `600 ns`:

`run_prog = 1`

Nesse momento, o CI 74157 deixa de utilizar os pinos externos e passa a encaminhar o endereço interno do MAR.

O MAR recebe os endereços necessários para:

- buscar a próxima instrução;
- acessar o operando da instrução;
- ler o endereço `E` durante `LDA E`;
- ler o endereço `F` durante `ADD F`.

### Exemplo: LDA E

1. A unidade de controle identifica a instrução `LDA E`.
2. O endereço `E` é colocado no MAR.
3. O CI 74157 está com `run_prog = 1`.
4. O endereço do MAR atravessa o multiplexador.
5. A RAM acessa o endereço `E`.
6. A RAM entrega o valor `01`.

### Exemplo: ADD F

1. A unidade de controle identifica a instrução `ADD F`.
2. O endereço `F` é colocado no MAR.
3. O CI 74157 encaminha o endereço do MAR para a RAM.
4. A RAM acessa o endereço `F`.
5. A RAM entrega `0C`, que vale `12`.
6. O registrador B recebe o valor.

---

## 11. Participação no cálculo 1 + 12 = 13

O CI 74157 não realiza a soma e também não armazena os operandos.

Sua participação ocorre no acesso correto à memória.

### Para obter o valor 1

O CI 74157 encaminha o endereço:

`E = 1110`

A RAM responde:

`01 = 1`

### Para obter o valor 12

O CI 74157 encaminha o endereço:

`F = 1111`

A RAM responde:

`0C = 12`

Depois disso:

- o CI 74173 guarda os operandos;
- o CI 7483 realiza a soma;
- o registrador de saída mantém o resultado `13`.

### Ideia principal

> O CI 74157 não calcula o resultado, mas garante que a RAM receba o endereço correto para fornecer os valores usados no cálculo.

---

## 12. Relação com o waveform

As linhas mais importantes para explicar o CI 74157 são:

| Linha | O que demonstra |
|---|---|
| `run_prog` | Determina qual fonte de endereço foi selecionada. |
| `ENDERECO` | Mostra o endereço externo usado durante a gravação. |
| `DADOS` | Mostra o conteúdo gravado no endereço selecionado. |
| `T1...T6` | Mostram as etapas da execução automática. |
| `REG_B` | Confirma que o acesso ao endereço `F` entregou o valor `12`. |
| `SAIDA` | Confirma que o programa terminou com o resultado `13`. |

### Como apresentar no waveform

1. Mostre o intervalo de `0 ns` a `600 ns`.
2. Aponte `run_prog = 0`.
3. Explique que o CI 74157 está selecionando o endereço externo.
4. Mostre os endereços `0, 1, 2, 3, E, F`.
5. Depois, mostre `run_prog` mudando para `1`.
6. Explique que o CI 74157 passa a selecionar o endereço interno do MAR.
7. Mostre que o processador encontra os operandos e produz `SAIDA = 13`.

---

## 13. Roteiro de fala completo

“O CI 74157 é um multiplexador quádruplo de duas entradas. Ele possui quatro canais e consegue selecionar uma entre duas palavras de quatro bits.

Cada canal possui duas entradas. No canal A temos A0 e A1, no canal B temos B0 e B1, e assim por diante. As saídas são Za, Zb, Zc e Zd.

O sinal S faz a seleção. Quando S vale zero, o conjunto de entradas terminado em zero é escolhido. Quando S vale um, o conjunto terminado em um é escolhido.

O componente também possui a entrada E, que é ativa em nível baixo. Quando E vale zero, o multiplexador funciona normalmente. Quando E vale um, as saídas são forçadas para zero.

No nosso projeto, E está ligado ao GND, então o CI 74157 fica sempre habilitado. O sinal S está ligado ao run_prog.

Quando run_prog vale zero, o SAP-1 está no modo de programação da memória. Nesse modo, o CI 74157 seleciona os pinos externos A0 até A3. Isso permite escolher manualmente os endereços onde serão gravadas as instruções e os operandos.

Quando run_prog passa para um, começa a execução automática. O CI 74157 muda a seleção e passa a encaminhar os sinais MAR_A0 até MAR_A3 para a memória RAM. A partir desse momento, o próprio processador escolhe os endereços.

Esse componente não possui memória e não realiza operações matemáticas. Sua função é controlar o caminho do endereço.

Na soma de 1 com 12, ele permite que o processador acesse o endereço E para obter o valor 1 e o endereço F para obter o valor 12. Depois, o registrador B guarda o 12 e o CI 7483 realiza a soma.

Portanto, o CI 74157 funciona como uma chave eletrônica de quatro bits que transfere o controle da memória entre o usuário e o processador.”

---

## 14. Perguntas que podem ser feitas

### O CI 74157 armazena dados?

Não. Ele é um circuito combinacional e apenas seleciona uma das entradas.

### O CI 74157 precisa de clock?

Não. A saída muda de acordo com as entradas, o sinal `S` e o sinal `E`.

### Por que ele possui quatro canais?

Porque o endereço da RAM do SAP-1 possui quatro bits. Os quatro bits precisam ser selecionados simultaneamente.

### Qual é a função de run_prog?

`run_prog` controla a entrada `S` e escolhe entre o endereço externo e o endereço interno do MAR.

### Por que a entrada E está ligada ao GND?

Porque `E` é ativa em nível baixo. Ligá-la ao GND mantém o componente sempre habilitado.

### O que acontece se E receber nível alto?

As quatro saídas são forçadas para zero, independentemente das entradas e de `S`.

### O que acontece se o CI 74157 for removido?

Não existiria uma seleção organizada entre o endereço externo e o endereço do MAR. A programação manual e a execução automática tentariam utilizar o mesmo caminho de forma inadequada.

### Qual é a diferença entre CI 74157 e CI 74173?

O CI 74157 escolhe um caminho e não armazena dados. O CI 74173 armazena dados e utiliza clock.

### O CI 74157 participa diretamente da soma?

Não. Ele participa indiretamente, selecionando os endereços corretos para que a RAM forneça os operandos.

### Por que as entradas possuem nomes A0, A1, B0 e B1?

As letras identificam os quatro canais. Os números zero e um identificam as duas possíveis fontes de cada canal.

---

## 15. Conclusão sugerida

### Fala para encerrar

“Para concluir, o CI 74157 é o componente que organiza o acesso à memória RAM.

Durante a programação, ele permite que os pinos externos controlem o endereço. Durante a execução, ele transfere esse controle para o MAR.

Ele faz essa troca usando o sinal run_prog, selecionando quatro bits ao mesmo tempo. Por ser um circuito combinacional, não precisa de clock e não armazena valores.

Mesmo sem realizar a soma, ele é essencial para o resultado final, porque garante que a RAM seja acessada nos endereços corretos e entregue os valores 1 e 12 ao processador.”

---

## Resumo em uma frase

> O CI 74157 é um multiplexador de quatro bits que seleciona se o endereço da RAM será controlado externamente ou pelo MAR do SAP-1.

O componente e sua utilização estão implementados somente em diagramas de bloco:

- `ci74157.bdf`: implementação interna do multiplexador;
- `SAP1.bdf`: conexão da instância `inst42` entre as fontes de endereço e a RAM.
