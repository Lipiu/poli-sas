# Notițe Java – Admitere Master SAS (Poli Securitate)

---

## 1. Tipuri de date în Java

Java este un limbaj **static typed** (tipurile se verifică la compilare) și distinge două mari categorii de tipuri:

### 1.1 Tipuri primitive (8 total)

| Tip | Dimensiune | Interval / observații |
|---|---|---|
| `byte` | 8 biți | -128 .. 127 |
| `short` | 16 biți | -32768 .. 32767 |
| `int` | 32 biți | ~ -2.1 mld .. 2.1 mld |
| `long` | 64 biți | folosește sufix `L` (ex: `100L`) |
| `float` | 32 biți | virgulă mobilă, sufix `f` |
| `double` | 64 biți | virgulă mobilă, implicit pentru literali zecimali |
| `char` | 16 biți | Unicode, valori 0..65535 |
| `boolean` | 1 bit (conceptual) | `true` / `false` |

**Caracteristici cheie:**
- Nu sunt obiecte, nu au metode, se stochează pe **stivă** (dacă sunt variabile locale) — mai eficiente decât obiectele.
- Java nu are `unsigned` (excepție parțială: operații unsigned pe `int`/`long` via metode statice din Java 8+).
- Conversii:
  - **Implicite (widening)**: `byte → short → int → long → float → double` (fără pierdere de informație, automat).
  - **Explicite (narrowing/cast)**: necesită cast manual, pot pierde date: `int i = (int) 3.99;` → 3.

### 1.2 Tipuri referință (reference types)

- **Clase**, **interfețe**, **array-uri**, **enum-uri**.
- Variabila reține o **referință** (adresă) către obiectul aflat pe **heap**, nu obiectul propriu-zis.
- Valoare implicită: `null`.
- Fiecare tip primitiv are un **wrapper** corespunzător (obiect): `Byte, Short, Integer, Long, Float, Double, Character, Boolean`.

### 1.3 Autoboxing / Unboxing

```java
Integer obj = 5;      // autoboxing: int -> Integer
int x = obj;           // unboxing: Integer -> int
```

- Introdus în Java 5, se face automat de compilator.
- Atenție la comparații: `Integer a = 127, b = 127; a == b` → `true` (cache -128..127), dar pentru valori în afara acestui interval `==` compară referințe, nu valori → folosiți `.equals()`.

### 1.4 Array-uri

- Sunt obiecte în Java (au `.length`, se alocă pe heap).
- Pot fi de tipuri primitive sau de referință, unidimensionale sau multidimensionale (array de array-uri).

```java
int[] v = new int[10];
int[][] matrice = new int[3][3];
```

---

## 2. Ascunderea implementării – pachete, specificatori de acces

### 2.1 Pachete (packages)

- Mecanism de organizare a claselor în **spații de nume** și de control al vizibilității.
- Declarare: `package com.exemplu.proiect;` (prima linie din fișier, opțional).
- Import: `import java.util.List;` sau `import java.util.*;`
- Corespund unei structuri de directoare pe disc.
- Rol dublu: organizare logică + **encapsulare la nivel de pachet**.

### 2.2 Specificatori de acces (access modifiers)

| Modificator | Aceeași clasă | Același pachet | Subclasă (alt pachet) | Oriunde |
|---|:---:|:---:|:---:|:---:|
| `private` | ✔ | ✘ | ✘ | ✘ |
| *(implicit / package-private)* | ✔ | ✔ | ✘ | ✘ |
| `protected` | ✔ | ✔ | ✔ | ✘ |
| `public` | ✔ | ✔ | ✔ | ✔ |

**Principiul encapsulării (data hiding):**
- Câmpurile (atributele) unei clase se declară de regulă `private`.
- Accesul din exterior se face prin metode publice (**getteri/setteri**), permițând controlul validării și ascunderea detaliilor de implementare.
- Beneficii: se poate schimba implementarea internă fără a afecta codul client; se pot impune invarianți (validare în setter).

```java
public class ContBancar {
    private double sold; // ascuns

    public double getSold() { return sold; }
    public void depune(double suma) {
        if (suma > 0) sold += suma; // validare
    }
}
```

- La nivel de **clasă** (nu membru), sunt permise doar `public` sau *implicit* (package-private) — o clasă top-level nu poate fi `private` sau `protected`.

---

## 3. Reutilizarea claselor – compunere (agregare) și moștenire

Două mecanisme principale de reutilizare a codului: **compunerea (has-a)** și **moștenirea (is-a)**.

### 3.1 Compunere / Agregare (has-a)

- O clasă conține ca membru o instanță a altei clase.
- Relație mai flexibilă, favorizată adesea în locul moștenirii ("*favor composition over inheritance*").

```java
class Motor { void porneste() { ... } }

class Masina {
    private Motor motor = new Motor(); // agregare
    void demareaza() { motor.porneste(); }
}
```

- **Agregare** vs **Compoziție** (nuanță UML): în agregare obiectul membru poate exista independent de container (relație slabă); în compoziție ciclul de viață al membrului depinde strict de container (relație puternică, "part-of").

### 3.2 Moștenire (is-a) – `extends`

```java
class Animal {
    protected String nume;
    void mananca() { System.out.println("mananca"); }
}

class Caine extends Animal {
    void latra() { System.out.println("ham"); }
}
```

- Java permite **moștenire simplă** pentru clase (o singură superclasă), dar **moștenire multiplă de interfețe**.
- Toate clasele moștenesc implicit din `Object` (dacă nu se specifică altă superclasă).
- Cuvântul cheie `super`:
  - `super()` – apelează constructorul clasei de bază (trebuie să fie prima instrucțiune din constructor).
  - `super.metoda()` – apelează versiunea din superclasă a unei metode suprascrise.
- `final` pe o clasă → nu poate fi extinsă; `final` pe o metodă → nu poate fi suprascrisă.
- Constructorii **nu se moștenesc**; dacă superclasa nu are constructor implicit, subclasa trebuie să apeleze explicit `super(...)`.

### 3.3 Ordinea de inițializare la moștenire

1. Se alocă memorie și se inițializează cu valori implicite (0, null, false).
2. Se apelează constructorul superclasei (implicit sau explicit prin `super`).
3. Se execută inițializatorii câmpurilor din subclasă, în ordinea declarării.
4. Se execută corpul constructorului subclasei.

---

## 4. Polimorfism – suprascriere vs. supraîncărcare

### 4.1 Supraîncărcare (Overloading) – polimorfism static (compile-time)

- Mai multe metode cu **același nume**, dar **semnătură diferită** (număr/tip parametri) în **aceeași clasă**.
- Se rezolvă la **compilare** (static binding / early binding).
- Tipul de return nu poate diferenția singur o supraîncărcare.

```java
class Calculator {
    int aduna(int a, int b) { return a + b; }
    double aduna(double a, double b) { return a + b; }
    int aduna(int a, int b, int c) { return a + b + c; }
}
```

### 4.2 Suprascriere (Overriding) – polimorfism dinamic (runtime)

- O subclasă redefinește o metodă **moștenită**, cu **aceeași semnătură** (nume, parametri, tip de return covariant).
- Se rezolvă la **runtime**, în funcție de tipul **real** al obiectului (dynamic/late binding), prin mecanismul de **dynamic dispatch** al JVM-ului.
- Reguli:
  - Nivelul de acces nu poate fi mai restrictiv decât în superclasă.
  - Nu se pot suprascrie metode `private`, `static` sau `final`.
  - Adnotarea `@Override` (recomandată, nu obligatorie) – ajută compilatorul să detecteze erori.

```java
class Animal {
    void faceSunet() { System.out.println("sunet generic"); }
}
class Pisica extends Animal {
    @Override
    void faceSunet() { System.out.println("miau"); }
}

Animal a = new Pisica();
a.faceSunet(); // "miau" -> decis la runtime (tipul real al obiectului)
```

### 4.3 Legarea metodelor (binding)

- **Static binding**: metode `private`, `static`, `final`, constructori, variabile (câmpurile **nu** sunt polimorfice, ele se leagă static după tipul declarat).
- **Dynamic binding**: metode de instanță suprascrise – JVM alege implementarea în funcție de obiectul efectiv din heap prin **vtable** (tabelă de metode virtuale).

### 4.4 Upcasting / Downcasting

```java
Animal a = new Pisica();          // upcasting - implicit, sigur
Pisica p = (Pisica) a;            // downcasting - explicit, poate arunca ClassCastException
if (a instanceof Pisica pis) { }  // pattern matching instanceof (Java 16+)
```

---

## 5. Clase abstracte, Interfețe, Clase interioare

### 5.1 Clase abstracte

```java
abstract class Forma {
    abstract double arie();          // metodă abstractă - fără implementare
    void afiseaza() {                // metodă concretă
        System.out.println("Aria: " + arie());
    }
}
class Cerc extends Forma {
    double raza;
    double arie() { return Math.PI * raza * raza; }
}
```

- Nu pot fi instanțiate direct (`new Forma()` → eroare de compilare).
- Pot conține atât metode abstracte, cât și implementate, precum și câmpuri, constructori.
- O clasă cu cel puțin o metodă abstractă **trebuie** declarată `abstract`.
- Prima subclasă concretă trebuie să implementeze toate metodele abstracte moștenite.

### 5.2 Interfețe

```java
interface Comparabil {
    int compara(Object o); // implicit public abstract

    default void afiseazaInfo() { // metodă default (Java 8+)
        System.out.println("Obiect comparabil");
    }
    static Comparabil identitate() { // metodă statică (Java 8+)
        return null;
    }
}
class Produs implements Comparabil {
    public int compara(Object o) { return 0; }
}
```

- Toate câmpurile sunt implicit `public static final` (constante).
- Metodele sunt implicit `public abstract`, cu excepția `default` și `static` (Java 8+) și `private` (Java 9+, folosite intern în interfață).
- O clasă poate implementa **mai multe interfețe** → simulează moștenirea multiplă.
- Diferență cheie **abstract class vs interface**:

| Aspect | Clasă abstractă | Interfață |
|---|---|---|
| Moștenire | simplă (`extends`, o singură superclasă) | multiplă (`implements`, mai multe) |
| Stare (câmpuri) | poate avea câmpuri de instanță | doar constante `static final` |
| Constructor | da | nu |
| Scop | relație "is-a" puternică, cod comun | contract / capabilitate ("can-do") |

### 5.3 Clase interioare (inner classes)

- **Member inner class** – legată de o instanță a clasei exterioare, are acces la membrii ei (inclusiv `private`).
```java
class Exterioara {
    private int x = 10;
    class Interioara {
        void afiseaza() { System.out.println(x); }
    }
}
Exterioara e = new Exterioara();
Exterioara.Interioara i = e.new Interioara();
```
- **Static nested class** – nu are referință implicită la instanța exterioară, se comportă ca o clasă top-level plasată în interior.
```java
class Exterioara {
    static class Statica { }
}
```
- **Local inner class** – definită în interiorul unei metode, vizibilă doar acolo.
- **Anonymous inner class** – clasă fără nume, definită și instanțiată pe loc, utilă pentru implementări rapide (de ex. listeners, comparatori).
```java
Comparator<Integer> c = new Comparator<Integer>() {
    public int compare(Integer a, Integer b) { return a - b; }
};
```
- Utilitate: încapsulare mai strânsă, organizare logică, acces direct la contextul clasei exterioare (util pentru callback-uri, iteratori interni etc.).

---

## 6. Șabloane de proiectare (Design Patterns)

### 6.1 Singleton (creațional)

Garantează o **singură instanță** a unei clase și oferă un punct global de acces.

```java
public class Singleton {
    private static volatile Singleton instanta;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instanta == null) {
            synchronized (Singleton.class) {
                if (instanta == null) instanta = new Singleton();
            }
        }
        return instanta;
    }
}
```
- Constructor `private` → nu poate fi instanțiat din exterior.
- Varianta *double-checked locking* pentru thread-safety cu performanță bună.
- Alternativă simplă și thread-safe: `enum Singleton { INSTANCE; }`

### 6.2 Factory Method (creațional)

Delegă crearea obiectelor unei metode/clase dedicate, ascunzând clasa concretă instanțiată de client.

```java
interface Forma { void deseneaza(); }
class Cerc implements Forma { public void deseneaza() { ... } }
class Patrat implements Forma { public void deseneaza() { ... } }

class FormaFactory {
    static Forma creeaza(String tip) {
        return switch (tip) {
            case "cerc" -> new Cerc();
            case "patrat" -> new Patrat();
            default -> throw new IllegalArgumentException();
        };
    }
}
```
- Clientul lucrează cu interfața/clasa abstractă, nu cu clasele concrete → cuplare redusă.

### 6.3 Observer (comportamental)

Definește o dependență de tip unu-la-mulți: când **subiectul** își schimbă starea, toți **observatorii** înregistrați sunt notificați automat.

```java
interface Observator { void actualizeaza(String eveniment); }

class Subiect {
    private List<Observator> observatori = new ArrayList<>();
    void adauga(Observator o) { observatori.add(o); }
    void notifica(String eveniment) {
        for (Observator o : observatori) o.actualizeaza(eveniment);
    }
}
```
- Stă la baza mecanismelor de evenimente/listeners (ex. `ActionListener` în Swing).

### 6.4 Visitor (comportamental)

Separă un algoritm de structura de obiecte pe care operează, permițând adăugarea de noi operații fără a modifica clasele elementelor (folosește **double dispatch**).

```java
interface Vizitator { void viziteaza(Cerc c); void viziteaza(Patrat p); }
interface Element { void accepta(Vizitator v); }

class Cerc implements Element {
    public void accepta(Vizitator v) { v.viziteaza(this); }
}
class Patrat implements Element {
    public void accepta(Vizitator v) { v.viziteaza(this); }
}

class VizitatorArie implements Vizitator {
    public void viziteaza(Cerc c) { /* calcul arie cerc */ }
    public void viziteaza(Patrat p) { /* calcul arie patrat */ }
}
```
- Util când există multe operații diferite pe o ierarhie de clase stabilă, dar operațiile se schimbă des.

**De reținut pentru examen:** clasificarea GoF – **creaționale** (Singleton, Factory, Builder, Prototype), **structurale** (Adapter, Decorator, Composite), **comportamentale** (Observer, Visitor, Strategy, Iterator).

---

## 7. Colecții (Java Collections Framework)

### 7.1 Ierarhia principală

```
Iterable
 └─ Collection
     ├─ List   (ordonată, permite duplicate)
     │    ├─ ArrayList   (array redimensionabil, acces rapid O(1))
     │    ├─ LinkedList  (listă dublu-înlănțuită, inserare/ștergere rapidă)
     │    └─ Vector       (sincronizat, legacy)
     ├─ Set    (nu permite duplicate)
     │    ├─ HashSet       (bazat pe hash table, neordonat)
     │    ├─ LinkedHashSet (păstrează ordinea inserării)
     │    └─ TreeSet       (ordonat, bazat pe arbore roșu-negru, implementează SortedSet)
     └─ Queue / Deque
          ├─ PriorityQueue
          └─ ArrayDeque

Map (NU extinde Collection)
 ├─ HashMap       (chei unice, neordonat, O(1) mediu)
 ├─ LinkedHashMap (păstrează ordinea inserării)
 └─ TreeMap        (ordonat după chei, SortedMap)
```

### 7.2 Interfețe cheie

- **Collection** – operații de bază: `add`, `remove`, `contains`, `size`, `iterator()`.
- **List** – acces indexat, permite duplicate: `get(i)`, `set(i, e)`.
- **Set** – colecție fără duplicate (folosește `equals()`/`hashCode()` pentru HashSet).
- **Map** – asociere cheie-valoare, chei unice.
- **Iterator** – parcurgere secvențială: `hasNext()`, `next()`, `remove()`.

### 7.3 Complexități uzuale (aproximativ)

| Structură | acces (get) | inserare | căutare |
|---|---|---|---|
| ArrayList | O(1) | O(1) amortizat / O(n) la mijloc | O(n) |
| LinkedList | O(n) | O(1) la capete | O(n) |
| HashMap/HashSet | - | O(1) mediu | O(1) mediu |
| TreeMap/TreeSet | - | O(log n) | O(log n) |

### 7.4 Generics și colecții

```java
List<String> nume = new ArrayList<>();
Map<String, Integer> varsteMap = new HashMap<>();
```

### 7.5 Contract `equals()` / `hashCode()`

- Obligatoriu de respectat pentru corectitudinea în `HashSet`/`HashMap`: dacă `a.equals(b) == true`, atunci `a.hashCode() == b.hashCode()`.
- `Comparable<T>` (`compareTo`) – ordine naturală, folosită de `TreeSet`/`TreeMap`/`Collections.sort`.
- `Comparator<T>` (`compare`) – ordine externă, alternativă/complementară, poate fi dată ca lambda.

```java
List<Persoana> lista = ...;
lista.sort(Comparator.comparing(Persoana::getVarsta));
```

### 7.6 Utilitare

- `Collections` – metode statice: `sort`, `reverse`, `max`, `min`, `unmodifiableList`, `synchronizedList`.
- `Arrays` – utilitare pentru array-uri: `sort`, `asList`, `binarySearch`.
- Stream API (Java 8+): `lista.stream().filter(...).map(...).collect(Collectors.toList())`.

---

## 8. Tratarea erorilor. Excepții

### 8.1 Ierarhia excepțiilor

```
Throwable
 ├─ Error               (erori grave JVM, nu se tratează în mod normal, ex: OutOfMemoryError)
 └─ Exception
      ├─ RuntimeException (unchecked) - ex: NullPointerException, ArrayIndexOutOfBoundsException,
      │                                     ArithmeticException, ClassCastException
      └─ (checked exceptions) - ex: IOException, SQLException
```

### 8.2 Checked vs Unchecked

- **Checked** (verificate la compilare): trebuie tratate cu `try-catch` sau declarate cu `throws` în semnătura metodei. Reprezintă condiții recuperabile (ex. fișier inexistent).
- **Unchecked** (`RuntimeException` și subclasele, plus `Error`): nu sunt obligatorii de tratat la compilare; de regulă indică erori de programare.

### 8.3 Sintaxă try-catch-finally

```java
try {
    int[] v = new int[5];
    System.out.println(v[10]);
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Index invalid: " + e.getMessage());
} catch (Exception e) {
    System.out.println("Alta eroare: " + e);
} finally {
    System.out.println("Se executa mereu (curatare resurse)");
}
```

- `finally` se execută întotdeauna (cu excepții rare: `System.exit()`, crash JVM).
- Se pot prinde mai multe tipuri într-un singur `catch` (Java 7+): `catch (IOException | SQLException e)`.

### 8.4 try-with-resources (Java 7+)

```java
try (BufferedReader br = new BufferedReader(new FileReader("f.txt"))) {
    System.out.println(br.readLine());
} catch (IOException e) {
    e.printStackTrace();
}
```
- Resursele trebuie să implementeze `AutoCloseable`; se închid automat, garantat, în ordine inversă declarării.

### 8.5 Aruncarea și propagarea excepțiilor

```java
void verifica(int varsta) throws IllegalArgumentException {
    if (varsta < 0) throw new IllegalArgumentException("Varsta invalida");
}
```

### 8.6 Excepții custom

```java
class SoldInsuficientException extends Exception {
    public SoldInsuficientException(String mesaj) { super(mesaj); }
}
```
- Se extinde de regulă `Exception` (pentru checked) sau `RuntimeException` (pentru unchecked).

### 8.7 Bune practici

- Nu prindeți excepții generice (`Exception`, `Throwable`) fără motiv – ascunde erorile reale.
- Nu lăsați blocuri `catch` goale ("swallowing exceptions").
- Folosiți excepții pentru situații excepționale, nu pentru controlul fluxului normal.

---

## 9. Sistemul de Intrare/Ieșire (I/O)

### 9.1 Stream-uri de octeți vs. caractere

- **Byte streams** (`InputStream` / `OutputStream`) – date binare.
  - `FileInputStream`, `FileOutputStream`, `BufferedInputStream`, `BufferedOutputStream`.
- **Character streams** (`Reader` / `Writer`) – text, gestionează automat encoding-ul.
  - `FileReader`, `FileWriter`, `BufferedReader`, `BufferedWriter`, `PrintWriter`.

```java
try (BufferedReader br = new BufferedReader(new FileReader("input.txt"));
     PrintWriter pw = new PrintWriter(new FileWriter("output.txt"))) {
    String linie;
    while ((linie = br.readLine()) != null) {
        pw.println(linie.toUpperCase());
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

### 9.2 Design pattern folosit: Decorator

- Clasele I/O folosesc pattern-ul **Decorator**: fiecare wrapper adaugă funcționalitate (buffering, conversie, etc.) peste un stream de bază, prin compunere (`new BufferedReader(new FileReader(...))`).

### 9.3 Serializare

```java
class Persoana implements Serializable {
    private String nume;
}

ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("p.dat"));
oos.writeObject(new Persoana());
```
- Interfața `Serializable` (marker interface, fără metode) semnalează că obiectul poate fi convertit într-un flux de octeți.
- `transient` – exclude un câmp de la serializare.
- `serialVersionUID` – identifică versiunea clasei pentru compatibilitate la deserializare.

### 9.4 NIO (New I/O – java.nio, Java 7+ NIO.2)

- `Path`, `Files`, `Paths` – API modern pentru manipularea fișierelor/directoarelor.
```java
Path p = Paths.get("test.txt");
List<String> linii = Files.readAllLines(p);
Files.write(p, "continut nou".getBytes());
```
- Oferă operații mai eficiente și mai bogate decât `java.io` clasic (copiere, mutare, atribute de fișier, watch service).

### 9.5 Console I/O

```java
Scanner sc = new Scanner(System.in);
int n = sc.nextInt();
```
- `System.in`, `System.out`, `System.err` – stream-uri standard predefinite.

---

## 10. Programare generică

### 10.1 Clase parametrizate (generice)

```java
class Cutie<T> {
    private T continut;
    public void seteaza(T continut) { this.continut = continut; }
    public T obtine() { return continut; }
}

Cutie<Integer> c = new Cutie<>();
c.seteaza(10);
```
- `T` este un **parametru de tip**, înlocuit efectiv la utilizare.
- Beneficii: **siguranță la tipuri la compilare** (type safety) — se elimină cast-urile manuale și erorile `ClassCastException` la runtime; cod reutilizabil pentru orice tip.
- Convenții de denumire: `T` (Type), `E` (Element), `K`/`V` (Key/Value), `N` (Number).

### 10.2 Type erasure

- Generics în Java sunt implementate prin **type erasure**: informația de tip generic există doar la compilare; la runtime, `T` este înlocuit cu `Object` (sau cu limita sa, dacă e specificată).
- Consecințe: nu se pot crea instanțe `new T()`, nu se pot crea array-uri generice `new T[10]`, nu există `T.class`.

### 10.3 Limitarea parametrizării (bounded type parameters)

```java
class CutieNumerica<T extends Number> {
    private T valoare;
    double dubla() { return valoare.doubleValue() * 2; }
}
```
- `T extends Number` – restricționează `T` la `Number` sau subclase ale sale (funcționează și pentru interfețe, cuvântul folosit este tot `extends`).
- **Multiple bounds**: `<T extends Number & Comparable<T>>` (o singură clasă + eventual mai multe interfețe).

### 10.4 Wildcards (`?`)

```java
void afiseaza(List<?> lista) { ... }                    // unknown type
void suma(List<? extends Number> lista) { ... }          // upper bounded - doar citire sigura
void adauga(List<? super Integer> lista) { ... }          // lower bounded - doar scriere sigura
```
- Regula **PECS** (*Producer Extends, Consumer Super*): folosiți `extends` când colecția **produce** (citiți din ea), `super` când colecția **consumă** (scrieți în ea).

### 10.5 Metode parametrizate (generic methods)

```java
public static <T> T primul(List<T> lista) {
    return lista.get(0);
}

public static <T extends Comparable<T>> T maxim(T a, T b) {
    return a.compareTo(b) > 0 ? a : b;
}
```
- Parametrul de tip se declară înaintea tipului de return: `<T>`.
- Pot fi independente de faptul că respectiva clasă este sau nu generică (ex: metode `static` generice într-o clasă non-generică).

### 10.6 Interfețe generice

```java
interface Comparabil<T> {
    int compareTo(T alt);
}
class Punct implements Comparabil<Punct> {
    public int compareTo(Punct alt) { return 0; }
}
```

### 10.7 Restricții importante

- Nu se pot folosi tipuri primitive ca argument generic direct (`List<int>` este invalid) → se folosesc wrapper-ii (`List<Integer>`).
- Nu se pot crea array-uri de tip generic parametrizat: `new List<String>[10]` → eroare.
- Membrii `static` ai unei clase generice **nu pot folosi** parametrul de tip al clasei (`T`), deoarece `T` e legat de instanță, nu de clasă.

---