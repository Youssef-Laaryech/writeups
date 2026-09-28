## 1. C'est quoi SMB ?

**SMB (Server Message Block)** est un protocole utilisé principalement pour permettre à des machines de **partager des fichiers, dossiers, imprimantes et certains services sur un réseau**.

Sur Linux, le logiciel le plus courant qui implémente SMB est **Samba**.

Dans ton scan :

```
139/tcp open  netbios-ssn Samba
445/tcp open  netbios-ssn Samba
```

Donc la machine `10.129.244.177` possède un **serveur Samba**.

Tu peux imaginer SMB comme un **serveur de fichiers distant** :

```
              SMB
┌─────────────┐        ┌─────────────────┐
│    Kali     │ ────── │ 10.129.244.177  │
│             │        │                 │
│ smbclient   │        │ Samba           │
└─────────────┘        │                 │
                       │ ├── public      │
                       │ ├── backup      │
                       │ └── documents   │
                       └─────────────────┘
```

Les dossiers partagés par SMB sont appelés des **shares**.

---

# 2. `smbclient`

C'est probablement le premier outil que tu dois apprendre.

### À quoi ça sert ?

`smbclient` est essentiellement un **client FTP pour SMB**.

Tu peux :

- voir les shares ;
- te connecter à un share ;
- naviguer dans les dossiers ;
- télécharger des fichiers ;
- envoyer des fichiers si tu as les permissions.

### Voir les shares

```
smbclient -L //10.129.244.177 -N
```

Décomposons :

```
smbclient
   │
   ├── -L       → List
   │
   ├── //IP     → serveur SMB
   │
   └── -N       → pas de mot de passe
```

Donc :

> **"Montre-moi les shares de cette machine sans fournir de mot de passe."**

Tu pourrais obtenir quelque chose comme :

```
Sharename       Type      Comment
---------       ----      -------
public          Disk      Public files
backup          Disk      Backup files
IPC$            IPC       IPC Service
```

### Se connecter à un share

Si tu trouves :

```
public
```

tu peux faire :

```
smbclient //10.129.244.177/public -N
```

Tu obtiendras :

```
smb: \>
```

Et là tu es **à l'intérieur du share**.

Quelques commandes importantes :

```
ls
```

→ liste les fichiers.

```
cd folder
```

→ entre dans un dossier.

```
get file.txt
```

→ télécharge un fichier.

```
mget *
```

→ télécharge plusieurs fichiers.

```
pwd
```

→ affiche ton emplacement actuel.

```
exit
```

→ quitte SMB.

---

# 3. `smbmap`

`smbmap` sert davantage à **énumérer rapidement les permissions SMB**.

Au lieu de te connecter manuellement à chaque share, il te donne une vue d'ensemble.

Par exemple :

```
smbmap -H 10.129.244.177
```

Tu pourrais obtenir :

```
Share       Permissions
-----       -----------
public      READ ONLY
backup      NO ACCESS
IPC$        READ ONLY
```

C'est très intéressant parce que tu veux savoir :

> **Qu'est-ce que mon utilisateur peut réellement faire ?**

Les permissions importantes sont généralement :

```
READ
WRITE
READ, WRITE
NO ACCESS
```

### Exemple

Imagine :

```
Share       Permissions
-----       -----------
public      READ
backup      READ, WRITE
admin       NO ACCESS
```

Tu sais immédiatement que :

- `public` → tu peux lire ;
- `backup` → tu peux lire **et écrire** ;
- `admin` → inaccessible.

Tu peux ensuite explorer un share particulier :

```
smbmap -H 10.129.244.177 -r public
```

`-r` signifie **recursive listing**.

---

# 4. Différence `smbclient` vs `smbmap`

C'est une distinction importante :

|Outil|Objectif|
|---|---|
|`smbclient`|Interagir directement avec un share|
|`smbmap`|Énumérer shares + permissions rapidement|

Donc :

```
smbmap
   ↓
"Quels shares existent et qu'est-ce que je peux faire ?"
   ↓
smbclient
   ↓
"Je vais maintenant explorer ce share."
```

---

# 5. `rpcclient`

Celui-ci est un peu différent.

**RPC = Remote Procedure Call.**

Avec Samba, certaines informations peuvent être obtenues via les interfaces RPC, notamment des informations sur :

- utilisateurs ;
- groupes ;
- domaine/workgroup ;
- informations système ;
- certaines politiques.

Tu peux essayer une connexion anonyme :

```
rpcclient -U "" -N 10.129.244.177
```

Si ça fonctionne, tu arrives sur :

```
rpcclient $>
```

Tu peux ensuite demander certaines informations.

Par exemple :

```
enumdomusers
```

→ énumérer les utilisateurs du domaine.

```
enumdomgroups
```

→ énumérer les groupes.

```
querydominfo
```

→ informations sur le domaine.

```
netshareenumall
```

→ énumérer les shares.

Et :

```
exit
```

pour quitter.

---

# 6. Pourquoi utiliser `rpcclient` si `smbclient` existe ?

Parce qu'ils ne répondent pas exactement à la même question.

Imagine que tu enquêtes sur cette machine.

### `smbclient`

Tu demandes :

> "Quels fichiers puis-je voir ?"

```
smbclient -L //10.129.244.177 -N
```

### `smbmap`

Tu demandes :

> "Quels shares existent et quelles sont mes permissions ?"

```
smbmap -H 10.129.244.177
```

### `rpcclient`

Tu demandes :

> "Quelles informations le serveur RPC accepte-t-il de me révéler ?"

```
rpcclient -U "" -N 10.129.244.177
```

---

# 7. Dans TA machine, je ferais ceci

Puisque tu débutes avec SMB, ne cherche pas encore à exploiter quoi que ce soit. Fais simplement cette chaîne :

### Étape 1 — Shares

```
smbclient -L //10.129.244.177 -N
```

### Étape 2 — Permissions

```
smbmap -H 10.129.244.177
```

### Étape 3 — Utilisateurs / domaine

```
rpcclient -U "" -N 10.129.244.177
```

Puis :

```
enumdomusers
enumdomgroups
querydominfo
netshareenumall
```

### Étape 4 — Explorer les shares accessibles

Si, par exemple, tu trouves :

```
public
```

fais :

```
smbclient //10.129.244.177/public -N
```

Puis :

```
ls
```

---

### 🧠 À retenir

Pense à SMB comme à une **bibliothèque distante** :

**`smbmap`** → _Quels rayons existent et ai-je le droit de les consulter ?_

**`smbclient`** → _Je rentre dans un rayon et je consulte les fichiers._

**`rpcclient`** → _Je pose des questions administratives au serveur : utilisateurs, groupes, domaine, etc._

Pour ta machine, **envoie-moi la sortie de `smbclient -L //10.129.244.177 -N`**, et je t'expliquerai chaque ligne et pourquoi elle est intéressante.