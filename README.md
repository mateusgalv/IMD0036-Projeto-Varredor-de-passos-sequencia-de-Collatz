# Varredor de passos da sequência de Collatz

Programa em C para calcular, para cada inteiro de um intervalo, quantos passos são necessários para a sequência de Collatz chegar a 1. O projeto permite comparar a execução sequencial com versões paralelas usando processos ou threads, com distribuição do trabalho em blocos ou de forma cíclica.

## Compilação

Na raiz do projeto, execute:

```sh
make
```

O executável será criado em `bin/varredor`. Para remover os arquivos de compilação e o executável:

```sh
make clean
```

Os resultados já existentes em `output/` não são removidos por `make clean`.

## Uso

```text
bin/varredor <A> <B> <W> <modo> <particao> <arquivo_saida>
```

- `A`: primeiro inteiro do intervalo (inclusivo).
- `B`: último inteiro do intervalo (inclusivo).
- `W`: número de trabalhadores. Com `W = 1`, o programa executa sequencialmente e não requer os argumentos seguintes.
- `modo`: `processo` ou `thread` (obrigatório quando `W` é maior que 1).
- `particao`: `bloco` ou `ciclico` (obrigatório quando `W` é maior que 1).
- `arquivo_saida`: nome do arquivo CSV a ser criado dentro de `output/` (obrigatório quando `W` é maior que 1; informe apenas o nome, sem o diretório).

Exemplos:

```sh
# Execução sequencial no intervalo [1, 100000]
bin/varredor 1 100000 1

# Quatro threads, dividindo o intervalo em blocos
bin/varredor 1 100000 4 thread bloco thread_bloco.csv

# Quatro processos, distribuição cíclica
bin/varredor 1 100000 4 processo ciclico processo_ciclico.csv
```

Também é possível compilar e executar com o alvo `run` do Makefile, passando os argumentos pela variável `ARGS`:

```sh
make run ARGS="1 100000 4 thread bloco thread_bloco.csv"
```

## Como funciona

Para cada número `n` do intervalo, aplica-se repetidamente a regra de Collatz até chegar a 1:

- se `n` é par, o próximo valor é `n / 2`;
- se `n` é ímpar, o próximo valor é `3n + 1`.

O programa soma a quantidade de passos calculada para os números atribuídos a cada trabalhador. Na partição `bloco`, cada trabalhador recebe um intervalo contíguo; na partição `ciclico`, os valores são distribuídos alternadamente, com incremento igual a `W`. Na execução sequencial, `W` deve ser 1.

## Saídas

O programa apresenta no terminal o modo de execução, a partição, o tamanho do intervalo e os tempos medidos. Também grava uma linha CSV em `output/`. O nome é definido pelo argumento `arquivo_saida` nas execuções paralelas; na execução sequencial, é `sequencial.txt`.

As colunas do CSV, sem cabeçalho, são:

| Coluna | Significado |
| --- | --- |
| `modo` | `sequencial`, `processo` ou `thread` |
| `particao` | `bloco` ou `ciclico` (a execução sequencial registra `bloco`) |
| `W` | Número de processos ou threads (1 na execução sequencial) |
| `L` | Quantidade de inteiros no intervalo, `B - A + 1` |
| `tempo_total` | Tempo total da execução, em segundos |
| `tempo_max_filho` | Maior tempo individual entre os trabalhadores, em segundos |
| `tempo_min_filho` | Menor tempo individual entre os trabalhadores, em segundos |
| `tempo_agregacao` | Tempo medido para a etapa final de agregação, em segundos |

Na execução sequencial, os campos de tempo dos trabalhadores e de agregação são registrados como `-1`, pois não se aplicam. Na execução com processos, arquivos parciais de tempo são usados temporariamente em `temp/`.

## Organização do projeto

- `src/`: implementação em C.
- `include/`: cabeçalhos e estruturas compartilhadas.
- `bin/`: executável compilado.
- `build/`: arquivos objeto da compilação.
- `output/`: resultados CSV.
- `temp/`: dados temporários usados pela execução com processos.
- `Makefile`: regras de compilação, execução e limpeza.
