# TP3 : Cloud Platform

## Mise en place

### 1. Frontend

Installation de OpenNebula:
Première étape, désactiver SELinux on modifion la ligne "SELINUX=disabled" dans /etc/selinux/config. Cela permet d'éviter des erreurs d'opérations fait par OpenNebula.

En suite on passe à l'installation du Repositories OpenNebula. On utilise la Community Edition avec la commande suivante 
cat << "EOT" > /etc/yum.repos.d/opennebula.repo
[opennebula]
name=OpenNebula Community Edition
baseurl=https://downloads.opennebula.io/repo/5.12/CentOS/7/$basearch
enabled=1
gpgkey=https://downloads.opennebula.io/repo/repo.key
gpgcheck=1
repo_gpgcheck=1
EOT

yum makecache fast

Pour finir l'installation du Software. On installe d'abord EPEL dont OpenNebula est dépendant 
"yum install epel-release".
On peut maintenant installer OpenNebula 
"yum install opennebula-server opennebula-sunstone opennebula-ruby opennebula-gate opennebula-flow"

#### Starting OpenNebula

Tout d'abord on change le mot de passe de oneadmin "echo 'oneadmin:PASSWORD' > /var/lib/one/.one/one_auth".

On lance ensuite le daemon OpenNebula 
"systemctl start opennebula
systemctl start opennebula-sunstone".

Pour vérifier que OpenNebula soit bien lancé on utilise la commande "oneuser show".

Maintenant on a accès à l'interface graphique via l'adresse http://10.100.100.11:9869

### 2. Virtu

Pour commencer on installe le repositorie d'OpenNebula sur KVM1.tp3.
SELinux doit aussi être désactivé.
on installe ensuite EPEL.

Sur kvm1.tp3 on installe les paquets Node:
sudo yum install opennebula-node-kvm
sudo systemctl restart libvirtd

La prochaine étape est de désactiver le password SSH.
Avec l'utilisateur Oneadmin on utilise la commande "ssh-keyscan 10.100.100.11 10.100.100.12 >> /var/lib/one/.ssh/known_hosts" pour créer le fichier Known_hosts.
Il faut ensuite copier le fichier known_hosts et la clé public anssi que l'id_rsa sur les autres nodes.

On désactive ensuite le mot de passe pour pouvoir ce connecter "ssh-copy-id -i /var/lib/one/.ssh/id_rsa.pub 10.100.100.12".
On distribue ensuite le fichier known_hosts avec la commande suivante "scp -p /var/lib/one/.ssh/known_hosts 10.100.100.12:/var/lib/one/.ssh/".
On distribue aussi la clé privé pour que les nodes se connectent au front end "scp -p /var/lib/one/.ssh/id_rsa 10.100.100.12:/var/lib/one/.ssh/"


### 3. Test

## 1.5 : Entracte Ansible

### 2. Adapt

Pour créer un vagrantFile pour déployer des VMs avec Ansible prêt au démarage, on configure d'abord une première VM avec pyhton et Ansible d'instller.
Il ne faut pas oublier d'installer le repo d'OpenNebula.
on créer ensuite les fichier hosts.ini et opennebula.yml

### 3. Act

installer OpenNebula avec ansible: opennebula.yml
      
Pour lancer Ansible on utilise la commande "ansible-playbook -i hosts.ini opennebula.yml -kK"
      
Il ne reste plus qu'à changer le mot de passe oneuser 
"echo 'oneadmin:PASSWORD' > /var/lib/one/.one/one_auth"
et lancer le service opennebula et opennebula-sunstone.

## II. Going further

### 1. Ajouter un noeud

On édite un nouveau fichier yml 
dans le fichier hosts.ini on renseigne l'ip de la nouvelle machine.

opennebula-node.yml installe le node package.