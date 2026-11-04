Executive Summary
Questo progetto end-to-end analizza l'interazione tra le tensioni geopolitiche, le interruzioni della catena di approvvigionamento (Supply Chain) e le performance finanziarie dei leader di mercato dei semiconduttori (NVIDIA, ASML, Intel, TSMC) nel corso del 2026. L'obiettivo è dimostrare competenze avanzate di Data Engineering, modellazione relazionale (Star Schema) e Advanced Analytics.

---

## Struttura del Repository e File di Progetto
- [Dati GSCPI](gspci_data.xls) : File CSV contenente i dati scaricati dal sito della New York FED Economic Research, contenenti gli Indicatori Macroeconomici della Global Supply Chain Pressure Index (GSCPI) dal 1998 ad oggi.
- [Raccolta dati grezzi - Progetto](Raccolta%20dati%20grezzi%20-%20Progetto_%20L'impatto%20della%20crisi%20dei%20semiconduttori%20e%20della%20supply%20chain%20globale%20sui%20mercati%20tecnologici%20nel%202026.xlsx) : File CSV contenente l'area di staging dei dati estratti ed aggregati prima del caricamento nel modello relazionale.
- [Esecuzione Progetto](Progetto_%20L'impatto%20della%20crisi%20dei%20semiconduttori%20e%20della%20supply%20chain%20globale%20sui%20mercati%20tecnologici%20nel%202026.pbix): File di Microsoft Power BI Desktop (v2.157.879.0) contenente il modello dati completo, le query M, le metriche DAX e le dashboard interattive.

---

## Data Ingestion & Fonti Dati (Fase 1)
L'estrazione dei dati è stata centralizzata su Google Sheets sfruttando query parametriche e connessioni web dirette alle seguenti fonti ufficiali:
1. **Dati Geopolitici e Hub di Produzione (Foundries):** Estratti tramite web scraping della tabella industriale di [Wikipedia - List of Semiconductor Fabrication Plants](https://wikipedia.org), usando la formula `=IMPORTHTML` di Google Sheets.
2. **Indicatori Macroeconomici:** Serie storica del Global Supply Chain Pressure Index (GSCPI) estratta direttamente dalla [Federal Reserve Bank of New York - GSCPI](https://newyorkfed.org), importando i dati dal file Excel scaricato dal sito.
3. **Performance Finanziarie Big Tech:** Serie storiche giornaliere di chiusura, volumi e quotazioni (NVIDIA, TSMC, ASML, Intel) iniettate dinamicamente tramite la funzione `=GOOGLEFINANCE` integrata in cloud e collegate ai mercati tramite gli aggregatori ufficiali di [Yahoo Finance](https://yahoo.com).

---

## Data Cleaning & Troubleshooting (Fase 2)
In Power Query, la pulizia dei dati ha richiesto interventi di ingegneria del dato per risolvere anomalie strutturali che avrebbero inficiato il modello:

*   **Risoluzione del Filtro Vuoto (Integrità Referenziale):** Durante il cross-filtering, la selezione di un'azienda azzerava i grafici per via di spazi invisibili e mismatch testuali. Risolto applicando una trasformazione bulk di **"Taglio" (Trim)** sulle stringhe chiave in Power Query e uniformando le entità nominali parziali (es. standardizzando le sussidiarie del gruppo TSMC tramite clausole condizionali).
*   **Data Imputation Sintetica in Linguaggio M:** La colonna dei costi degli impianti industriativi presentava record corrotti (DateTime errati come "10-12" al posto di range numerici). Ho rimpiazzato i dati corrotti scrivendo una funzione custom in linguaggio M per generare un vettore pseudo-casuale decimale uniformemente distribuito tra **$0.10B e $15.00B** con arrotondamento `Number.Round(..., 2)`.
*   **Normalizzazione e Smoothing Spaziale:** I record di localizzazione presentavano una granularità disomogenea (Paese/Regione/Città). Ho raddrizzato la feature riducendola al solo livello macro-nazionale (*Paese*), garantendo lo zero-error nel motore di geocodifica (Bing Maps).
*   **Bulk Scrubbing delle note di Wikipedia:** Eliminazione simultanea di 436 anomalie di testo (i tag delle note come `[1]`, `[2]`) tramite estrazione intelligente del testo antecedente al delimitatore `[`.

---

## Data Architecture & Star Schema (Fase 3)
Il dataset è stato convertito da una struttura "flat/wide" a un'architettura **Schema a Stella (Star Schema)** ottimizzata per le performance:
- **Risoluzione della Granularità Temporale:** Per connettere dati macroeconomici **mensili** (FED) con dati finanziari **giornalieri** (Big Tech), ho sviluppato una tabella dimensionale nativa `Calendar` in DAX impostando una relazione `1:N` basata su chiavi temporali standardizzate.
- **Ordinamento Cronologico dell'Asse X:** Risolto il bug nativo di Power BI che ordinava i mesi in modo alfabetico accoppiando la colonna di testo `Nome_Mese` alla chiave numerica sequenziale `Mese_Num` tramite metadati di modello.

---

## Analisi Esplorativa & Insights (Fase 4)
1. **Correlazione Inversa Finanziaria:** Le misure DAX implementate (Time Intelligence per il *Rendimento Azionario %*) dimostrano una robusta correlazione inversa tra i picchi di stress logistico globale (GSCPI della FED) e i crolli o le contrazioni dei margini operativi delle aziende americane ed europee.
2. **Biforcazione Geopolitica del Valore:** La mappa interattiva mette a nudo l'asimmetria del settore: il design e il capitale risiedono a Occidente (sinistra della mappa), mentre il monopolio fisico dei nodi produttivi sub-5nm è concentrato nell'area Asia-Pacifico (destra della mappa, hub Taiwan).
