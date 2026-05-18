# Labo Docker - Rapport

## Docker c'est quoi ?
1. quellle est la différence entre un conteneur et un simple processus linux ?
Un conteneur est une technologie de virtualisation légère qui permet d'exécuter des applications de manière isolée. Il met a disposition un environnement d'exécution complet, y compris les bibliothèques et les dépendances nécessaires.

2. Pourquoi dit-on qu'un conteneur est plus léger qu'une machine virtuelle ?
Le noyau du système d'exploitation est partagé entre les conteneurs, ce qui réduit considérablement la consommation de ressources par rapport à une machine virtuelle qui nécessite un système d'exploitation complet pour chaque instance.

3. Exemple d'utilisation d'un conteneur :
 - Serveur web
 - Base de données
 - app python avec ses dépendances


4. Quels sont les mecanisme du noyau linux que docker utilise pour cree des conteneurs isolés ?
 - chroot : pour isoler le système de fichiers
    - chroot : pour isoler le système de fichiers
    - namespaces : pour isoler les processus, les utilisateurs, les réseaux, etc.
    - cgroups : pour limiter les ressources utilisées par les conteneurs
    - OverlayFS : permet de gagner en efficacité en utilisant des couches en lecture seule partagées entre plusieurs conteneurs ; chaque conteneur ajoute sa propre couche modifiable, et seules les modifications sont copiées (copy-on-write).


## Isolation et securite
1. Un processus linux classique benefice t il deja d'un certains niveau d'isolation ? 
Oui, un processus Linux classique bénéficie d'un certain niveau d'isolation grâce au système de permissions et de droits d'accès du noyau. Cependant, cette isolation est limitée par rapport à celle offerte par les conteneurs, qui utilisent des mécanismes supplémentaires pour garantir une séparation plus complète entre les applications et les ressources du système. Par exemple, un processus Linux classique peut accéder à d'autres processus via des signaux ou des fichiers, tandis qu'un conteneur utilise des namespaces pour isoler complètement les processus et les ressources, empêchant ainsi toute interaction non autorisée entre les conteneurs et le système hôte.

2. Pourquoi l'isolation réseau est elle particullierement pour un service web ?
En isolant le réseau d'un conteneur, on peut limiter l'accès aux ports et aux services exposés, ce qui réduit la surface d'attaque. De plus, cela permet de contrôler le trafic entrant et sortant du conteneur, empêchant ainsi les communications non autorisées avec d'autres services ou avec l'extérieur. 

3. Quel est le risque de deux programme different qui utilisent des fichiers temporaires ?
Le risque est que les deux programmes puissent interférer l'un avec l'autre en utilisant les mêmes fichiers temporaires, ce qui peut entraîner des conflits, des erreurs ou même des pertes de données. Par exemple, si les deux programmes écrivent dans le même fichier temporaire, ils pourraient écraser les données de l'autre programme. En utilisant des conteneurs, chaque programme peut avoir son propre espace de fichiers temporaires isolé, évitant ainsi ces risques d'interférence.

## Cree un conteneur a la main
1. Que se passerait il si un programme malveillant dans le conteneur essayait de modifer les fichier de l'hote ? par exemple /etc/passwd ?


2. Pourquoi est il important que debootstrap verifie les signatures GPG des paquets telechargés ?
Cela permet de s'assurer de l'intégrité et de l'authenticité des paquets téléchargés, empêchant ainsi les attaques de type "man-in-the-middle" et garantissant que les paquets proviennent de sources fiables.

3. Que contient le dossier /dev et /etc dans un systme linux ?
- Le dossier /dev contient les fichiers de périphériques, qui représentent les interfaces vers les périphériques matériels et les ressources du système.
- Le dossier /etc contient les fichiers de configuration du système, tels que les paramètres de réseau, les utilisateurs, les services, etc.

4. chroot isole t il les processus ? depuis le conteneur pouvez vous voir les processus de l'hote avec ps -aux ?
Non car le dossier /proc est que utilise ps -aux est le dossier /proc du conteneur.

5. chroot isole t il le reseau ? depuis le conteneur pouvez vous ping google.com ?
Non, chroot n'isole pas le réseau. Depuis le conteneur on peut ping qqch.

6. Est il possible de chrooter depuis un autre utilisateur que root ?
Non, il n'est pas possible de chrooter depuis un autre utilisateur que root, car la commande chroot nécessite des privilèges élevés pour modifier l'environnement d'exécution du processus.

## mount
1. que ce passe t il si on monte dans un dosssier qui n'existe pas ?
❯ sudo mount -o loop mydisk.img mydis 
mount: mydis: mount point does not exist.
       dmesg(1) may have more information after failed mount system call.

2. Que se passe t il si on monte dans un dossier qui contient deja des fichiers ?
Si on monte dans un dossier qui contient déjà des fichiers, les fichiers existants seront masqués tant que pas unmount.

3. Comment pouvez verifier quels systemes de fichiers sont montés sur votre système ?
On peut vérifier les systèmes de fichiers montés en utilisant la commande `mount` ou en consultant le fichier `/proc/mounts`.

## Unshare
1. Que se passe t il si on omet d'utiliser l'option --pid avec unshare ?
Si on omet d'utiliser l'option --pid avec unshare, les processus ne sont pas isolé et on peut voir les processus de l'hôte avec ps -aux.

2. A quoi sert --uts ?
L'option --uts permet d'isoler le nom d'hôte.

3. Quel option de unshare permet d'isoler le reseau ?
L'option --net permet d'isoler le réseau.

## Reseau
1. Pourquoi est-il nécessaire d'activer le forwarding IP sur l'hôte pour que les conteneurs puissent accéder à Internet ?
Pour que le conteneur puisse accéder à Internet, il doit être capable de faire du routage vers l'extérieur. En activant le forwarding IP sur l'hôte, on permet au trafic provenant du conteneur d'être acheminé vers Internet via l'interface réseau de l'hôte.

2. Que fait --net dans unshare ? Pourquoi le conteneur n'a-t-il que l'interface de loopback ?
L'option --net dans unshare crée un nouvel espace de noms réseau pour le conteneur, ce qui signifie que le conteneur a son propre réseau isolé. Par défaut, le conteneur n'a que l'interface de loopback (lo) car il n'est pas connecté à un réseau externe.

3. A quoi sert la regle MASQUERADE ? Que se passerait-il si on ne l'utilisait pas ?
La règle MASQUERADE est utilisée pour masquer l'adresse IP source des paquets sortants du conteneur, en les remplaçant par l'adresse IP de l'hôte. Si on ne l'utilisait pas, les paquets sortants du conteneur conserveraient leur adresse IP source d'origine, ce qui pourrait entraîner des problèmes de routage et d'accès à Internet pour le conteneur.

## Cgroups
faire page 11