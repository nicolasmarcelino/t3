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
| 0-4 (5, train) | 0+5 = 5 | 5 > 5 falso | ignorada (empate) |

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
| 1-2 (2, road) | 1+2 = 3 | 3 > 3 falso | ignorada (empate) |

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

**Decisão sobre os trens.** Depois do Dijkstra, cada rota de trem `(s, y)` é comparada com `distTo[s-1]`:

* se `y > distTo[s-1]`, o trem é estritamente pior, então pode ser fechado;
* se `y == distTo[s-1]`, o trem só pode ser fechado se existir uma **estrada** `u-s` com `distTo[u] + x == distTo[s-1]`, isto é, outro caminho de mesmo custo que não dependa do trem. Entre vários trens empatados para a mesma cidade, mantém-se apenas um.

| Rota (cidade, comprimento) | Vértice | distTo[] | Comparação | Decisão | Motivo |
|:--------------------------:|:-------:|:--------:|:----------:|:-------:|:-------|
| (3, 5) | 2 | 3 | 5 > 3 | fecha | a estrada `0-2` já chega em 3, e o trem é pior |
| (4, 5) | 3 | 5 | 5 = 5 | mantém | a única estrada que chega a 3 é `2-3`, e `distTo[2] + 4 = 7 > 5`, então só o trem garante a distância 5 |
| (5, 5) | 4 | 5 | 5 = 5 | fecha | a estrada `0-4` (5) empata com o trem e já garante a distância mínima |

**Resultado.** Apenas 1 rota é necessária (a da cidade 4). O número máximo de rotas que podem ser fechadas é `k - necessárias = 3 - 1 = 2`, o que coincide com a saída esperada do exemplo.

**Complexidade.** O Dijkstra continua com custo O((V + E) log V), e a verificação dos trens percorre as `k` rotas e as arestas incidentes uma vez, com custo O(V + E + k), sem alterar a ordem de complexidade total.
