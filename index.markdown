---
layout: default
---

<h2 id="intro">Introduzione</h2>
Piccolo tutorial sulla programmazione assembly per il processore [MOS 6502](https://en.wikipedia.org/wiki/MOS_Technology_6502)

Questo tutorial è un adattamento a cura di Diego Cortassa del tutorial originale di Nick Morgan pubblicato su [easy6502](http://skilldrick.github.io/easy6502/)

Si utilizza un simulatore di 6502 scritto in Javascript.

- La memoria emulata va da `$0000` a `$ffff` (l'intero spazio indirizzabile da un 6502)

- La locazione di memoria `$fe` contiene un numero casuale che cambia ad ogni istruzione del processore.

- La locazione di memoria `$ff` contiene il codice ascii dell'ultimo tasto premuto.

- La zona di memoria dalla locazione `$200` alla locazione `$5ff` e mappata sui pixel dello schermo simulato. Inserendo differenti valori si disegneranno pixel di diverso colore (questa è una semplificazione di un display reale). Il display è composto da 33 (`$21`) righe di 31 (`$1f`) colonne.

I colori utilizzabili sono:

    $0: Nero
    $1: Bianco
    $2: Rosso
    $3: Ciano
    $4: Viola
    $5: Verde
    $6: Blu
    $7: Giallo
    $8: Arancione
    $9: Marrone
    $a: Rosso chiaro
    $b: Grigio scuro
    $c: Grigio
    $d: Verde chiaro
    $e: Blue chiaro
    $f: Grigio chiaro


<h2 id="first-program">Il primo programma</h2>


Clicca su **Assemble** e poi **Run** per assemblare ed eseguire il frammento di codice assembly.

{% include start.html %}
LDA #$01
STA $0200
LDA #$05
STA $0201
LDA #$08
STA $0202
{% include end.html %}

Sullo schermo simulato nero appariranno tre "pixel" colorati in alto a sinistra.

Cosa fa realmente questo programma? E' possibile eseguirlo passo per passo utilizzando il debugger. Premi **Reset** e attiva la spunta **Debugger** per far partire il debugger. Premi **Step** per avanzare un istruzione alla volta. noterai che  `A=` è cambiato da `$00` a `$01`, ae `PC=` è cambiato da `$0600` a`$0602`.

In assembly 6502 qualsiasi numero preceduto da  `$` è da intendersi in formato esadecimale (hex). Inoltre qualsiasi valore preceduto da `#` rappresenta un valore esplicito. Qualsiasi altro numero si riferisce ad una locazione in memoria.

In base a queste informazioni si può intuire che l'istruzione `LDA #$01` carica (load) il valore hex `$01` nel registro `A`. 

Premi **Step** di nuovo per eseguire la seconda istruzione. Il pixel in alto a sinistra del display del simulatore è diventato bianco. L'istruzione  `STA $0200` salva il valore del registro `A` nella locazione di memoria  `$0200`. Premi **Step** altre quattro volte per eseguire il resto delle istruzioni tenendo d'occhio il registro `A` .

###Esercizi###

1. Cambia il colore dei tre pixel.
2. Cambia un pixel nell'angolo in fondo a destra (memory location `$05ff`).
3. Aggiungi altre istruzioni per disegnare altri pixel.


<h2 id='registers'>Registri e flag</h2>

Sulla prima riga del simulatore vengono mostrato i registri `A`, `X` e `Y`  (`A` viene di solito chiamato "accumulatore"). Ogni registro può contenere un singolo byte. La maggior parte delle operazioni avvengono sul contenuto di questi registri.

`SP` è lo stack pointer. Viene decrementato ogni volta che un byte viene inserito nello stack (push) e incrementato quando un byte viene prelevato dallo stack (push).

`PC` è il program counter - Il modo col quale il processore tiene traccia del punto in cui si trova nell'esecuzione del programma. Nel simulatore il codice viene assemblato partendo dalla locazione di memoria `$0600`,  e il  program counter parte sempre da tale valore.

L'ultimo registro contiene i flag del processore. Ogni flag è  un bit. I flag vengono settati dal processore per dare informazioni sull'esecuzione della precedente istruzione.

<h2 id='instructions'>Istruzioni</h2>

Le istruzioni sono un set di piccoli comandi o funzioni predefiniti.
Tutte le istruzioni accettano zero o un argomento. Ecco alcuni esempi di istruzioni con una piccola spiegazione:

{% include start.html %}
LDA #$c0  ;Load the hex value $c0 into the A register
TAX       ;Transfer the value in the A register to X
INX       ;Increment the value in the X register
ADC #$c4  ;Add the hex value $c4 to the A register
BRK       ;Break - we're done
{% include end.html %}

Assembla il codice, attiva il debugger e osserva i registri `A` e `X`   durante l'esecuzione per step. Alla linea `ADC #$c4` succede qualcosa di strano.
Ci si aspetterebbe che somma di `$c4` e `$c0` sia `$184`, ma il processore risponde `$84`. Come mai?

`$184` è un numero troppo alto per essere contenuto in un byte (il massimo è `$FF`) e il registro può contenere un solo byte ma il processore ha rilevato il problema settando a `1`   il  carry flag (flag di riporto) dopo l'operazione.

Un altro test:

{% include start.html %}
LDA #$80
STA $01
ADC $01
{% include end.html %}

Nota la distinzione fra l'istruzione `ADC #$01` che aggiunge `$01` al registro `A` mentre `ADC $01` aggiunge il valore contenuto all'indirizzo di memoria `$01` al registro `A`.

Assembla, attiva la spunta **Monitor**, ed esegui le tre istruzioni. Il monitor mostra una sezione della memoria. `STA $01` salva il valore del registro `A` nella locazione di memoria `$01`, e `ADC $01` aggiunge il valore memorizzato nella locazione di memoria `$01` al registro `A`. `$80 + $80` dovrebbe risultare `$100`, ma, di nuovo, essendo un numero troppo grande per un byte, il registro `A` viene riempito col valore `$00` e il carry flag (flag di riporto) viene settato a 1. Inoltre lo zero flag viene settato da tutte le istruzioni che hanno un risultato di zero.

 [Qui](http://www.6502.org/tutorials/6502opcodes.html) e [qui](http://www.obelisk.me.uk/6502/reference.html) sono disponibili due liste dell'intero set di istruzioni del 6502. 

###Esercizi###

1. Hai imparato ad usare l'istruzione `TAX`. Poi probabilmente immaginare cosa fanno le istruzioni `TAY`, `TXA` e `TYA`, scrivi del codice per testare il loro funzionamento.
2. Riscrivi il primo esempio di questa sezione utilizzando il registro `Y` al posto di `X`.
3. L'istruzione opposta di `ADC` è `SBC` (subtract with carry, sottrai con riporto). Scrivi un programma che usi questa istruzione.


<h2 id='branching'>Branching</h2>

Fino ad ora abbiamo scritto codice lineare senza alcuna logica di branching.

L'assembly 6502 possiede una serie di istruzioni di di branching che eseguono un salto in base al valore di alcuni flag. Nel prossimo esempio diamo uno sguardo a  `BNE`: "Branch on not equal".

{% include start.html %}
  LDX #$08
decrement:
  DEX
  STX $0200
  CPX #$03
  BNE decrement
  STX $0201
  BRK
{% include end.html %}

Innanzitutto carichiamo il valore  `$08` nel registro `X`.
La riga successiva è una label (etichetta). Le label identificano dei punti nel programma ai quali possiamo saltare .
Dopo la label decrementiamo `X` e salviamo il suo valore in `$0200` (il pixel in alto a sinistra), poi lo confrontiamo con il valore `$03`.
[`CPX`](http://www.obelisk.me.uk/6502/reference.html#CPX) confronta il valore del registro `X`  con un altro valore. Se sono uguali il flag `Z` viene impostato ad `1`, altrimenti a `0`.

La linea successiva, `BNE decrement`, salta alla label `decrement` se il valore del flag `Z` è settato a `0` (cioè se i due valori confrontati da `CPX` non erano uguali), altrimenti il programma continua e salva il contenuto di `X` in `$0201` (il secondo pixel).
in pseudo codice le ultime due istruzioni significano:
`if X == 3 goto decrement`

Di solito, in assembly, con le istruzioni di branch si usano le label ma quando il programma viene assemblato queste vengono convertite in byte che rappresentano lo scostamento (offset), il numero di locazioni in memoria di cui retrocedere o procedere per eseguire la prossima istruzione questo significa che ci si può muovere solo in un intervallo di 256 bytes, se volessimo saltare più lontano dobbiamo usare una istruzione si jump.

###Esercizi###

1. L'opposto di `BNE` è `BEQ`. Scrivi un programma che usi `BEQ`.
2. `BCC` e `BCS` ("branch on carry clear" and "branch on carry set") vengono usate per saltare in base al flag di riporto . Scrivi un programma che usi una di queste istruzioni.


<h2 id='stack'>Lo stack</h2>

Lo stack (pila) del 6502 funziona come un stack classico, i valori possono venir depositati in cima alla pila (push) o prelevati dalla cima della stessa (pop o pull in gergo 6502). La profondità corrente dello stack viene misurata dal registro stack pointer. Lo stack viene memorizzato fra `$0100` and `$01ff`. Lo stack pointer è inizializzato a `$ff`, e punta quindi alla locazione di memoria `$01ff`. Quando un byte viene depositato sulla pila (push) lo stack pointer viene decrementato a `$fe`, puntando alla locazione `$01fe`, e così via per i successivi byte.

`PHA` and `PLA` sono due istruzioni per l'uso dello stack, "push accumulator" e "pull accumulator". Ecco un esempio di utilizzo di queste due istruzioni:

{% include start.html %}
  LDX #$00
  LDY #$00
firstloop:
  TXA
  STA $0200,Y
  PHA
  INX
  INY
  CPY #$10
  BNE firstloop ;loop until Y is $10
secondloop:
  PLA
  STA $0200,Y
  INY
  CPY #$20      ;loop until Y is $20
  BNE secondloop
{% include end.html %}

`X` memorizza il colore del pixel e `Y` la posizione pixel corrente.
Il primo ciclo disegna un pixel del colore corrente utilizzando il registro `A`, poi deposita (push) il colore sullo stack quindi incrementa colore e posizione.
Il secondo ciclo preleva i valori dallo stack (pop), disegna un pixel del colore ottenuto e infine incrementa la posizione.
Il risultato è un pattern speculare.


<h2 id='jumping'>Salti</h2>

Il salto (jump) è simile al branching ma differisce per due ragioni. Prima, i salti non sono eseguiti condizionalmente. Seconda accettano un come argomento un indirizzo assoluto in memoria lungo 2 byte. Il secondo dettaglio non è molto importante per piccoli programmi ma per programmi di grandi dimensioni il salto è l'unica soluzione per spostarsi da una sezione all'altra del codice.

###JMP###

`JMP` esegue un salto incondizionato. Ecco un esempio di utilizzo:

{% include start.html %}
  LDA #$03
  JMP there
  BRK
  BRK
  BRK
there:
  STA $0200
{% include end.html %}


###JSR/RTS###

`JSR` and `RTS` ("jump to subroutine" e "return from subroutine") sono utilizzate insieme. `JSR` salta dalla locazione attuale ad un altro punto del codice. `RTS` torna alla posizione precedente. E' simile ala chiamata e ritorno da una funzione (in basic GOSUB e RETURN).

Il processore sa dove tornare perché `JSR` deposita l'indirizzo della locazione della prossima istruzione meno uno sullo stack prima di saltare alla locazione richiesta mentre `RTS` preleva la locazione aggiunge uno e salta al risultato ottenuto.
Ecco un esempio:

{% include start.html %}
  JSR init
  JSR loop
  JSR end

init:
  LDX #$00
  RTS

loop:
  INX
  CPX #$05
  BNE loop
  RTS

end:
  BRK
{% include end.html %}

La prima istruzione salta alla label `init`, qui viene settato `X` a `$00` poi ritorna all'istruzione successiva, `JSR loop` che salta alla label `loop`, dove `X` viene incrementato fino a che diventa `$05`, si ritorna quindi alla prossima istruzione, `JSR end`, che salta alla fine del programma.
Questo esempio illustra come sia possibile scrivere codice modulare usando `JSR` and `RTS`.


<h2 id='constants'>Costanti (o simboli)</h2>


In assembly si possono definire delle costanti (o simboli) che rappresentano dei numeri cono dei nomi descrittivi. Nel resto del codice si potranno usare tali costanti invece dei numeri rendendo più leggibile il codice.
Nei nomi dei simboli si possono usare lettere, numero o underscore (trattino basso).

Ecco un esempio. Nota che per gli operandi letterali occorre comunque usare il prefisso `#`.
{% include start.html %}
  define  sysRandom  $fe ; an address
  define  a_dozen    $0c ; a constant
  define red $2  ; the color red
 
  LDA sysRandom  ; equivalent to "LDA $fe"

  LDX #a_dozen   ; equivalent to "LDX #$0c"

  LDA #red
  STA $021f  ; plot a red pixel
{% include end.html %}

<h2 id='snake'>Snake</h2>

In ultimo ecco un esempio di un programma complesso, il gioco snake:
{% include snake.html %}



