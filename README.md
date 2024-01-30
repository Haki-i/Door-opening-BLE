# Door-opening-BLE

# **Objectif du projet**

L’objectif est de contrôler l’ouverture et la fermeture d’une porte grâce à la proximité d’un appareil BLE (nos écouteurs) et de partager des données par Wifi entre ESP32 et ESP8266.

Vidéo du projet : <https://www.tiktok.com/@__hakii__/video/7284935015522127137>

# I- **Conception électronique**

## a) ESP32 BLE

Le Bluetooth Low Energy est une technologie de réseau personnel sans fil utilisée pour transmettre des données sur de courtes distances. Comme son nom l'indique, il est conçu pour une faible consommation d'énergie et un faible coût, tout en conservant une portée de communication similaire au Classic Bluetooth.

Nous utiliserons le module ESP32 Wrover-CAM possédant cette technologie pour notre projet. Celui-ci agira en tant que client, attendant de recevoir des données issues d’un serveur en scannant les appareils autour de lui.

Le serveur sera une technologie beacons : les écouteurs Bluetooth BUDS Live agiront en tant que serveur, émettant en permanence un signal.

Lorsque le boitier des écouteurs sera allumé, le module ESP32 détectera sa présence et récupérera comme donnée la valeur RSSI du signal soit la puissance du signal reçu.

Nous pourrons ainsi connaître la proximité entre le serveur et le client.

## b) ESP32 BLE + ESP8266

Le module ESP32 ne sera pas situé à côté de la porte, il nous sera donc impossible de le brancher au vérin électrique pour commander son ouverture.

Il nous faudra communiquer en Wifi avec un ESP8266 qui sera, lui, chargé de contrôler le mécanisme.

## c) Vérin électrique

Le vérin électrique est commandé en 12V à l’aide d’une batterie et d’un relais branché à l’ESP8266.

Lorsque la carte reçoit par Wifi comme indication si les écouteurs sont proches ou éloignés, elle agira sur le vérin.

# II- **Conception informatique**

## a) Mise en place du client BLE

Pour notre projet, nous utiliserons la bibliothèque ESP32 BLE Arduino. Nous souhaitons donc mettre en place un client de manière à récupérer les informations du boitier des écouteurs.

Dans un  premier temps, nous devons récupérer l’adresse MAC de nos écouteurs afin d’identifier le bon appareil. Nous pouvons télécharger une application comme BLE scanner pour récupérer cet UUID et la mettre dans une variable.

Nous emploierons principalement le code fourni en exemple dans la bibliothèque utilisée. Une fonction callback est appelée à chaque fois que des appareils disposant de la technologie Bluetooth sont détectés à un intervalle de temps que l’on choisira. Celle-ci les scanne et nous renvoie leurs informations.

Dans le loop, nous ne regardons alors que leur adresse MAC pour les comparer à l’adresse de nos écouteurs. Ce n’est que lorsque celles-ci sont identiques que l’on affiche une nouvelle information de l’appareil : le RSSI soit la force du signal reçu par le client.

Ainsi grâce au RSSI, nous pouvons déterminer la proximité ou non de l’appareil avec l’ESP32 puis déterminer une action à réaliser en fonction.

## b) Communication Wifi entre appareil

Comme mentionné, l’actionneur et l’ESP32 ne seront pas situés à côté. Il sera donc nécessaire à l’ESP32 d’envoyer des données à une carte ESP8266 qui, elle, interagira avec le vérin. Pour les faire communiquer, nous utilisons le protocole ESP-NOW.

**ESP32 : sender**

L’ESP32 va être chargé d’envoyer la donnée suivante : les écouteurs sont à proximité de l’ESP32, oui/non.

Préalablement, tout comme pour le boitier, nous devons récupérer l’adresse MAC de l’ESP8266. Nous utiliserons un programme Arduino qui permettra de l’obtenir.

Pour envoyer nos données, nous déclarons une structure pouvant contenir n’importe quel type de variable. Dans notre cas, simplement un type char pour 0 ou 1.

Enfin dans le loop, nous utilisons esp_now_send() avec comme paramètres l’adresse MAC de l’appareil avec lequel on cherche à communiquer, l’adresse de la structure contenant nos données et sa taille.

**ESP8266 : receiver :**

On créé une structure.

Dans le setup, on initialise l'ESP-NOW. On met également en place une fonction callback qui sera appelée à chaque fois que la carte recevra des informations de l’envoyeur.

Pour manipuler la structure de données envoyée par la carte, on la copie dans notre propre structure créée.

## c) Activation de la porte

Nous recevons donc dans notre structure les informations 1 ou 0 et avec une simple condition, nous actionnons le vérin électrique.

En s’approchant avec les écouteurs, celui-ci déverrouille la porte et inversement en s’éloignant.

# Rendu final

![ezgif com-gif-maker (6)](https://user-images.githubusercontent.com/92324336/169152523-1fde116c-a971-4a41-909e-4755c3ce8719.gif)

![ezgif com-gif-maker (7)](https://user-images.githubusercontent.com/92324336/169152541-9dbc8e03-631b-4c4d-b821-3ed2c6dfcece.gif)
