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

### Securitatea sistemului

    Sistem sigur = funcționează conform specificațiilor.
    Bug = funcționare greșită.
    Vulnerabilitate = bug exploatabil de atacator.
    Exploit = metodă de exploatare a unei vulnerabilități.
    Atacatorul urmărește: furt de informații, DoS sau controlul sistemului.

### Securitatea memoriei

    Se ocupă de protejarea citirii/scrierii datelor și executării codului.
    Atacurile urmăresc de obicei modificarea fluxului de execuție (control flow hijack).
    Vulnerabilități comune:
        Buffer overflow
        Index out of bounds

### Control Flow Graph (CFG)

    Nod = basic block (secvență liniară de instrucțiuni).
    Arc = salt/branch.
    Atacul poate:
        adăuga/modifica arce → code reuse
        adăuga cod nou → code injection

### Zone de memorie

    R-X → cod (.text)
    R-- → date constante (.rodata)
    RW- → .data, .bss, heap, stack
    Executabilul conține .text, .rodata, .data, .bss.
    Atacatorul vizează mai ales zonele RW, pentru suprascrierea datelor/code pointerilor.

### Buffer Overflow

    Buffer = zonă continuă de memorie cu adresă + dimensiune.
    Buffer overflow = scriere peste limita bufferului.
    Out of bounds = acces în afara limitelor (ex. index negativ sau prea mare).
    Pot fi suprascrise date critice, inclusiv adresa de retur.
    Stack buffer overflow → afectează stack-ul; heap buffer overflow → heap-ul.

### Code Pointers

Pointeri care conțin adrese de cod:

    adresa de retur
    function pointers

### Suprascrierea lor poate redirecționa execuția:

    Code injection → se execută cod nou injectat (shellcode).
    Code reuse → se reutilizează cod existent din .text sau biblioteci.
        Return-to-libc
        ROP (Return-Oriented Programming)
        JOP (Jump-Oriented Programming)

### Shellcode

    Cod mașină injectat pentru execuție.
    De obicei combinat cu buffer overflow + suprascrierea unui code pointer.
    Exemplu clasic: deschiderea unui shell.
    Pentru execuție, memoria trebuie să fie writable + executable.

### Protecții principale

    Stack Canary / SSP → valoare între buffer și adresa de retur; detectează suprascrierea.
    Safe Stack → code pointerii sunt separați de datele vulnerabile.
    DEP / NX → zonele RW nu pot fi executate → blochează shellcode-ul.
    ASLR → randomizează adresele heap/stack/biblioteci.
    PIE → permite randomizarea și a zonelor executabilului.
    CFI → verifică respectarea CFG și blochează fluxuri de execuție nepermise.
    AddressSanitizer → detectează erori de memorie; util în dezvoltare/testare, dar cu overhead.

### Ideea de reținut

### Vulnerabilitate → exploit → control flow hijack → code injection / code reuse

### Protecții:
Canary + Safe Stack + DEP/NX + ASLR/PIE + CFI + sanitizers.

##

# Curs 9 - Fire de executie

### Tipuri de acțiuni

    I/O intensive → folosesc frecvent disc, rețea etc.
    CPU intensive → folosesc intens procesorul.
    O acțiune = input → procesare → output.

### Procese vs. Thread-uri
Procese

    Un proces = program în execuție + spațiu de adrese + I/O + thread-uri.
    Procesele pot rula în paralel pe mai multe core-uri.
    Avantaje: izolare între procese.
    Dezavantaje: mai multă memorie, creare și context switch mai costisitoare, partajarea datelor mai dificilă.

### Thread-uri

    Un proces poate avea mai multe thread-uri.
    Thread-ul = unitatea de execuție/abstractizarea procesorului.
    Thread-urile aceluiași proces partajează spațiul de adrese.
    Au proprii:
        registre
        stack
        Thread Local Storage (TLS)
    Avantaje: creare rapidă, overhead mic, partajarea datelor simplă, bune pentru paralelizare.
    Dezavantaje: lipsă de izolare → un thread poate corupe memoria celorlalte; necesită sincronizare; debugging mai dificil.

### Thread: concepte esențiale

    Definit printr-un TCB (Thread Control Block): TID, stare, stack, proces, timp de execuție, prioritate etc.
    Execuția începe de la o funcție și primește o stivă proprie.
    Se termină când:
        funcția se încheie
        procesul se încheie
        se apelează pthread_exit()
    join = așteaptă terminarea thread-ului și recuperează rezultatul.
    Similar: wait() pentru procese.

### Memorie partajată

    Thread-urile aceluiași proces văd aceeași memorie.
    O modificare făcută de un thread este vizibilă celorlalte.
    Stack-ul și TLS-ul sunt proprii fiecărui thread.
    Pentru acces sigur la date comune → sincronizare.

### Implementarea thread-urilor
**Kernel-level threads**

    Gestionate și planificate de kernel.
    Pot rula simultan pe mai multe core-uri.
    Bune pentru CPU-intensive.
    Context switch implică kernel-ul.

**User-level threads**

    Implementate în user-space, fără suport direct din kernel.
    Activare rapidă și overhead mic.
    Bune pentru I/O-intensive.
    Un thread blocat poate bloca întreg procesul → se folosesc I/O asincrone.
    Green threads / fibers = forme de user-level threads.
    Thread pool = reutilizarea thread-urilor pentru reducerea costului de creare.

### De reținut

Proces = izolare + overhead mai mare
Thread = partajare + overhead mic

Proces → spațiu de adrese + I/O + unul sau mai multe thread-uri
Thread → execuție (IP + SP + registre)

Kernel threads → multi-core + CPU intensive
User threads → rapide + I/O intensive

##

# Curs 10 - Sincronizare

### Tipuri de acțiuni

O acțiune presupune date de intrare → prelucrare → date de ieșire.

    I/O intensive – folosesc predominant discul, rețeaua etc.
    CPU intensive – folosesc predominant procesorul.

### Procese

Un proces este o abstractizare pentru executarea unei acțiuni și este planificat pe un procesor.

Pe sisteme multi-core/multi-procesor putem:

    executa mai multe acțiuni diferite în paralel;
    executa aceeași acțiune în paralel, folosind mai multe procese.

Dezavantaje ale proceselor:

    consum mai mare de memorie;
    partajarea datelor este mai dificilă;
    crearea și schimbarea între procese au overhead mai mare.

### Thread-uri

Un thread (fir de execuție) este o variantă „lightweight” a procesului. Un proces poate avea unul sau mai multe thread-uri.

Thread-ul reprezintă în principal contextul de execuție al procesorului:

    instruction pointer;
    stack pointer;
    registre;
    stivă proprie.

Thread-urile din același proces partajează spațiul de adrese, ceea ce face comunicarea între ele rapidă și simplă.

**Avantaje:**

    creare mai rapidă;
    consum redus de memorie;
    schimbare de context mai rapidă;
    partajarea datelor este simplă;
    pot folosi mai multe core-uri.

**Dezavantaje:**

    izolare redusă: o eroare a unui thread poate afecta întreg procesul;
    necesită sincronizare pentru accesul la date comune;
    programele multithreaded sunt mai greu de depanat.

### Memoria thread-urilor

Thread-urile aceluiași proces partajează întreg spațiul de adrese.

Fiecare thread are însă propria:

    stivă (stack);
    zonă TLS (Thread Local Storage);
    stare și registre.

Pentru acces corect la datele comune sunt necesare primitive de sincronizare.
### Crearea și terminarea

La crearea unui thread se specifică funcția pe care o va executa.

Un thread se poate termina:

    când se termină funcția sa;
    când se termină procesul;
    prin pthread_exit().

Pentru a aștepta terminarea unui thread se folosește join, similar cu wait pentru procese.
### Kernel-level vs. User-level threads
Kernel-level	User-level\
Suportate de sistemul de operare	Implementate în user space\
Sunt planificate de kernel	Planificatorul este în user space\
Pot rula simultan pe mai multe core-uri	Nu necesită suport în kernel\
Potrivit pentru CPU intensive	Activare foarte rapidă, bune pentru I/O intensive\
Schimbarea de context implică kernel-ul	Schimbarea poate fi mai rapidă

##

# Curs 11 - Dispozitive de intrare/iesire

### Resursele unui sistem de calcul

Cele 3 resurse principale sunt:

    Procesorul (CPU) – prelucrarea datelor.
    Memoria – păstrarea datelor și codului.
    I/O – comunicarea cu exteriorul și cu alte procese.

I/O poate însemna tastatură, mouse, monitor, disc, rețea, senzori, actuatori etc.
### Cum funcționează I/O la nivel hardware

Comunicarea urmează în general:

CPU → magistrală → controller → dispozitiv

    Controller-ul este cipul care controlează dispozitivul și conține registre pentru date, comenzi și stare.
    CPU comunică cu aceste registre prin:
        Memory-mapped I/O – registrele sunt mapate în spațiul de adrese fizice.
        Port-mapped I/O – există un spațiu separat de adrese pentru I/O.
    Memory-mapped I/O este metoda folosită cel mai frecvent.

### Polling vs. întreruperi

CPU trebuie să știe când dispozitivul este pregătit.

    Polling – CPU verifică permanent starea controller-ului.
        simplu, dar consumă CPU;
        util la trafic foarte mare.
    Întreruperi (interrupts) – controller-ul notifică CPU când are nevoie de atenție.
        mai eficient pentru CPU;
        preferat de obicei la trafic redus/normal.

O întrerupere declanșează o ISR (Interrupt Service Routine) din kernel.
### DMA

DMA (Direct Memory Access) permite transferul unor blocuri mari de date între dispozitiv și memoria principală fără ca CPU să copieze fiecare dată.

→ Scade încărcarea procesorului și crește performanța I/O.
### Device drivers

Un device driver este componenta kernelului care face legătura dintre sistemul de operare și dispozitiv.

Primește:

    cereri de la aplicații prin system calls (read(), write() etc.);
    întreruperi de la hardware.

Driverul traduce operațiile generale ale OS în comenzi specifice dispozitivului.
### Niveluri intermediare

Între aplicație și driver pot exista componente precum:

    networking stack – gestionează protocoale precum IP/TCP;
    filesystem – gestionează fișierele;
    block I/O layer – gestionează accesul la disc.

Buffer cache păstrează în RAM datele de pe disc pentru a evita accesul lent la disc.

La citire se poate folosi read-ahead: sistemul aduce dinainte date care probabil vor fi folosite.
### Interfața pentru aplicații

Aplicațiile folosesc de obicei file descriptors pentru I/O.

**Operațiile principale sunt:**

    open() – deschide/resursează un descriptor;
    read() – citește;
    write() – scrie;
    close() – închide;
    seek() – schimbă poziția în dispozitive cu acces aleator;
    ioctl() – operații specifice dispozitivului.

În Linux, dispozitivele sunt de obicei în /dev/ și sunt identificate prin major + minor.
### I/O și performanța

**Operații sincrone și blocante (read/write)**

    thread-ul așteaptă până când operația poate fi realizată;
    simplu de programat, dar poate reduce performanța.

**Operații non-blocante**

    operația se întoarce imediat cu ceea ce este disponibil;
    aplicația poate încerca din nou ulterior.

**Operații asincrone**

    aplicația trimite cererea și continuă să lucreze;
    OS notifică ulterior finalizarea;
    performanță bună, dar programare mai complicată.

Pentru multe conexiuni/cereri se folosesc API-uri scalabile precum epoll (Linux).
### Tehnici pentru reducerea overhead-ului

    Scatter/Gather I/O – mai multe buffere sunt procesate printr-un singur system call.
    Zero-copy – datele sunt transferate fără copii inutile între kernel și user space.

##

# Curs 12 - Implementarea sistemelor de fisiere

### FCB / inode

FCB (File Control Block) = metadata unui fișier. În sistemele Unix/Linux se numește de obicei inode.

Conține:

    identificatorul (inode number / ino);
    permisiuni și proprietar;
    timestamp-uri;
    tipul fișierului;
    dimensiunea;
    pointeri către blocurile de date.

**Important: FCB/inode nu conține numele fișierului.**
### Dentry și hard link-uri

Dentry (directory entry) asociază:
nume fișier → inode

Pot exista mai multe dentry-uri care indică același inode. Acestea sunt hard link-uri.

    Hard link = dentry
    Fișierul = inode + blocurile sale de date
    rm / unlink() șterge un dentry, nu direct inode-ul.
    Fișierul este eliminat efectiv când nu mai există hard link-uri către inode.

### Directoare

Un director este tot un fișier, dar blocurile sale de date conțin dentry-uri.

Fiecare director are:

    . → referință către el însuși;
    .. → referință către directorul părinte.

Un director cu N subdirectoare are în general N + 2 link-uri.

Ierarhia sistemului de fișiere este construită prin urmărirea dentry-urilor pornind de la directorul rădăcină /.
### Symbolic link

Un symbolic link (symlink) este un inode al cărui conținut este o cale către alt fișier.

    Symlink = inode care conține o cale
    Hard link = dentry care indică un inode
    Un symlink poate fi dangling dacă calea indicată nu mai există.
    Symlink-urile pot indica fișiere de pe alte sisteme de fișiere; hard link-urile nu pot traversa sisteme de fișiere diferite.

### Alte tipuri de fișiere

Pe lângă fișierele obișnuite și directoare există:

    symbolic links;
    UNIX sockets;
    FIFO/named pipes;
    character devices;
    block devices.

### Gestionarea spațiului

Sistemul de fișiere ține evidența:

    inode-urilor ocupate/libere;
    blocurilor de date ocupate/libere.

De obicei folosește bitmap-uri:

    0 = liber;
    1 = ocupat.

Când se creează un fișier:

    se găsește un inode liber;
    este marcat ca ocupat;
    se completează inode-ul;
    se alocă blocuri de date când este nevoie.

### Structura pe disc

Un sistem de fișiere conține în general:

Superblock → inode map → data map → inode-uri → blocuri de date

    Superblock – descrie structura sistemului de fișiere.
    Inode map – evidențiază inode-urile ocupate.
    Data map – evidențiază blocurile ocupate.
    Inode-uri – metadata + pointeri către date.
    Data blocks – conținutul efectiv al fișierelor.

### Formatare și montare

    Formatarea creează structura sistemului de fișiere și inode-ul rădăcină.
    Montarea leagă inode-ul rădăcină al unui sistem de fișiere de un director din alt sistem de fișiere.
    Sistemul de fișiere principal este montat în /.
    Demontarea face sistemul de fișiere inaccesibil prin acel punct de montare.

După o oprire bruscă pot apărea inconsistențe. fsck verifică și repară aceste probleme.
### Descriptorul de fișier

În lanțul de acces:

file descriptor → structura fișierului deschis → FCB/inode → blocurile de date

Descriptorul este un index în tabela de descriptori a procesului.

##

# Curs 13 - Networking in OS

ChatGPT said:
Rezumat – Interacțiunea hardware–OS și networking
### NIC și comunicarea cu OS

O placă de rețea (NIC) comunică cu sistemul de operare prin perechi de ring buffers RX/TX.

    RX = primirea pachetelor.
    TX = transmiterea pachetelor.
    Comunicarea folosește memory-mapped I/O, întreruperi/polling și DMA.
    DMA permite transferul pachetelor între NIC și memoria RAM fără ca CPU să copieze fiecare byte.

La inițializare, OS alocă memoria pentru pachete și ring-uri și transmite NIC-ului adresele acestora.
### Recepția unui pachet (RX)

Fluxul esențial:

NIC primește pachet → RX ring → DMA → memorie → notifică OS → stiva de networking

Dacă nu există slot liber în RX ring, pachetul este drop-uit.

OS este notificat prin:

    întrerupere, la trafic normal;
    polling, de obicei la trafic foarte mare.

### Transmiterea unui pachet (TX)

Fluxul este:

Aplicație → kernel → IP/network stack → TX ring → DMA → NIC → rețea

NIC copiază datele din memoria sistemului în bufferul său intern și apoi transmite pachetul.

Placa poate face hardware offload, de exemplu:

    calcularea checksum-ului;
    TSO (TCP Segmentation Offload) – spargerea unui pachet mare în pachete mai mici;
    calcularea FCS.

### Mai multe cozi RX/TX

NIC-urile moderne au mai multe perechi de cozi RX/TX.

La recepție:

    NIC folosește de obicei un hash pentru a decide coada RX;
    cozile pot fi asociate unor core-uri diferite.

→ Scopul principal este distribuirea traficului pe mai multe core-uri și creșterea performanței.
### Procesarea în OS

Când OS primește un pachet, verifică adresa IP destinație:

    dacă este pentru sistemul local → intră în host/network stack;
    dacă nu este local și există rutare → poate fi forwardat;
    altfel → este drop-uit.

Pentru un pachet local, OS trebuie să găsească socketul căruia îi aparține.

Identificarea se face în principal folosind 5-tuple:

IP sursă + port sursă + IP destinație + port destinație + protocol
### UDP

Pentru UDP:

pachet → socket UDP → receive queue → recvfrom()

Fiecare socket UDP are o coadă de pachete primite.

    recvfrom() ia următorul pachet din coadă.
    Dacă nu există pachete, procesul poate fi blocat până când apare unul.

### TCP

TCP are două tipuri importante de socket-uri:

    listener – așteaptă conexiuni;
    connection socket – reprezintă o conexiune TCP efectivă.

Fluxul este:

socket() → bind() → listen() → accept()

Clientul folosește connect().

La o conexiune nouă:

    clientul trimite SYN;
    serverul răspunde SYN/ACK;
    clientul trimite ACK;
    conexiunea devine disponibilă pentru accept().

accept() returnează un nou socket de conexiune, în timp ce socketul listener continuă să accepte alte conexiuni.
### TCP receive/send buffers

Pentru fiecare conexiune TCP există buffere în kernel:

    receive buffer – date primite de la rețea;
    send buffer – date trimise de aplicație către kernel.

recv() citește din receive buffer.

Important: TCP livrează datele în ordine. Dacă există o „gaură” cauzată de un pachet pierdut, datele următoare nu sunt livrate aplicației până când pachetul lipsă nu este retransmis.

send() nu înseamnă că datele au ajuns la destinație. Înseamnă că datele au fost copiate cu succes în bufferul TCP al kernelului.
### UDP vs. TCP

**UDP:**

    sendto() → produce un pachet;
    recvfrom() → primește un pachet;
    fiecare pachet implică în general un system call.

**TCP:**

    send() copiază datele în send buffer;
    kernelul decide când și cum le grupează în segmente;
    recv() copiază datele din receive buffer;
    TCP gestionează automat retransmisia, ordinea etc.

### Performanță și offloading

System call-urile sunt costisitoare, mai ales când se fac pentru fiecare pachet.

Pentru TCP putem trimite/recepționa mulți bytes per system call, reducând overhead-ul.

Problema este că rețeaua lucrează cu pachete, de obicei de aproximativ 1500 B, iar procesarea per pachet este costisitoare.

De aceea se folosesc:

    TSO – placa de rețea sparge segmente TCP mari în pachete.
    GSO – kernelul face această segmentare.
    LRO – combină mai multe pachete primite pentru a reduce numărul de operații.

→ Toate reduc munca stivei de networking și cresc performanța.
### Servere cu multe conexiuni

Există trei modele principale:

    1 proces / conexiune
        izolare bună;
        overhead mare.

    1 thread / conexiune
        mai eficient decât procesele;
        multe thread-uri înseamnă totuși overhead și context switches.

    1 thread/proces pentru mai multe conexiuni
        folosește epoll pentru a detecta socket-urile pregătite pentru I/O;
        poate folosi mai multe thread-uri pentru toate core-urile;
        este mai eficient și scalabil;
        programarea este mai complicată deoarece trebuie păstrată o stare pentru fiecare client.

##

# Curs 14 - Analiza performantei

### Ce vrem să aflăm?

Pentru un server, principalele criterii sunt:

    Latenta – cât durează să răspundă la o cerere.
    Throughput – câte cereri poate procesa pe secundă.

Performanța depinde de:

    numărul de clienți;
    dimensiunea fișierelor;
    distribuția cererilor;
    hardware-ul serverului.

### Identificarea bottleneck-ului

Înainte de teste trebuie să estimăm ce componentă limitează performanța:

    Storage (HDD) – cel mai lent; ~1–4 Gbps citire secvențială.
    Rețeaua – 10 Gbps.
    CPU – depinde de numărul de core-uri și de operațiile efectuate.

Pentru serverul analizat:

    ~60.000 fișiere;
    dimensiune medie ~1,7 MB;
    maxim 7 MB;
    minim 32 KB.

### Timpul de citire de pe HDD

Timpul poate fi aproximat:

**T = To + Ts + Size / Speed**

unde:

    To = overhead-ul constant;
    Ts = timpul de seek al HDD-ului;
    Size / Speed = timpul efectiv de citire.

Cu hdparm se poate măsura viteza de citire.

Pentru HDD-ul analizat:

    viteza ≈ 200 MB/s;
    seek ≈ 5 ms.

Consecința importantă: dacă fiecare client cere un fișier diferit, seek-ul HDD-ului limitează puternic numărul de cereri.
### HDD vs. buffer cache

Dacă datele nu sunt în cache:

    seek ≈ 5 ms;
    un fișier mediu → ~11 ms;
    aproximativ 100 cereri/s.

Dacă datele sunt deja în buffer cache:

    accesul este mult mai rapid (~2 ms pentru exemplul analizat);
    HDD-ul nu mai este bottleneck;
    devin importante CPU-ul, syscall-urile și rețeaua.

Ideea esențială:

Cache → CPU/rețea devin bottleneck
Fără cache → HDD poate fi bottleneck
### Rețeaua ca bottleneck

Dacă serverul poate servi suficient de repede din cache, limita devine conexiunea de 10 Gbps.

De exemplu, pentru fișiere mici, CPU-ul poate ajunge bottleneck înaintea rețelei; pentru fișiere mai mari, rețeaua poate deveni limita.
### Testarea serverului

Se compară implementări precum:

    sendfile-server;
    threaded-server.

Testarea se face:

    serial – un singur download la un moment dat;
    parallel – mai mulți clienți simultan.

Testarea serială poate fi înșelătoare: dacă clientul este bottleneck, diferențele dintre servere nu sunt relevante.

La download paralel se poate observa mai bine capacitatea serverului.
### Variabilitatea rezultatelor

Când crește numărul de clienți:

    throughput-ul crește;
    la un moment dat rezultatele devin variabile.

O cauză este contensionarea între procesele client și thread-urile serverului.

Se poate folosi taskset pentru a pune serverul și clienții pe core-uri diferite.

Dar această soluție poate modifica experimentul, deoarece nu mai testează utilizarea tuturor core-urilor.
### Metodologia corectă de testare

**Ordinea importantă este:**

    Ce vreau să măsor?
        latență;
        throughput.

    La ce nivel de încărcare?
        câți clienți;
        ce dimensiuni de fișiere;
        ce distribuție a cererilor.

    Aleg setup-ul
        aceeași mașină;
        rețea;
        tipul de rețea relevant pentru scenariu.

    Aleg workload generator-ul
        open;
        closed;
        partly open.

    Estimez bottleneck-ul înainte de experiment.

    Rulez experimentul
        schimb un singur parametru;
        măsor rezultatul;
        repet pentru ceilalți parametri importanți.

    Vizualizez și verific rezultatele
        dacă sunt credibile → experimentul este terminat;
        dacă sunt suspecte → verific setup-ul sau metodologia și repet testul.