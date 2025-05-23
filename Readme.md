# tutorial sottomoduli git

- [tutorial sottomoduli git](#tutorial-sottomoduli-git)
  - [gerarchia repo e submodules:](#gerarchia-repo-e-submodules)
  - [aggiungere sottomoduli](#aggiungere-sottomoduli)
    - [nota per visual studio](#nota-per-visual-studio)
  - [rimuovere un sottomodulo:](#rimuovere-un-sottomodulo)
  - [clonare repository che contengono sottomoduli](#clonare-repository-che-contengono-sottomoduli)
  - [checkout](#checkout)
    - [nuovo branch](#nuovo-branch)
  - [configurare git globalmente](#configurare-git-globalmente)
    - [nuovo submodule nel repo remoto](#nuovo-submodule-nel-repo-remoto)
      - [esempio](#esempio)
  - [merge](#merge)
    - [hook post merge](#hook-post-merge)
      - [esempio](#esempio-1)
        - [senza Hook](#senza-hook)
        - [con hook](#con-hook)
  - [suggerimenti](#suggerimenti)
  - [note](#note)


## gerarchia repo e submodules:
Si usa la seguente gerarchia di sottomoduli per gli esempi seguenti.  
```
A
├─ B
├─ C
│  └─ B
└─ D
   ├─ B
   └─ E
```

## aggiungere sottomoduli
Per aggiungere un sottomodulo ad un repository git, è sufficiente posizionarsi nella cartella dove si vuole inserire il repository (sottomodulo) e usare il comando: 
> `$ git submodule add <URL>`

(Tutorial e info aggiuntive [qui](https://git-scm.com/book/it/v2/Git-Tools-Submodules) e [qui](https://www.atlassian.com/it/git/tutorials/git-submodule)).

**ATTENZIONE**: quando si clona un repo contenente sottomoduli, questi vengono scaricati in modalità [_detached HEAD_](https://github.com/gitextensions/gitextensions/issues/10794): nel tempo, il repo del sottomodulo potrebbe avanzare.   
Se lo si vuole aggiornare all'ultima versione, si dovrà fare manualmente il _checkout_ nel `main` locale (che dovrebbe essere stato creato automaticamente da Git quando è stato clonato il repo `A` e che punta a `origin/main`), o crearlo [a piacimento](https://stackoverflow.com/questions/10914022/how-do-i-check-out-a-specific-version-of-a-submodule-using-git-submodule) in un commit specifico o in `origin/main` (oppure seguire un altro branch).  
In questo modo, essendo ora in un branch specifico (`main`), questo verrà costantemente tracciato e verranno notificati eventuali aggiornamenti presenti in `origin/main`.  
Il comportamento, che può sembrare strano, di per sè è corretto: il repo `A` deve puntare sempre ad un commit specifico dei suoi sottomoduli per ovvi motivi di versionamento/compatibilità del codice esistente, e tali sottomoduli non devono aggiornarsi autonomamente.  
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

### nota per visual studio
Se si inseriscono sottomoduli in un repo, il comportamento di VS sembra essere il seguente (con riferimento al repo principale `A`):
* scarica automaticamente tutti i sottomoduli del repo A in maniera ricorsiva. 
* non ne traccia le modifiche e non viene neppure visualizzato che esistono dei sottomoduli a meno che non esista un progetto di VS all'interno del submodule e che questo progetto sia incluso nella solution del repo principale! A questo punto i submodules vengono rilevati e tracciati.
  * VSCode visualizza invece tutti i repo che trova in una cartella e quindi sarà sempre visibile, in qualche modo, se ci sono modifiche su un repo o in uno dei submodules.


## rimuovere un sottomodulo:
La [rimozione completa di un sottomodulo](https://stackoverflow.com/questions/1260748/how-do-i-remove-a-submodule/36593218#36593218) è più complessa e richiede più step.  
Il sottomodulo deve prima essere de-inizializzato, poi rimossi i file dalla cartella .git e il riferimento in .gitmodules e infine cancellata la cartella fisica in cui è presente il sottomodulo:
```
# Remove the submodule entry from .git/config
git submodule deinit -f path/to/submodule

# Remove the submodule directory from the superproject's .git/modules directory
rm -rf .git/modules/path/to/submodule

# Remove the entry in .gitmodules (manually edit the file)
# ...
# and remove the submodule directory located at path/to/submodule
git rm -f path/to/submodule
```

## clonare repository che contengono sottomoduli
Il comando standard per clonare un repository è:  
> `$ git clone <URL>`

Di default, però, `git clone` **NON** scarica i sottomoduli!  
Per scarica automaticamente tutti i sottomoduli in maniera ricorsiva è **sempre** necessario aggiungere l'opzione **`--recurse-submodules`** al comando di clone, anche se sono globalmente impostati i parametri `submodule.recurse` e `submodule.stickyRecursiveClone` ([info](https://stackoverflow.com/questions/70249838/git-setting-submodule-recurse-is-not-working-git-bash)).  

> `$ git clone <URL> --recurse-submodules`


## checkout
Anche per il `checkout` tra commit e branch nel repository principale git **non** aggiorna automaticamente i sottomoduli, serve il comando:
> `$ git submodule update --init --recursive`

[link](https://www.vogella.com/tutorials/GitSubmodules/article.html)

Oppure, per eseguire un `pull` o un `checkout` in modo che anche i submodules siano sincronizzati, aggiungere l'opzione **`--recurse-submodules`** al comando:
> `$ git pull origin <branch-name> --recurse-submodules`

È anche possibile aggiornare i sottomoduli al commit più recente del branch remoto attualmente tracciato:
> `$ git submodule update --remote`

### nuovo branch
Se esiste un nuovo branch remoto, con un nuovo sottomodulo, e facendo il chekout compare un errore del tipo:  
> fatal: not a git repository: ../.git/modules/<submodule name>  
> fatal: could not reset submodule index

Provare a creare prima un branch locale con lo stesso nome
> $ `git checkout -b <nome branch>`

poi legarlo al branch remoto
> $ `git branch -u origin/<nome branch>`

e infine aggiornarlo con un `pull` (eventualmente con opzione `--recurse-submodule`).


## configurare git globalmente
La soluzione (forse) migliore è [configurare Git globalmente]((https://stackoverflow.com/questions/1899792/why-is-git-submodule-not-updated-automatically-on-git-checkout)) in modo che l'update automatico dei sottomoduli sia il comportamento predefinito:
> `$ git config --global submodule.recurse true` 

Per rileggere l'impostazione:
> `$ git config --get submodule.recurse`

A scelta, si può configurare anche il setting [link](https://stackoverflow.com/questions/70249838/git-setting-submodule-recurse-is-not-working-git-bash): 
> `$ git config --global submodule.stickyRecursiveClone`

Spiegazione del parametro (da chatgpt):
  >Il parametro submodule.stickyRecursiveClone è una configurazione di Git che riguarda il comportamento dei sottomoduli durante un'operazione di clonazione ricorsiva.
  >
  >Descrizione di submodule.stickyRecursiveClone
  >Questo parametro, se impostato su true, indica a Git di utilizzare il flag --recurse-submodules di default quando si clona un repository con sottomoduli, ma con un comportamento specifico legato al checkout delle branch dei sottomoduli.
  >
  >Quando è abilitato, Git clona i sottomoduli mantenendo la branch attuale di ciascun sottomodulo come quella predefinita, invece di bloccarsi al commit specifico registrato nel repository principale.
  >
  >Dettagli sul comportamento
  >Sottomoduli e commit specifici: Per impostazione predefinita, Git tratta i sottomoduli come repository "puntati" a un commit specifico registrato dal superproject (repository principale). Questo significa che quando cloni un repository con --recurse-submodules, i sottomoduli vengono clonati e posizionati sul commit esatto richiesto.
  >
  >Comportamento con submodule.stickyRecursiveClone = true: Se abiliti questo parametro, quando cloni un repository con sottomoduli ricorsivamente, i sottomoduli non si limitano al commit specifico, ma rimangono sulla branch originale a cui appartengono (ad esempio, main, master, o una branch specificata). Questo può essere utile se desideri lavorare sui sottomoduli con i branch attuali, invece di bloccarli a un determinato commit.
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

### nuovo submodule nel repo remoto
Un altro caso particolare si verifica quando nel repository remoto viene aggiunto un commit che contiene un nuovo submodule. Quando, in locale, si eseguirà il `pull` del commit remoto (o, se siamo in un branch differente, il checkout nel branch che contiene questo nuovo sottomodulo), il sottomodulo verrà aggiunto solo come "riferimento", ma **NON verrà automaticamente scaricato**!

Come nel [caso già esamitato](#clonare-repository-che-contengono-sottomoduli) in cui si clona un repository (comando `clone`), anche in questo caso git, per scelta, non scarica automaticamente i sottomoduli ma è necessario farlo manualmente con il comando:
> `$ git submodule update --init --recursive`

In alternativa al comando manuale, si può creare un hook `post_merge` come descritto nel capitolo [merge](#merge), che si occupa di scaricare automaticamente i nuovi submodules dopo un `pull`.

Lo stesso problema si presenta anche se il sottomodulo si trova in un branch differente, nuovo e presente solamente sul repo remoto, e vogliamo fare il `checkout` o `swicth` in questo nuovo branch. Anche in questo caso è possibile evitare di dover scaricare manualmente il sottomodulo creando un hook dal nome `post_checkout` che verrà eseguito automaticamente dopo un `checkout` o `swicth`.  
L'hook sarà identico a quello per il merge, dovrà contenere solamente il comando:
> `$ git submodule update --init --recursive`

#### esempio
Supponiamo di essere nella situazione iniziale seguente: vogliamo spostarci in un nuovo branch remoto (con nuovi sottomoduli).  
![alt text](<Immagine 2025-01-11 211854.jpg>)

Facendo lo `switch` sul nuovo branch (**attenzione**, preferire `git switch` rispetto a `git checkout`), troveremo in locale le cartelle dei nuovi sottomoduli ma saranno vuote. Dovremo aggiornare manualmente i nuovi repository.
![alt text](<Immagine 2025-01-11 212044.jpg>)

![alt text](<Immagine 2025-01-11 212448.jpg>)

Se invece creiamo un nuovo script `post_checkout`, l'aggiornamento avviene automaticamente: questa volta viene eseguito l'hook (si nota il testo "Running post-checkout hook") e i sottomoduli vengono scaricati.
![alt text](<Immagine 2025-01-11 213334.jpg>)

**NOTA**: questo script ha però un effetto collaterale: viene eseguito ad **ogni checkout** e quindi, se ci si sposta spesso tra commit o branch con sottomoduli diversi, potrebbe richiedere del tempo per completare l'aggiornamento di tutti i sottomoduli, specialmente se sono piuttosto grossi.  
Inoltre, anche il checkout di un singolo file da un altro commit (comando `git checkout <SHA> -- path/to/your/file`) triggera l'esecuzione dello script (viene pur sempre usato il comando `checkout`).  
Da valutare quindi se vale la pena o meno inserirlo globalmente (probabilmente no).


## merge
**Problema**: quando si fa il merge in `main` (o altro branch) di un branch che è linkato ad una versione più recente di un submodule (supponiamo non ci siano conflitti tra le versioni del sottomodulo), git **non** aggiorna automaticamente il sottomodulo nel branch dove è stato fatto il merge, ma bisogna **sempre** dare il comando:  
> `$ git submodule update --init --recursive` 

Il problema è simile al [caso precedente](#nuovo-submodule-nel-repo-remoto), ma mentre prima avevamo a che fare con un **nuovo** sottomodulo, qui parliamo di un sottomodulo **già esistente** che, in un altro branch, è stato **aggiornato ad un commit più recente** (o semplicemente diverso).  
Di default git non sposta il sottomodulo ma lo manitiene alla versione attuale, notificando però che è necessario un nuovo commit per riportare il sottomodulo alla versione attuale, in seguito al merge ([link](https://www.reddit.com/r/git/comments/170la7k/why_isnt_my_submodule_uptodate_after_a_merge/)).
Una soluzione automatica è quella di usare un [_**post-merge hook**_](#hook-post-merge) per risolvere il problema. [link](https://www.reddit.com/r/git/comments/170la7k/why_isnt_my_submodule_uptodate_after_a_merge/), [link](https://gist.github.com/ejmr/453edc19dd596e472e90)

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

**NOTA**: Questo comando reinizializza solo il repository principale, ma non i sottomoduli (lo si vede perchè nella cartella `.git/modules/<nome modulo>/hooks/` del repo principale, **non** è stato copiato il file di hook `post-merge`).  
Per reinizializzare anche i sottomoduli, bisogna entrare nelle loro cartelle e, per ognuno, eseguire lo stesso comando precedente.  
L'alternativa più veloce è farlo con il comando:  
> `$ git submodule foreach --recursive 'git init'`

Riassumendo, per fare tutto con **un unico comando** direttamente dalla cartella del repo principale:
> `$ git init && git submodule foreach --recursive 'git init'`

Se la procedura è stata eseguita correttamente, essendo un parametro globale, ogni nuovo repository che verrà clonato con `git clone` conterrà automaticamente lo script di hook `post-merge` senza bisogno di ulteriori azioni.

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

----

In alternativa, esiste anche il comando (non testato):
> `$ git config --global core.hooksPath /path/to/global/hooks`  

che definisce una cartella globale per gli hook che verrà usata da tutti i repository. In questo caso, i repository non eseguiranno più gli hook all'interno di .git/hooks ma solamente quelli della cartella globale.  
Per eseguire _anche_ quelli locali, serve inserire del codice aggiuntivo negli script globali (istruzioni da chatgpt, non verificato):
```bash
#!/bin/bash

# Esegui logica globale
echo "Esecuzione dell'hook globale post-merge"

# Se esiste un hook locale, eseguilo
if [ -x "$(git rev-parse --git-dir)/hooks/post-merge-local" ]; then
    "$(git rev-parse --git-dir)/hooks/post-merge-local"
fi
```


## suggerimenti

**VERSIONAMENTO e COMMIT**
Per un versioning più agevole, adottare un approccio **bottom-up**:  
**se il sottomodulo `A` dipende dal sottomodulo `C` e il sottomodulo `C` dipende dal sottomodulo `B`**, e si devono fare alcune modifiche al codice di tutti questi moduli, **aggiornare per primo `B`** e fare il commit. Poi, fare il **pull del modulo `B` dal modulo `C`**, aggiornare il codice in `C` e fare il **commit di `C`** (il commit aggiornerà anche il riferimento all'attuale commit di `B`). Infine, fare il **pull di `C` da `A`** (in questo modo, `B` inerno a `C` sarà automaticamente aggiornato grazie al commit di `C`), fare attenzione a **sincronizzare** (se necessario) **tutti i `B`** alla stessa versione, aggiornare il codice di `A` e fare il **commit di `A`**, che conterrà quindi i riferimenti aggiornati di tutti gli altri repository a cascata.

## note

**ATTENZIONE**:  
Per i nostri progetti che generano come output una .dll, l'unica precauzione da prendere è di ricordarsi che la cartella di output per le .dll è la stessa per tutta la solution!
Pertanto, se un repository ha più submodules che a loro volta dipendono da altri repository (submodules annidati), è probabile che questi debbano essere allineati manualmente TUTTI alla stessa versione/commit per evitare conflitti a runtime!

Questo perchè se il progetto `B` è incluso in VS nella solution, come dipendenza di altri progetti (più di uno), quando viene compilato solo la .dll genrata dall'ultimo progetto compilato verrà mantenuta! È quindi obbligatorio che tutti i repository `B` puntino allo stesso commit!  
Se invece alcuni files sorgenti di `B` vengono inclusi direttamente all'interno di altri progetti e compilati, potrebbe non essere un problema. Tutto dipende dal fatto che alla fine di `B` venga usata una .dll esterna oppure no.  
Inoltre, se un repository (`D`) ha una dipendenza di link da un sottomodulo (`E`), ovviamente VS non compilerà `E` a meno che non venga incluso manualmente nella solution (la solution di `D` include ovviamente `E`, ma la solution di `A` non può saperlo).
Se ad un primo momento questa può sembrare una cosa complicata, in realtà in fase di build verrà probabilmente generato un errore di link che permette di individuare la dipendenza e correggerla facilmente (a meno che non si usino stringhe hard-coded per invocare metodi ecc. come nel caso di Qt).  
È comunque utile perchè, se le dll sono linkate nel progetto, permette di individuare tutte e sole le dipendenze necessarie.
