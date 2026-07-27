Hai ragione: **questa parte mancava nella sintesi precedente** ed è probabilmente il filone più concreto dal punto di vista progettuale.

## 3. Rilevatore ottico basato su A‑D Strips e MQTT

L’idea successiva non era misurare direttamente la concentrazione istantanea di acido acetico con un sensore gas, ma **digitalizzare e automatizzare la lettura delle A‑D Strips**.

Le A‑D Strips cambiano progressivamente colore dal blu al verde fino al giallo in presenza dei vapori acidi prodotti dal degrado dell’acetato. La lettura manuale prevista dal produttore avviene confrontando la strip con una scala cromatica di cinque riferimenti, associati ai livelli da 0 a 3. ([filmcare.org][1])

### Obiettivo del sistema

Il rilevatore avrebbe dovuto:

1. illuminare la A‑D Strip in condizioni ottiche ripetibili;
2. rilevarne periodicamente il colore;
3. convertire il colore in un valore numerico;
4. calcolare il cambiamento rispetto alla condizione iniziale;
5. individuare la velocità di variazione;
6. associare il risultato a una classe di allarme;
7. trasmettere misura e diagnostica a un broker MQTT.

La finalità principale non era quindi produrre direttamente una misura in **ppm di CH₃COOH**, ma ottenere un **indice digitale dell’acidità e della sua evoluzione**.

## Correzione terminologica

Chiamerei il dispositivo:

> **Rilevatore ottico della variazione dell’acidità mediante A‑D Strips**

piuttosto che “rilevatore di varianza”.

La **varianza**, in senso statistico, misura la dispersione di una serie di dati. Può essere una delle metriche calcolate dal sistema, ma ciò che volevamo principalmente rilevare era:

* il livello cromatico;
* la variazione rispetto al baseline;
* il gradiente temporale, cioè `Δcolore/Δtempo`;
* eventualmente la varianza del segnale in una finestra temporale, per identificare instabilità o anomalie.

## Architettura ipotizzata

```text
A-D Strip
    ↓
Camera ottica schermata
    ├── illuminazione controllata
    ├── sensore RGB / multispettrale / camera
    ├── riferimento cromatico
    └── sensore temperatura e umidità
            ↓
Microcontrollore / Edge Node
    ├── normalizzazione del colore
    ├── conversione RGB → CIELAB
    ├── calcolo ΔE
    ├── classificazione livello A-D
    ├── calcolo trend e allarmi
    └── diagnostica della misura
            ↓
MQTT Broker
    ↓
Node-RED / database storico / dashboard / allarmi
```

### Parte ottica

Erano state considerate genericamente fotocellule, fotoresistenze, fotodiodi, sensori di riflettanza o una piccola camera. La soluzione tecnicamente più adatta, però, è un **sensore RGB o multispettrale**, perché il fenomeno da misurare non è una semplice variazione di luminosità ma uno spostamento cromatico tra blu, verde e giallo.

La testa ottica dovrebbe comprendere:

* camera chiusa alla luce esterna;
* illuminazione LED controllata;
* distanza e angolo fissi;
* supporto meccanico ripetibile per la strip;
* riferimento bianco o cromatico permanente;
* misura di temperatura e umidità relativa;
* identificativo della strip e possibilmente del lotto.

La compensazione della luce ambientale non dovrebbe essere affidata solo al software: **la luce ambientale va fisicamente eliminata** con una camera ottica chiusa.

## Elaborazione del dato

Il processo previsto può essere formalizzato così:

### 1. Acquisizione

Rilevazione di:

* valori grezzi R, G e B;
* intensità luminosa;
* temperatura;
* umidità relativa;
* tempo di esposizione;
* identificativo dispositivo;
* identificativo strip.

### 2. Normalizzazione

Il colore misurato viene corretto rispetto a:

* riferimento cromatico interno;
* intensità dei LED;
* dark reading del sensore;
* eventuale deriva della sorgente luminosa;
* temperatura della camera ottica.

### 3. Conversione cromatica

È preferibile convertire il dato da RGB a **CIELAB**, perché lo spazio Lab consente di calcolare in modo più significativo la distanza tra due colori.

La metrica principale diventerebbe:

```text
ΔE = distanza cromatica tra la strip corrente e il riferimento iniziale
```

### 4. Classificazione

Il valore misurato viene confrontato con i cinque riferimenti della scheda A‑D:

```text
Livello 0
Livello 1
Livello 1,5
Livello 2
Livello 3
```

La classificazione non dovrebbe essere soltanto discreta. Il dispositivo potrebbe pubblicare anche un valore interpolato, per esempio:

```text
A-D level = 1,37
```

Questo valore avrebbe utilità come **indice strumentale**, ma non dovrebbe essere presentato come livello ufficiale senza una validazione sperimentale.

### 5. Analisi temporale

Oltre al livello assoluto, il sistema potrebbe calcolare:

```text
delta_ad       = livello_attuale - livello_iniziale
rate_ad_h      = variazione_livello / ore
delta_e_h      = variazione_ΔE / ore
rolling_mean   = media mobile
rolling_var    = varianza nella finestra
confidence     = affidabilità della misura
```

Il valore più utile per l’allarme sarebbe probabilmente il **rateo di variazione**, non la varianza statistica.

## Trasmissione MQTT

Nelle conversazioni erano stati citati:

* un Edge Node;
* broker come **EMQX** o **NanoMQ**;
* Node-RED;
* `node-red-contrib-sparkplug`;
* pubblicazione in formato **Sparkplug B**.

Non risultava però finalizzato uno schema preciso di topic e payload.

Sparkplug può essere appropriato in una successiva industrializzazione perché standardizza namespace MQTT, payload e gestione dello stato delle sessioni. ([Eclipse Sparkplug][2])

Per il primo prototipo, userei invece MQTT con JSON leggibile:

```text
museum/acetate/cabinet-01/adstrip/telemetry
museum/acetate/cabinet-01/adstrip/status
museum/acetate/cabinet-01/adstrip/alarm
```

Esempio di payload:

```json
{
  "timestamp": "2026-07-16T10:15:00Z",
  "device_id": "ADREADER-001",
  "cabinet_id": "CABINET-01",
  "strip_id": "STRIP-20260716-001",
  "exposure_hours": 24.0,
  "rgb": {
    "r": 86,
    "g": 121,
    "b": 109
  },
  "delta_e": 13.8,
  "ad_level": 1.42,
  "ad_rate_per_hour": 0.031,
  "temperature_c": 18.6,
  "relative_humidity_pct": 42.3,
  "measurement_confidence": 0.94,
  "quality_flags": [],
  "alarm_state": "warning"
}
```

Solo dopo avere validato la misura ottica passerei a Sparkplug B.

## Il vincolo critico delle A‑D Strips

Le istruzioni ufficiali prevedono che la strip:

* sia conservata al buio in un sacchetto sigillato;
* sia esposta per un tempo determinato in funzione di temperatura e umidità;
* sia confrontata immediatamente con la scala cromatica;
* sia eliminata dopo un solo utilizzo. ([filmcare.org][3])

Questo cambia la natura del prodotto.

Il dispositivo non sarebbe un sensore continuo e rigenerabile come un MOS. Sarebbe piuttosto un:

> **lettore automatico di un indicatore chimico consumabile e integrativo**

Monitorare continuamente la stessa strip durante l’esposizione è tecnicamente possibile, ma la relazione tra velocità del cambiamento cromatico e concentrazione di acido acetico dovrebbe essere caratterizzata sperimentalmente. Il protocollo ufficiale è infatti una lettura finale dopo un tempo di esposizione definito, non una misura cinetica continua.

## Conseguenza progettuale

Per realizzare un vero monitoraggio nel tempo servirebbe una delle seguenti strategie:

* sostituzione manuale periodica della strip;
* cartuccia con più strip attivate in sequenza;
* caricatore automatico;
* lettura di una nuova strip a intervalli programmati;
* utilizzo della strip come riferimento lento e di un MOS come rilevatore rapido.

La soluzione più robusta resta quindi una configurazione ibrida:

```text
MOS/VOC → variazioni rapide e allarmi preliminari
A-D Strip ottica → conferma lenta e indice di degrado
T/RH → contestualizzazione e compensazione
MQTT → storicizzazione e integrazione
```

## Decisione

**La scelta migliore ora è: realizzare un MVP con sensore RGB, illuminazione chiusa e controllata, sensore T/RH, microcontrollore e MQTT JSON, senza tentare inizialmente di convertire il colore in ppm di acido acetico.**

È una decisione **reversibile**: MQTT JSON potrà essere sostituito da Sparkplug B e il sensore RGB potrà essere aggiornato senza cambiare il principio del sistema.

**Cosa guadagni:** un prototipo semplice, verificabile e focalizzato sul rischio tecnico reale, cioè la ripetibilità della misura cromatica.

**Cosa sacrifichi:** nella prima versione non avrai una concentrazione certificata in ppm né piena interoperabilità Sparkplug.

**Cosa rischi:** costruire un sistema otticamente preciso che misura bene il colore, ma il cui colore non sia correlabile in modo univoco alla concentrazione istantanea di acido acetico.

**Prossimo passo entro 24 ore:** predisporre una prova con una strip, i cinque riferimenti cromatici ufficiali, illuminazione LED fissa e sensore RGB; acquisire almeno 30 letture per ciascun riferimento e verificare se le cinque classi risultano separabili con margine sufficiente.

👉 **Ecco la mia analisi onesta e il punto cieco che potresti non aver notato:**

* Il prodotto non misura direttamente l’acido acetico: misura la risposta cromatica di un indicatore a vapori acidi.
* La strip è monouso; senza un meccanismo di sostituzione, il dispositivo non è realmente un monitor continuo.
* La priorità non è MQTT o Sparkplug: è dimostrare la ripetibilità ottica e la relazione tra colore, tempo, temperatura, umidità e acidità.
* Una lettura numericamente molto stabile può comunque essere chimicamente ambigua; serve almeno una campagna di calibrazione con un riferimento indipendente.

[1]: https://www.filmcare.org/ad_strips?utm_source=chatgpt.com "Using A-D Strips - FilmCare.org"
[2]: https://sparkplug.eclipse.org/specification/?utm_source=chatgpt.com "The Sparkplug Specification | The Eclipse Foundation"
[3]: https://www.filmcare.org/pdf/adstrips_instructions.pdf?utm_source=chatgpt.com "USER’S GUIDE F - FilmCare.org"
