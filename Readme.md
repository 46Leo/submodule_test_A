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
Per aggiungere un sottomodulo ad un repository git, usare il comando: `git submodule add <URL>`. (Tutorial e info aggiuntive [qui](https://git-scm.com/book/it/v2/Git-Tools-Submodules) e [qui](https://www.atlassian.com/it/git/tutorials/git-submodule)).  
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
Se voglio "tornare indietro" nel repository principale facendo il `checkout` ad un commit o branch specifico, potrei aver bisogno di spostarmi anche nei suoi sottomoduli. Questo **non** viene fatto automaticamente ma serve il comando:

> git submodule update --init --recursive

[link](https://www.vogella.com/tutorials/GitSubmodules/article.html)

Di default, Git non aggiorna i sottomoduli quando ci si sposta nel repo principale.  
Per eseguire un `pull` o un `checkout` in modo che anche i submodules siano sincronizzati, aggiungere l'opzione **`--recurse-submodules`** oppure [configurare Git]((https://stackoverflow.com/questions/1899792/why-is-git-submodule-not-updated-automatically-on-git-checkout)) in modo che sia il comportamento predefinito:
> `$ git config --global submodule.recurse true` 

Un altro caso particolare è quando un submodule non è presente in un branch ma lo è in un altro. Potrebbe non venire scaricato, è utile allora il comando:
> git submodule update --remote

## Rimuovere un submodule:
per rimuovere completamente un submodule ([link](https://stackoverflow.com/questions/1260748/how-do-i-remove-a-submodule/36593218#36593218)):
```
# Remove the submodule entry from .git/config
git submodule deinit -f path/to/submodule

# Remove the submodule directory from the superproject's .git/modules directory
rm -rf .git/modules/path/to/submodule

# Remove the entry in .gitmodules and remove the submodule directory located at path/to/submodule
git rm -f path/to/submodule
```

## Note
**ATTENZIONE**:  
Con i nostri progetti che hanno come output una `.dll`, l'unica accortezza da avere è che la cartella di output delle .dll è la stessa per tutta la _solution_, quindi se un repo ha più sottomoduli che dipendono a loro volta da altri repo (sottomoduli dei sottomoduli), questi dovranno essere manualmente allineati **TUTTI** alla stessa versione/commit per evitare incompatibilità, dato che quando viene compilato il repo `B` (per 3 volte, essendo sottomodulo di `A`, `C` e `D`) verrà tenuta solo la .dll dell'ultimo che è stato compilato! È quindi fondamentale che tutti i repo `B` puntino allo stesso commit.

Ogni modulo (repo) deve clonare i sottomoduli al suo stesso livello di directory (che sarà dentro la cartella più esterna della solution) e fare riferimento da VS con path relativi.