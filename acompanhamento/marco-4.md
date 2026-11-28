# Marco 4

## Solução
A solução foi implementada em Java e pode ser acessada pela pasta /src na raíz do repositório.

```
Main.java
├── Main                   (leitura e saída de dados/solução aplicada)
├── EdgeWeightedGraph      (grafo não direcionado ponderado como lista de adjacência)
├── Edge                   (aresta ponderada, com tipo "road" ou "train")
├── DijkstraUndirectedSP   (algoritmo de caminhos mínimos adaptado)
├── IndexMinPQ             (fila de prioridade indexada, usada pelo Dijkstra)
└── Bag                    (coleção usada nas listas de adjacência)
```

# Execução

## Opção 1: Compilar e depois executar
*Primeiro compila o código e gera o arquivo `Main.class`.*

```bash
javac ./src/Main.java
java -cp ./src Main < ./dados/casos-de-teste.txt
```

## Opção 2: Executar diretamente do código-fonte

*Compila em memória e executa imediatamente, sem gerar arquivos `.class`. Requer Java 11 ou superior.*

```bash
java ./src/Main.java < ./dados/casos-de-teste.txt
```

# Adaptações do *algs4*

A implementação teve como referência [algs4-java](https://github.com/carubbi/RPG/tree/main/algs4-java).

## Edge
A classe `Edge` recebe um tipo como atributo, podendo ser uma string "train" ou "road".

```java
class Edge implements Comparable<Edge> {

    private final int v;
    private final int w;
    private final double weight;
    private final String type; // tipo train/road
```

## DijkstraUndirectedSP
A classe `DijkstraUndirectedSP` implementa o algoritmo de Dijkstra, e seu método `relax` foi modificado para tratar o empate entre duas distâncias. Quando a distância até a cidade `w` pela aresta atual é igual à que já está registrada em `distTo[w]`, a prioridade de `w` na fila não muda, mas é preciso decidir qual aresta permanece em `edgeTo[w]`. Se a aresta registrada for um trem e a nova for uma estrada, a estrada a substitui, pois ela alcança a mesma distância mínima sem depender da rota de trem.

Com isso, o vetor `edgeTo[]` responde diretamente à pergunta central do problema. Como toda rota de trem liga a capital diretamente à cidade de destino, se uma rota consta em `edgeTo[]`, nenhuma estrada alcança aquela cidade pela mesma distância mínima, e a rota é indispensável.


```java
private void relax(Edge e, int v) {
        int w = e.other(v);
        if (distTo[w] > distTo[v] + e.weight()) {
            distTo[w] = distTo[v] + e.weight();
            edgeTo[w] = e;
            if (pq.contains(w)) pq.decreaseKey(w, distTo[w]);
            else                pq.insert(w, distTo[w]);
        }
        else if (distTo[w] == (distTo[v] + e.weight())
            && "road".equals(e.getType())
            && "train".equals(edgeTo[w].getType())) {
            edgeTo[w] = e;   // empate: estrada substitui trem em edgeTo[]
        }
    }
```