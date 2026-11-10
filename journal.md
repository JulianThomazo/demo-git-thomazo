# TP 04 — Maîtriser les fusions

Dépôt : `C:\\Users\\julia\\depots\\merge-lab`

## 1\. L’avance rapide

### Étapes 1 et 2

J’ai créé le dépôt `merge-lab` avec un premier commit contenant `fichier.txt` (trois lignes), puis la branche `feature-a`, sur laquelle j’ai ajouté une quatrième ligne :

```powershell
git switch -c feature-a
Add-Content fichier.txt "ligne 4 ajoutee sur feature-a"
git commit -am "Ajoute la ligne 4 sur feature-a"
```

### Étape 3 — Prédiction et résultat

**Prédiction :** J’attendais un commit de fusion, car je pensais que toute fusion de branches en créait un.

**Résultat :**

```text
PS C:\\Users\\julia\\depots\\merge-lab> git merge feature-a
Updating f5a85bb..f513c92
Fast-forward
 fichier.txt | 1 +
 1 file changed, 1 insertion(+)
```

Première sortie de `git lg` :

```text
\* f513c92 (HEAD -> main, feature-a) Ajoute la ligne 4 sur feature-a
\* f5a85bb Premier commit avec fichier.txt
```

Git a affiché `Fast-forward` et aucun commit de fusion n’a été créé : `main` et `feature-a` pointent sur le même commit.

### Étape 4 — Pourquoi pas de commit de fusion ?

Git n’a pas créé de commit de fusion parce que `main` n’avait reçu aucun nouveau commit depuis la création de `feature-a` : `main` était un ancêtre direct de `feature-a`, il suffisait donc d’avancer l’étiquette `main` jusqu’au dernier commit de `feature-a`.

## 2\. Forcer le commit de fusion

### Étape 5 — Faire diverger les branches

J’ai créé `feature-b` avec un commit sur `fichier.txt`, puis j’ai fait un commit sur `main` dans un autre fichier, `autre.txt` :

```powershell
git switch -c feature-b
Add-Content fichier.txt "ligne 5 ajoutee sur feature-b"
git commit -am "Ajoute la ligne 5 sur feature-b"
git switch main
Set-Content autre.txt "fichier cree sur main"
git add autre.txt
git commit -m "Ajoute autre.txt sur main"
```

Les deux branches ont alors divergé :

```text
\* 125a95d (HEAD -> main) Ajoute autre.txt sur main
| \* 9c27343 (feature-b) Ajoute la ligne 5 sur feature-b
|/
\* f513c92 (feature-a) Ajoute la ligne 4 sur feature-a
\* f5a85bb Premier commit avec fichier.txt
```

### Étape 6 — Fusion avec `--no-ff`

```powershell
git merge --no-ff feature-b
```

Git a ouvert l’éditeur (Notepad) avec le message `Merge branch 'feature-b'`, que j’ai conservé.

```text
PS C:\\Users\\julia\\depots\\merge-lab> git cat-file -p HEAD | Select-String parent

parent 125a95dfbd28c96185eb128910133c2d3e9ba88d
parent 9c273430efff1cbff6061a17c0899fa13e0fbb51
```

**Réponse :** le commit de fusion a **deux parents** :

* `125a95d`, le dernier commit de `main` (premier parent : la branche sur laquelle j’étais) ;
* `9c27343`, le dernier commit de `feature-b` (second parent : la branche fusionnée).

Un commit ordinaire n’a qu’un parent, et le premier commit du dépôt n’en a aucun. Sous PowerShell, j’ai remplacé `grep` par `Select-String`.

Remarque : les branches ayant divergé, Git aurait créé un commit de fusion même sans `--no-ff`. Cette option sert surtout lorsqu’une avance rapide serait possible : elle force la création du commit de fusion.

### Étape 7 — La forme visée

Deuxième sortie de `git lg` : `main` et `feature-b` se séparent après `f513c92`, puis se rejoignent dans le commit de fusion `5300f4b`.

```text
\*   5300f4b (HEAD -> main) Merge branch 'feature-b'
|\\
| \* 9c27343 (feature-b) Ajoute la ligne 5 sur feature-b
\* | 125a95d Ajoute autre.txt sur main
|/
\* f513c92 (feature-a) Ajoute la ligne 4 sur feature-a
\* f5a85bb Premier commit avec fichier.txt
```

## 3\. Le conflit, provoqué puis résolu

### Étape 8 — Modifier la même ligne sur deux branches

Sur `feature-c`, la ligne 1 est devenue `ligne 1 version feature-c` (commit `9216c00`). Sur `main`, la même ligne est devenue `ligne 1 version main` (commit `2c4d401`).

```text
\* 2c4d401 (HEAD -> main) Modifie la ligne 1 sur main
| \* 9216c00 (feature-c) Modifie la ligne 1 sur feature-c
|/
\*   5300f4b Merge branch 'feature-b'
|\\
| \* 9c27343 (feature-b) Ajoute la ligne 5 sur feature-b
\* | 125a95d Ajoute autre.txt sur main
|/
\* f513c92 (feature-a) Ajoute la ligne 4 sur feature-a
\* f5a85bb Premier commit avec fichier.txt
```

### Étape 9 — Le conflit

Message exact de Git :

```text
PS C:\\Users\\julia\\depots\\merge-lab> git merge feature-c
Auto-merging fichier.txt
CONFLICT (content): Merge conflict in fichier.txt
Automatic merge failed; fix conflicts and then commit the result.
```

Contenu exact de `fichier.txt` en conflit :

```text
<<<<<<< HEAD
ligne 1 version main
=======
ligne 1 version feature-c
>>>>>>> feature-c
ligne 2
ligne 3
ligne 4 ajoutee sur feature-a
ligne 5 ajoutee sur feature-b
```

* Entre `<<<<<<< HEAD` et `=======` : la version de la branche courante (`main`).
* Entre `=======` et `>>>>>>> feature-c` : la version de la branche fusionnée.
* Les lignes 2 à 5 sont hors des marqueurs : Git les a fusionnées sans difficulté.

`git status` indiquait :

```text
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   fichier.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

### Étape 10 — Résolution

J’ai réécrit le fichier avec une ligne 1 qui combine les deux versions, sans marqueurs :

```text
ligne 1 version main et feature-c
ligne 2
ligne 3
ligne 4 ajoutee sur feature-a
ligne 5 ajoutee sur feature-b
```

**Si l’on oublie `git add`**, Git refuse de commiter :

```text
PS C:\\Users\\julia\\depots\\merge-lab> git commit -m "Merge branch 'feature-c'"
U       fichier.txt
error: Committing is not possible because you have unmerged files.
hint: Fix them up in the work tree, and then use 'git add/rm <file>'
hint: as appropriate to mark resolution and make a commit.
fatal: Exiting because of an unresolved conflict.
```

Le `U` signifie *unmerged* : le fichier reste marqué en conflit dans l’index. Git ne vérifie pas le contenu du fichier : c’est `git add` qui lui indique que le conflit est résolu.

Après `git add fichier.txt`, `git status` indiquait :

```text
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)
```

**La commande qui termine une fusion est `git commit`** (ou, de façon équivalente, `git merge --continue`) :

```text
PS C:\\Users\\julia\\depots\\merge-lab> git commit -m "Merge branch 'feature-c'"
\[main 56cce19] Merge branch 'feature-c'
```

### Étape 11 — Le commit de fusion et ses deux parents

```text
\*   56cce19 (HEAD -> main) Merge branch 'feature-c'
|\\
| \* 9216c00 (feature-c) Modifie la ligne 1 sur feature-c
\* | 2c4d401 Modifie la ligne 1 sur main
|/
\*   5300f4b Merge branch 'feature-b'
|\\
| \* 9c27343 (feature-b) Ajoute la ligne 5 sur feature-b
\* | 125a95d Ajoute autre.txt sur main
|/
\* f513c92 (feature-a) Ajoute la ligne 4 sur feature-a
\* f5a85bb Premier commit avec fichier.txt
```

Le commit de fusion `56cce19` a pour parents `2c4d401` (dernier commit de `main`) et `9216c00` (dernier commit de `feature-c`).

Vérification de l’absence de marqueurs (équivalent PowerShell de `grep -r '<<<<<<<' .`) :

```powershell
Get-ChildItem -Recurse -File | Select-String '<<<<<<<'
```

La commande ne renvoie rien : il ne reste aucun marqueur de conflit.

## 4\. S’entraîner à la forme (learngitbranching)

J’ai fait les niveaux 1 à 4 de la séquence « Introduction » sur `learngitbranching.js.org`.

### Commandes du niveau 4

```text
git branch bugFix
git checkout bugFix
git commit
git checkout main
git commit
git checkout bugFix
git rebase main
```

### Ce que je retiens sur `rebase`

`git rebase main` prend les commits propres à la branche courante (`bugFix`) et les rejoue un par un au bout de `main`. Au lieu d’une fourche refermée par un commit de fusion, on obtient un historique en ligne droite, comme si le travail sur `bugFix` avait commencé après le dernier commit de `main`.

### Pourquoi le commit rejoué porte un prime (C2 devient C2’)

Le commit rejoué contient la même modification que C2, mais il n’a plus le même parent. Or l’empreinte d’un commit est calculée à partir de tout son contenu, parent compris : c’est donc un nouveau commit, avec une nouvelle empreinte, d’où le prime. L’ancien C2 n’est pas modifié, il est abandonné (le site l’affiche en transparence). C’est le même mécanisme que `git commit --amend` au TP 03.

### Quand il ne faut pas rebaser

Il ne faut pas rebaser des commits déjà poussés et partagés avec d’autres personnes. Le rebase remplace ces commits par de nouveaux : les collègues qui ont travaillé à partir des anciens se retrouvent avec un historique qui ne correspond plus à celui du serveur, ce qui provoque doublons et conflits. On réserve donc `rebase` à ses propres commits locaux, pas encore partagés.

## Conclusion

Ce TP m’a montré que la forme de l’historique décide du type de fusion :

* si la branche de destination n’a pas bougé, Git fait une **avance rapide** (pas de commit de fusion) ;
* si les branches ont divergé, ou avec `--no-ff`, Git crée un **commit de fusion à deux parents** ;
* si les deux branches modifient la même ligne, Git ne peut pas choisir : il signale un **conflit**, qu’on résout à la main, qu’on marque comme résolu avec `git add`, puis qu’on conclut avec `git commit`.

