# Configurer Linux Ubuntu

`[]` = Contenue optionel ou variable (ne pas écrire les `[]`)

## Commandes utiles

[Commmandes Utiles](https://juliend.github.io/linux-cheatsheet/#chmod)

- `cat [nomdu fichier]` : Consulter un fichier
- `nano [nom du fichier]` : Editer un fichier dans l'éditeur nano(s'ouvre dans la console)
- `echo [PORT = 3500] > [.env]` : Edite et crée un fichier à la volée si il n'existe pas (directement dans la console)
  - Un `>` crée le fichier ou l'écrase si il existe
  - Deux `>>` insère dans le fichier
- `mkdir [dossier ou chemin dossier]`: Créer un dossier
- `touch [fichier ou chemin fichier]`: créer un fichier
- `rm -rf [dossier]` : supprime dossier nom vide
- `/~` : Se rendre dans son dossier utilisateur (équivalent à `/home/[user]`), dans ce dossier on a les droits de lecture, d'écriture, d'execution.

#### WSL:

- Basculer entre la VM et WSL :

  - Passer sur la VM: `bcdedit /set hypervisorlaunchtype off`
  - Passer sur WSL: `bcdedit /set hypervisorlaunchtype auto`
  - redémarer à chaque fois

- `notepad.exe [nomdufichier]` : Ouvre le fichier avec l'éditeur de texte windows`
- `explorer.exe .` : Ouvre l'explorer windows dans le dossier courant
- `code .` : Ouvre VsCode dans le dossier courant

---

---

<br>

# Config Linux de zéro

- Au premier lancement renseigner le nom du `user` et un` mdp`
- Mettre à jour le package : `sudo apt update`

## **Namespaces et Cgroup**

_Manip à faire uniquement sur un linux WSL_

#### Les namespcaces et Cgroup sont des fonctionalitées manquante des versions WSL de linux et poutant indispensable aux fonctionnement de certaines applications notament `Docker ` et `mongoDB`

[Plus d'info](https://blog.ineat-group.com/2021/02/comment-bien-demarrer-avec-wsl2-windows-10/)
<br>

### Cgroup

- Rentrer seulement ces deux lignes de commandes
  - `sudo mkdir /sys/fs/cgroup/systemd`
  - `sudo mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd`

### NameSpaces

[genie](https://github.com/arkane-systems/genie)

- `sudo apt install apt-transport-https`
- `sudo wget -O /etc/apt/trusted.gpg.d/wsl-transdebian.gpg https://arkane-systems.github.io/wsl-transdebian/apt/wsl-transdebian.gpg`
- `chmod a+r /etc/apt/trusted.gpg.d/wsl-transdebian.gpg`
- `sudo touch /etc/apt/sources.list.d/wsl-transdebian.list` : Crée le fichier
- `sudo nano /etc/apt/sources.list.d/wsl-transdebian.list` : Editer le fichier
- Ecrire ces deux lignes :
  - `deb https://arkane-systems.github.io/wsl-transdebian/apt/ bullseye main`
  - `deb-src https://arkane-systems.github.io/wsl-transdebian/apt/ bullseye main`
- Enregistrer et fermer l'éditeur (`ctrl + x ` puis `y` puis `Entrée`)
- `sudo apt update`
- `sudo apt install -y systemd-genie`
- Couper l'instance de wsl : `wsl --shutdown`
- Lancer la commande `wsl genie -s` depuis le shell windows
- A partir de maintenant il faut toujours lancer WSL avec cette commande si l'on utilise des appli comme Docker, MongoDB ou d'autres appli qui necessites les NameSpaces

**ATTENTION: Quand on lance wsl avec "genie" le PATH windows n'est plus disponible et donc les commandes comme `explorer.exe` ou `code .` ne sont plus utilisable. Une solution est d'installé une extention vscode `remote-WSL` qui permet d'ouvrir le répectoire WSL en 1 clic.**

---

## **Serveur apache2**

[Apache2](https://doc.ubuntu-fr.org/apache2)
<br>
[Permissions](https://doc.ubuntu-fr.org/permissions)

- `sudo apt install apache2`
- `sudo service apache2 start`

- le dossier `/var/www/html` est celui qui est utilisé par défault pour le serveur mais comme il fait partie des dossiers racine on a pas les droits d'écrtiture dessus...

  Pour s'attribuer les droit d'écriture:

- ` sudo chown [geoffrey] /var/www/html`

---

## **Installer Git**

- `sudo apt install git`
- `git config --global user.name "username"`
- `git config --global user.email "mail@gmail.com"`
- `git config --global core.editor nano`
- `git config --global color.ui true `
- `git config -l`

- créer clé SSH:
  - `ssh-keygen`: Générer la clé ssh
  - Se rendre à `~/.ssh`
  - Consulter le fichier id_rda.pub: `cat id_rsa.pub`
  - Copier la clé: Commence par `ssh-rsa` et finis par le nom de l'utilisateur
  - Copier la clé la ou on en a besoin( github, AWS )

---

## **postgresQl**

- `sudo apt install postgresql postgresql-contrib` : Installer postgresql
- `sudo service postgresql star` : Lancer le service
- Configurer le fichier `pg_hba.conf` pour povoir se connecter par mot de passe:
  - Se rendre dans le dossier `/etc/postgresql/[12]/main` ,
  - `sudo nano pg_hba.conf`: Ouvre le fichier avec l'éditeur
  - Modifier la ligne `local all all peer ` par `local all all md5`
  - Sauvergarder les modifs, fermer le fichier et relancer le serveur:
  - `sudo service postgresql restart`
- Créer la base:

  - Se connecter en superUser postgres : `sudo -i -u postgres` puis `psql`
  - Créer un rôle : ` CREATE ROLE nomOwner WITH LOGIN PASSWORD 'unMdp'`
  - Créer une base : `DATABASE nomDB OWNER nomOwner`
  - Quitter la BDD `\q` puis quitter l'instance postgres `exit`
  - Se connecter à la base nouvelement créer: `psql -h localhost -U nomowner [-d nomBase]`

---

## **NodeJS**

#### Installer la dernière version de node

[Nodejs](https://docs.aws.amazon.com/fr_fr/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html)

- ` curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash` : installer NVM
- `. ~/.nvm/nvm.sh` : Activer NVM
- `nvm install node` : Installer Node
- `node -e "console.log('Running Node.js ' + process.version` : Tester le fonctionement de Node

#### Lancer une version spécifique de Node

- ` curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash`
- `nvm ls` : Voir les verions de Node disponibles
- `nvm [14.16]` : Utiliser une version spécifique de Node

---

- ## **MongoDB**

#### Instaler la denière version de MongoDB

_Ne pas utiliser npm, le package n'est pas à jour et n'est plus maintenus (v3.6 au lieu de 4.4) _

_Faire absolument l'étape NameSpaces avant!_

[MongoDB](https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-20-04-fr)

- `curl -fsSL https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add - `
- `echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list`
- `sudo apt update`
- `sudo apt install mongodb-org`
- Lancer le service : `sudo systemctl start mongod.service`
- Commandes services mongod: `sudo systemctl [stop, start, restart, status] mongod`
- Lancer le service automatiquement: `sudo systemctl enable mongod`
- Désactiver le lancement auto `sudo systemctl disable mongod`
- Vérifier l'état et les connections de la BDD `mongo --eval 'db.runCommand({ connectionStatus: 1 })'`

<br>

- Lancer le client : `mongo`
- Voir les bases : `show dbs`
- Se placer dans une table : `use [table]`
- Voir la/les collections de la table: `show collections`
