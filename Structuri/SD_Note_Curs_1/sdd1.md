# Curs 1

### Structura de date abstracta
- O descriere logica a unui mod de organizare a datelor impreuna
cu operatiile specifice asociate
- Permite abstractizarea operatiilor fara a tine cont de detalii de implementare
cat si incapsularea datelor `(multime, stiva, coada, graf)`

### Colectii si multimi
**Colectie** - grup de elemente de acelasi tip in care **pot exista duplicate**
**Multime** - o colectie ce **NU** contine duplicate

##

# Curs 2

### Liste inlantuite
-  O serie de obiecte ordonate
- Fiecare nod are data si link (cu exceptia ultimului pt next si primului pt prev)

### Liste vs Vectori
Vector
- Necesita estimare nr elemente
- Cautarea se face prin index -> O(1)
- Inserare/Stergere -> O(n)
- Compacte in memorie, se aloca static sau dinamic cu dimensiune fixa

Lista inlantuita
- Nu necesita sa stim nr elemente
- Cautare -> O(n)
- Inserare/Stergere -> O(1)
- Nu sunt compacte in memorie, au nevoie de spatiu pt legaturi

##

# Curs 3

### Lista dublu inlantuita
- Are un extra camp prev (pe langa data si next)
- Se poate parcurge in ambele sensuri

### Aplicatii ale listelor duble inlantuite
- Sisteme de operare - thread scheduler
- Aplicatii de redat melodii de pe CD
- Cache-ul unui browser
- Undo/Redo (CTRL + Z)

### Dictionare
- perechi de tip **chere - valoare**

##

# Curs 4

### Stiva vs Coada (Stack vs Queue)
**Stiva**
- Tip LIFO
- Push
- Pop
- Top

**Coada**
- FIFO
- Enqueue
- Dequeue

##

# Curs 5

### Arbori
Un arbore este format dintr-un nod radacina, caruia ii este
atasat un numar finit de arbori (subarbori)

**Terminologie arbori**
1. Ordin - numar de subarbori atasati nodului
2. Noduri interne - noduri care **AU** subarbori
3. Noduri externe - noduri frunza, care **NU** au subarbori
4. Nivel/Adancime - numarul de arce de la radacina la nod (radacina = nivel 0)
5. Inaltime - nivelul maxim din arbore

### Arbore multicai
Un arbore in care nodurile pot avea mai mult de 2 subarbori

### Arbori oarecare
- Structură ierarhică, aciclică, conexă, cu un singur nod **rădăcină**; fiecare alt nod are exact un **părinte** și 0..k **copii**.
- **Reprezentare**: „copil–frate" (fiecare nod are pointer către primul copil și către următorul frate) — transformă orice arbore în echivalentul unui arbore binar.
- **Parcurgere**: preordine (rădăcină apoi subarbori, stânga la dreapta) sau pe nivel (BFS).

### Arbori binari
- Fiecare nod are cel mult **2 copii** (stâng, drept).
- **Reprezentare**: înlănțuită (`{data, left, right}`) sau prin vector (util la heap: copiii nodului `i` sunt `2i+1` și `2i+2`).
- **Parcurgeri**:
  - **Preordine (RSD)**: rădăcină → stânga → dreapta
  - **Inordine (SRD)**: stânga → rădăcină → dreapta (la ABC dă elementele **sortate**)
  - **Postordine (SDR)**: stânga → dreapta → rădăcină
  - **Pe nivel (BFS)**: folosind o coadă

**Tipuri de arbori binari**
- Arbore binar plin - fiecare nod are exact 0 sau 2 fii
- Arbore binar complet - arbore complet umplut cu posibila exceptie a ultimului
nivel care este umplut de la stanga la dreapta


##

# Curs 6

### Structuri de cautare
**Reprezentari:**
1. Vector de elemente (sortate/nesortate)
2. Liste inlantuite (sortate/nesortate)
3. Cautare prin adresa directa

### Arbori binari de căutare (ABC / BST)
**Proprietate**: pentru orice nod, toate cheile din subarborele stâng < cheia nodului < toate cheile din subarborele drept.

- **Căutare**: compari cheia cu nodul curent, mergi stânga/dreapta — O(h), unde `h` = înălțimea arborelui.
- **Inserare**: cauți poziția (ca la căutare) până ajungi la `NULL`, atașezi noul nod acolo — O(h).
- **Ștergere nod** — 3 cazuri:
  1. Nod frunză → se elimină direct.
  2. Nod cu un singur copil → copilul îi ia locul.
  3. Nod cu doi copii → se înlocuiește cu **succesorul in-order** (cel mai mic din subarborele drept) sau **predecesorul in-order** (cel mai mare din subarborele stâng), apoi se șterge succesorul/predecesorul (care are cel mult un copil).

**Performanță**: O(h) pentru toate operațiile. În cel mai rău caz (arbore degenerat, ca o listă) `h = O(n)`. Într-un arbore echilibrat `h = O(log n)`.

##

# Curs 7

### Arbori AVL (Adelson-Velsky/Landis)
- ABC în care, pentru fiecare nod, **factorul de echilibru** = `înălțime(subarbore stâng) − înălțime(subarbore drept) ∈ {−1, 0, 1}`.
- **Inserare**: se inserează normal ca în ABC, apoi se actualizează înălțimile pe drumul spre rădăcină; dacă un nod devine dezechilibrat (factor ±2), se aplică **rotații**:
  - **Rotație simplă stânga** / **dreapta** (caz stânga-stânga / dreapta-dreapta)
  - **Rotație dublă** (stânga-dreapta / dreapta-stânga) — o rotație pe copil urmată de una pe nod
- **Performanță**: înălțimea unui AVL cu `n` noduri este garantat **O(log n)** (mai precis ≈ `1.44 log₂ n`), deci căutare/inserare/ștergere sunt O(log n) *în cel mai rău caz* — spre deosebire de un ABC neechilibrat.

BF(N) < 0 - left-heavy
BF(N) > 0 - right-heavy
BF(N) = 0 - balanced

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

##

# Curs 8

### Cozi cu prioritate
- Fiecare element are asociata o prioritate, iar elementele sunt extrase
in functie de aceasta prioritate.
- Primul element extras este cel cu cea mai mare prioriate (heap max)
sau elementul cu cea mai mica prioritate (heap min)

### Implementare si utilizari
**Implementari:**
-  C++: priority_queue
- Java: PriorityQueue
- Python: heapq

**Algoritmi ce utilizeaza PQ:**
- Djikstra
- Prim
- Huffman
- Heapsort

|  | Insert | Extract Max |
|---|---|---|
| Vector sau lista (nesortata) | O(1) | O(n) |
| Vector sau lista (sortata) | O(n) | O(n) |

### Heap
- Permite implementarea eficienta a operatiilor cu cozi de prioritate
- Heap-max binar = arbore binar cu proprietatea:
pentru orice nod, **cheia nodului este mai mare decat cheile din nodurile copii, daca exista copii**

### HeapSort
- Se construiește un **max-heap** (arbore binar aproape complet, unde părintele ≥ copii, reprezentat în vector) din tot vectorul — O(n).
- Se extrage repetat maximul (rădăcina), se pune la finalul zonei sortate, se „coboară" (`heapify`) noul element din vârf — O(log n) per extragere, de `n` ori.
- **Complexitate**: O(n log n) în toate cazurile (best/avg/worst). In-place, dar **nu e stabil**.

### Treaps (Tree + Heap)
- Structura de date ce combina BST cu Heap binar
- Fiecare nod contine (info, prioritate)
- Are proprietatea de arbore binar de cautare 
(info)
- Are proprietatea de ordonare din Heap 
binar (prioritate)

### Aplicatii Treaps
- Se pot implementa usor pe structuri paralele
- Folosite in multe aplicatii, de ex: wireless networking, memory allocation, fast parallel aggregate set
operations
- Se pot utizila pentru implementarea unor structuri de date avansate cum ar fi weighted trees, interval trees

##

# Curs 9 - part 1

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

##

# Curs 10

### Continuare grafuri

**Algoritmul lui Djikstra** - afla toate drumurile de cost minim de la un nod (sursa)
la celelalte noduri din graf
- Graful poate fi orientat sau neorientat
- Costurile trebuie sa fie non-negative
- Graful sa fie conex sau tare conex
- **Principiul algoritmului:** orice subcale a unei cai de cost minim este o cale de cost minim

**Algoritmul lui Prim:**
- Determină arborele de acoperire de cost minim 
- Incepe cu un arbore vid și încearcă să adauge 
pe rând câte un arc 
- Selectează arbitrar un nod pe post de rădăcină 
- Cât timp arborele nu conține toate nodurile din 
graf, se alege un arc de cost minim legat la 
arborele parțial construit și se adaugă dacă nu 
formează cicluri 
- Se mentin 2 mutimi de noduri: cele introduse in arbore (S) si cele neintroduse inca (V-S)

**Algoritmul lui Kruskal:**
- Creeaza o padure de arbori A in care, initial, fiecare 
nod din graf este un arbore 
- Creeaza o multime S cu toate arcele din graf 
cat timp S nu este vida si se mai pot adauga in A 
repeta 
- Elimina arcul de cost minim din S 
daca arcul conecteaza 2 arbori diferiti din A 
atunci adauga arcul in A 
- Padurea contine 1 sau mai multi AAM – unul daca 
graful este conex

**Algoritmul lui Warshall:**
- Pentru fiecare pereche de noduri (x,y) a.î. y este 
accesibil din x, adăugăm (dacă nu există) un  arc x → y 
- **Algoritmul lui Warshall**
- Reprezentarea grafurilor prin **matrice de 
adiacență**
- Daca exista un k a.i. x → k si k → y 
atunci x → y 
- Găsește închiderea tranzitivă în O(V3) 
- Algoritm Warshall – inchidera tranzitiva – matrice 
de adiacenta 
- Algoritm Floyd – cele mai scurte cai intr-un graf 
orientat ponderat matrice de adiacenta

##

# Curs 11

### Tabele de dispersie

Datele sunt grupate in forma: key-value
In cazul unei coliziuni (doua chei arata spre acelasi index) se aplica adresare prin inlantuire (cea mai simpla varianta)

**Rezolvarea coliziunilor:**
- Dispersie deschisa (open hashing)
- Dispersie inchisa (closed hashing sau open adressing) - memoreaza inregistrarile (sau cheile si pointer la inregistrari) chiar in tabela hash - EX: bucket hashing
- Grupuri de M/B celule in tabela hash, B - numar de grupuri + un grup de overflow (overflow bucket)

**Aplicatii:**
- Compilatoare - tabele de simboluri
- Determinarea rapida a elementelor cu chei egale in colectii mari de date
- Determinarea rapida a grupurilor de elemente cu chei similare
- Algoritmul de cautare Rabin-Karp a sirurilor de caractere in siruri lungi de caractere
- Algoritmul de compresie a sirurilor de caractere LZW

### Algoritmi de cautare in siruri

**Algoritmul cautarii directe:** 
- Cazul cel mai nefavorabil -> timp N*M
-Cazul mediu: N+M
- Unde **M, N:** nr. de caractere din siruri diferite

**Rabin-Karp:**
- Considera textul ca fiind o memorie mare si trateaza fiecare secventa de M caractere a textului
ca o cheie intr-o tabela de dispersie (hash)
- Trebuie  sa  calculam  functia  de  dispersie  pentru toate secventele posibile de M caractere 
consecutive din text si sa verificam daca valorile  obtinute  sunt  egale  cu  functia  de  dispersie  a 
sablonului
- Are destul de probabil o complexitate **liniara O(n)**
- Algoritmul are timp proportional cu N+M

**Compresie Lempel-Ziv-Welch:**
- Algoritm de compresie a sirurilor de caractere foarte folosit
- Foloseste un dictionar prin care asociaza unor siruri de caractere de lungimi diferite coduri numerice
intregi si inlocuieste secvente de caractere din fisierul initial prin aceste numere
- Sirul initial (care trebuie comprimat) este analizat si codificat intr-o singura trecere, fara revenire
- La **stanga** pozitiei curente sunt subsiruri deja codificate, iar la **dreapta** cursorului se cauta cea mai lunga secventa care exista deja in dictionar
- Odata gasita secventa, va fi inlocuita prin codul asociat deja si se adauga la dictionar o secventa cu un caracter mai lunga

**Algoritmul Knuth-Morris-Pratt:**
- Presupune precompilarea sablonului
- Nu decrementeaza indexul in sir
- Da rezultate bune daca o nepotrivire a aparut dupa o identificare partiala de o anumita lungime

##

# Curs 12

### Metode de sortare

**Metoda de sortare stabila:**
- Mentine ordinea relativa a inregistrarilor cu chei egale
- Daca stabilitatea este importanta se poat ecrea o noua cheie - cheie extinsa

**Selection Sort:**
- Gaseste cel mai mic element si il schimba cu elementul de pe prima pozitie, se repeta procesul asta.
- Metoda buna pentru sortarea secventelor **mici**

**Insertion Sort:**
- Considera fiecare element pe rand, inserand elementul la locul potrivit printre cele deja sortate

**Shell Sort:**
- Extindere a Insertion Sort - creste viteza permitand interschimbari intre elemente neadiacente

**HeapSort**
- Se construiește un **max-heap** (arbore binar aproape complet, unde părintele ≥ copii, reprezentat în vector) din tot vectorul — O(n).
- Se extrage repetat maximul (rădăcina), se pune la finalul zonei sortate, se „coboară" (`heapify`) noul element din vârf — O(log n) per extragere, de `n` ori.
- **Complexitate**: O(n log n) în toate cazurile (best/avg/worst). In-place, dar **nu e stabil**.

**QuickSort**
- Se alege un **pivot (de obicei ultimul element)**, se partiționează vectorul astfel încât elementele mai mici să fie înaintea lui, cele mai mari după, apoi se sortează recursiv (Divide et Impera) cele două părți.
- **Complexitate**: medie O(n log n); worst-case **O(n²)** (când pivotul e mereu extrem — ex. vector deja sortat cu pivot = primul element). In-place, nu e stabil, dar are constante mici → foarte rapid în practică.

**MergeSort**
- Se împarte vectorul recursiv în jumătăți (Divide et Impera) până la elemente unice, apoi se **interclasează** (merge) subvectorii sortați.
- **Complexitate**: O(n log n) garantat în **toate** cazurile. Necesită O(n) memorie suplimentară. Este **stabil**.

**Radix Sort:**
- Este un algoritm de sortare liniară (pentru un număr fix de cifre) care sortează elementele procesându-le cifră cu cifră.


### Performanta metodelor de sortare

| Algorithm | Best | Average | Worst | Auxiliary Space |
| ----- | -----: | ----: | ----: | ----: |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) |
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(n) |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) |
| Shell Sort | O(n log n) | O(n log n) | O(n²) | O(1) |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(n) |
| Radix Sort | O(Nk) | O(Nk) | O(Nk) | O(N+k) |
