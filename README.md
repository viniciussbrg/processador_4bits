Este repositório contém a implementação de uma CPU modular de 4 bits desenvolvida no simulador Logisim. O projeto foi estruturado com base na separação rigorosa entre o Caminho de Dados (Datapath) e a Unidade de Controle.

Integrantes: 
•	Eduardo Urbanovicz de Souza – RA: 082230018
•	Lucas Daiki Honda Kuniyosi – RA: 082230020
•	Ronaldo de Oliveira Santos – RA: 082230031



Vídeo para explicação em voz do projeto e execução :

[![Assistir no YouTube](https://img.shields.io/badge/YouTube-Assistir_Apresentação-red?style=for-the-badge&logo=youtube)](https://www.youtube.com/watch?v=BBcqaQcUtOk)

## 1. Descrição Geral da Arquitetura
A CPU possui uma arquitetura de 4 bits focada em processamento sequencial. O hardware é composto pelos seguintes blocos principais:
* **Program Counter (PC):** Registrador de 8 bits responsável por apontar o endereço da instrução atual.
* **Memória de Instruções (ROM):** Armazena o código de máquina. Capacidade de endereçamento de 8 bits (256 linhas) e saída de dados de 10 bits.
* **Unidade de Controle:** Circuito combinacional puramente decodificador que roteia os sinais da instrução para o datapath.
* **Banco de Registradores:** Contém 4 registradores de uso geral (R0, R1, R2, R3) de 4 bits. Permite leitura simultânea de dois operandos e escrita isolada com proteção por decodificador.
* **ULA (Unidade Lógica e Aritmética):** Processador central de 4 bits com multiplexador de 8 canais, sem pinos de flag/status habilitados.

## 2. Conjunto de Instruções Desenvolvido
A arquitetura suporta 8 operações fundamentais, divididas em duas categorias (Aritméticas e Lógicas):

**Aritméticas:**
* `ADD`: Soma dois registradores.
* `SUB`: Subtrai um registrador do outro.
* `INC`: Incrementa (+1) o valor de um registrador.
* `DEC`: Decrementa (-1) o valor de um registrador.

**Lógicas:**
* `AND`: Operação E bit a bit.
* `OR`: Operação OU bit a bit.
* `XOR`: Operação OU Exclusivo bit a bit.
* `NOT`: Inversão lógica dos 4 bits de um registrador.

## 3. Formato das Instruções
Todas as instruções possuem tamanho fixo de **10 bits**. A decodificação segue a estrutura da esquerda para a direita (MSB para LSB):

* **Opcode (Bits 9-8):** Define a categoria da operação (`00` para Aritmética, `01` para Lógica).
* **R1 / Destino (Bits 7-6):** Endereço do registrador onde o resultado será salvo.
* **R2 / Operando 1 (Bits 5-4):** Endereço do primeiro registrador fonte.
* **R3 / Operando 2 (Bits 3-2):** Endereço do segundo registrador fonte.
* **Funct (Bits 1-0):** Define a operação específica dentro da categoria escolhida pelo Opcode.

## 4. Explicação do Ciclo de Instrução
O processador opera em um ciclo contínuo de três fases, sincronizado pela borda de subida do *clock*:
1. **Busca (Fetch):** O PC envia o endereço atual para a ROM, que disponibiliza a instrução de 10 bits no barramento.
2. **Decodificação (Decode):** A Unidade de Controle recebe os 10 bits, separa o Opcode/Funct para gerar o seletor da ULA (3 bits) e envia os endereços R1, R2 e R3 para o Banco de Registradores.
3. **Execução (Execute & Write-back):** O Banco de Registradores emite os valores dos operandos. A ULA realiza o cálculo e devolve o resultado de 4 bits para a porta de entrada de dados do Banco, que é salvo no registrador de destino (R1) no exato instante do próximo pulso de clock.

## 5. Passo a Passo para Abrir o Circuito
1. Faça o download do software simulador **Logisim** (requer Java instalado).
2. Clone ou faça o download deste repositório para o seu computador.
3. Abra o Logisim, vá em `File` > `Open...` e selecione o arquivo do circuito principal (processador4bits.circ).
4. O circuito principal será carregado. Para visualizar os componentes internos, dê um duplo clique em `ULA`, `UC` ou `BancoReg` na árvore de navegação à esquerda.

## 6. Como Executar o Programa na ROM
1. No circuito principal, clique com o botão direito sobre o componente **ROM**.
2. Selecione **Edit Contents...** (Editar Conteúdo).
3. Na janela hexadecimal, insira a seguinte sequência de instruções na primeira linha:
   `042 084 0d8 02c 070 084 0d8`
4. Feche a janela de edição.
5. Certifique-se de que o pino `CLEAR` está em `0` e que o Chip Select da ROM está em `1`.
6. Selecione a ferramenta **Poke** (ícone de mãozinha) no menu superior.
7. Clique sequencialmente no pino de `CLK` para avançar a execução do programa ciclo a ciclo.

## 7. Exemplos de Execução Passo a Passo
O programa inserido na ROM calcula a **Sequência de Fibonacci** utilizando os limites da arquitetura. 

Abaixo, a evolução do estado dos registradores a cada ciclo de *clock* (representado nos displays Hexadecimais):

* **Ciclo 1 (PC = 01):** R1 = `1` (R0 foi incrementado)
* **Ciclo 2 (PC = 02):** R2 = `1` (R0 + R1)
* **Ciclo 3 (PC = 03):** R3 = `2` (R1 + R2)
* **Ciclo 4 (PC = 04):** R0 = `3` (R2 + R3)
* **Ciclo 5 (PC = 05):** R1 = `5` (R3 + R0)
* **Ciclo 6 (PC = 06):** R2 = `8` (R0 + R1)
* **Ciclo 7 (PC = 07):** R3 = `D` (Equivalente a 13 em decimal, R1 + R2)

*Nota: Se o programa continuar, o ciclo seguinte (8+13 = 21) causará um **overflow** limitante do hardware, pois com 4 bits só teremos os valores de 0 a 15.
