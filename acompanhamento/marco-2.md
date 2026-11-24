# Marco 2

# Justificando o Dijkstra
O problema apresentado pertence à classe dos problemas de **caminho mínimo**. A solução será baseada no algoritmo de **Dijkstra**, sendo o algoritmo ideal pelos seguintes motivos:

- pesos positivos: todas as arestas (estradas e trens) têm peso maior ou igual a 1, então Bellman-Ford não se faz necessário.
- origem única: o problema pede distâncias mínimas a partir de um único vértice, a capital (cidade 1), restringindo o problema ao cenário ideal do Dijkstra e descartando algoritmos de todos os pares (como o Floyd-Warshall).
- eficiência: com fila de prioridade (heap binário), a complexidade é O((V + E) * log V).

# Representando a instância pequena

Como o problema será modelado como um grafo ponderado, cada posição na lista de adjacência representa um vértice e armazena uma lista com as arestas incidentes a ele. Além disso, será necessário diferenciar as arestas que representam rotas de trem das que representam estradas. Com base em um dos exemplos do enunciado, segue um esboço da lista de adjacência.

```
0: 0-1(1,road)  0-2(3,road)  0-4(5,road)  0-2(5,train)  0-3(5,train)  0-4(5,train)
1: 1-0(1,road)  1-2(2,road)
2: 2-1(2,road)  2-0(3,road)  2-3(4,road)  2-0(5,train)
3: 3-2(4,road)  3-0(5,train)
4: 4-0(5,road)  4-0(5,train)
```