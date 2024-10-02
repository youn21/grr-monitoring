# TP Grafana

[TOC]

## Observer les metrics depuis PLMShift

Openshift/OKD propose un menu permettant de faire de consulter les données de monitoring (métriques) et d'en faire des graphiques. 
En mode *Developer*, rendez vous dans le menu Observe. Plusieurs onglets sont disponibles. Nous allons nous interesser aux deux premiers : 
### Dashboard
Cet onglet propose une série de dashboard prédéfinis. Ils permettent d'avoir des informations sur les ressources (CPU, mémoire, réseau) consommées par votre projet actuel. 

>:bulb:
>En cliquant sur le bouton **inspect** d'un graphique puis **Show PromQL** vous pouvez voir la requete utilisée. Cela peut etre utile pour créer vos propres requetes !


### Metrics
Cet onglet vous donne plus de liberté et permet d'afficher une graphe d'une métrique simple ou d'une requete plus complexes. 
Vous avez acces à des métriques standards (CPU Usage, Memory Usage, etc.)
En sélectionnant le champ **Custom query**, vous pouvez interroger n'importe quelle métriques disponibles dans votre projet. 

>:bulb:
>Le champ **Custom query** fait une auto-complétion des métriques que vous rentrez, vous pouvez par exemple taper "cpu" pour avoir l'ensemble des métriques qui contiennent le mot "cpu". 


## Visualisation des métrics depuis Grafana

### Configuration de la DataSource

### Création/import d'un dashboard

### Création des resources avec l'opérateur Grafana

#### CRD datasource
#### CRD dashboard