# Relatório do Laboratório 2 - Chamadas de Sistema

---

## Exercício 1a - Observação printf() vs 1b - write()

### Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 9 syscalls
- ex1b_write: 7 syscalls

**2. Por que há diferença entre printf() e write()?**

```
O printf() não é uma syscall, ele utiliza buffering, ou seja, guarda os dados em memória e só chama write() quando precisa ou quando acontece algum evento específico, como o fim do programa.

Já o write é uma syscall direta, ou seja, realiza comunicação com kernel. Além disso, cada chamada write() é uma transição de modo usuário para modo kernel.
```

**3. Qual implementação você acha que é mais eficiente? Por quê?**

```
Acredito que depende da situação, porém na implementação desse laboratório a diferença entre printf() e write() não foi tão grande devido a maneira em que foi construida, já que as mensagens estavam em uma estrutura parecida e cada chamada de printf() acabou resultando em uma chamada de write(). Porém, usando o printf() em contextos onde a saída possui a necessidade de formatação de tipos, como de int para string, acretido que ela seja a melhor escolha pela praticidade, ou até em uso de loops, onde o buffer reduz a quantidade de syscalls e aumenta o desempenho, já que em cada syscall ocorre uma transição de modos.
```

---

## Exercício 2 - Leitura de Arquivo

### Resultados da execução:
- File descriptor: 3
- Bytes lidos: 127

### Comando strace:
```bash
strace -e open,read,close ./ex2_leitura
```

### Análise

**1. Por que o file descriptor não foi 0, 1 ou 2?**

```
O file descriptor não foi 0, 1 ou 2 por que esses números são reservados pelo sistema, no meu caso o número 3 era o primeiro disponível para uso.

- 0: Entrada padrão.
- 1: Saída padrão.
- 2: Erro padrão.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
O arquivo foi lido completamente, pois read() retornou 127 bytes, a mesma quantidade presente no buffer, indicando que todo o conteúdo foi armazenado. Além disso, o arquivo foi fechado com close(), e em seguida foi exibida a mensagem "Arquivo fechado!".
```

---

## Exercício 3 - Contador com Loop

### Resultados (BUFFER_SIZE = 64):
- Linhas: 24 (esperado: 25)
- Caracteres: 1299
- Chamadas read(): 21
- Tempo: 0.000092 segundos

### Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          | 82              | 0.000203  |
| 64          | 21              | 0.000092  |
| 256         | 6               | 0.000068  |
| 1024        | 2               | 0.000067  |

### Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Analisando a tabela acima, podemos concluir que um buffer maior permite ler mais bytes por syscall, reduzindo a necessidade de realizar múltiplas chamadas de read(). Por outro lado, buffers menores exigem mais chamadas para processar todo o arquivo, aumentando a frequência de syscalls.
```

**2. Como você detecta o fim do arquivo?**

```
O fim do arquivo é detectado quando read() retorna 0, indicando que não há mais conteúdo para processar. No ex3, ao usar strace, podemos observar que a última chamada de read() retornou 0. 
read(3, "", 64)                         = 0

No código, é representado pelo loop:
while ((bytes_lidos = read(fd, buffer, BUFFER_SIZE)) > 0)
```

---

## Exercício 4 - Cópia de Arquivo

### Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0.000201 segundos
- Throughput: 6627.02 KB/s

### Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [x] Idênticos [ ] Diferentes

### Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Verificar se bytes_escritos == bytes_lidos é importante para garantir que todos os dados lidos do arquivo de origem foram realmente escritos no arquivo de destino. Caso ocorra algum erro, ele tem que ser tratado.
```

**2. Que flags são essenciais no open() do destino?**

```
fd_destino = open("dados/destino.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);

- O_WRONLY: apenas para escrita.
- O_CREAT: criar o arquivo.
- O_TRUNC: apaga o conteúdo se o arquivo já existir.
```

---

## Análise Geral

### Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
O programa normalmente está ativo no modo usuário. Quando usamos comandos específicos como read() e write(), ocorre uma syscall, que troca o programa para o modo kernel para realizar operações específicas. Depois, o controle retorna ao modo usuário. 
Portanto, cada syscall representa uma transição entre os modos usuário e kernel.
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
Os File Descriptors são identificadores e normalmente possuem valores inteiros não negativos. Eles trabalham como intermediários entre o programa e os arquivos, assim, garantindo uma segurança maior das informações internas.

Existem 3 File Descriptors reservados pelo sistema:
- 0: Entrada padrão.
- 1: Saída padrão.
- 2: Erro padrão.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Quanto maior o buffer, mais dados podem ser processados por syscall, reduzindo o número de chamadas necessárias e aumentando o desempenho. O contrário ocorre com buffers menores, pois exigem mais syscalls, e cada chamada envolve a transição entre o modo usuário e o modo kernel, o que reduz a performance do programa.
```

### Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** ex4_copia

**Por que você acha que foi mais rápido?**

```
Acredito que a diferença esteja na quantidade de syscalls realizadas por cada programa. O ex4_copia é um programa básico de cópia de arquivos e provavelmente não executa todas as verificações que o comando de terminal cp realiza.
```

---

## Entrega

Certifique-se de ter:
- [x] Todos os códigos com TODOs completados
- [x] Traces salvos em `traces/`
- [x] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```