# Aquaponie = Aquaculture + Hydroponie

![Aquaponie](images/aquaponie_homepage.png "Aquaponie")
https://github.com/LabAixBidouille/aquaponie/blob/master/pdf/Projet_domotique_Aquaponie.pdf

# Objectifs

`TODO @François`

# Matériel nécessaire

`TODO @François @Guy`

# Nature des données mesurées

Les données mesurées au fil de l'eau seront :
* [Conductivité](http://aquatechnique.pagesperso-orange.fr/Techniques/page_%20conduc.htm)
* [Ph](https://fr.wikipedia.org/wiki/Potentiel_hydrog%C3%A8ne)
* [Redox](https://fr.wikipedia.org/wiki/Indicateur_r%C3%A9dox)
* Température
* Niveau
* Oxygène dilué
* Nitrate
* Je sais pas

`todo : il est nécessaire de documenter chacun des points pour expliciter ces choix`

# Logiciels

## Interface de contrôle

`TODO @François`

## Extraction des données vivantes depuis MQTT

Un programme python fourni par le LabAixBidouille permet d'extraire les données précédemmemnt mesurées et de les formater sous format CSV. Les différents formats des CSV seront décrits ultérieurement.

## Stockage des données

Nous utilisons les outils opensource de la stack InfluxData
* InfluxDB pour le stockage des données
* Telegraf pour remonter les données mesurées dans InfluxDB
* Kapacitor pour déclencher des alertes sur les données présentes dans InfluxDB
 
## Monitoring temps réel

L'outil utilisé pour la restitution graphique sera [Grafana](http://grafana.org/), un produit opensource.
Il s'agit d'une solution clé en main, facile à prendre en main.

![Monitoring](images/grafana_homepage.png "Monitoring")
