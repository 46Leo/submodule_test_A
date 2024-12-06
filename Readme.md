# tutorial sottomoduli git

- [tutorial sottomoduli git](#tutorial-sottomoduli-git)
  - [gerartchia repo e submodules:](#gerartchia-repo-e-submodules)
  - [aggiungere sottomoduli](#aggiungere-sottomoduli)
  - [clonare repository che contengono sottomoduli](#clonare-repository-che-contengono-sottomoduli)
  - [checkout](#checkout)
  - [configurare git globalmente](#configurare-git-globalmente)
  - [merge](#merge)
    - [hook post merge](#hook-post-merge)
      - [esempio](#esempio)
        - [senza Hook](#senza-hook)
        - [con hook](#con-hook)
  - [rimuovere un sottomodulo:](#rimuovere-un-sottomodulo)
  - [note](#note)


## gerartchia repo e submodules:
Si usa la seguente gerarchia come esempio di sottomoduli.  
```
A
├─ B
├─ C
│  └─ B
└─ D
   ├─ B
   └─ E
```
Fare attenzione con i submoduli: **se il submodulo A dipende dal submodulo C e il submodulo C dipende dal submodulo B**, e si deve apportare qualche modifica al codice di tutti, prima aggiornare B e fare il commit. Poi, eseguire il pull del commit di B da C, aggiornare il codice in C e fare il commit di C (il commit aggiornerà anche il riferimento all'attuale commit di B). Infine, eseguire il pull del commit di C da A (in questo modo, B è già aggiornato nell'ultimo commit di C), aggiornare il codice di A e fare il commit (il commit conterrà i riferimenti aggiornati di tutti gli altri repository).


## aggiungere sottomoduli
Per aggiungere un sottomodulo ad un repository git, usare il comando: 
> `$ git submodule add <URL>`

(Tutorial e info aggiuntive [qui](https://git-scm.com/book/it/v2/Git-Tools-Submodules) e [qui](https://www.atlassian.com/it/git/tutorials/git-submodule)).

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

## clonare repository che contengono sottomoduli
Di default, `git clone` NON scarica i sottomoduli. È **sempre** necessario aggiungere l'opzione **`--recurse-submodules`** al comando di clone, anche se sono globalmente impostati i parametri `submodule.recurse` e `submodule.stickyRecursiveClone` ([link](https://stackoverflow.com/questions/70249838/git-setting-submodule-recurse-is-not-working-git-bash)).  

## checkout
Anche per il `checkout` tra commit e branch nel repository principale git **non** aggiorna automaticamente i sottomoduli, serve il comando:
> `$ git submodule update --init --recursive`

[link](https://www.vogella.com/tutorials/GitSubmodules/article.html)

Oppure, per eseguire un `pull` o un `checkout` in modo che anche i submodules siano sincronizzati, aggiungere l'opzione **`--recurse-submodules`** al comando.

## configurare git globalmente
La soluzione (forse) migliore è [configurare Git globalmente]((https://stackoverflow.com/questions/1899792/why-is-git-submodule-not-updated-automatically-on-git-checkout)) in modo che l'update automatico dei sottomoduli sia il comportamento predefinito:
> `$ git config --global submodule.recurse true` 

Per rileggere l'impostazione:
> `$ git config --get submodule.recurse`

Forse è utile anche il setting [link](https://stackoverflow.com/questions/70249838/git-setting-submodule-recurse-is-not-working-git-bash): 
> `$ git config --global submodule.stickyRecursiveClone`

Spiegazione di chatgpt:
  >Il parametro submodule.stickyRecursiveClone è una configurazione di Git che riguarda il comportamento dei sottomoduli durante un'operazione di clonazione ricorsiva.
  >
  >Descrizione di submodule.stickyRecursiveClone
  >Questo parametro, se impostato su true, indica a Git di utilizzare il flag --recurse-submodules di default quando si clona un repository con sottomoduli, ma con un comportamento specifico legato >al checkout delle branch dei sottomoduli.
  >
  >Quando è abilitato, Git clona i sottomoduli mantenendo la branch attuale di ciascun sottomodulo come quella predefinita, invece di bloccarsi al commit specifico registrato nel repository >principale.
  >
  >Dettagli sul comportamento
  >Sottomoduli e commit specifici: Per impostazione predefinita, Git tratta i sottomoduli come repository "puntati" a un commit specifico registrato dal superproject (repository principale). Questo >significa che quando cloni un repository con --recurse-submodules, i sottomoduli vengono clonati e posizionati sul commit esatto richiesto.
  >
  >Comportamento con submodule.stickyRecursiveClone = true: Se abiliti questo parametro, quando cloni un repository con sottomoduli ricorsivamente, i sottomoduli non si limitano al commit >specifico, ma rimangono sulla branch originale a cui appartengono (ad esempio, main, master, o una branch specificata). Questo può essere utile se desideri lavorare sui sottomoduli con i branch >attuali, invece di bloccarli a un determinato commit.
  >
  >Come impostarlo
  >Per configurare submodule.stickyRecursiveClone, utilizza il comando:
  >
  >Per configurarlo a livello globale:
  >
  >bash
  >Copia codice
  >git config --global submodule.stickyRecursiveClone true
  >Per configurarlo solo a livello di repository locale:
  >
  >bash
  >Copia codice
  >git config submodule.stickyRecursiveClone true
  >Verifica della configurazione
  >Puoi verificare se il parametro è attivo con:
  >
  >bash
  >Copia codice
  >git config --get submodule.stickyRecursiveClone
  >Se restituisce true, la configurazione è attiva.


Un altro caso particolare è quando un submodule non è presente in un branch ma lo è in un altro. Potrebbe non venire scaricato, è utile allora il comando (da verificare se serve questo o se va bene `git submodule update --init --recursive`):
> `$ git submodule update --remote`

## merge
**Problema**: quando si fa il merge in `main` (o altro branch) di un branch che è linkato ad una versione più recente di un submodule (supponiamo non ci siano conflitti tra le versioni del sottomodulo), git **non** aggiorna automaticamente il sottomodulo nel branch dove è stato fatto il merge, ma bisogna **sempre** dare il comando:  
> `$ git submodule update --init --recursive` 

Di default git non sposta il sottomodulo ma lo manitiene alla versione attuale, notificando però che è necessario un nuovo commit per riportare il sottomodulo alla versione attuale, in seguito al merge ([link](https://www.reddit.com/r/git/comments/170la7k/why_isnt_my_submodule_uptodate_after_a_merge/)).
Una soluzione automatica è quella di usare un **post-merge hook** per risolvere il problema. [link](https://www.reddit.com/r/git/comments/170la7k/why_isnt_my_submodule_uptodate_after_a_merge/), [link](https://gist.github.com/ejmr/453edc19dd596e472e90)

### hook post merge
Per automatizzare l'aggiornamento dei sottomoduli dopo un merge, è utile creare un hook globale che esegue automaticamente dopo un merge.  
Info ai link: [istruzioni](https://coderwall.com/p/jp7d5q/create-a-global-git-commit-hook), [problema](https://www.reddit.com/r/git/comments/170la7k/why_isnt_my_submodule_uptodate_after_a_merge/) ed [esempio di hook-file](https://gist.github.com/ejmr/453edc19dd596e472e90).  

**Istruzioni**
* abilitare globalmente i template e crearne uno per il post-merge:
  > `$ git config --global init.templatedir '~/.git-templates'`  
  > `$ mkdir -p ~/.git-templates/hooks`  
  > `$ touch ~/.git-templates/hooks/post-merge`  
  > `$ chmod a+x ~/.git-templates/hooks/post-merge`  
* aprire il file e inserire il seguente testo:  
  ```bash
  #!/bin/sh
  git submodule update --init --recursive
  ```
* infine, reinizializzare i repository già presenti in locale per i quali si vuole applicare questo hook, entrando nella cartella del repository ed eseguendo il comando:
  > `$ git init`

Se la procedura è stata eseguita correttamente, essendo un parametro globale, ogni nuovo repository che verrà clonato con `git clone` conterrà automaticamente lo script di hook `post-merge` senza bisogno di ulteriori comandi.

#### esempio
Questo esempio mostra cosa succede inizialmente, senza l'hook, e dopo averlo applicato.

##### senza Hook
Inizialmente, il repository A e B (che è sottomodulo di A) si trovano entrambi in `main`: 
![alt text](<Screenshot 2024-12-06 115817.png>)
![alt text](<Screenshot 2024-12-06 115907.png>)

In entrambi viene creato un nuovo branch: in B viene eseguito un commit e in A viene registrato l'avanzamento di B:
![alt text](<Screenshot 2024-12-06 120004.png>)
![alt text](<Screenshot 2024-12-06 120041.png>)

A questo punto, in A viene fatto il merge del nuovo branch in master: per fare questo, si fa il checkout in master e si esegue:
> `$ git merge subm-test --no-ff`

Il merge va a buon fine (**1**) ma il sottomodulo **non** viene automaticamente aggiornato al riferimento importato dal branch appena mergiato (**2**).  

![alt text](<Screenshot 2024-12-06 141002.png>)

Anzi, in realtà è come se lo spostamento fosse stato eseguito durante il merge, salvo poi spostare nuovamente il sottomodulo al commit precedente che c'era già in main.  
Questo infatti porta alla presenza di una notifica (**3**) di modifiche non committate: tale modifica riporterebbe il sottomodulo dalla versione corretta `b4b98` (aggiornata tramite il merge) a quella precedente `354ee`, annullando di fatto l'avanzamento appena eseguito dal merge (qualora questa modifica venisse committata).  
A questo punto, però, basta eseguire il comando: 
> `$ git submodule update --init --recursive`

Il sottomodulo si porta alla versione corretta e la notifica scompare.

![alt text](<Screenshot 2024-12-06 141214.png>)


##### con hook
Dopo aver generato correttamente l'hook e aver re-inizializzato il repository, nella cartella `.git/hooks/` troveremo un nuovo file `post-merge` con il nostro script.  
A questo punto, l'operazione di merge aggiornerà automaticamente il sottomodulo senza bisogno di altri comandi. 
![alt text](<Screenshot 2024-12-06 143141.png>)

I messaggi aggiuntivi stampati sul terminale sono inseriti nello script, integralmente riportato di seguito:  
```bash
#!/bin/sh
### see https://gist.github.com/ejmr/453edc19dd596e472e90
#
# This simple shell script is a Git hook that will automatically
# update all submodules any time you perform a merge or pull.
# This can be useful because you can do things like...
#
#     $ git checkout master && git pull
#
# ...without having to remember to potentially update any
# submodules in the repository.
#
# To use this script save it as `.git/hooks/post-merge` in
# your repository and make the script executable, e.g. via
# `chmod +x .git/hooks/post-merge`.
##################################################################

echo "Running post-merge hook:"
echo "Updating submodules..."

git submodule update --init --recursive

echo "Submodules updated"
```

## rimuovere un sottomodulo:
Per [rimuovere completamente un sottomodulo](https://stackoverflow.com/questions/1260748/how-do-i-remove-a-submodule/36593218#36593218):
```
# Remove the submodule entry from .git/config
git submodule deinit -f path/to/submodule

# Remove the submodule directory from the superproject's .git/modules directory
rm -rf .git/modules/path/to/submodule

# Remove the entry in .gitmodules and remove the submodule directory located at path/to/submodule
git rm -f path/to/submodule
```

## note
**ATTENZIONE**:  
Per i nostri progetti che producono un .dll, l'unica precauzione da prendere è di ricordarsi che la cartella di output per le .dll è la stessa per tutta la solution!
Pertanto, se un repository ha più submodules che a loro volta dipendono da altri repository (submodules annidati), questi devono essere allineati manualmente TUTTI alla stessa versione/commit per evitare conflitti a runtime!

Questo perché quando il repository B viene compilato interamente (se il progetto è incluso nella solution) o in parte (vengono inclusi direttamente alcuni suoi files), solo la .dll genrata dall'ultimo progetto compilato verrà mantenuta! È quindi obbligatorio che tutti i repository B puntino allo stesso commit.
Inoltre, se un repository (D) ha una dipendenza di link da un sottomodulo (E), ovviamente VS non compilerà E a meno che non venga incluso manualmente nella solution (la solution di D include ovviamente E, ma la solution di A non può saperlo).
Se ad un primo momento questa può sembrare una cosa complicata, in realtà in fase di build verrà generato un errore di link che permette di individuare la dipendenza e correggerla facilmente.
È comunque utile perchè permette di individuare tutte e sole le dipendenze necessarie.

**VERSIONAMENTO e COMMIT**
Prestare attenzione con i sottomoduli: se il submodule A dipende dal submodule B e il submodule C dipende dal submodule B, e si devono fare alcune modifiche al codice di tutti questi moduli, aggiornare per primo B e fare il commit. Poi, fare il pull del modulo B dal modulo C, aggiornare il codice in C e fare il commit di C (il commit aggiornerà anche il riferimento all'attuale commit di B). Infine, fare il pull di C da A (in questo modo, B inerno a C sarà automaticamente aggiornato grazie al commit di C), fare attenzione a sincronizzare (se necessario) tutti i B alla stessa versione, aggiornare il codice di A e fare il commit di A, che conterrà quindi i riferimenti aggiornati di tutti gli altri repository a cascata.
