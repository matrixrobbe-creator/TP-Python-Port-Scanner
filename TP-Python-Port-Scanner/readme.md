# TP - Scanner de Ports TCP en Python

Projet réalisé dans le cadre du module **Bloc 3 (B3.5.4-SLAM - Prévenir les attaques)**.

Ce script Python permet d'auditer une machine cible afin d'identifier ses ports TCP ouverts (plage standard 1 à 1023) à l'aide de threads concurrents.

## ⚠️ Avertissement légal

L'utilisation d'outils de scan de ports sur des systèmes distants sans autorisation préalable explicite est interdite par la loi. La tentative d'accès frauduleux à un système de traitement automatisé de données est réprimée par l'**article 323-7 du Code pénal français**, qui sanctionne les actes préparatoires des mêmes peines que l'infraction consommée.

Ce script doit être utilisé exclusivement sur votre machine locale (`127.0.0.1`) ou sur des environnements de laboratoire autorisés (VMs de test).

## 🛠️ Prérequis et Dépendances

Le script utilise uniquement des modules de la bibliothèque standard de Python :

* `socket` : gestion des connexions réseau TCP.
* `threading` : parallélisation du scan pour accélérer l'analyse.
* `queue` : file d'attente synchronisée pour distribuer les numéros de ports entre les threads.

Aucune installation de dépendance tierce (`pip`) n'est requise.

## 🚀 Utilisation

1. Cloner ou télécharger le dépôt :
   ```bash
   git clone <url-du-depot>
   cd TP-Python-Port-Scanner
   ```

2. Exécuter le script dans un terminal interactif :
   ```bash
   python scanner.py
   ```

3. Renseigner l'adresse IP de la cible lorsque le programme le demande :
   * Pour tester votre propre machine : `127.0.0.1`
   * Pour tester une machine virtuelle locale autorisée : renseigner son adresse IP LAN.

## 💡 Explication du fonctionnement du code

Le script repose sur le modèle de concurrence **Producteur-Consommateur** (ou *Worker Pool*) pour scanner les ports sans attendre séquentiellement la réponse de chacun :

1. **Initialisation et saisie** : Le programme sollicite l'utilisateur pour obtenir l'adresse IP de la cible (`target`), puis prépare une structure de données synchronisée (`Queue`) et une liste vide (`open_ports`) destinée à stocker les résultats positifs.
2. **Distribution des tâches** : La fonction `fill_queue()` parcourt l'intervalle des ports réservés (`range(1, 1024)`) et les injecte un à un dans la file d'attente.
3. **Parallélisation (multi-threading)** : 500 threads ouvriers exécutant la boucle `executor()` sont créés et démarrés. Chaque thread dépile un numéro de port de la file d'attente et appelle `port_scan(port)`.
4. **Test de connexion TCP** : La fonction `port_scan()` tente d'établir une poignée de main TCP (*three-way handshake*) via un socket `AF_INET / SOCK_STREAM`. Si `sock.connect()` réussit, la fonction retourne `True` ; si la connexion est refusée, filtrée ou arrive à expiration, une exception est levée et interceptée pour retourner `False`.
5. **Collecte et synchronisation** : Dès qu'un port répond positivement, il est affiché dans la console et ajouté à `open_ports`. L'instruction `thread.join()` force le programme principal à patienter jusqu'à ce que tous les threads aient terminé de vider la file d'attente, avant d'afficher la liste récapitulative des ports ouverts.

## ⚙️ Détail des composants du code

* **`target = input(...)`** : Récupère la cible à tester.
* **`fill_queue(port_list)`** : Insère les numéros de ports dans la file `queue`.
* **`port_scan(port)`** : Tente la connexion TCP brute sur le couple `(target, port)`.
* **`executor()`** : Boucle de travail consommée par les threads tant que `queue` n'est pas vide.
* **Gestion des threads** : Instanciation de 500 threads, lancement avec `.start()` et synchronisation finale avec `.join()`.

## 🔗 À voir

* **Nmap (Network Mapper)** : outil de référence en audit réseau et détection de services.