Ho recuperato **due filoni di conversazione chiaramente pertinenti**, entrambi del 2025. Non risultano altre chat specifiche sull’acido acetico nel contesto storico disponibile.

## 1. Monitoraggio dell’acido acetico nelle pellicole in acetato di cellulosa

**Periodo:** luglio 2025
**Contesto:** degrado delle pellicole in acetato di cellulosa, “vinegar syndrome” e necessità di individuare precocemente l’emissione di acido acetico in contenitori o ambienti chiusi.

### Tecnologie considerate

| Soluzione                             | Utilizzo discusso                                    | Punti di forza                                                        | Limiti                                                                    |
| ------------------------------------- | ---------------------------------------------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| **BIOS1 – ossido di tungsteno, WO₃**  | Sensore MOS sperimentale per acido acetico/VOC       | Sensibilità indicata fino a circa **30 ppb**, risposta rapida         | Selettività, deriva e disponibilità commerciale da verificare             |
| **BIOS2 – ossido di stagno, SnO₂**    | Alternativa MOS a BIOS1                              | Sensibilità indicata fino a circa **30 ppb**                          | Come sopra: risposta influenzata da umidità, temperatura e altri VOC      |
| **Strisce A-D**                       | Screening passivo dell’acidità delle pellicole       | Semplici, economiche, molto diffuse in conservazione                  | Risposta lenta, circa **24 ore**; soglia indicativa intorno a **500 ppb** |
| **Figaro TGS2602**                    | Rilevamento generico di VOC e contaminanti dell’aria | Commerciale, integrabile, relativamente economico                     | Non specifico per l’acido acetico                                         |
| **MiCS-5524 / MICS-VZ-87**            | Monitoraggio VOC                                     | Piccoli e adatti a sistemi embedded                                   | Richiedono calibrazione applicativa                                       |
| **Aeroqual SM50**                     | Modulo di misura ambientale                          | Soluzione più pronta all’uso                                          | Costo e selettività da valutare                                           |
| **AlphaSense PID-AH**                 | Misura di VOC tramite fotoionizzazione               | Sensibile e veloce                                                    | Misura il carico VOC complessivo, non distingue necessariamente CH₃COOH   |
| **AlphaSense MOX-LC**                 | Sensore MOS a basse concentrazioni                   | Potenzialmente adatto a monitoraggio continuo                         | Deriva e interferenze tipiche dei MOS                                     |
| **PEM2 IPI o datalogger equivalenti** | Temperatura e umidità relativa                       | Necessari per interpretare sia il degrado sia la risposta dei sensori | Non misurano direttamente l’acido acetico                                 |

### Conclusioni emerse

I sensori **MOS** risultavano la soluzione più promettente per ottenere un allarme rapido e seguire l’andamento nel tempo. Nelle discussioni veniva citato un tempo di risposta dell’ordine di **15 minuti**, contro le circa **24 ore** delle strisce A-D.

Le **strisce A-D** restavano però utili come metodo semplice di screening e come riscontro qualitativo, soprattutto quando non serve una misura continua.

I sensori commerciali analizzati erano prevalentemente dispositivi per **VOC generici**. Il problema principale non era quindi la sensibilità, ma la **selettività verso l’acido acetico**: solventi, formaldeide, acetone, alcool, prodotti lignei e materiali dell’armadio potrebbero produrre segnali simili.

Non risultava presa una decisione definitiva su un singolo produttore o modello.

---

## 2. Armadi BlockFire e assorbimento dell’acido acetico

**Periodo:** agosto 2025
**Contesto:** prove sperimentali per verificare l’efficacia di assorbitori all’interno di armadi destinati alla conservazione.

### Risultati discussi

Erano state considerate **tre prove**, con differenti quantità di acido acetico, condizioni di ventilazione e tempi di osservazione fino a circa 24–30 ore.

Il confronto principale era tra:

* **Armadio A**, dotato di assorbitore;
* **Armadio B**, senza assorbitore.

L’assorbitore mostrava una riduzione più rapida della concentrazione, soprattutto dopo carichi elevati, fino a un valore teorico iniziale dell’ordine di **200 ppmV**.

Anche l’armadio privo di assorbitore mostrava però una progressiva diminuzione. Dopo circa 24 ore i valori tendevano verso un livello residuo indicato intorno a **20 ppmV**.

### Interpretazione

La riduzione non poteva essere attribuita esclusivamente all’assorbitore. Erano stati identificati almeno tre fenomeni concomitanti:

1. azione dell’assorbitore sperimentale;
2. perdita di acido dovuta alla non perfetta ermeticità dell’armadio;
3. assorbimento da parte dell’isolante e dei materiali costruttivi.

L’isolante conteneva componenti a base di **carbonato e silicato di calcio**. Il carbonato di calcio può reagire con l’acido acetico formando **acetato di calcio**, sottraendo quindi acido alla fase gassosa.

Questo significava che una diminuzione misurata dal sensore non dimostrava automaticamente l’efficacia del solo assorbitore: parte dell’acido poteva essere trattenuta dalle pareti o dispersa all’esterno.

Era stata anche considerata, come ipotesi da verificare, una possibile efficacia dell’assorbitore verso altri VOC, come formaldeide e acetone.

---

## Sintesi trasversale delle conversazioni

Il problema è stato affrontato come un sistema composto da tre elementi:

**Sorgente → ambiente/materiali → sensore**

Le pellicole emettono acido acetico; armadio, isolante, ventilazione e assorbitore modificano la concentrazione; il sensore misura infine ciò che rimane nella fase gassosa.

Per questo motivo, la sola sensibilità nominale del sensore non è sufficiente. I criteri realmente importanti emersi sono:

* selettività verso CH₃COOH;
* tempo di risposta;
* deriva nel lungo periodo;
* dipendenza da temperatura e umidità;
* interferenza degli altri VOC;
* capacità di recuperare dopo un’esposizione elevata;
* correlazione tra segnale e concentrazione reale;
* posizione del sensore all’interno dell’armadio;
* interazione dell’acido con pareti, isolanti e assorbitori.

## Questioni rimaste aperte

Non risulta definita una procedura completa per:

* calibrare il sensore con concentrazioni note di acido acetico;
* compensare temperatura e umidità;
* verificare formaldeide, acetone, alcool e altri interferenti;
* confrontare MOS, PID e strisce A-D sulla stessa prova;
* separare l’effetto dell’assorbitore da perdite e assorbimento delle pareti;
* stabilire soglie operative di preallarme e allarme;
* selezionare definitivamente un modello commerciale.

## Decisione operativa

**La scelta migliore ora è: utilizzare un sensore MOS continuo come rilevatore di trend e allarme, affiancato da strisce A-D periodiche e da uno strumento di riferimento durante la fase di validazione.**

Non sceglierei ancora un unico sensore come strumento quantitativo autonomo. I dati discussi dimostrano che il segnale dipende troppo da interferenze, umidità, deriva e comportamento dei materiali dell’armadio.

**Cosa guadagni:** risposta rapida, storico continuo, possibilità di individuare variazioni e confrontare armadi o assorbitori.

**Cosa sacrifichi:** semplicità e immediatezza; serve una fase iniziale di caratterizzazione.

**Cosa rischi:** scambiare un aumento di VOC generici per acido acetico oppure interpretare come “rimozione” quella che è in realtà dispersione o assorbimento temporaneo nelle pareti.

**Prossimo passo entro 24 ore:** trasformare le prove già svolte in una matrice unica con almeno controllo vuoto, armadio senza assorbitore e armadio con assorbitore, registrando contemporaneamente MOS, temperatura, umidità e una misura di riferimento.

👉 **Ecco la mia analisi onesta e il punto cieco che potresti non aver notato:**

* Il collo di bottiglia non è trovare un sensore abbastanza sensibile: è dimostrare che il suo segnale rappresenti davvero l’acido acetico.
* Il valore residuo di circa 20 ppmV potrebbe essere un equilibrio del sistema o un limite sperimentale, non necessariamente il “fondo” reale.
* L’isolante può apparire utile come assorbitore, ma potrebbe saturarsi o rilasciare nuovamente sostanze in condizioni ambientali diverse.
* Senza un riferimento indipendente, confrontare due sensori MOS rischia di produrre due misure coerenti tra loro ma entrambe non quantitative.
