# **Progetto Easy Maze**

Link alla consegna del progetto: [PDF](EasyMaze.pdf)

## Informazioni generali:
### Legenda:
- S (Start): posizione iniziale dell’agente
- E (Exit) : uscita
- X : muro invalicabile
- 1/4 : cella calpestabile e costo per attraversarla

### Dominio e vincoli:
L'agente all'interno del labirinto è soggetto ai seguenti **vincoli**:
- v1 = "può compiere solo un passo alla volta";
- v2 = "può muoversi solo tra caselle adiacenti";
- v3 = "può muoversi solo Left/Right/Up/Down";
- v4 = "non è ammesso lo spostamento in diagonale";
- v5 = "non è permesso transitare dentro le caselle con la X all'interno".

Invece il labirinto deve rispettare le seguenti **caratteristiche**:  
- v6 = "la matrice può anche non essere quadrata, $n ≠ m$";
- v7 = "deve avere 1 ingresso S per la posizione iniziale dell'agente";
- v8 = "può avere 1 o più uscite E";
- v9 = "le celle calpestabili sono solo le celli con i numeri all'interno";
- v10 = "gli unici numeri ammessi vanno da 1 a 4 compresi".

Il costo di una azione è composto dallo spostamento da una cella ad un'altra ed è uguale al percorso di spostamento tra caselle in base al peso assegnato w(n):

$Path Cost = g(n) = g(p)+w(n)$

Le celle transitabili hanno peso che va da 1 a 4:

$1 <= w(n) <= 4$

Le celle *S* e *E* hanno peso rispettivo uguale a 0 e 1, tutti i muri hanno peso 0 ma sono invalicabili.

eg:

```
[S 1 1 X
 4 2 2 3
 3 1 3 2
 2 X 2 E]
```
$PathCost(S -> E)= w([0,0]) + w([0,1]) + w([1,1]) + w([1,2]) + w([2,2]) + w([2,3]) + w([3,3])=0 + 1 + 2 + 2 + 3 + 2 + 1 = 11$

# **Rappresentazione dei dati e metodi**

## Classi
Le principali classi utilizzate sono **EasyMazeClass(Problem)** e **Node()**:
## **EasyMazeClass(Problem)**
Deriva la classe Problem, definita da AIMA dentro **search.py**, ridefinendo alcuni metodi per la gestione dei dati. Verrà creato un oggetto problem, con al suo interno tutto il labirinto, che è formato da un array di celle, che sono oggetti di un'altra classe.
### Attributi
- *problem.start* = rappresenta lo stato iniziale S;
- *problem.goal* = rappresenta lo stato finale E.

### Metodi:
- *problem.actions(problem.cell[n])* = riporta tutte le direzioni in cui si può muovere l'agente in quella cella;
- *problem.result(problem.cell[n],action(problem.cell[n]))* = ritorna tutte le celle e le rispettive posizioni in cui la cella passata può spostarsi tramite l'actions;
- *problem.goal_test(problem.cell[n])* = ritorna True o False, se la cella passata corrisponde al nodo goal oppure no.

## **Node()**
Rappresenta una cella all'interno del labirinto, con tutte le caratteristiche che essa ha, come peso, lettera/numero presente.
Viene trattata anche come un nodo di un albero, dato che al suo interno salveremo dei dati utili come g, h ed f.
### Attributi:
- *node.letter* = rappresenta la lettera o numero in formato string;
- *node.weight* = rappresenta il peso della cella in base alla lettera;
- *node.position* = rappresenta le coordinate all'interno del labirinto;
- *node.parent* = indica il nodo parent al momento della ricerca del cammino;
- *node.g* = indica il peso del cammino dallo start a node;
- *node.h* = indica il peso della funzione euristica da node a goal;
- *node.f* = indica la somma di g + h.

### Metodi:
- *node.path()* = ritorno il path dal nodo iniziale a node, in un array di coordinate;
- *node.set_weight_letter()* = imposta il peso e la lettera del nodo andando a controllare la posizione sul labirinto.

# **Funzione Euristica**
La funzione euristica che andremo ad utilizzare sarà l'euristica della **distanza di Manhattan** secondo il quale la distanza tra due punti è la somma del valore assoluto delle differenze delle loro coordinate.

## La distanza di Manhattan è un'euristica ottimale:

### **Ammissibilità:**
definisce un'euristica che non sovrastima mai la distanza tra un nodo e il nodo di destinazione. Ciò significa che l'euristica di distanza di Manhattan non può mai riportare che la distanza tra un nodo e il nodo di destinazione è maggiore di quanto non sia in realtà.

Vale la seguente proprietà: $h(n) ≤ c(n, E) ∀n$

eg:

```
[S 1 1 X
 4 2 2 3
 3 1 3 2
 2 X 2 E]
```
$h(n)$ = 6

$c(n,E)$ = 11

Anche se ci fosse 1 singola strada

```
[S 1 X X
 X 1 X X
 X 1 1 X
 X X 1 E]
```
$h(n)$ = 6

$c(n,E)$ = 6


### **Consistenza:**
definisce una funzione di euristica tale che la differenza tra l'euristica per un nodo e la distanza effettiva tra il nodo e il nodo di destinazione non aumenta mai quando il nodo viene espanso.

Vale la seguente proprietà: $h(n) ≤ c(n,n') + h(n')$

## Definizione:
Possiamo definire la distanza $L$ tra 2 punti $P1$ di coordinate $(x1,y1)$ e un punto $P2$ di cordinate $(x2,y2)$ nel piano in questo modo:

$L(P1,P2) = |x2-x1| + |y2-y1|$

### Caso d'uso:
Nel nostro specifico caso questa euristica è perfetta dato che gli unici movimenti che possiamo effettuare sono orizzontali e verticali, non è permesso lo spostamento in diagonale.


```
def manhattan_distance(start, goal):

  # Calculate the horizontal and vertical distances between the two nodes.
  # Return the sum of the two distances.
  return abs(start[0] - goal[0]) + abs(start[1] - goal[1])

```
# Algoritmo A*
L'algoritmo A* è un tipo di algoritmo di ricerca informata ed è utilizzato per trovare il percorso più breve tra due nodi in un grafo.

Lo utilizzeremo per trovare il cammino più breve per raggiungere l'uscita E dalla posizione di partenza S, se esiste.

A* utilizza la funzione $f(n)$ per selezionare il nodo con il costo stimato più basso non ancora visitato

$f(n) = g(n)+h(n)$

dove:
- $g(n)$ è il costo effettivo per raggiungere il nodo n.
- $h(n)$ è la funzione di approssimazione euristica, nel nostro caso Manhattan
