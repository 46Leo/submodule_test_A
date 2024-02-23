## Gerartchia repo e submodules:
```
A
├─ B
├─ C
│  └─ B
└─ D
   └─ B
```

## Info
Se inserisco sottomoduli in un repo, il comportamento di VS è il seguente (con riferimento ad `A`):
* scarica automaticamente tutti i sottomoduli del repo A in maniera ricorsiva. Git `git clone`, di default, non lo fa; serve l'opzione **`--recurse-submodules`**.
* non ne traccia le modifiche e non viene neppure visualizzato che esistono dei sottomoduli a meno che non esista un progetto di VS all'interno del submodule e che questo progetto sia incluso nella solution del repo principale! A questo punto i submodules vengono rilevati e tracciati.
* **ATTENZIONE**: quando si clona un repo contenente submodules, questi vengono scaricati in modalità [_detached HEAD_](https://github.com/gitextensions/gitextensions/issues/10794): quindi, se nel frattempo il repo del sottomodulo è avanzato, si dovrà creare un branch locale `main` e aggiornarlo [a piacimento](https://stackoverflow.com/questions/10914022/how-do-i-check-out-a-specific-version-of-a-submodule-using-git-submodule) ad un commit specifico o ad `origin/main`.  
  Il comportamento di per sè è corretto: il repo `A` punta ad un commit specifico dei suoi sottomoduli per ovvi motivi di versionamento/compatibilità del codice esistente.
* VSCode visualizza invece tutti i repo che trova in una cartella e quindi sarà sempre visibile, in qualche modo, se ci sono modifiche su un repo o in uno dei submodules.

## Note
**ATTENZIONE**:  
Con i nostri progetti che hanno come output una .dll, l'unica accortezza da avere è che la cartella di output delle .dll è la stessa per tutta la _solution_, quindi se un repo ha più sottomoduli che dipendono a loro volta da altri repo (sottomoduli dei sottomoduli), questi dovranno essere manualmente allineati **TUTTI** alla stessa versione/commit per evitare incompatibilità, dato che quando viene compilato il repo `B` (per 3 volte) verrà tenuta solo la .dll dell'ultimo che è stato compilato!