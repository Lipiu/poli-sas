# Curs 1

### Arbori binari de căutare (ABC / BST)

Un arbore binar nevid este **arbore de cautare** daca indeplineste urmatoarele conditii:
**Proprietate 1:** cei doi arbori (daca exista) sunt, la randul lor, **arbori de cautare**
**Proprietate 2:** pentru orice nod, toate cheile din subarborele stâng < cheia nodului < toate cheile din subarborele drept.

- **Căutare**: compari cheia cu nodul curent, mergi stânga/dreapta — O(h), unde `h` = înălțimea arborelui.
- **Inserare**: cauți poziția (ca la căutare) până ajungi la `NULL`, atașezi noul nod acolo — O(h).
- **Ștergere nod** — 3 cazuri:
  1. Nod frunză → se elimină direct.
  2. Nod cu un singur copil → copilul îi ia locul.
  3. Nod cu doi copii → se înlocuiește cu **succesorul in-order** (cel mai mic din subarborele drept) sau **predecesorul in-order** (cel mai mare din subarborele stâng), apoi se șterge succesorul/predecesorul (care are cel mult un copil).

**Performanță**: O(h) pentru toate operațiile. În cel mai rău caz (arbore degenerat, ca o listă) `h = O(n)`. Într-un arbore echilibrat `h = O(log n)`.


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

# Curs 2

### Arbori binari
- Fiecare nod are cel mult **2 copii** (stâng, drept).
- **Reprezentare**: înlănțuită (`{data, left, right}`) sau prin vector (util la heap: copiii nodului `i` sunt `2i+1` și `2i+2`).
- **Parcurgeri**:
  - **Preordine (VSD)**: rădăcină → stânga → dreapta
  - **Inordine (SVD)**: stânga → rădăcină → dreapta (la ABC dă elementele **sortate**)
  - **Postordine (SDV)**: stânga → dreapta → rădăcină
  - **Pe nivel (BFS)**: folosind o coadă

### Arbori oarecare
- Structură ierarhică, aciclică, conexă, cu un singur nod **rădăcină**; fiecare alt nod are exact un **părinte** și 0..k **copii**.
- **Reprezentare**: „copil–frate" (fiecare nod are pointer către primul copil și către următorul frate) — transformă orice arbore în echivalentul unui arbore binar.
- **Parcurgere**: preordine (rădăcină apoi subarbori, stânga la dreapta) sau pe nivel (BFS).

### Arbore multicai
- Arbore in care nodurile pot avea mai **mult** de **2** subarbori

**Ordin (grad):** numarul de subarbori atasati nodului
**Noduri interne:** nodurile care au subarbori
**Noduri externe:** noduri frunza
**Nivelul unui nod:** distanta la care se afla fata de radacina (radacina are nivel 0)
**Inaltime:** nivelul maxim din arbore

### Parcurgeri
**DFS (Depth-First Search)** — mergi cât de „adânc" posibil pe o ramură înainte de a face backtrack. Se implementează recursiv sau iterativ cu o **stivă**. Complexitate: O(V + E).

**BFS (Breadth-First Search)** — parcurge graful „în lățime", nivel cu nivel, folosind o **coadă**. Găsește cel mai scurt drum (în număr de muchii) în grafuri neponderate. Complexitate: O(V + E).

##

# Curs 3

### Arbori AVL (Adelson-Velsky/Landis)
- ABC în care, pentru fiecare nod, **factorul de echilibru** = `înălțime(subarbore stâng) − înălțime(subarbore drept) ∈ {−1, 0, 1}`.
- **Inserare**: se inserează normal ca în ABC, apoi se actualizează înălțimile pe drumul spre rădăcină; dacă un nod devine dezechilibrat (factor ±2), se aplică **rotații**:
  - **Rotație simplă stânga** / **dreapta** (caz stânga-stânga / dreapta-dreapta)
  - **Rotație dublă** (stânga-dreapta / dreapta-stânga) — o rotație pe copil urmată de una pe nod
- **Performanță**: înălțimea unui AVL cu `n` noduri este garantat **O(log n)** (mai precis ≈ `1.44 log₂ n`), deci căutare/inserare/ștergere sunt O(log n) *în cel mai rău caz* — spre deosebire de un ABC neechilibrat.