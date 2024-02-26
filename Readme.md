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
  * VSCode visualizza invece tutti i repo che trova in una cartella e quindi sarà sempre visibile, in qualche modo, se ci sono modifiche su un repo o in uno dei submodules.
* **ATTENZIONE**: quando si clona un repo contenente sottomoduli, questi vengono scaricati in modalità [_detached HEAD_](https://github.com/gitextensions/gitextensions/issues/10794): nel frattempo il repo del sottomodulo potrebbe essere avanzato.   
Se lo si vuole aggiornare all'ultima versione, si dovrà fare manualmente il _checkout_ nel `main` locale (che dovrebbe essere stato creato automaticamente da Git quando è stato clonato il repo `A` e punta a `origin/main`), o crearlo [a piacimento](https://stackoverflow.com/questions/10914022/how-do-i-check-out-a-specific-version-of-a-submodule-using-git-submodule) in un commit specifico o in `origin/main`.  
In questo modo, essendo ora in un commit specifico (`main`), questo verrà costantemente tracciato e verranno notificati eventuali aggiornamenti presenti in `origin/main`.  
Il comportamento di per sè è corretto: il repo `A` deve puntare sempre ad un commit specifico dei suoi sottomoduli per ovvi motivi di versionamento/compatibilità del codice esistente.  
Esempio:
  ```
  $ git submodule foreach 'git branch -a'
  Entering 'submodule_test_B'
  * (HEAD detached at 13dae44)
    main
    remotes/origin/HEAD -> origin/main
    remotes/origin/main
  Entering 'submodule_test_C'
  * (HEAD detached at 4d60542)
    main
    remotes/origin/HEAD -> origin/main
    remotes/origin/main
  Entering 'submodule_test_D'
  * (HEAD detached at 6570e69)
    main
    remotes/origin/HEAD -> origin/main
    remotes/origin/main
  ```

## Checkout
Se voglio "tornare indietro" nel repository principale facendo il `checkout` ad un commit o branch specifico, potrei aver bisogno di spostarmi anche nei suoi sottomoduli. Questo non viene fatto automaticamente ma serve il comando:

> git submodule update

[link](https://www.vogella.com/tutorials/GitSubmodules/article.html)

## Note
**ATTENZIONE**:  
Con i nostri progetti che hanno come output una .dll, l'unica accortezza da avere è che la cartella di output delle .dll è la stessa per tutta la _solution_, quindi se un repo ha più sottomoduli che dipendono a loro volta da altri repo (sottomoduli dei sottomoduli), questi dovranno essere manualmente allineati **TUTTI** alla stessa versione/commit per evitare incompatibilità, dato che quando viene compilato il repo `B` (per 3 volte, essendo sottomodulo di `A`, `C` e `D`) verrà tenuta solo la .dll dell'ultimo che è stato compilato!