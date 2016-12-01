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

Ciclo di vita:
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
- **Stato:** lo stato del gioco, rapresentato da un led è salvato in una memoria da 3 bits che conterrà:
    - *000* se il giocatore 1 sta selezionando la casella da attaccare
    - *001* se il giocatore 1 ha cliccato 'attacca' e il circuito sta processando la mossa
    - *011* se il giocatore 1 ha completato il posizionamento delle navi
    - *010* se il giocatore 2 sta selezionando la casella da attaccare
    - *011* se il giocatore 2 ha cliccato 'attacca' e il circuito sta processando la mossa
    - *011* se il giocatore 2 ha completato il posizionamento delle navi
- **Contatore Navi:** questi contatori, uno per giocatore, hanno un molteplice utilizzo: quello di tenere traccia delle sezioni di nave che sono state posizionate dai giocatori durante la fase di preparazione, e quello di tenere traccia delle sezioni di nave ancora integre per giocatore. Questo ci permette di reagire al completamento della fase di preparazione e alla vitoria di un giocatore, nonche di visualizzare lo score di ogni giocatore.


## Componenti
Ogni componente di questo pregetto, fatta eccezzione per le porte logiche e led, è stato realizzato personalmente.

Componenti base:
- **Flip-Flop D**
- **Selettore** da 2 vie da 6 bit
- **Selettore** da 64 vie da 2 bit

Sottocomponenti:
- **RGB led**
- **Led Driver**

Componenti:
- **Keypad** (x2)
- **Display** (x2)
- **Memoria** (x1)
- **Unita centrale** (x1)
- **Controllore di stato** (x2)























# cosiderazioni finali (da approfondire)
- ho deciso di utilizzare un indirizzamenot che lasciare dei bichi in memoria perche evitarlo avvrebbe portato a complicare inutilmente il curcuito
- utilizzare un componente proprio per lo stato invece di metterlo nel attak keeper
- scelta dei 3 bit di stato da 2 e prima da 1