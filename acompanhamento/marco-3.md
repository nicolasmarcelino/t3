# Marco 3 - Rastreamento manual

O rastreamento usa a classe `DijkstraUndirectedSP` do algs4, com as estruturas `distTo`, `edgeTo` e `pq`, sobre o primeiro exemplo de entrada. Como no Marco 2, a cidade `c` do enunciado é o vértice `c-1`, e a capital é o vértice 0.

```
5 5 3
1 2 1
2 3 2
1 3 3
3 4 4
1 5 5
3 5
4 5
5 5
```

## 1. Estruturas de dados e estados iniciais

```java
private double[] distTo;          // distTo[v] = distância do menor caminho s->v
private Edge[] edgeTo;            // edgeTo[v] = última aresta do menor caminho s->v
private IndexMinPQ<Double> pq;
```

**Lista de adjacência** (grafo não direcionado: cada aresta aparece nos dois extremos; o tipo da aresta vem do Marco 2):

```
0: 0-1(1,road)  0-2(3,road)  0-4(5,road)  0-2(5,train)  0-3(5,train)  0-4(5,train)
1: 1-0(1,road)  1-2(2,road)
2: 2-1(2,road)  2-0(3,road)  2-3(4,road)  2-0(5,train)
3: 3-2(4,road)  3-0(5,train)
4: 4-0(5,road)  4-0(5,train)
```

**Estruturas e seus estados iniciais:**

| Estrutura | Função | Estado inicial |
|:---------:|:-------|:---------------|
| `distTo[v]` | menor distância conhecida de 0 até `v` | `distTo[0] = 0.0` e `∞` nos demais |
| `edgeTo[v]` | última `Edge` do melhor caminho até `v` | `null` em todos |
| `pq` | `IndexMinPQ<Double>` com os vértices e suas distâncias provisórias | `{0:0.0}` |

| v | distTo[] | edgeTo[v] |
|:-:|:--------:|:---------:|
| 0 |    0     |   null    |
| 1 |    ∞     |   null    |
| 2 |    ∞     |   null    |
| 3 |    ∞     |   null    |
| 4 |    ∞     |   null    |

`pq = {0:0}`

**Relaxamento da aresta `e` a partir de `v`:** seja `w = e.other(v)`. Se `distTo[w] > distTo[v] + e.weight()`, então `distTo[w]` e `edgeTo[w]` são atualizados, e `w` é inserido na `pq` (`insert`) ou tem a chave reduzida (`decreaseKey`) caso já esteja nela. Caso contrário, a aresta é ignorada. A comparação é estrita, então empates não alteram nada.

## 2. Rastreamento das operações

### Passo 1: `delMin()` retorna 0 (distância 0)

| Aresta | distTo[v] + p | Teste `distTo[w] >` | Ação |
|:------:|:-------------:|:-------------------:|:-----|
| 0-1 (1, road)  | 0+1 = 1 | ∞ > 1 verdadeiro | atualiza `distTo[1]` e `edgeTo[1]`, `insert(1, 1)` |
| 0-2 (3, road)  | 0+3 = 3 | ∞ > 3 verdadeiro | atualiza `distTo[2]` e `edgeTo[2]`, `insert(2, 3)` |
| 0-4 (5, road)  | 0+5 = 5 | ∞ > 5 verdadeiro | atualiza `distTo[4]` e `edgeTo[4]`, `insert(4, 5)` |
| 0-2 (5, train) | 0+5 = 5 | 3 > 5 falso | ignorada |
| 0-3 (5, train) | 0+5 = 5 | ∞ > 5 verdadeiro | atualiza `distTo[3]` e `edgeTo[3]`, `insert(3, 5)` |
| 0-4 (5, train) | 0+5 = 5 | 5 > 5 falso | **empate**: a nova aresta dá a mesma distância que a registrada. Como `distTo[4]` não muda, a fila não precisa ser atualizada. Resta decidir qual aresta fica registrada em `edgeTo[4]` (ver observação abaixo). |

**Observação sobre o empate.** Quando duas arestas dão a mesma menor distância até um vértice, a escolha de qual delas fica em `edgeTo` **não pode depender da ordem** em que as arestas são percorridas. Se dependesse, o mesmo grafo poderia dar respostas diferentes. Aqui, se o trem 0-4 fosse processado antes da estrada 0-4, ele ficaria registrado primeiro, e a estrada chegaria depois com o mesmo valor. Por isso é necessária uma **regra de desempate** que dê o mesmo resultado nas duas ordens: no empate, a estrada tem preferência sobre o trem. Assim, `edgeTo[4]` termina como `road` de qualquer forma, e o trem 0-4 é dispensável. A fila não é afetada, pois ela só enxerga `distTo[w]`.

| v | distTo[] | edgeTo[v] |
|:-:|:--------:|:---------:|
| 0 |    0     |   null    |
| 1 |    1     | 0-1 (road)  |
| 2 |    3     | 0-2 (road)  |
| 3 |    5     | 0-3 (train) |
| 4 |    5     | 0-4 (road)  |

`pq = {1:1, 2:3, 3:5, 4:5}`

### Passo 2: `delMin()` retorna 1 (distância 1)

| Aresta | distTo[v] + p | Teste `distTo[w] >` | Ação |
|:------:|:-------------:|:-------------------:|:-----|
| 1-0 (1, road) | 1+1 = 2 | 0 > 2 falso | ignorada |
| 1-2 (2, road) | 1+2 = 3 | 3 > 3 falso | ignorada (empate): `distTo[2]` já vale 3, então a fila não muda. O `edgeTo[2]` já registra uma estrada (`0-2`), então não há trem a substituir. |

**Observação sobre o empate.** Este é um empate entre duas estradas, diferente do passo 1, em que o empate era entre trem e estrada. Como a regra de desempate só precisa garantir que o trem não permaneça registrado quando existe uma estrada que o cubra, aqui a aresta que já estava registrada permanece.

Nenhuma estrutura é alterada.

`pq = {2:3, 3:5, 4:5}`

### Passo 3: `delMin()` retorna 2 (distância 3)

| Aresta | distTo[v] + p | Teste `distTo[w] >` | Ação |
|:------:|:-------------:|:-------------------:|:-----|
| 2-1 (2, road)  | 3+2 = 5 | 1 > 5 falso | ignorada |
| 2-0 (3, road)  | 3+3 = 6 | 0 > 6 falso | ignorada |
| 2-3 (4, road)  | 3+4 = 7 | 5 > 7 falso | ignorada |
| 2-0 (5, train) | 3+5 = 8 | 0 > 8 falso | ignorada |

Nenhuma estrutura é alterada.

`pq = {3:5, 4:5}`

### Passo 4: `delMin()` retorna 3 (distância 5)

Os vértices 3 e 4 empatam em 5. Assumindo que o 3 sai primeiro, a ordem não altera o resultado, pois nenhum dos dois melhora algum vizinho.

| Aresta | distTo[v] + p | Teste `distTo[w] >` | Ação |
|:------:|:-------------:|:-------------------:|:-----|
| 3-2 (4, road)  | 5+4 = 9  | 3 > 9 falso  | ignorada |
| 3-0 (5, train) | 5+5 = 10 | 0 > 10 falso | ignorada |

`pq = {4:5}`

### Passo 5: `delMin()` retorna 4 (distância 5)

| Aresta | distTo[v] + p | Teste `distTo[w] >` | Ação |
|:------:|:-------------:|:-------------------:|:-----|
| 4-0 (5, road)  | 5+5 = 10 | 0 > 10 falso | ignorada |
| 4-0 (5, train) | 5+5 = 10 | 0 > 10 falso | ignorada |

`pq = {}`, e o Dijkstra termina. Nenhuma chamada a `decreaseKey` ocorreu: todas as atualizações foram `insert`.

### Estado final

| v | distTo[] | edgeTo[v] |
|:-:|:--------:|:---------:|
| 0 |    0     |   null    |
| 1 |    1     | 0-1 (road)  |
| 2 |    3     | 0-2 (road)  |
| 3 |    5     | 0-3 (train) |
| 4 |    5     | 0-4 (road)  |

## 3. Justificativa das decisões e do resultado

**Ordem de extração.** A `pq` sempre devolve o vértice de menor distância provisória (0, 1, 2, 3, 4). Como todos os pesos são positivos, a distância de um vértice que sai da fila já é definitiva.

**Decisão sobre os trens.** Durante o relaxamento, o Dijkstra registra em `edgeTo[w]` a aresta que define a menor distância até `w`. Em caso de empate, a estrada substitui o trem, sem alterar a fila de prioridade, pois `distTo[w]` não muda. Ao final, cada cidade cujo `edgeTo` é um trem depende dele para manter sua distância mínima, e esses são os trens necessários. A resposta é `k - necessárias`, sendo `k` o total de rotas de trem.

**Complexidade.** O Dijkstra continua com custo O((V + E) * log V). A troca no empate custa O(1) por relaxamento, e a contagem final percorre os vértices uma vez, com custo O(V), sem alterar a ordem de complexidade total.