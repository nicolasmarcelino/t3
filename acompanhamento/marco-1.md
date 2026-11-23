# Problema: Jzzhu and Cities
Cenário: um país com cidades ligadas por estradas e rotas de trem.

# Descrição
O país possui _n_ cidades, enumeradas de 1 a _n_.
A cidade 1 é a capital.
Existem _m_ estradas, que conectam duas cidades e podem ser percorridas nos dois sentidos, cada uma com um comprimento.
Existem _k_ rotas de trem, que também são bidirecionais, mas sempre ligam a capital a uma outra cidade, cada uma com um comprimento.
Pode haver várias estradas entre o mesmo par de cidades e várias rotas de trem para a mesma cidade.
É garantido que existe pelo menos um caminho de cada cidade até a capital.
O presidente quer economizar fechando algumas rotas de trem, mas a menor distância de cada cidade até a capital não pode mudar.
Objetivo: descobrir o maior número de rotas de trem que podem ser fechadas ao mesmo tempo respeitando essa condição.

# Modelagem
Podemos considerar o país como um **grafo não direcionado e ponderado**, em que as cidades são vértices e as estradas e rotas de trem são arestas com pesos positivos. As rotas de trem são um tipo especial de aresta, pois sempre partem do vértice 1 e são as únicas que podem ser removidas. O objetivo é contar quantas dessas arestas são dispensáveis, isto é, quantas podem ser removidas simultaneamente sem alterar as distâncias mínimas a partir da capital.

**Solução proposta**: executar o algoritmo de Dijkstra a partir da capital, tratando estradas e trens como arestas comuns, para obter a distância mínima até cada cidade...

# Entradas e saídas

**Significado das entradas**: quantas cidades, estradas e rotas de trem existem, e quais são seus extremos e comprimentos.

**Significado da saída**: um único inteiro, o número máximo de rotas de trem que podem ser fechadas.

A primeira linha contém três inteiros _n_, _m_ e _k_. As próximas _m_ linhas descrevem as estradas, cada uma com três inteiros _u_, _v_ e _x_, indicando uma estrada entre as cidades _u_ e _v_ de comprimento _x_. As últimas _k_ linhas descrevem as rotas de trem, cada uma com dois inteiros _s_ e _y_, indicando uma rota entre a capital e a cidade _s_ de comprimento _y_.

## Exemplo de entrada
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

## Exemplo de saída
```
2
```

## Segundo exemplo de entrada
```
2 2 3
1 2 2
2 1 3
2 1
2 2
2 3
```

## Segundo exemplo de saída
```
2
```

Fonte: https://codeforces.com/problemset/problem/449/B