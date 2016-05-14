Messages RS485
--------------

Les messages qui circulent sur le réseau RS485 sont des chaines de caractères avec le format suivant: "Topic=Payload"

La communication série s'effectue en SoftSerial (D4, D5 et D6) pour préserver les sorties vers le moniteur.

A terme, ces messages seront remplacés par un protocole Modbus.

Fonctionnement de base
----------------------

Les messages en sortie vers le broker et vers le 485 sont stockés provisoirement dans des files (FIFO). Celles-ci sont vidées message par message dans la boucle loop(). Ceci assure la non-collision des échanges.

Le processus de fonctionnement consiste en :

Si on reçoit un message MQTT on l'envoie vers l'UC.
Si on n'en a pas, toutes les 10s on envoie un message de vie à chacun des UC

L'UC doit répondre dans tous les cas.

 - S'il s'agit d'une requête MQTT, l'UC répond avec un topic spécifique qui est renvoyé au broker.

 - S'il s'agit d'une commande/affectation MQTT, l'UC répond avec le même topique et 'OK' en payload, ce qui certifie que le message est bien arrivé et l'action effectuée. Il peut y avoir ensuite des tests internes à l'UC comme une détection de retour de commande. Si ces tests sont négatifs, l'UC renvoie dans ce cas une alarme spécifique de dysfonctionnement à la place de 'OK'.

 - S'il s'agit d'un message de vie, l'UC répond avec le même topique et:
	- Soit 'OK' en payload, ce qui signifie qu'il est content et que tout va bien, 
	- Soit avec un topique différent, ce qui signifie alors qu'il a un message à transmettre au broker.

 - Si le message reçu par l'UC a été corrompu, il répondra avec le même topique et 'KO' en payload.

Si l'UC a plusieurs messages à transmettre, il peut les assembler en un seul avec:

	- Topique = "01/MM" ou "02/MM"  (Multi-Messages)
	- Payload = "T1=P1;T2=P2;..."

   Dans ce cas le GM dissociera les messages pour les envoyer individuellement au broker.

Si l'UC ne répond pas, 2 autres envois sont effectués avant de déclarer qu'il est en panne (message MQTT d'alarme et led rouge allumée). S'il répond avant le troisième envoi le processus est annulé. Le test de vie est maintenu, mais si la Led d'alarme est allumée, on n'envoie plus de message d'alarme pour ne pas saturer le broker.

Dans le cas où l'UC a été déclaré en panne, un message d'alarme MQTT ("01/AL/FC=1" ou "02/AL/FC=1") a donc été envoyé. A son retour en fonction (après maintenance), le GW qui continue à tester cet UC envoie un message d'alarme à 0 pour signifier un retour à la normale.

Il est du rôle du serveur RPi de tester le GW et vérifier qu'il est en vie. Un message peut lui être destiné (topique spécifique) auquel il répond comme le fait un UC (même topique, 'OK' en payload).

Le GW filtre les messages MQTT qui ne le concerne pas. Il s'abonne aux messages concernant ses UC.

Messages MQTT propres au GW
----------------------

Le GW répond au message :  "00/RQ" par "00/EL=enPanne"
          bit 0 = GW perte réseau TCP       |            
          bit 1 = UC1 ne répond pas         |
          bit 2 = UC2 ne répond pas         | allument la led d'alarme D7
          ...                               |
          bit 8 = dysfonctionnement sur GW  |
          bit 9 = dysfonctionnement sur UC1
          bit 10 = dysfonctionnement sur UC2
          ...

Cadencement
-----------

Des messages de vie sont envoyés toutes les 5 secondes. Les messages qui sont envoyés font partie d'une liste qui est scrutée régulièrement et qui est bouclée.

Cette liste n'a pas de dimension figée et peut contenir une liste de messages autres que des messages de vie. Ceci peut être utilisé pour actualiser des données régulièrement.

Avec un afficheur LCD 2x16 en I2C
---------------------------------

L'affichage de données peut utiliser la liste de cadencement précédente. Il suffit d'inclure dans la liste, des topiques de requête vers les UCs, de traiter les réponses et d'afficher les valeurs après une mise en forme. On bénéficie avec processus d'un cadencement des différents affichages.

Note: Un UC déporté et spécialisé dans les affichages serait plus souple à mettre en oeuvre.

RTC & NTP
---------

Il n'est pas prévu que le GW possède de RTC. Par contre, l'EEPROM peut être utile pour stocker des messages pour l'affichage LCD.

Il pourrait être inclus que la GW ait en charge de mettre à jour les RTC de ses UCs. Il utilisera alors sa connexion internet pour récupérer les données NTP et les distribuer vers les UCs.

