# Progetto d'esame - Battaglia navale
## Carlo Vassallo - matricola: 892613
### Architettura degli elaboratori I - anno 2016/17
<br>

## Specifiche del progetto
Il progetto ha come obbiettivo quello di simulare, in logisim, un circuito che permetta a due utenti di giocare a battaglia navale.

Input/output:
- **Display:** ogni giocatore potrà visualizzare lo stato delle sue navi sul proprio display che mostra la griglia di gioco di dimenzioni 4x6.
- **Keypad:** una tastiera composta da 11 tasti, contenenti le lettere dalla 'A' alla 'D', i numeri dall '1' al '5', e un tasto 'attacca'. Ogni tasto, eccetto quello di invio, ha sopra di se un led che indica la sua selezione. Il tasto 'attacca' effettua la mossa del giocatore "tirando una bomba" nella casella del giocatore avversario selezionata attraverso gli altri tasti.
- **Led di stato:**: al lato di ogni display è posto, uno che indica lo stato del gioco:
    
    - *led blu:* se siamo nella fase di preparazione al gioco e il giocatore sta posizionando le proprie navi
    - *led verde:* se la partita è avviata ed è il proprio turno di gioco
    - *led rosso:* se la partita è avviata ma è il turno dell'avversario

Ciclo di gioco:
1. **Preparazione al gioco:** in questo stato i giocatori posizioneranno le navi a disposizione (per un totale di 6 caselle riempite) selezionando la posizione di destinazione e premendo invio per ogni singola casella da occupare. Questo stato termina automaticamente quando ogni giocatore ha finito il posizionamento.
2. **Turno giocatore:** è il turno del primo giocatore. Quest'ultimo deve selezionare la casella da colpire attraverso il keypad e cliccare 'attacca'. L'attacco avra effetto sulla griglia dell'avversario che potrà visulaizzare sul proprio display se è stato colpito o meno. Al click del bottone 'attacca' il turno vinene automaticamente passato all'avversario. Questo stato termina quando uno dei due giocatori vince.
3. **Fine del gioco:** uno dei giocatori ha vinto e la partita è terminata.

Memorizzazione:
- **Griglie dei giocatori:** ogniuna delle due griglie è salvata in una memoria indirizzabile composta da 48 celle da 2 bit ciascuna. Gli indirizzi sono da 6bit di cui:
    - *1° bit* rappresenta il giocatore; 0 per il 1°, 1 per il 2°
    - *2°, 3° bits* raprresentano la riga sulla griglia
    - *4°, 5°, 6° bits* raprresentano la collonna sulla griglia.

    Ogni cella puo contentere:
    - *00* in presenza di acqua
    - *01* in presenza di un segmento di nave integro
    - *10* in presenza di un segmento di nave colpita
- **Stato:** lo stato del gioco, rapresentato da un led è salvato in una memoria da 3 bits con il seguente formato:
    - *1° bit* rappresenta la fase di gioco; 0 posizionamento delle navi, 1 partita.  
    - *2° bit* turno del giocatore; 0 per il primo giocatore, 1 per il secondo.
    - *3° bit* stato della mossa, 0 se in attesa, 1 se è stata inviata.
- **Contatore Navi:** questi contatori, uno per giocatore, hanno un molteplice utilizzo: quello di tenere traccia delle sezioni di nave che sono state posizionate dai giocatori durante la fase di preparazione, e quello di tenere traccia delle sezioni di nave ancora integre per giocatore. Questo ci permette di reagire al completamento della fase di preparazione e alla vitoria di un giocatore, nonche di visualizzare lo score di ogni giocatore.


## Componenti
Ogni componente di questo progetto, fatta eccezzione per le porte logiche e i led, è stato realizzato personalmente.

Componenti base:
- **Flip-Flop D**
- **Selettore** da 2 vie da 6 bit
- **Selettore** da 64 vie da 2 bit

Componenti:
- **Keypad** (x2)
- **Display** (x2)
- **Memoria** (x1)
- **Unita centrale** (x1)
- **Controllore di stato** (x2)

Principio di progettazine:

Ogni componente è stato sviluppato in funzione dello stato. Infatti lo stato definisce come e quando i componenti devono agire. Di base essi si limitano a fornire in output il valore che hanno ricevuto in input durante gli stati di attesa (3° bit di stato a 0) e solo quando lo stato muta in stato di "attacco" (3° bit a 1) l'output sarà un nuovo valore.

### Display
![Display Appearance](./screenshot/display.outer.png)
![Display inner](./screenshot/display.driver.inner.png)
Questo componente ha il compito di mostrare ai giocatori lo stato delle proprie navi ed è composto da:
- **96 led:** in gruppi da quatto che rappresentano le celle della griglia di gioco. Ogni cella puo assumere la colorazione interamente blu, se contiene acqua, interamente grigia se contiene un segmento di nave, metà grigia e meta blu a formare una croce se rappresenta un segmento di nave che è stato colpito. Per ottenere 2 colori è stato settato come colore del led spento il colore celeste e grigio per il led acceso. La tensione su ogni led è gestita dal driver del display.
- **Display Driver:** questo componente, che possiamo immaginare come lo chassis del display, ha il compito di accendere a dovere ogni led in modo da rispecchiare la griglia di gioco salvata in memoria. Questo driver ha 4 input da 12bit ogniuno, ovvero la rappresentazione binaria di ogni riga della griglia di gioco. In oltre presenta 96 pin di output posizionati a dovere in modo da potervici posizionare sopra i led, immaginando che siano saldati sopra. Internamente questo componente si limita a leggere dagli input il valore che ogni cella (ovvero ogni gruppo di 4 led) deve visulaizzare, e accendere a dovere i led. Come da specifiche in memoria ogni cella è rappresentata su 2 bit come:
    - *00* acqua
    - *01* nave
    - *10* nave colpita

    Per fare ciò è stato creato un circuito cosi composto:

    | i1 | i2 |   | A | B | C | D |
    |----|----|---|---|---|---|---|
    | 0  | 0  |   | 0 | 0 | 0 | 0 |
    | 0  | 1  |   | 1 | 1 | 1 | 1 |
    | 1  | 0  |   | 1 | 0 | 0 | 1 |
    | 1  | 1  |   | x | x | x | x |

    - B = C = i2
    - A = D = ( !i1 AND i1 ) OR ( i1 AND !i2 ) = i1 XOR i2

![Display cell circuit](./screenshot/display.driver.inner.cell.png)

#### Considerazione
Durante la progettazione di questo display ho incontrate delle difficolta puramente strumentali. Infatti logism non mette a disposizione dei led che possono assumere piu colori, cosa che nel mio caso sarebbero seviti per rappresentare i vari stati delle navi. Dopo delle ricerche ho trovato una soluzione che prevedeva la sovrapposizione di led con colori semi trasparenti in modo da sovrappore piu colori e attraverso le possibili combinazioni formare gli svariati colori. tuttavia logism non permette la sovrapposizione di led, infatti la soluzione sopracitata prevedeva lo sfruttamento di quello che sembra essere un bug. Dovendo scartare questa opzione ho pensato di utilizzare quattro led per ogni cella come sopra descritto. A questo punto pero collegare i led alla memoria utilizzando dei fili non avrebbe reso graficamente in quanto la distanza fra le celle sarebbe stata troppa e non avrei ottenuto un "simil-diplay", motivo per cui ho creato il driver con gli input sulla parte superiore del componente, invece che sui lati come è solito fare.
    
### Memory
![Memory Appearance](./screenshot/memory.outer.png)
![Memory inner](./screenshot/memory.inner.png)
Questo componente ha il compito di memorizzare i dati del gioco come le due griglie dei giocatori, il conto delle navi posizionate e rimanenti, e lo stato della partita. La struttura del circuito è molto simile ad un register file di una cpu.
- **Griglie di gioco:** le due griglie di gioco sono memorizzate in una struttura ispirata al register file; infatti la scrittura avviene specificando l'indirizzo di registro e il suo valore e analogamente la lettura avviene specificando il registro da leggere. Per fare cio sono stati creati 64 registri da 2 bit, 1 decoder da 64 vie a 2 bit e 1 multiplexer da 64 vie a 2 bit. Il piano di indirizzamento e cosi formato: il 1° bit, più significativi, indica il giocatore, il 2° e 3° indicano la riga sulla griglia, il 4°, 5°, 6° indicano la colonna sulla griglia. Essendo 6 le righe e 3 i bit utilizzati per rappresentarle, per ogni riga rimangono 2 registri inutilizzati per un totale di 16 registi superflui che sono stati riportati nello screenshot ma che sono stati rimossi durante la fase di rifinitura finale. I dati qui memorizzati devono essere rappresentati sul display, motivo per cui sono state create delle uscite dedicate.
- **Stato della partita:** lo stato della partita è memorizzato in un registro da 3 bit appositamente creato, esso è accessibile in scrittura e lettura attraverso un ingresso e un uscita dedicate.
- **Conto delle navi:** Il contatore delle navi di ogni giocatore è memorizzato in 2 dedicati registri da 3 bit che sono accessibili in scrittura attraverso, un ingresso di selezione per specificare su quale giocatore agire, e un ingresso ove immettere il nuovo valore da memorizzare. Per le operazioni di lettura invece sono presenti 2 uscite dedicate per ogni contatore

#### Considerazioni
La progettazione di questo componente non è stata particolarmente difficoltosa in quanto è per la maggiorparte ispirata al registerfile, le difficolta riscontrate sono state piu che altro relative alla costruzione fisica inquanto sia per il multiplexer e il decoder, sia per il componente in se è stato necessario collegare molti fili, operazione delicata che spesso ha portato a degli errori involtari e di conseguenza ad una fase di "debugging".

### AttackProcessor
![AttackProcessor Appearance](./screenshot/attack.processor.outer.png)
![AttackProcessor inner](./screenshot/attack.processor.inner.png)

Questo componente è adibito al processing dell attacco. Come da principio durante gli stati di attesa loutput equivarrà a l'input. Durante la fase di attacco questo componente distingue due casi:
- *Stato di preparazione:* in questo stato (1° bit di stato a 0) l'AttackProcessor fornirà in output 01 (valore che rappresenta una sezione di nave) se il valore in input è 00 (valore che rappresenta l'acqua) in modo da permettere il posizionamento delle navi.
- *Stato di gioco:* in questo stato (1° di stato a 1) l'AttackProcessor fornirà in output il valore 10 (valore che rappresenta una sezione di nave affondata) se il valore in input è 01 (sezione di nave) in modo da permettere l'affondamento delle navi.

| i1 | i0 | s2 | s1 |   | o1 | o0 |
|----|----|----|----|---|----|----|
| 0  | 0  | 0  | 0  |   | 0  | 0  |
| 0  | 0  | 0  | 1  |   | 0  | 1  |
| 0  | 0  | 1  | 0  |   | 0  | 0  |
| 0  | 0  | 1  | 1  |   | 0  | 0  |
| 0  | 1  | 0  | 0  |   | 0  | 1  |
| 0  | 1  | 0  | 1  |   | 0  | 1  |
| 0  | 1  | 1  | 0  |   | 0  | 1  |
| 0  | 1  | 1  | 1  |   | 1  | 0  |
| 1  | 0  | 0  | 0  |   | 1  | 0  |
| 1  | 0  | 0  | 1  |   | 1  | 0  |
| 1  | 0  | 1  | 0  |   | 1  | 0  |
| 1  | 0  | 1  | 1  |   | 1  | 0  |
| 1  | 1  | 0  | 0  |   | x  | x  |
| 1  | 1  | 0  | 1  |   | x  | x  |
| 1  | 1  | 1  | 0  |   | x  | x  |
| 1  | 1  | 1  | 1  |   | x  | x  |


|    | 00 | 01 | 11 | 00 |
|----|----|----|----|--- |
| 00 | 0  | 0  | 0  | 0  |
| 01 | 0  | 0  | *1*  | 0  |
| 11 | x  | x  | x  | x  |
| 01 | *1*  | *1*  | *1*  | *1*  |

- o1 = (i0 AND s2 AND s0) OR (i1 AND i0)

|    | 00 | 01 | 11 | 00 |
|----|----|----|----|--- |
| 00 | 0  | *1*  | 0  | 0  |
| 01 | *1*  | *1*  | 0  | *1*  |
| 11 | x  | x  | x  | x  |
| 01 | 0  | 0  | 0  | 0  |

- o0 = (!i1 AND !s2 AND s0) OR (!i1 AND i0 AND !s0) =    
    = !i0 AND ( ( !s2 AND s0 ) OR ( i0 AND !s0 ) )

#### Considerazioni
Lo sviluppo di questo componente, una volda definito il principio di progettazione e quindi il comportamento dell'AttackProcessor è stato banale.

### ShipCountManager
![ShipCountManager Appearance](./screenshot/ship.count.manager.outer.png)
![ShipCountManager inner](./screenshot/ship.count.manager.inner.png)

Lo ShipCountManager è admibito alla gestione del contatore delle navi di ogni giocatore. Il suo compito è quello di incrementare o sottrarre al corretto contatore in base alla fase di gioco. 
Per lo scopo è stato realizzato un sommatore a 4 bit. Il concetto di base è quello di sommare zero in stato di attesa mentre sommare +1 o -1 in base alla situazione; per fare cio, essendo  i contatori a 3 bit e dovendovi sommarci un valore negativo, internamente espandiamo il valore a 4 bit in modo da poter utilizzare il -1 (1111) in complento a 2.
Il circuito è diviso in 2 parti:
- *Selezione dell'operazione* (blu) che seleziona +1 in stato di preparazione alla partita e -1 in stato di gioco, in modo tale da incrementare il contatore ogni volta che un giocatore posiziona una nave e decrementarlo ogni volta che una nave viene affondata 
- *Selezione del contatore:* (rosso) che in base allo stato della partita e al giocatore corrente seleziona l'appropriato contatore, overro il medesimo del giocatore se siamo nella fase di preparazione, o il contatore opposto al giocatore che detiene il turno se siamo in fase di partita.
Il cicuito prende quindi in input lo stato e l'output del AttackProcessor in modo tale da essere consapevole dell'effetto della mossa corrente e agire di conseguenza

Selezione dell'operazione

| s2 | s1 | o1 | o0 |   | A3 | A2 | A1 | A0 |    |
|----|----|----|----|---|----|----|----|----|----|
| 0  | 1  | 0  | 1  |   | 0  | 0  | 0  | 1  | +1 |
| 1  | 1  | 1  | 0  |   | 1  | 1  | 1  | 1  | -1 |

- A3 = A2 = A1 = s1 AND s2 AND o1 AND !o0
- A0 = (!s2 AND s1 AND !O1 AND o1) OR (s2 AND s1 AND o1 AND o0) =

     = s1((!s2 AND !O1 AND o1) OR (s2 AND o1 AND o0))

     = s1 AND (o1 XOR o0) AND (!S2 + S2)

     = s1 AND (o1 XOR o0)

Selezione del constatore
Mettendo in XOR i primi 2 bit dello stato otteniamo il giocatore su cui agire, questa informazione viene utilizzita per selezionare uno dei due input riguardanti i contatori attraverso lo stesso meccanismo del multiplexer.

### ShipCountManager
![AttackKeeper Appearance](./screenshot/attack.keeper.outer.png)
![AttackKeeper inner](./screenshot/attack.keeper.inner.png)

Questo componente ha lo scopo di selezionare la cella di memoria su cui scrivere il valore dato in output dall'AttackProcessor.
Attraverso lo stesso meccanismo del multiplexer selezioniamo l'input della mossa del giocatore corrente e la portiamo sugli ultimi 5 bit in output.
Analogamente allo ShipCountManager utilizziamo una XOR fra turno e stato di gioco per ottenere il giocatore su cui attuare la mossa, valore che portiamo in output sulbit piu sognificativo.
cosi facendo otteniamo l'indirizzo di memoria corrispondente alla cella della griglia da attaccare.





# cosiderazioni finali (da approfondire)
- ho deciso di utilizzare un indirizzamenot che lasciare dei bichi in memoria perche evitarlo avvrebbe portato a complicare inutilmente il curcuito
- utilizzare un componente proprio per lo stato invece di metterlo nel attak keeper
- scelta dei 3 bit di stato da 2 e prima da 1