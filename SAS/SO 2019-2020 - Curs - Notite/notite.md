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

