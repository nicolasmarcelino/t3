# Marco 3 - Rastreamento manual

O rastreamento a seguir usa o algoritmo de Dijkstra (`DijkstraSP` e `IndexMinPQ` do algs4) sobre o primeiro exemplo de entrada do problema. Como no Marco 2, os vértices são numerados de 0 a n-1, então a cidade `c` do enunciado corresponde ao vértice `c-1`. A capital é o vértice 0.

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

**Lista de adjacência** (`adj[]`): cada posição guarda as arestas incidentes ao vértice, com peso e tipo (estrada ou trem). Como o grafo é não direcionado, cada estrada aparece nos dois extremos, e cada rota de trem aparece na capital e na cidade de destino.

```
0: 0-1(1,road)  0-2(3,road)  0-4(5,road)  0-2(5,train)  0-3(5,train)  0-4(5,train)
1: 1-0(1,road)  1-2(2,road)
2: 2-1(2,road)  2-0(3,road)  2-3(4,road)  2-0(5,train)
3: 3-2(4,road)  3-0(5,train)
4: 4-0(5,road)  4-0(5,train)
```

**Estruturas auxiliares do algoritmo:**

| Estrutura | Função | Estado inicial |
|:---------:|:-------|:---------------|
| `distTo[v]` | menor distância conhecida da capital até `v` | `distTo[0] = 0` e `∞` nos demais |
| `edgeTo[v]` | última aresta do melhor caminho até `v` | vazio em todos |
| `trem[v]` | indica se `edgeTo[v]` é uma rota de trem (`v`) ou uma estrada (`f`) | `f` em todos |
| `pq` | fila de prioridade indexada (`IndexMinPQ`) com os vértices de distância provisória | `{0:0}` |

**Estado inicial:**

| v | distTo[] | edgeTo[v] | trem[v] |
|:-:|:--------:|:---------:|:-------:|
| 0 |    0     |           |    f    |
| 1 |    ∞     |           |    f    |
| 2 |    ∞     |           |    f    |
| 3 |    ∞     |           |    f    |
| 4 |    ∞     |           |    f    |

`pq = {0:0}`

**Regra de relaxamento da aresta `v-w` com peso `p`**, incluindo a decisão sobre os trens:

* se `distTo[v] + p < distTo[w]`: atualiza `distTo[w]` e `edgeTo[w]`, e define `trem[w]` conforme o tipo da aresta;
* se `distTo[v] + p == distTo[w]` e a aresta é uma **estrada** enquanto `trem[w] = v`: troca `edgeTo[w]` pela estrada e faz `trem[w] = f`, sem mudar a distância;
* nos demais casos, a aresta é ignorada.

No final, as rotas necessárias são os vértices com `trem[w] = v`, e a resposta é `k - (número de rotas necessárias)`.

## 2. Rastreamento das operações

### Passo 1: delMin() retorna 0 (distância 0)

Relaxamento das arestas de 0, na ordem da lista:

| Aresta | distTo[v] + p | Comparação | Ação |
|:------:|:-------------:|:----------:|:-----|
| 0-1 (1, road)  | 0+1 = 1 | 1 < ∞ | atualiza `distTo[1]`, insere 1 na `pq` |
| 0-2 (3, road)  | 0+3 = 3 | 3 < ∞ | atualiza `distTo[2]`, insere 2 na `pq` |
| 0-4 (5, road)  | 0+5 = 5 | 5 < ∞ | atualiza `distTo[4]`, insere 4 na `pq` |
| 0-2 (5, train) | 0+5 = 5 | 5 > 3 | ignorada (caminho pior) |
| 0-3 (5, train) | 0+5 = 5 | 5 < ∞ | atualiza `distTo[3]` com trem, insere 3 na `pq` |
| 0-4 (5, train) | 0+5 = 5 | 5 = 5 | empate, mas o atual já é estrada: mantém |

| v | distTo[] | edgeTo[v] | trem[v] |
|:-:|:--------:|:---------:|:-------:|
| 0 |    0     |           |    f    |
| 1 |    1     |    0-1    |    f    |
| 2 |    3     |    0-2    |    f    |
| 3 |    5     |    0-3    |    v    |
| 4 |    5     |    0-4    |    f    |

`pq = {1:1, 2:3, 3:5, 4:5}`

### Passo 2: delMin() retorna 1 (distância 1)

| Aresta | distTo[v] + p | Comparação | Ação |
|:------:|:-------------:|:----------:|:-----|
| 1-0 (1, road) | 1+1 = 2 | 2 > 0 | ignorada |
| 1-2 (2, road) | 1+2 = 3 | 3 = 3 | empate entre estradas: mantém `0-2` |

Nenhuma estrutura é alterada.

`pq = {2:3, 3:5, 4:5}`

### Passo 3: delMin() retorna 2 (distância 3)

| Aresta | distTo[v] + p | Comparação | Ação |
|:------:|:-------------:|:----------:|:-----|
| 2-1 (2, road)  | 3+2 = 5 | 5 > 1 | ignorada |
| 2-0 (3, road)  | 3+3 = 6 | 6 > 0 | ignorada |
| 2-3 (4, road)  | 3+4 = 7 | 7 > 5 | ignorada |
| 2-0 (5, train) | 3+5 = 8 | 8 > 0 | ignorada |

Nenhuma estrutura é alterada.

`pq = {3:5, 4:5}`

### Passo 4: delMin() retorna 3 (distância 5)

Os vértices 3 e 4 empatam com distância 5. Assumindo que a `IndexMinPQ` retorna o 3 primeiro, a ordem não altera o resultado, pois nenhum dos dois melhora algum vizinho.

| Aresta | distTo[v] + p | Comparação | Ação |
|:------:|:-------------:|:----------:|:-----|
| 3-2 (4, road)  | 5+4 = 9  | 9 > 3  | ignorada |
| 3-0 (5, train) | 5+5 = 10 | 10 > 0 | ignorada |

`pq = {4:5}`

### Passo 5: delMin() retorna 4 (distância 5)

| Aresta | distTo[v] + p | Comparação | Ação |
|:------:|:-------------:|:----------:|:-----|
| 4-0 (5, road)  | 5+5 = 10 | 10 > 0 | ignorada |
| 4-0 (5, train) | 5+5 = 10 | 10 > 0 | ignorada |

`pq = {}`, e o algoritmo termina.

### Estado final

| v | distTo[] | edgeTo[v] | trem[v] |
|:-:|:--------:|:---------:|:-------:|
| 0 |    0     |           |    f    |
| 1 |    1     |    0-1    |    f    |
| 2 |    3     |    0-2    |    f    |
| 3 |    5     |    0-3    |    v    |
| 4 |    5     |    0-4    |    f    |

## 3. Justificativa das decisões e do resultado

**Por que a ordem de extração da fila está correta.** A `pq` sempre devolve o vértice de menor distância provisória (0, 1, 2, 3, 4). Como todos os pesos são positivos, quando um vértice sai da fila sua distância já é definitiva, e nenhum caminho passando por vértices ainda não processados poderia ser menor.

**Decisões sobre cada rota de trem** (as cidades abaixo seguem a numeração do enunciado):

| Rota (cidade, comprimento) | Vértice | Menor distância | Decisão | Motivo |
|:--------------------------:|:-------:|:---------------:|:-------:|:-------|
| (3, 5) | 2 | 3 | fecha | o trem tem 5 e a estrada `0-2` chega em 3, então ele é estritamente pior |
| (4, 5) | 3 | 5 | mantém | é o único caminho com custo 5 até o vértice 3, e fechá-lo aumentaria a distância para 7 (`0-2-3`) |
| (5, 5) | 4 | 5 | fecha | empata com a estrada `0-4` (5), que já garante a distância mínima |

**Resultado.** Apenas `trem[3] = v` ao final, então 1 rota é necessária. O número máximo de rotas que podem ser fechadas é `k - necessárias = 3 - 1 = 2`, o que coincide com a saída esperada do exemplo.

**Por que a decisão não aumenta a complexidade.** O teste de empate e a marcação `trem[]` acontecem dentro do relaxamento, em tempo O(1) por aresta. A complexidade permanece O((V + E) log V), como no Dijkstra tradicional.
