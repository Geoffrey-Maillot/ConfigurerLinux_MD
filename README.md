# Configurer Linux Ubuntu (wsl2)

`[]` = Contenue optionel ou variable retirer les crochets quand on copie la ligne de code.

`$` = Indique une commande, copier tous ce qui suit jusqu'au commentaire ou au prochain `$`, Il ne fait pas partie de la commande ne pas le copier.

## Commandes utiles

[Commmandes Utiles](https://juliend.github.io/linux-cheatsheet/#chmod)

```bash
$cat [nomdu fichier] #Consulter un fichier
$nano [nom du fichier] #Editer un fichier dans l'éditeur nano(s'ouvre dans la console)
$echo [PORT = 3500] > [.env] #Edite et crée un fichier à la volée si il n'existe pas (directement dans la console)
      #Un `>` crée le fichier ou l'écrase si il existe
      #Deux `>>` insère dans le fichier
$mkdir [dossier ou chemin dossier] #Créer un dossier
$touch [fichier ou chemin fichier] #créer un fichier
$rm -rf [dossier] #supprime dossier nom vide
$ /~ #Se rendre dans son dossier utilisateur (équivalent à `/home/[user]`), dans ce dossier on a les droits de lecture, d'écriture, d'execution.
```

#### WSL:

- Basculer entre la VM et WSL :

  ```bash
  $bcdedit /set hypervisorlaunchtype off #Passer sur la VM
  $bcdedit /set hypervisorlaunchtype auto #Passer sur WSL
  ```

  - redémarer à chaque fois

```bash
$notepad.exe [nomdufichier] #Ouvre le fichier avec l'éditeur de texte windows
$explorer.exe . #Ouvre l'explorer windows dans le dossier courant
$code . #Ouvre VsCode dans le dossier courant
```

- Se rendre dans les fichiers wsl depuis l'explorateur, taper `\\wsl$` dans la barre d'adresse de l'explorateur puis `Enter`

---

---

<br>

# Config Linux de zéro

- Au premier lancement renseigner le nom du `user` et un` mdp`
- Mettre à jour le package :

```bash
$sudo apt update
```

- ## **Namespaces et Cgroup**

_Manip à faire uniquement sur un linux WSL_

#### Les namespcaces et Cgroup sont des fonctionalitées manquante des versions WSL de linux et poutant indispensable aux fonctionnement de certaines applications notament `Docker ` et `mongoDB`

[Plus d'info](https://blog.ineat-group.com/2021/02/comment-bien-demarrer-avec-wsl2-windows-10/)
<br>

### Cgroup

- Rentrer seulement ces deux lignes de commandes

```bash
  $sudo mkdir /sys/fs/cgroup/systemd
  $sudo mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd
```

### NameSpaces

[genie](https://github.com/arkane-systems/genie)

```bash
$sudo apt install apt-transport-https
$sudo wget -O /etc/apt/trusted.gpg.d/wsl-transdebian.gpg https://arkane-systems.github.io/wsl-transdebian/apt/wsl-transdebian.gpg
$chmod a+r /etc/apt/trusted.gpg.d/wsl-transdebian.gpg
$sudo touch /etc/apt/sources.list.d/wsl-transdebian.list  #Crée le fichier
$sudo nano /etc/apt/sources.list.d/wsl-transdebian.list  #Editer le fichier
```

- Ecrire ces deux lignes :
  - `deb https://arkane-systems.github.io/wsl-transdebian/apt/ bullseye main`
  - `deb-src https://arkane-systems.github.io/wsl-transdebian/apt/ bullseye main`
- Enregistrer et fermer l'éditeur (`ctrl + x ` puis `y` puis `Entrée`)

```bash
$sudo apt update
$sudo apt install -y systemd-genie
```

- Depuis le bash windows:

```bash
$wsl --shutdown #Couper l'instance de wsl
$wsl genie -s #Lance Ubuntu avec genie
```

- A partir de maintenant il faut toujours lancer WSL avec cette commande si l'on utilise des appli comme Docker, MongoDB ou d'autres appli qui necessites les NameSpaces

**ATTENTION: Quand on lance wsl avec "genie" le PATH windows n'est plus disponible et donc les commandes comme `explorer.exe` ou `code .` ne sont plus utilisable. Une solution est d'installé une extention vscode `remote-WSL` qui permet d'ouvrir le répectoire WSL en 1 clic.**

---

## **Serveur apache2**

[Apache2](https://doc.ubuntu-fr.org/apache2)
<br>
[Permissions](https://doc.ubuntu-fr.org/permissions)

```bash
$sudo apt install apache2
$sudo service apache2 start # lancer le servive
```

- le dossier `/var/www/html` est celui qui est utilisé par défault pour le serveur mais comme il fait partie des dossiers racine on a pas les droits d'écrtiture dessus... (donc pas de git clone, de npm i etc...)

  Pour s'attribuer les droit d'écriture:

```bash
 $sudo chown [user] /var/www/html
```

---

## **Installer Git**

```bash
$sudo apt install git
$git config --global user.name "Votre Nom"
$git config --global user.email "Votre Mail"
$git config --global core.editor nano
$git config --global color.ui true
$git config -l # Controler les configs
```

- créer clé SSH:

```bash
$ssh-keygen #Générer la clé ssh
$cat ~/.ssh/id_rsa.pub #Consulter le fichier id_rsa.pub
```

- Copier la clé: Commence par `ssh-rsa` et finis par le nom de l'utilisateur
- Copier la clé la ou on en a besoin( github, AWS )

---

## **postgresQl**

```bash
$sudo apt install postgresql postgresql-contrib #Installer postgresql
$sudo service postgresql star #Lancer le service
```

Configurer le fichier `pg_hba.conf` pour pouvoir se connecter aux BDD par mot de passe:

```bash
 $cd /etc/postgresql/[12]/main #Se rendre dans le dossier de config
$sudo nano pg_hba.conf #Ouvre le fichier avec l'éditeur
```

- Modifier la ligne `local all all peer ` par `local all all md5`
- Sauvergarder les modifs, fermer le fichier et relancer le serveur:

```bash
  $sudo service postgresql restart
```

- Créer la base:
  - Se connecter en superUser postgres :

```bash
   $sudo -i -u postgres #puis
   $psql
   $CREATE ROLE nomOwner WITH LOGIN PASSWORD 'unMdp' # Créer un rôle
   $DATABASE nomDB OWNER nomOwner # Créer une base
   $\q # Quitter la bdd
   $exit # Quitter l'instance postgres
```

- Se reconnecter avec le role nouvellement créer:

```bash
 $psql -h localhost -U nomowner [-d nomBase]  #puis le mot de passe
```

---

## **NodeJS**

#### Installer la dernière version de node

[Nodejs](https://docs.aws.amazon.com/fr_fr/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html)

```bash
$curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash # installer NVM
$ . ~/.nvm/nvm.sh #Activer NVM
$ nvm install node # Installer Node
$node -e "console.log('Running Node.js ' + process.version)"  #Tester le fonctionement de Node
```

#### Lancer une version spécifique de Node

```bash
$curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash # A refaire même si ça a été fait à l'install de Node
$nvm ls #Voir les verions de Node disponibles
$nvm install [14.16] #Installer une version spécifique
$nvm use [14.16] #Utiliser la version spécifique
```

---

## **MongoDB**

#### Instaler la denière version de MongoDB

_Ne pas utiliser npm, le package n'est pas à jour et n'est plus maintenus (v3.6 au lieu de 4.4)_

_Faire absolument l'étape NameSpaces avant!_

[MongoDB](https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-20-04-fr)

```bash
$curl -fsSL https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
$echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
$sudo apt update
$sudo apt install mongodb-org
$sudo systemctl start mongod.service #Lancer le service
$sudo systemctl [stop, start, restart, status] mongod # Services Mongod
$sudo systemctl enable mongod #Lancer le service automatiquement
$sudo systemctl disable mongod #Désactiver le lancement auto
$mongo --eval 'db.runCommand({ connectionStatus: 1 })' #Vérifier l'état et les connections de la BDD
```

<br>

```bash
$mongo #Lancer le client
$show dbs #Voir les bases
$use [table] #Se placer dans une table
$show collections #Voir la/les collections de la table
```

## **Strapi**

_Préférer la version du gerstionaire de paquet `yarn`_

- Switcher sur la dernière version comptatible de Node(14.16):

```bash
$nvm install 14.16 # Installe et switch sur cette version en même temps.
```

- Installer le gestionnaire de paquet yarn:

```bash
$npm install --global yarn

```

- Installer Strapi:

```bash
$yarn add strapi
```

- Lancer un projet Strapi:

  - Créer un nouveau rôle et une nouvelle BDD sur postgresql

  - Se placer dans dossier et lancer le projet:

  ```bash
  $yarn create strapi-app [nomProjet]
  ```

  -Répondre aux questions:

```bash
? Choose your installation type Custom (manual settings) # Choisir une instalation custom
? Choose your default database client postgres
? Database name: oblog # nom BDD
? Host: 127.0.0.1 # localhost
? Port: 5432 #port par défault strapi
? Username: oblog # Owner
? Password: ***** #Mot de passe Bdd
? Enable SSL connection: No # répondre No
```

- Lancer le projet:

```bash
$yarn strapi develop
```

- Le projet se lance et une page html s'ouvre à cet adresse: `http://localhost:1337/admin/auth/register-admin`

Pour la suite : [strapi](https://strapi.io/documentation/developer-docs/latest/getting-started/introduction.html)
