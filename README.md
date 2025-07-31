# Modello di previsione del benessere giornaliero con sistema di raccomandazione personalizzato

Deep Learning project per la previsione del benessere e sistema di raccomandazione personalizzato basato su dati di attività fisica e sonno

---

## Introduzione

Negli ultimi anni, l’uso di dispositivi indossabili (wearables) per il monitoraggio della salute è diventato sempre più diffuso. Attraverso sensori integrati, questi strumenti sono in grado di raccogliere dati fisiologici quotidiani, come il numero di passi, la velocità del passo e la durata delle diverse fasi del sonno. Tali informazioni possono essere analizzate per stimare il livello di benessere dell’utente e fornire raccomandazioni personalizzate.

In questo progetto è stato utilizzato un dataset contenente dati giornalieri raccolti da dispositivi wearable relativi a molteplici utenti. L’obiettivo principale è sviluppare un modello predittivo basato su tecniche di deep learning per il forecasting, in grado di stimare il benessere quotidiano di un utente, espresso tramite un indice sintetico chiamato **activity index**.

Sulla base della previsione dell’activity index del giorno successivo, è stato anche implementato un sistema di raccomandazione intelligente che suggerisce azioni personalizzate per migliorare lo stile di vita dell’utente.

---

## 🔎 Analisi del problema

Il problema viene formulato come un task di previsione temporale personalizzata. Il modello riceve in input sequenze di 15 giorni consecutivi con dati relativi a sonno e attività fisica, e prevede l’activity index del giorno 16.

Questa impostazione consente al modello di apprendere relazioni temporali tra abitudini recenti e condizione futura dell’utente, con applicazioni in e-health e raccomandazione intelligente.

---

## 📊 Dataset originale

Dati raccolti da dispositivi Nokia su 11.615 utenti (01/04/2016 → 31/03/2017), con informazioni giornaliere su sonno e attività fisica. Il dataset è strutturato in:

* `user id`
* `date`
* `data type`: tipo di metrica (1→27)
* `value`: valore della metrica

Esempi di `data type` includono: numero di passi, durata del sonno, frequenza cardiaca, risvegli notturni, velocità dell'andatura, ecc.

---

## Dataset personalizzato per la previsione

Sono state selezionate solo le variabili rilevanti legate a sonno e attività fisica. I passaggi principali sono:

* **Filtraggio**: scelta dei data type rilevanti
* **Pivot**: trasformazione da formato long a wide
* **Rinomina feature**: nomi coerenti e leggibili

Variabili finali: `sleepduration`, `bedin`, `nbawake`, `awakeduration`, `deepduration`, `remduration`, `activity calories`, `stepsgaitspeed`, `distancegaitspeed`, ecc.

---

## Calcolo dell’activity index

Indice numerico sintetico per rappresentare il benessere giornaliero. Viene calcolato combinando:

* Calorie bruciate
* Durata attività fisica
* Qualità e durata del sonno
* Sonno profondo e REM
* Numero e durata dei risvegli
* Andatura (passi/min, km/h)

Ogni componente contribuisce con un punteggio. Il risultato è un indice interpretabile da usare come target.

---

## Costruzione sequenze temporali

Vengono create finestre temporali di 15 giorni consecutivi per ciascun utente, ammettendo al massimo un giorno mancante. Il giorno 16 rappresenta il target `y` da prevedere.

Questa strategia è ideale per modelli di forecasting e consente una rappresentazione temporale stabile.

---

## Dataset finale per NeuralForecast

Il dataset viene ristrutturato secondo il formato richiesto dalla libreria **NeuralForecast**:

* `unique_id` per identificare l’utente
* `ds` come colonna temporale
* `y` come target: activity index

Solo le sequenze con target disponibile vengono conservate.

---

## Modelli utilizzati

### Informer

* Basato su Transformer ottimizzato per sequenze lunghe
* `ProbSparse Attention` + `Distillation`
* Encoder/Decoder separati
* Embedding posizionali e temporali

### PatchTST

* Derivato da Vision Transformer (ViT)
* Solo encoder, senza decoder
* Sequenze suddivise in patch temporali
* Training efficiente, forecast diretto

---

## Risultati: Informer vs PatchTST

| Modello  | MSE   | MAE   | RMSE  | R²    |
| -------- | ----- | ----- | ----- | ----- |
| Informer | 0.923 | 0.702 | 0.961 | 0.555 |
| PatchTST | 1.341 | 0.880 | 1.158 | 0.354 |

Informer si dimostra più preciso e stabile nel forecasting dell’activity index.

---

## Cross-Validation

 Cross-validation temporale con 10 finestre (n=10) per valutare la stabilità del modello Informer nel tempo.

Andamento dell’errore MAE smussato mostra un buon bilanciamento tra accuratezza e consistenza nel tempo, con evidenza del minimo di errore.

---

## Sistema di raccomandazione personalizzato

Basato sulle previsioni dell’activity index, il sistema fornisce suggerimenti personalizzati.
Valuta: durata sonno, attività fisica, risvegli notturni, sonno REM/deep, andatura, calorie.
Restituisce un messaggio del tipo:

> **Livello moderato. Attenzione: dormi di più, fai più attività fisica.**

Utilizza la funzione `detailed_fitness_recommendation()` per analizzare punti deboli e suggerire miglioramenti.

---

## Confronto con altri modelli

| Modello    | MSE   | MAE   | RMSE  | R²     |
| ---------- | ----- | ----- | ----- | ------ |
| Informer   | 0.923 | 0.702 | 0.961 | 0.555  |
| PatchTST   | 1.341 | 0.880 | 1.158 | 0.354  |
| TimesNet   | 1.906 | 0.989 | 1.381 | 0.082  |
| NHITS      | 2.739 | 1.153 | 1.655 | -0.319 |
| Autoformer | 1.983 | 1.006 | 1.408 | 0.045  |

Informer si conferma il modello più efficace per accuratezza e robustezza predittiva.

---

## Conclusioni e sviluppi futuri

Il progetto ha dimostrato come l’uso del deep learning e dei dispositivi wearable possa supportare la previsione del benessere quotidiano e fornire raccomandazioni personalizzate.

Informer è risultato il miglior modello tra quelli testati, e il sistema di raccomandazione ha mostrato utilità concreta nella generazione di suggerimenti intelligenti.

**Sviluppi futuri**:

* Personalizzazione dei modelli su base individuale
* Integrazione di variabili biometriche
* Introduzione di feedback dinamici dagli utenti

> Il progetto rappresenta un passo concreto verso soluzioni intelligenti per il monitoraggio e il miglioramento della salute personale.
