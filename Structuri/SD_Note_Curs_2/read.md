# Pregătire admitere — Algoritmi și Structuri de Date

---

## I. Analiza asimptotică a performanței

Măsoară cum crește timpul de execuție / memoria folosită de un algoritm în funcție de dimensiunea `n` a intrării, ignorând constantele și detaliile de implementare (limbaj, mașină).

**O (big-O) — margine superioară**
`f(n) = O(g(n))` dacă există constante `c > 0` și `n0` astfel încât `f(n) ≤ c·g(n)` pentru orice `n ≥ n0`.
→ descrie **cel mai defavorabil caz** (worst-case) sau, general, o limită *de sus* a creșterii.

**Ω (big-Omega) — margine inferioară**
`f(n) = Ω(g(n))` dacă există `c > 0`, `n0` astfel încât `f(n) ≥ c·g(n)` pentru `n ≥ n0`.
→ descrie **cel mai bun caz** (best-case) sau o limită *de jos*.

**Θ (big-Theta) — margine exactă (strânsă)**
`f(n) = Θ(g(n))` dacă `f(n) = O(g(n))` **și** `f(n) = Ω(g(n))` simultan.
→ algoritmul crește *exact* în ritmul lui `g(n)`, atât ca limită de sus cât și de jos.

**Ordine uzuale (crescător):**
`O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!)`

**De reținut la interviu:** O/Ω/Θ pot fi aplicate oricărui caz (best/average/worst) — greșeala comună e să spui „O = worst-case, Ω = best-case" ca definiție; de fapt O/Ω sunt margini matematice, iar best/worst-case sunt scenarii de intrare. Cel mai des în practică: O = worst-case, Ω = best-case, Θ = când ambele coincid.

---

## II. Structuri lineare

### Alocare statică vs. dinamică
- **Vector (array)** — alocare statică/contiguă, acces direct O(1) prin index, dimensiune fixă (sau realocare costisitoare), inserare/ștergere costisitoare O(n) (necesită deplasarea elementelor).
- **Listă înlănțuită** — alocare dinamică, elementele (noduri) nu sunt contigue în memorie, fiecare nod conține **date** + **pointer(i)** către nodul/nodurile vecine. Acces secvențial O(n), dar inserare/ștergere O(1) *dacă ai deja poziția*.

### Listă simplu înlănțuită
- Nod = `{data, next}`
- `head` = primul nod, `tail` = ultimul (`next = NULL`)
- **Traversare**: pornești de la `head`, avansezi prin `next` până la `NULL` — O(n)
- **Căutare**: traversare + comparare cheie — O(n)
- **Inserare**:
  - la început: O(1)
  - la sfârșit: O(1) dacă păstrezi pointer `tail`, altfel O(n)
  - la mijloc (după un nod dat): O(1) — dar găsirea poziției costă O(n)
- **Ștergere**: similar inserării — necesită pointer către nodul *anterior* (pentru a-i rescrie `next`)

### Listă dublu înlănțuită
- Nod = `{prev, data, next}`
- Avantaj: parcurgere în ambele direcții, ștergerea unui nod dat e O(1) fără a mai căuta predecesorul
- Dezavantaj: memorie suplimentară per nod (încă un pointer)

### Tipuri particulare de liste
- **Listă circulară**: ultimul nod (`tail.next`) indică spre `head` în loc de `NULL`. Utilă la buffere circulare, scheduling round-robin.
- **Listă circulară dublă**: combină ambele proprietăți.
- **Listă cu sentinel/gardă**: nod fictiv la început, simplifică tratarea cazurilor de margine.

### Structuri lineare cu restricții I/O

**Stivă (Stack)** — LIFO (Last In, First Out)
- Operații: `push` (adaugă în vârf), `pop` (elimină din vârf), `peek/top` — toate O(1)
- Implementare: vector sau listă înlănțuită
- **Aplicații**: evaluarea expresiilor aritmetice (infix→postfix, evaluare postfix), verificarea parantezelor echilibrate, apeluri de funcții/recursivitate (stiva de execuție), algoritmul DFS (varianta iterativă), undo/redo

**Coadă (Queue)** — FIFO (First In, First Out)
- Operații: `enqueue` (adaugă la coadă), `dequeue` (elimină din față) — O(1)
- **Aplicații**: BFS, scheduling de procese (CPU, print jobs), buffere de date (streaming)
- **Coadă circulară**: reutilizează spațiul unui vector cu doi indici (`front`, `rear`) mod `n`
- **Deque (double-ended queue)**: inserare/ștergere la ambele capete
- **Coadă cu priorități**: fiecare element are o prioritate; se extrage mereu elementul cu prioritate maximă/minimă — implementată de obicei cu un **heap** (vezi HeapSort)

| Structură / Operație | Best case | Average case | Worst case |
|---|---|---|---|
| Vector — acces prin index | O(1) | O(1) | O(1) |
| Vector — căutare | O(1) | O(n) | O(n) |
| Vector — inserare/ștergere (mijloc) | O(1)* | O(n) | O(n) |
| Listă simplu înlănțuită — traversare/căutare | O(1) | O(n) | O(n) |
| Listă înlănțuită — inserare la început | O(1) | O(1) | O(1) |
| Listă înlănțuită — inserare la sfârșit (cu `tail`) | O(1) | O(1) | O(1) |
| Listă înlănțuită — inserare la sfârșit (fără `tail`) | O(n) | O(n) | O(n) |
| Listă înlănțuită — ștergere (poziție cunoscută) | O(1) | O(1) | O(1) |
| Listă dublu înlănțuită — ștergere (nod dat) | O(1) | O(1) | O(1) |
| Stivă — push/pop/peek | O(1) | O(1) | O(1) |
| Coadă — enqueue/dequeue | O(1) | O(1) | O(1) |

\* inserare/ștergere la capătul unde nu e nevoie de deplasare

---

**De reținut**: un ABC "obișnuit" poate degenera într-o listă (inserare de chei deja sortate → `h = n`), pe când AVL garantează `h = O(log n)` prin rotații, indiferent de ordinea de inserare. Cea mai comună întrebare-capcană la interviu: "de ce nu folosim mereu ABC simplu?" → răspuns: worst-case-ul devine liniar fără echilibrare.

---

## III. Grafuri

### Definiții
- **Graf** `G = (V, E)` — mulțime finită de noduri (vârfuri) `V` și mulțime de muchii `E` care leagă perechi de noduri.
- **Graf neorientat**: muchia `(u,v)` = `(v,u)` (fără sens).
- **Graf orientat (digraf)**: muchia `(u,v)` are sens — `u` = sursă, `v` = destinație.
- **Graf ponderat**: fiecare muchie are un cost/greutate asociată.
- **Grad al unui nod**: numărul de muchii incidente (la digraf: grad de intrare `in-degree` și grad de ieșire `out-degree`).
- **Drum (path)**: succesiune de noduri legate prin muchii, fără repetarea nodurilor.
- **Ciclu**: drum care se întoarce la nodul de start.
- **Buclă (loop/self-loop)**: muchie de la un nod la el însuși.
- **Graf conex**: există drum între orice pereche de noduri.
- **Graf conex tare** (la digraf): există drum în ambele sensuri între orice pereche.
- **Subgraf**: submulțime de noduri și muchii ale unui graf.

### Reprezentare
| Reprezentare | Spațiu | Verificare muchie `(u,v)` | Enumerare vecini |
|---|---|---|---|
| Matrice de adiacență (`n×n`) | O(n²) | O(1) | O(n) |
| Listă de adiacență | O(n+m) | O(grad) | O(grad) |
| Matrice de incidență (`n×m`) | O(n·m) | — | — |

Listele de adiacență sunt preferate la grafuri rare (`m << n²`); matricea la grafuri dense sau când ai nevoie de acces rapid la existența unei muchii.

### Parcurgeri
**DFS (Depth-First Search)** — mergi cât de „adânc" posibil pe o ramură înainte de a face backtrack. Se implementează recursiv sau iterativ cu o **stivă**. Complexitate: O(V + E).

**BFS (Breadth-First Search)** — parcurge graful „în lățime", nivel cu nivel, folosind o **coadă**. Găsește cel mai scurt drum (în număr de muchii) în grafuri neponderate. Complexitate: O(V + E).

**Aplicații**: detectarea conexității, detectarea ciclurilor, sortare topologică (DFS pe digraf aciclic), componente conexe, cel mai scurt drum neponderat (BFS).

| Operație | Complexitate |
|---|---|
| DFS | O(V + E) — aceeași în toate cazurile |
| BFS | O(V + E) — aceeași în toate cazurile |
| Verificare muchie (matrice adiacență) | O(1) |
| Verificare muchie (listă adiacență) | O(grad) |

---

## IV. Structuri arborescente

### Arbori oarecare
- Structură ierarhică, aciclică, conexă, cu un singur nod **rădăcină**; fiecare alt nod are exact un **părinte** și 0..k **copii**.
- **Reprezentare**: „copil–frate" (fiecare nod are pointer către primul copil și către următorul frate) — transformă orice arbore în echivalentul unui arbore binar.
- **Parcurgere**: preordine (rădăcină apoi subarbori, stânga la dreapta) sau pe nivel (BFS).

### Arbori binari
- Fiecare nod are cel mult **2 copii** (stâng, drept).
- **Reprezentare**: înlănțuită (`{data, left, right}`) sau prin vector (util la heap: copiii nodului `i` sunt `2i+1` și `2i+2`).
- **Parcurgeri**:
  - **Preordine (VSD)**: rădăcină → stânga → dreapta
  - **Inordine (SVD)**: stânga → rădăcină → dreapta (la ABC dă elementele **sortate**)
  - **Postordine (SDV)**: stânga → dreapta → rădăcină
  - **Pe nivel (BFS)**: folosind o coadă

### Arbori binari de căutare (ABC / BST)
**Proprietate**: pentru orice nod, toate cheile din subarborele stâng < cheia nodului < toate cheile din subarborele drept.

- **Căutare**: compari cheia cu nodul curent, mergi stânga/dreapta — O(h), unde `h` = înălțimea arborelui.
- **Inserare**: cauți poziția (ca la căutare) până ajungi la `NULL`, atașezi noul nod acolo — O(h).
- **Ștergere nod** — 3 cazuri:
  1. Nod frunză → se elimină direct.
  2. Nod cu un singur copil → copilul îi ia locul.
  3. Nod cu doi copii → se înlocuiește cu **succesorul in-order** (cel mai mic din subarborele drept) sau **predecesorul in-order** (cel mai mare din subarborele stâng), apoi se șterge succesorul/predecesorul (care are cel mult un copil).

**Performanță**: O(h) pentru toate operațiile. În cel mai rău caz (arbore degenerat, ca o listă) `h = O(n)`. Într-un arbore echilibrat `h = O(log n)`.

### Căutarea binară (în vector sortat)
- Compari elementul din mijloc, elimini jumătate din spațiul de căutare la fiecare pas.
- Complexitate: **O(log n)** — echivalentă cu adâncimea unui ABC perfect echilibrat construit din același vector.

### Arbori AVL (Adelson-Velsky/Landis)
- ABC în care, pentru fiecare nod, **factorul de echilibru** = `înălțime(subarbore stâng) − înălțime(subarbore drept) ∈ {−1, 0, 1}`.
- **Inserare**: se inserează normal ca în ABC, apoi se actualizează înălțimile pe drumul spre rădăcină; dacă un nod devine dezechilibrat (factor ±2), se aplică **rotații**:
  - **Rotație simplă stânga** / **dreapta** (caz stânga-stânga / dreapta-dreapta)
  - **Rotație dublă** (stânga-dreapta / dreapta-stânga) — o rotație pe copil urmată de una pe nod
- **Performanță**: înălțimea unui AVL cu `n` noduri este garantat **O(log n)** (mai precis ≈ `1.44 log₂ n`), deci căutare/inserare/ștergere sunt O(log n) *în cel mai rău caz* — spre deosebire de un ABC neechilibrat.


| Structură / Operație | Best case | Average case | Worst case |
|---|---|---|---|
| Căutare binară (vector sortat) | O(1) | O(log n) | O(log n) |
| ABC — căutare | O(1) | O(log n) | O(n) (arbore degenerat) |
| ABC — inserare | O(1) | O(log n) | O(n) |
| ABC — ștergere | O(1) | O(log n) | O(n) |
| AVL — căutare | O(1) | O(log n) | O(log n) (garantat) |
| AVL — inserare | O(1) | O(log n) | O(log n) |
| AVL — ștergere | O(1) | O(log n) | O(log n) |
| Rotație AVL | O(1) | O(1) | O(1) |

---

## V. Algoritmi de sortare (mulțimi statice — vectori)

**HeapSort**
- Se construiește un **max-heap** (arbore binar aproape complet, unde părintele ≥ copii, reprezentat în vector) din tot vectorul — O(n).
- Se extrage repetat maximul (rădăcina), se pune la finalul zonei sortate, se „coboară" (`heapify`) noul element din vârf — O(log n) per extragere, de `n` ori.
- **Complexitate**: O(n log n) în toate cazurile (best/avg/worst). In-place, dar **nu e stabil**.

**QuickSort**
- Se alege un **pivot**, se partiționează vectorul astfel încât elementele mai mici să fie înaintea lui, cele mai mari după, apoi se sortează recursiv (Divide et Impera) cele două părți.
- **Complexitate**: medie O(n log n); worst-case **O(n²)** (când pivotul e mereu extrem — ex. vector deja sortat cu pivot = primul element). In-place, nu e stabil, dar are constante mici → foarte rapid în practică.

**MergeSort**
- Se împarte vectorul recursiv în jumătăți (Divide et Impera) până la elemente unice, apoi se **interclasează** (merge) subvectorii sortați.
- **Complexitate**: O(n log n) garantat în **toate** cazurile. Necesită O(n) memorie suplimentară. Este **stabil**.

| Algoritm | Best | Average | Worst | Spațiu | Stabil |
|---|---|---|---|---|---|
| HeapSort | O(n log n) | O(n log n) | O(n log n) | O(1) | Nu |
| QuickSort | O(n log n) | O(n log n) | O(n²) | O(log n) | Nu |
| MergeSort | O(n log n) | O(n log n) | O(n log n) | O(n) | Da |

### Limita inferioară a sortării bazate pe comparații
Orice algoritm de sortare care decide ordinea **doar prin comparații** între chei poate fi modelat printr-un **arbore de decizie** binar: fiecare nod intern = o comparație, fiecare frunză = o permutare posibilă a rezultatului.
- Există `n!` permutări posibile → arborele are cel puțin `n!` frunze.
- Un arbore binar cu `n!` frunze are înălțime minimă `⌈log₂(n!)⌉`.
- Prin aproximarea lui Stirling, `log₂(n!) = Θ(n log n)`.
- **Concluzie**: orice algoritm de sortare prin comparații are complexitate **Ω(n log n)** în cazul cel mai defavorabil — deci HeapSort/MergeSort sunt optimale asimptotic. (Sortările non-bazate pe comparații, precum Counting Sort/Radix Sort, pot depăși această limită, dar folosesc informații suplimentare despre chei.)

| Algoritm | Best | Average | Worst | Spațiu | Stabil |
|---|---|---|---|---|---|
| HeapSort | O(n log n) | O(n log n) | O(n log n) | O(1) | Nu |
| QuickSort | O(n log n) | O(n log n) | O(n²) | O(log n) | Nu |
| MergeSort | O(n log n) | O(n log n) | O(n log n) | O(n) | Da |

---

## VI. Arbori binari stricți cu ponderi — Algoritmul lui Huffman

- **Arbore binar strict**: fiecare nod are exact 0 sau 2 copii (niciodată doar unul).
- **Problemă**: dat un set de simboluri cu frecvențe (ponderi) de apariție, se caută o **codificare binară prefix-free** (niciun cod nu e prefixul altuia) care minimizează lungimea medie a mesajului codificat.

**Algoritm (greedy)**:
1. Se pune fiecare simbol într-o coadă cu priorități, ordonată după frecvență.
2. Repetat: se extrag cele două noduri cu frecvența minimă, se creează un nod părinte nou cu frecvența = suma lor, cei doi devin copiii lui (stâng/drept), nodul nou se reintroduce în coadă.
3. Se repetă până rămâne un singur nod = rădăcina arborelui Huffman.
4. Codul fiecărui simbol = drumul de la rădăcină la frunza lui (ex. stânga=0, dreapta=1).

**Complexitate**: O(n log n) folosind un heap pentru coada cu priorități.

**Aplicații**: compresia fără pierderi (ZIP, JPEG folosește Huffman ca etapă finală, MP3), transmisie de date — simbolurile frecvente primesc coduri scurte, cele rare coduri lungi, minimizând lungimea totală a mesajului.

| Operație | Complexitate |
|---|---|
| Construire arbore Huffman | O(n log n) — aceeași în toate cazurile (cu heap) |

---

## VII. Tehnici generale de programare

**Greedy**
- La fiecare pas se alege soluția *local optimă*, fără a reveni asupra deciziei.
- Funcționează doar când problema are **proprietatea de alegere greedy** + **substructură optimă**.
- Exemple: algoritmul lui Huffman, algoritmul lui Dijkstra (drum minim), arbore parțial de cost minim (Prim, Kruskal), problema rucsacului fracționar.

**Backtracking**
- Se construiește soluția pas cu pas; dacă o alegere parțială nu poate duce la o soluție validă, se **revine** (backtrack) și se încearcă altă alternativă.
- Explorează practic un arbore de stări, cu „tăiere" (pruning) a ramurilor imposibile.
- Exemple: problema celor N regine, Sudoku, generarea permutărilor/submulțimilor, colorarea grafurilor.

**Divide et Impera**
- Se împarte problema în subprobleme *independente*, mai mici, de același tip; se rezolvă recursiv fiecare subproblemă; se combină rezultatele.
- Exemple: MergeSort, QuickSort, căutarea binară, înmulțirea rapidă a matricelor (Strassen).
- Complexitate tipică analizată prin **relații de recurență** (ex. teorema masterului): `T(n) = a·T(n/b) + f(n)`.

**Programare dinamică**
- Similară cu Divide et Impera, dar subproblemele **se suprapun** (nu sunt independente) — se rezolvă fiecare subproblemă **o singură dată** și se reține rezultatul (**memoizare** top-down sau tabel bottom-up), evitând recalcularea.
- Se aplică problemelor cu **substructură optimă**: soluția optimă a problemei se construiește din soluții optime ale subproblemelor.
- Exemple: șirul lui Fibonacci (memoizat), problema rucsacului 0/1, cel mai lung subșir comun (LCS), drumul minim în graf DAG, distanța de editare (Levenshtein).

**Diferența cheie D&I vs. Programare dinamică**: D&I — subprobleme independente, fără suprapunere (nu are rost să reții rezultate); PD — subprobleme se repetă, reținerea rezultatelor economisește timp exponențial.

---

## Sfaturi rapide pentru interviu
- Când ți se cere un algoritm, spune întâi **ideea** în 1-2 fraze, apoi complexitatea, apoi (dacă se cere) pașii/pseudocodul.
- Fii pregătit să faci diferența clar: **O vs Ω vs Θ** nu sunt "worst/best/average" per definiție, ci margini matematice — dar în practică majoritatea profesorilor acceptă asocierea O=worst, Ω=best, Θ=exact.
- La structuri, poți fi întrebat "de ce alegi lista înlănțuită și nu vectorul" — răspunsul standard: cost inserare/ștergere vs. cost acces direct.
- La arbori, memorează cele 3 cazuri de ștergere din ABC și rotațiile AVL — sunt cele mai frecvente întrebări "pe tablă/verbal".
- La sortări, memorează tabelul de complexități — e aproape sigur întrebat.