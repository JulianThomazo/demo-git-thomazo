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



---

# TP 05 — Git distant avec GitHub (socle)

Dépôt GitHub : https://github.com/JulianThomazo/demo-git-thomazo

## Étape 1 — Le dépôt distant

J’ai créé sur GitHub le dépôt public `demo-git-thomazo`, sans README, sans `.gitignore` et sans licence, pour qu’il soit vide et que le premier push ne soit pas refusé.

Pour utiliser SSH, j’ai d’abord créé une paire de clés et ajouté la clé publique à mon compte GitHub :

```powershell
ssh-keygen -t ed25519 -C "julianthomazo.chevannes@gmail.com"
Get-Content $HOME\.ssh\id_ed25519.pub | Set-Clipboard
ssh -T git@github.com
```

Le test a répondu :

```text
Hi JulianThomazo! You've successfully authenticated, but GitHub does not provide shell access.
```

## Étape 2 — Relier le dépôt du TP 04

J’ai relié mon dépôt `merge-lab` (TP 04) au dépôt GitHub :

```text
PS C:\Users\julia\depots\merge-lab> git remote add origin git@github.com:JulianThomazo/demo-git-thomazo.git
PS C:\Users\julia\depots\merge-lab> git push -u origin main
Enumerating objects: 26, done.
Counting objects: 100% (26/26), done.
Delta compression using up to 12 threads
Compressing objects: 100% (21/21), done.
Writing objects: 100% (26/26), 5.64 KiB | 1.41 MiB/s, done.
Total 26 (delta 5), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (5/5), done.
To github.com:JulianThomazo/demo-git-thomazo.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

Seule la branche `main` a été publiée ; les branches `feature-a`, `feature-b` et `feature-c` restent locales.

## Étape 3 — `git remote -v` et `git branch -vv`

```text
PS C:\Users\julia\depots\merge-lab> git remote -v
origin  git@github.com:JulianThomazo/demo-git-thomazo.git (fetch)
origin  git@github.com:JulianThomazo/demo-git-thomazo.git (push)
PS C:\Users\julia\depots\merge-lab> git branch -vv
  feature-a f513c92 Ajoute la ligne 4 sur feature-a
  feature-b 9c27343 Ajoute la ligne 5 sur feature-b
  feature-c 9216c00 Modifie la ligne 1 sur feature-c
* main      45b9280 [origin/main] Ajoute le journal du TP 04
```

**Ce que signifie `origin/main` :** c’est une branche de suivi distante, c’est-à-dire la copie locale de l’état de la branche `main` du dépôt `origin` (GitHub), telle que mon dépôt l’a vue lors du dernier échange (`push`, `fetch` ou `pull`). Ce n’est pas GitHub en direct : elle n’est mise à jour que lors de ces échanges. Les crochets `[origin/main]` indiquent que ma branche `main` locale la suit ; l’absence de mention `ahead` ou `behind` signifie que les deux sont au même commit. Les branches `feature-*` n’ont rien entre crochets, car elles ne suivent aucune branche distante.

**Pourquoi `-u` ne se met qu’une fois :** `-u` (`--set-upstream`) enregistre le lien entre `main` et `origin/main` dans la configuration du dépôt (`.git/config`). Ce lien est permanent : ensuite, `git push` et `git pull` sans argument savent vers quel distant et quelle branche aller.

## Étape 4 — L’autre protocole

J’avais utilisé SSH ; j’ai ajouté un second remote en HTTPS :

```text
PS C:\Users\julia\depots\merge-lab> git remote add https-origin https://github.com/JulianThomazo/demo-git-thomazo.git
PS C:\Users\julia\depots\merge-lab> git remote -v
https-origin    https://github.com/JulianThomazo/demo-git-thomazo.git (fetch)
https-origin    https://github.com/JulianThomazo/demo-git-thomazo.git (push)
origin  git@github.com:JulianThomazo/demo-git-thomazo.git (fetch)
origin  git@github.com:JulianThomazo/demo-git-thomazo.git (push)
PS C:\Users\julia\depots\merge-lab> git push https-origin main
info: please complete authentication in your browser...
Everything up-to-date
```

**Ce que chaque protocole demande pour s’authentifier :**

- **SSH** (`origin`) : une paire de clés. La clé publique est enregistrée sur le compte GitHub, la clé privée reste sur mon ordinateur et ne doit jamais être partagée. Une fois la clé ajoutée, il n’y a ni mot de passe ni navigateur à chaque push.
- **HTTPS** (`https-origin`) : le compte GitHub lui-même. Git pour Windows (Git Credential Manager) a ouvert le navigateur pour que je me connecte, puis a mémorisé un jeton d’accès. GitHub n’accepte plus le mot de passe du compte en ligne de commande : il faut passer par le navigateur ou par un jeton d’accès personnel.

Remarque : plus tard, `git lg` affichait `https-origin/main` resté sur `45b9280`, alors que `origin/main` avait avancé. Chaque remote a ses propres branches de suivi, mises à jour seulement lors d’un échange avec ce remote : cela confirme qu’`origin/main` est un souvenir de GitHub, pas GitHub lui-même.

## Étape 5 — Le README, aller simple

```text
PS C:\Users\julia\depots\merge-lab> Set-Content README.md "# demo-git-thomazo","Julian Thomazo"
PS C:\Users\julia\depots\merge-lab> git add README.md
PS C:\Users\julia\depots\merge-lab> git commit -m "Ajoute le README"
[main abafdfa] Ajoute le README
 1 file changed, 2 insertions(+)
 create mode 100644 README.md
PS C:\Users\julia\depots\merge-lab> git push
...
To github.com:JulianThomazo/demo-git-thomazo.git
   45b9280..abafdfa  main -> main
```

Grâce au `-u` de l’étape 2, `git push` n’a eu besoin d’aucun argument. J’ai vérifié sur GitHub que le commit `abafdfa` et le README y apparaissaient.

## Étape 6 — Modification en ligne

Depuis l’interface GitHub, j’ai modifié `README.md` (icône crayon, puis **Commit changes**). Deux commits ont été créés en ligne : `56c6f6b Update README.md` (message par défaut de GitHub) puis `41ccdcb Complete le README en ligne`. Mon dépôt local était alors en retard sur GitHub.

## Étape 7 — Prédiction et `git pull`

**Prédiction (14:47) :** avance rapide, car je n’ai fait aucun commit local depuis mon dernier push ; seul GitHub a avancé.

**Résultat réel :**

```text
PS C:\Users\julia\depots\merge-lab> git pull
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 4 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
Unpacking objects: 100% (4/4), 1.84 KiB | 50.00 KiB/s, done.
From github.com:JulianThomazo/demo-git-thomazo
   abafdfa..41ccdcb  main       -> origin/main
Updating abafdfa..41ccdcb
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
```

La prédiction est confirmée : Git a fait une avance rapide, sans commit de fusion ni conflit. La sortie montre les deux temps de `git pull` : d’abord le téléchargement (`From github.com…`, `origin/main` passe de `abafdfa` à `41ccdcb`), puis la fusion (`Updating… Fast-forward`, `main` avance jusqu’à `origin/main`).

```text
* 41ccdcb (HEAD -> main, origin/main, origin/HEAD) Complete le README en ligne
* 56c6f6b Update README.md
* abafdfa Ajoute le README
* 45b9280 (https-origin/main) Ajoute le journal du TP 04
```

## Étape 8 — `git fetch` seul

J’ai fait une seconde modification en ligne (`Modifie encore le README en ligne`), puis j’ai lancé `git fetch` sans `git pull` :

```text
PS C:\Users\julia\depots\merge-lab> git status
On branch main
Your branch is behind 'origin/main' by 1 commit, and can be fast-forwarded.
  (use "git pull" to update your local branch)

nothing to commit, working tree clean
PS C:\Users\julia\depots\merge-lab> git lg
* 9b38381 (origin/main, origin/HEAD) Modifie encore le README en ligne
* 41ccdcb (HEAD -> main) Complete le README en ligne
* 56c6f6b Update README.md
* abafdfa Ajoute le README
* 45b9280 (https-origin/main) Ajoute le journal du TP 04
*   56cce19 Merge branch 'feature-c'
|\
| * 9216c00 (feature-c) Modifie la ligne 1 sur feature-c
* | 2c4d401 Modifie la ligne 1 sur main
|/
*   5300f4b Merge branch 'feature-b'
|\
| * 9c27343 (feature-b) Ajoute la ligne 5 sur feature-b
* | 125a95d Ajoute autre.txt sur main
|/
* f513c92 (feature-a) Ajoute la ligne 4 sur feature-a
* f5a85bb Premier commit avec fichier.txt
```

(`git lg` est mon alias de `git log --oneline --graph --decorate --all`.)

**Où se trouve le nouveau commit ?** Après `git fetch`, le commit `9b38381` est sur `origin/main` uniquement ; ma branche `main` est restée sur `41ccdcb`, et `git status` indique qu’elle est en retard d’un commit. `git fetch` télécharge les commits et met à jour les branches de suivi distantes, mais ne touche ni à mes branches locales ni à mes fichiers. `git pull`, lui, fait ce même `fetch` puis fusionne `origin/main` dans `main` : c’est ce qu’on voit dans les deux parties de sa sortie à l’étape 7.

## Vérification finale

```text
PS C:\Users\julia\depots\merge-lab> git pull
Updating 41ccdcb..9b38381
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
PS C:\Users\julia\depots\merge-lab> git status
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
PS C:\Users\julia\depots\merge-lab> git log --oneline -1
9b38381 (HEAD -> main, origin/main, origin/HEAD) Modifie encore le README en ligne
```

Ce dernier `git pull` n’a pas de partie `From github.com…` : le `git fetch` précédent avait déjà tout téléchargé, il ne restait que l’avance rapide. Mon poste et GitHub affichaient alors la même empreinte de tête, `9b38381`.

## Conclusion

Ce TP m’a montré que le dépôt local n’est qu’une copie parmi d’autres. Les branches de suivi comme `origin/main` sont le souvenir local de l’état du dépôt distant, mis à jour uniquement par `push`, `fetch` et `pull`. `git fetch` permet de regarder ce qui est arrivé sans rien modifier, et `git pull` l’intègre ensuite. Enfin, SSH et HTTPS mènent au même dépôt, mais s’authentifient différemment : par une paire de clés pour SSH, par le compte GitHub et un jeton pour HTTPS.
