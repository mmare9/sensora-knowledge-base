Esatto. Questo costituiva il **banco prova controllato** per validare il rilevatore ottico basato su A‑D Strips prima dell’integrazione definitiva con MQTT.

## 4. Ambiente di test con campana isolata

L’architettura definita era sostanzialmente questa:

```text
┌──────────────── Campana di prova ────────────────┐
│                                                  │
│  Punto di rilascio                               │
│  dell’acido acetico                              │
│          │                                       │
│          ▼                                       │
│      [Sorgente]               [Ventola]          │
│                                  │               │
│                                  ▼               │
│                    miscelazione dell’atmosfera   │
│                                                  │
│        [A-D Strip + rilevatore ottico]           │
│        [Sensore temperatura/umidità]             │
│                                                  │
└──────────────────────────────────────────────────┘
                         │
                         ▼
               Edge device / MCU
                         │
                         ▼
                    MQTT Broker
```

### Funzione dei componenti

**Campana di isolamento**

* separa il volume di prova dall’ambiente esterno;
* limita le interferenze dovute a correnti d’aria e contaminanti;
* permette di lavorare con un volume interno noto;
* rende confrontabili prove successive, purché tenuta, geometria e temperatura rimangano costanti.

**Ventola**

* riduce i gradienti locali di concentrazione;
* accelera l’omogeneizzazione dell’atmosfera interna;
* evita che il sensore misuri solo la concentrazione presente nel punto in cui è installato;
* deve funzionare sempre alla stessa velocità e nella stessa posizione.

**Punto di rilascio dell’acido acetico**

* introduce il composto nella campana;
* deve essere separato fisicamente dal sensore e non trovarsi direttamente davanti alla ventola;
* deve consentire un rilascio il più possibile ripetibile.

**Rilevatore ottico**

* contiene la A‑D Strip;
* acquisisce il colore a intervalli prefissati;
* calcola RGB normalizzato, coordinate Lab, ΔE, livello A‑D e velocità di variazione;
* invia i risultati al sistema di acquisizione.

**Temperatura e umidità**

Devono essere registrate in ogni misura perché influenzano sia l’evaporazione dell’acido sia la risposta temporale della strip. Non le userei inizialmente per “correggere” matematicamente il dato: prima devono essere trattate come variabili sperimentali osservate.

## Correzione critica al punto di rilascio

Un contenitore aperto con acido acetico non costituisce una sorgente controllata.

La velocità di evaporazione dipenderebbe da:

* superficie esposta;
* temperatura del liquido;
* velocità dell’aria sopra il liquido;
* posizione rispetto alla ventola;
* quantità residua;
* assorbimento e condensazione sulle pareti.

In particolare, se la ventola investe direttamente la sorgente, svolge contemporaneamente due funzioni:

1. mescola l’aria;
2. aumenta l’evaporazione.

A quel punto non sarebbe possibile capire se una risposta più rapida della strip dipenda da una migliore miscelazione o da una quantità maggiore di acido rilasciata.

### Configurazione più ripetibile

Per l’MVP userei:

* un piccolo contenitore chiuso dentro la campana;
* un setto o un sistema di apertura controllata;
* introduzione di un’aliquota nota;
* ventola già in funzione prima del rilascio;
* sorgente schermata dal flusso diretto della ventola;
* stessa posizione geometrica in tutte le prove.

Non serve inizialmente un dosatore sofisticato. Serve soprattutto che la procedura di introduzione sia identica da una prova all’altra.

## Sequenza di una prova

### 1. Preparazione

* pulizia e aerazione della campana;
* installazione di una A‑D Strip nuova;
* verifica del riferimento cromatico;
* registrazione di identificativo e lotto della strip;
* controllo della posizione di sensore, ventola e sorgente;
* acquisizione di temperatura e umidità iniziali.

### 2. Baseline

Con ventola accesa e senza acido:

* acquisire il colore iniziale;
* verificare la stabilità dell’illuminazione;
* misurare il rumore del sensore;
* controllare che ΔE non mostri una deriva significativa.

Questa fase permette di separare il cambiamento reale della strip dalla deriva di LED, elettronica o temperatura.

### 3. Rilascio

Al tempo `t0`:

* attivare il rilascio;
* registrare l’evento nel sistema;
* mantenere costanti ventola, illuminazione e frequenza di campionamento.

Il messaggio MQTT dovrebbe contenere anche l’evento di rilascio:

```json
{
  "timestamp": "2026-07-16T10:00:00Z",
  "event": "acetic_acid_release",
  "test_id": "TEST-ACOH-001",
  "dose_id": "DOSE-LOW-01"
}
```

### 4. Esposizione

Durante la prova vengono registrati:

```text
timestamp
test_id
strip_id
raw_rgb
normalized_rgb
Lab
delta_e_from_baseline
estimated_ad_level
temperature_c
relative_humidity_pct
fan_state
elapsed_time
quality_flags
```

La variabile principale da osservare è:

```text
ΔE(t)
```

Da questa curva si possono ricavare:

* variazione complessiva;
* pendenza iniziale;
* tempo di superamento di una soglia;
* velocità massima di cambiamento;
* stabilizzazione finale;
* ripetibilità tra prove equivalenti.

### 5. Fine prova

* interrompere l’esposizione;
* salvare il risultato finale;
* rimuovere la strip;
* evacuare in sicurezza l’atmosfera della campana;
* riportare il sistema alle condizioni iniziali.

La stessa strip non dovrebbe essere utilizzata per una seconda prova, perché la risposta cromatica è cumulativa e non è ragionevole aspettarsi un ritorno affidabile al colore iniziale.

## Matrice minima di validazione

Prima di variare umidità, ventilazione o geometria, congelerei tutte le condizioni e proverei soltanto quattro livelli:

| Serie  | Rilascio nominale | Ripetizioni | Obiettivo                           |
| ------ | ----------------: | ----------: | ----------------------------------- |
| Blank  |           Nessuno |           3 | Misurare deriva e falso cambiamento |
| Low    |             Basso |           3 | Verificare la sensibilità           |
| Medium |        Intermedio |           3 | Valutare andamento e separabilità   |
| High   |              Alto |           3 | Verificare saturazione e dinamica   |

Totale: **12 prove**, ciascuna con una strip nuova.

Non modificherei contemporaneamente velocità della ventola, dose, temperatura e umidità. Cambiare più parametri insieme impedirebbe di attribuire il risultato a una causa precisa.

## Concentrazione nominale e concentrazione reale

Con volume della campana noto e quantità introdotta nota è possibile stimare una concentrazione teorica:

[
C_{nominale} =
\frac{nRT}{PV}\times 10^6
]

dove:

* `n` è la quantità di acido in moli;
* `T` è la temperatura assoluta;
* `P` è la pressione;
* `V` è il volume libero della campana.

Questo valore deve però essere chiamato **concentrazione nominale teorica**, non concentrazione misurata. Presuppone infatti:

* evaporazione completa;
* assenza di perdite;
* assenza di adsorbimento sulle pareti;
* miscelazione uniforme;
* comportamento ideale del gas.

La campana, le guarnizioni, il supporto del sensore e gli altri materiali potrebbero trattenere una parte dell’acido.

Per correlare realmente ΔE a ppm servirebbe un metodo indipendente di misura dell’acido acetico. Non utilizzerei automaticamente un PID generico come riferimento: OSHA riporta per l’acido acetico un potenziale di ionizzazione di circa 10,66 eV, quindi un comune PID con lampada da 10,6 eV potrebbe risultare inadatto o poco affidabile per questa applicazione. OSHA dispone invece di metodi di campionamento e analisi specifici, con interferenze che devono comunque essere considerate. ([osha.gov][1])

## MQTT per il banco prova

Separerei i dati sperimentali dagli eventi:

```text
lab/acetic-acid/testbench/telemetry
lab/acetic-acid/testbench/event
lab/acetic-acid/testbench/status
lab/acetic-acid/testbench/alarm
```

Esempio di telemetria:

```json
{
  "timestamp": "2026-07-16T10:22:00Z",
  "test_id": "TEST-ACOH-001",
  "elapsed_s": 1320,
  "strip_id": "STRIP-001",
  "rgb_raw": [82, 116, 134],
  "lab": [46.8, -11.2, -8.6],
  "delta_e": 7.4,
  "ad_level_estimated": 0.83,
  "temperature_c": 22.1,
  "relative_humidity_pct": 47.6,
  "fan": {
    "state": true,
    "speed_pct": 50
  },
  "source_state": "released",
  "quality_flags": []
}
```

Il broker non deve essere il punto in cui viene calcolato il dato principale. La lettura normalizzata e ΔE dovrebbero essere prodotti localmente dal dispositivo; MQTT dovrebbe trasportare sia il risultato sia abbastanza dati grezzi da permettere una verifica successiva.

## Sicurezza del banco prova

La campana di isolamento **non equivale a una cappa chimica**. La campana contiene temporaneamente il volume di prova; una cappa chimica deve invece aspirare e gestire i vapori, soprattutto durante caricamento, apertura e spurgo. L’ISS descrive la cappa come dispositivo di protezione collettiva destinato all’aspirazione e filtrazione o espulsione di vapori e fumi. ([ISS][2])

L’acido acetico può provocare irritazione e, a concentrazioni elevate, ustioni a occhi, pelle e vie respiratorie. Il banco deve quindi essere impiegato secondo la scheda dati di sicurezza della concentrazione utilizzata, con contenimento secondario e una procedura controllata di evacuazione; la campana non dovrebbe essere aperta direttamente nel locale dopo la prova. ([Centri per il Controllo Malattie][3])

## Decisione

**La scelta migliore ora è: congelare il banco prova con volume noto, ventola a regime fisso e rilascio mediante aliquota controllata, eseguendo prima 12 prove di ripetibilità senza cercare ancora una conversione in ppm.**

**Cosa guadagni:** dati confrontabili, possibilità di misurare la vera ripetibilità del rilevatore e identificazione della soglia minima cromaticamente distinguibile.

**Cosa sacrifichi:** nella prima fase otterrai livelli relativi Low/Medium/High, non una calibrazione metrologica della concentrazione.

**Cosa rischi:** che la risposta sia ripetibile ma dipenda più dal tempo, dall’umidità o dall’assorbimento della campana che dalla sola quantità di acido.

**Prossimo passo entro 24 ore:** scrivere una scheda prova unica che fissi volume libero della campana, geometria, velocità della ventola, modalità di rilascio, durata, cadenza di acquisizione, identificativo della strip e criterio di accettazione tra le tre ripetizioni.

👉 **Ecco la mia analisi onesta e il punto cieco che potresti non aver notato:**

* La ventola può essere contemporaneamente miscelatore e acceleratore dell’evaporazione: la sorgente deve essere schermata dal flusso diretto.
* La concentrazione calcolata dalla quantità introdotta è solo nominale; pareti, guarnizioni e supporti possono sottrarre acido alla fase gassosa.
* Prima di calibrare la strip occorre qualificare la campana: tenuta, omogeneità dell’atmosfera e memoria chimica tra una prova e la successiva.
* MQTT non è il rischio tecnico principale; il rischio è produrre curve digitali molto precise da un ambiente di prova non ripetibile.

[1]: https://www.osha.gov/chemicaldata/469?utm_source=chatgpt.com "ACETIC ACID† - Occupational Safety and Health Administration"
[2]: https://www.iss.it/documents/20126/1446946/5_Rischio_Chimico.pdf/51c229d7-c90a-679d-a172-91e105ddd7b6?download=true&t=1576452634299&version=1.2&utm_source=chatgpt.com "RISCHIO CHIMICO- DIAPOSITIVE_versione_ufficiale.ppt [modalità ..."
[3]: https://www.cdc.gov/niosh/npg/npgd0002.html?utm_source=chatgpt.com "CDC - NIOSH Pocket Guide to Chemical Hazards - Acetic acid"
