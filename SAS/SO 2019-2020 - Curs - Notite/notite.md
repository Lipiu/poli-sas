# Curs 1 - Nevoia de OS

### Sistemul de operare
**Sistemul de operare** este componenta software folosită pentru a asigura buna funcționare a sistemului. 
Are rolul de arbitru și gardian al resurselor sistemului. 
Aplicațiile trebuie să aibă acces sigur (izolat), echitabil (să nu acapareze o singură aplicație) la resursele sistemului.

**A syscall**, or system call, is a way for a computer program to request a service from the operating system, such as accessing hardware or managing processes. It serves as an essential interface between a program and the operating system.

### Nevoia de sistem de operare
**Nevoia:**
Ofera mai multe functionalitati
Mai multe aplicatii

**Consecinte:**
Hardware mai puternic, mai performant
Medierea accesului la resursele hardware (indeplinit de OS)


### OS Overhead
**Operating system overhead** refers to the resources consumed by the operating system to manage hardware and software interactions, which are not directly related to executing user applications.

##

# Curs 2 - Fisiere

### Sistem de fisiere
Datele sunt stocate ca fisiere, de obicei pe disc (suport persistent)
Persistenta, organizare, eficienta, partajare
Aplicatii (utilizare) -> Sistem de fisiere (organizare) -> date (suport persistent)

### Fisiere
**Fisierul** este o secventa de octeti (byte stream) stocati pe disc
Fisier binar -> format care nu este human readable
Fisier text -> format human readable

**Metadata (data about data):** nume, identificator, dimensiune, user, group, permisiuni, timpi de acces
si se gasesc in structura numita **File Control Block (FCB)**

### Fisier deschis
**Comenzi:** open, fopen, CreateFile (Windows)
Proces -> apel de sistem de deschidere -> structura de fisier deschis -> structura de fisier pe disc (FCB + date)

### Deschiderea unui fisier
**Permisiuni:**
R -> read
W -> write
X -> execute

### Creare fisier
Trebuie sa avem permisiuni de scriere in directorul in care vom creea fisierul
- Avem optiunea **O_CREAT** -> este un flag al functiei open() care creeaza un fisier daca acesta nu exista deja
- Numele este dat ca parametru
- user/group este al procesului care a efectual syscall
- dimensiunea = 0
- **Permisiuni** -> date ca al treilea parametru

### Descriptorul de fisiere / Tabela descriptorilor de fisiere
- **Descriptorul de fisiere** -> un index intr-o tabela numita tabela descriptorilor de fisiere (file descriptor table) retinuta in kernel/supervisor space
- Exista o tabela de descriptori pt fiecare proces
- Intrarea in tabela este **validata/initializata** in momentul deschiderii unui fisier
- Inchiderea unui fisier rezulta in **invalidarea** acestuia
- Intrarile in tabela se pot referi nu doar la fisiere, ci si la: 
**socket:** (create: socket | inchidere: close) 
**pipe:** (creare: pipe | inchidere: close) 
**terminal**
- Tabela are dimensiune limitata pt a preveni abuzuri de prea multe fisiere deschise
- **Descriptori standard:**
    - 0 (std input)
    - 1 (std output)
    - 2 (std error)

**lsof (list open file):** report a list of all open files and the processes that opened them

### Operatii cu date si cursorul de fisier
**Operatii:**
    - read - citire date din fisier intr-un buffer (user/application space)
    - write - scriere date dintr-un buffer (din user/application space) in fisier
Citirea **NU** avanseaza dincolo de dimensiunea fisierului
Scirerea poate trece peste si modifica/creste dimensiunea fisierului

##

# Curs 3 - Procese

### Procese
- Incapsularea unei actiuni in sistemul de calcul: date si cod in memorie, rulare instructiuni pe procesor,
interactiune cu I/O
- Permite multi-programare
- Procesele sunt izolate: memoria este separata, ruleaza separat pe procesor
- Izolarea si planificarea proceselor este handled de OS

### Atributele unui proces
- PID (Process ID)
- Resurse:
    - spatiu virtual de adrese (memorie)
    - timp de lucru pe procesor
    - fisiere deschise
    - user/group
    - starea unui proces
    - structura de proces - PCB (Process Control Block) retinuta in kernel space

### Creare proces
Dintr-un proces existent (parinte), uzual un shell, se creeaza o ierarhie de procese
Un proces are un singur parinte, dar un proces parinte poate avea oricate procese copii
**fork()** 
-> creeaza un proces copil ca fiind o copie a procesului parinte 
-> partajeaza info precum tabela de descriptori de fisiere, pornesc de la acelasi cod
-> se apeleaza o data si se intoarce de doua ori: o data in procesul copil si o data in procesul parinte

**exec()**
-> invoca loaderul si incarca o noua imagine de executabil
-> modifica spatiul virtual de adrese al procesului, fara a schimba PID-ul acestuia

### Incheierea unui proces
Poate fi normala sau anormala
**Normala** -> se ajunge la sfarsitul codului sau apel exit()
**Anormala** -> actiune invalida si omorata de OS sau procesul este omorat de alt proces
**Rolul procesului parinte:** a se ocupa de colectarea de info legate de incheierea procesului copil
Procesul parinte asteapta dupa procesul copil - wait()
Daca rulam cu & la sfarsit -> nu se asteapta incheierea comenzii
**Proces orfan:** proces al carui proces parinte s-a incheiat
**Proces zombie:** proces care si-a incheiat executia dar nu a fost asteptat de parintele sau

### Procese daemon
- Procese detasate de terminal, stdin, stdout, stderr, de obicei /dev/null
- Procesul parinte - **init**
- Nu sunt interactive - ofera servicii sau realizeaza mentenanta

### Starea proceselor
**Cele 3 stari principale:**
- Proces care ruleaza - **running state**
- Proces care executa operatie I/O se blochgeaza in asteptarea incheierii operatiei: **blocking/waiting state**
- Proces care poate rula dar nu are alocat un procesor - **ready state**

**Tranzitii:**
**running -> ready** - se intampla cand unui proces ii expira cuanta
**running -> blocking** - cand un proces realizeaza o operatie I/O blocanta
**blocking -> ready** - cand operatia I/O blocanta s-a definitivat
**ready -> running** - cand se elibereaza un procesor

##

# Curs 4 - Multitasking

**Proces:** un program caruia i se ataseaza un context de executie
**Schimbare de context:** salvarea informatiilor procesului anterior intr-o zona din OS si restaurarea informatiilor noului proces
Schimbarea de context inseamna **overhead** si se poate produce:
- **voluntar:**
    - un proces decide sau cauzează schimbarea de context
    - un proces cedează de bună voie procesorul: yielding
    - un proces execută o operație blocantă (de exemplu citire de la un dispozitiv care nu are încă date)
un proces își încheie execuția
- **nevoluntar:**
    - planificatorul sistemului de operare decide forțarea unui proces de pe procesor
    - un proces a stat prea mult timp pe procesor
    - apare în sistem un proces mai important
    - demo cu schimbări voluntare și nevoluntare la rularea comenzii find

### Criterii de evaluare a unui planificator
**Metrici:**
    1. throughput (productivitate)
    2. fairness (echitate)
**Planificator mai productiv** -> mai **putin** timp consumat schimband contextul si mai **mult** timp ruland procese
**Planificator mai echitabil** -> fiecare proces **are acces** la procesor -> procesele stau **cat mai putin** timp in coada **READY** pana sa intre in **RUNNING**
**Sistem inechitabil** -> procesul sta **foarte mult** timp in coada **READY** si nu este planificat de procesor: **starvation**
**Sistem neproductiv** -> schimbari de context foarte dese si procesoarele fac foarte putina treaba
**turnaround time:** timpul din care un proces intra in sistem pana cand isi incheie executia
**avg turnaround time:** media turnaround time  pt toate procesele din sistem
**waiting time:** suma timpurilor in care un proces asteapta in starea **READY, NU WAITING**
**avg waiting time:** media waiting time pt toate procesele din sistem
**Sistem productiv** -> avg turnaround time mic
**Sistem echitabil** -> avg waiting time mic

### Cuanta de timp
Cuanta de timp se asociaza unui proces in planificatoare preemptive
**cuanta mare** -> schimbari de context mai rare, **productivitate sporita**
**cuanta mica** -> schimbari de context mai dese, **echitate sporita**

##

# Curs 5 - Gestiunea memoriei

Procesul preia date din memorie sau din I/O -> le prelucreaza -> le stocheaza inapoi in memorie sau I/O
**Ciclu executie:** instruction fetch -> instruction decode -> data fetch -> execution -> data writeback
Memoria este legata de procesor/procesoare prin magistrala bus (**FSB:** Front Side Bus): **magistrala de date** si **magistrala de adrese**

**Citirea din memorie:**
1. procesorul plaseaza adresa (indexul) pe magistrala de adrese
2. trimite mesajul de read
3. asteapta plasarea informatiei pe magistrala de date de unitate de memorie
4. citeste informatia de pe magistrala de date

**Scrierea in memorie:**
1. procesorul plaseaza adresa pe magistrala de adrese si informatia pe magistrala de date
2. trimite mesajul de scriere
3. asteapta ca unitatea de memorie sa ia informatia si sa o scrie in memorie la adresa indicata

### Internele memoriei
Static and dynamic memory
RAM, ROM (PROM, EPROM, EEPROM)
**Latency** refers to the delay before data transfer begins following an instruction. 
**Bandwidth** is the amount of data that can be transmitted in a given time.
**Frequency** indicates how often the memory can perform operations per second.

### Memoria Cache
**Cache memory** is a small, fast memory (often inside the CPU) that stores copies of frequently used data so future requests can be served faster than going to RAM

### Spatiul virtual de adrese al unui proces
**Este unic pentru fiecare proces**
Spatiul virtual ocupa 2^32 octeti (4GiB) pe sistem de 32 biti

### Segmentare vs Paginare si Tabela de Pagini
**Segmentare (Legacy):** compartimentarea spatiului virtual de adrese in zone (precum text, heap, stiva) si asocierea fiecarei zone la spatiul fizic
**Paginare:** compartimentarea spatiului virtual al fiecarui proces si al spatiului fizic al sistemului in componente de dimensiune fixa, numite pagini: **pagini virtuale (pages)** si **pagini fizice (frames)**
**Tabela de pagini:**
A Page Table is a vital part of a computer’s memory management system. It acts like a guide for the CPU, translating the virtual addresses used by programs into actual physical addresses in RAM. 
This ensures that each process can access its own memory safely and efficiently, without interfering with other processes.
1. The operating system (OS) maintains a page table for each process, and the **Memory Management Unit (MMU)** uses it to automatically handle address translation. 
2. The page number from a virtual address acts as an index into the table, and each row—called a **Page Table Entry (PTE)** stores the mapping to physical memory.

### Tabela de pagini ierarhica
Are mai multe niveluri (4-5) pt sistemele pe 64 biti
Daca o zona lipseste, intrarea in page directory este nevalida si nu refera page table
**Avantaj:** spatiu redus
**Dezavantaj:** mai mult overhead de translatare

### Translation Lookaside Buffers (TLB)
Combate dezavantajul overhead-ului de translatare (nevoie de acces la memorie)
este nevoie de TLB flush la address space switch, când se schimbă tabelele de pagini
O operație cu memoria înseamnă un acces la tabela de pagini (în memoria fizică) pentru extragerea mapării și apoi un acces la memoria efectivă pentru extragerea informației (2 accese)
Pentru a reduce overhead-ul, TLB reține cele mai recent accesate intrări în tabela de pagini; este un cache
are 128-256 intrări cu cele mai recente mapări

##

# Curs 6 - Memoria Virtuala

Un proces are un spatiu virtual de adrese propriu\
Adresele virtuale sunt asociate cu adresele fizice prin intermediul **mecanismului de memorie**, la nivel de pagini\
**Zone de memorie virtuala:**
- alocate static (la load time): cod/text, rodata, data, /, biblioteci
- alocate dinamic (la runtime): biblioteci, stiva, heap

**load time:** momentul in care un proces este pornit, lansarea in executie, cnad se foloseste ./a.out in cmd\
**runtime:** momentul in care procesul ruleaza, se afla deja in executie; bibliotecile se incarca dinamic cu apeluri de tip **dlopen(POSIX)** sau **LoadLibrary(Windows)**\

**Memorie partajata:**\
- Doua sau mai multe procese pot partaja pagini de memorie
- Intrarile din fiecare tabela de pagini refera aceeasi pagina fizica
- Paginile virtuale pot diferi

**copy-on-write:**
- imediat după fork() fiecare spațiu virtual (al procesului părinte și al procesului copil) referă același spațiu fizic; nu se alocă spațiu fizic suplimentar (în afară de cel pentru noua tabelă de pagini)
- rezolva problema fork() exec(), unde duplicarea fork() era degeaba deoarece apelul exec() inlocuia tot spatiul virtual si fizic

**swapping:**
- zona din memoria secundara (zona persistena) folosita ca suport de stocare a paginilor
- atunci cand nu exista pagina fizica disponibila, se alege o pagina fizica si se evacueaza in spatiul de swap: **swap out**

**algoritmi de inlocuire de pagini:**
- paginile au un dirty bit care spune ca a fost modificata
- se prefera paginile care au fost cel mai putin recent utilizate
- thrashing (operatii dese de swap out / swap in) care consuma timp

**Maparea fisierelor:**
- scrierea intr-o pagina virtuala conduce la scrierea in blocul corespunzator de pe disc al procesului
- are loc uzual pt fisiere executabile si biblioteci partajate
- **pmap** - arata numele fisierelor
- **lsof** si vizualizarea zonelor de tip txt
- avantaje: overhead scazut temporal si spatial
- dezavantaj: fisierele trebuie sa aiba dimensiunea stiuta pt mapare, nu se poate creste dimensiunea

##

# Curs 7 - Analiza executabilelor si proceselor

### Analiza statica si dinamica
- **Analiza statica:** analiza pe program fara ca acesta sa ruleze
- **Analiza dinamica:** are loc in momentul rularii in proces si are ca tinta principala procesul si resursele folosite de acesta: memorie, registre, fisiere, syscalls, execution flow

- Motive pt folosirea analizei statice/dinamice (3 motive)
    - 1. testare/validare/depanare aplicatie
    - 2. imbunatatirea unei aplicatii: analiza consum resurse, zonelor critice, imbunatatiri latenta, viteza, dimensiune
    - 3. intelegerea functionarii unei aplicatii

**Avantaj analiza statica:** privirea completa a programului\
**Dezavantaj analiza statica:** lipsa de focus, nu se poate construi un flux de executie al programului\
**Avantaj analiza dinamica:** precisa, urmareste un flux de executie cert\
**Dezavantaj analiza dinamica:** nu are o privire in ansamblu si nu poate acoperi toate cazurile\

### Proces de compilare
- compilare program: cod sursa -> limbaj de asamblare
- asamblare program: limbaj de asamblare -> modul / cod obiect
- legare (linking): module obiect + biblioteci -> **executabil**
- incarcarea programului: executabil -> proces

### Interpretare
În cazul interpretării, interpretorul este un executabil existent care este încărcat. În cadrul acestui proces se interpretează codul sursă.

### Fisiere obiect si fisiere executabile
**Fisier obiect:** relocabil, fara main, referinte nerezolvate catre simboluri externe\
**Fisier executabil:** obtinut dupa linking, contine main, are referinte rezolvate, are adrese stabilite
**Asemanari:** fisier obiect/executabil: acelasi format (ELF, PE, Mach-O, COFF): header, date, cod, simboluri

### Legare (Linking)

    Linker-ul combină module obiect + biblioteci → executabil.
    Face:
        comasarea secțiunilor (.text, .data etc.)
        atribuirea adreselor simbolurilor
        rezolvarea referințelor
        stabilirea entry point-ului (_start → main)

**Static vs. Dynamic**

    Static linking: toate bibliotecile și referințele sunt incluse/rezolvate la link time.
            portabil
            pornire mai rapidă
        − executabil mai mare, consum mai mare de memorie
    Dynamic linking: bibliotecile sunt încărcate și referințele rezolvate la load time.
            executabil mai mic
            bibliotecile pot fi partajate între procese
        − pornire mai lentă
        − depinde de existența versiunilor compatibile ale bibliotecilor
    Linux: bibliotecile dinamice = shared objects (.so).

**Analiză statică**

Se face fără rularea programului:

    dezasamblare
    simboluri și adrese
    șiruri
    entry point
    call graph
    biblioteci necesare
    simboluri importate/exportate

Unelte: objdump, readelf, nm, strings, ldd, radare2, IDA, Ghidra.
Loading → Proces

**Loader-ul:**

    mapează executabilul în memoria virtuală
    încarcă .text, .rodata, .data, .bss
    pregătește stiva, argc, argv și variabilele de mediu
    începe execuția de la entry point

La executabile dinamice, încarcă și bibliotecile necesare.

Bibliotecile/zonele read-only (.text, .rodata) pot fi partajate între procese.

Bibliotecile pot fi încărcate și la runtime:

    Linux: dlopen() / dlclose()
    Windows: LoadLibrary() / FreeLibrary()
    util pentru plugin-uri

**Analiză dinamică**

Se face în timpul execuției unui proces:

    breakpoint + stepping
    inspectare/modificare memorie și registre
    urmărirea execuției (tracing)
    inspectarea resurselor
    instrumentare

Unelte:

    Debugger: GDB, LLDB, WinDbg
    Instrumentare: Valgrind, AddressSanitizer, Intel Pin
    Profilare: perf, gcov, Intel VTune
    Tracing: strace, ltrace, ftrace, Dtrace

**Fluxul complet**

Cod sursă → compilator → asamblare → cod mașină → linker → executabil → loader → proces → runtime

Static = analizezi fișierul.
Dynamic = analizezi programul în execuție.

##

# Curs 8 - securitatea memoriei