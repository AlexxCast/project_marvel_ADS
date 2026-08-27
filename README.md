# Universo Marvel: chi fa da ponte tra le comunità?

Progetto d'esame per il corso di **Network Science & Text Mining**
(Università di Udine).

## Domanda di ricerca

Per fare da *boundary spanner* (ponte tra comunità diverse) nella rete dei
personaggi Marvel, conta di più l'**intelligenza** o la **forza fisica**?

## Approccio

Il progetto integra **network analysis** e **text mining**:
- costruzione della rete di co-apparizione dei personaggi;
- individuazione dei personaggi-ponte tramite **betweenness centrality**;
- rilevamento delle comunità con l'algoritmo di **Louvain**;
- analisi statistica (correlazione di Pearson, test d'ipotesi, regressione
  lineare multipla) per rispondere alla domanda di ricerca;
- caratterizzazione delle comunità tramite **TF-IDF** sulle affiliazioni.

## Risultato principale

L'intelligenza risulta correlata alla capacità di fare da ponte
(r ≈ 0.26, p < 0.001), mentre la forza no (r ≈ 0.01, p = 0.86). L'effetto è
statisticamente significativo ma di entità modesta (R² ≈ 0.07).

## Contenuto del repository

### File del progetto
- `progetto-marvel.Rmd` — documento R Markdown (analisi completa e commentata)
- `progetto-marvel.html` / `progetto-marvel.pdf` — documento esportato
- `Universo_Marvel__chi_fa_da_ponte_.pdf` — presentazione
- `script_marvel.R` — script R del codice

### Dataset utilizzati
- `hero-network.csv` — rete di co-apparizione dei personaggi
- `superHeroes.csv` — attributi dei personaggi (intelligenza, forza, affiliazioni)

Gli altri file `.csv` presenti nel repository sono dataset di partenza esplorati
in fase iniziale ma **non utilizzati** nell'analisi finale.

## Nota sull'uso dell'AI

I blocchi di codice sviluppati con assistenza AI (ponderazione TF-IDF,
co-occorrenza, estrazione e arricchimento del sotto-grafo) sono evidenziati nel
documento con appositi commenti, come previsto dalla policy del corso, e
costituiscono una porzione minoritaria del codice complessivo.

## Autore

Alessio Castronovo
