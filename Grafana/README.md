# TP Grafana

[TOC]

## Observer les metrics depuis PLMShift

Openshift/OKD propose un menu permettant de consulter les données de monitoring (metrics) et d'en faire des graphiques. 
En mode *Developer*, rendez vous dans le menu Observe. Plusieurs onglets sont disponibles. Nous allons nous interesser aux deux premiers : 
### Dashboard
Cet onglet propose une série de dashboard prédéfinis. Ils permettent d'avoir des informations sur les ressources (CPU, mémoire, réseau) consommées par votre projet actuel. 

>:bulb:
>En cliquant sur le bouton **inspect** d'un graphique puis **Show PromQL** vous pouvez voir la requete utilisée. Cela peut etre utile pour créer vos propres requetes !


### Metrics
Cet onglet vous donne plus de liberté et permet d'afficher un graphe d'une metric simple ou d'une requete plus complexe. 
Vous avez acces à des metrics génerales (CPU Usage, Memory Usage, etc.)
En sélectionnant le champ **Custom query**, vous pouvez interroger n'importe quelle metrics disponibles dans votre projet. 

>:bulb:
>Le champ **Custom query** fait une auto-complétion des metrics que vous rentrez, vous pouvez par exemple taper "cpu" pour avoir l'ensemble des metrics qui contiennent le mot "cpu". 

Pour tester, vous pouvez afficher le nombre d'utilisateur de votre site GRR à l'aide de la métric `grr_utilisateurs`. Appuyer sur la touche **Entrer** pour valider.


## Visualisation des metrics depuis Grafana

Grace à l'operateur Grafana installé sur le cluster Openshift, nous pouvons facilement creer une instance Grafana dans notre projet : 
- en mode 'Developer', cliquez sur le bouton '+Add' dans le menu à gauche
- dans le 'Developer Catalog', choisissez 'All services'
- dans le champ recherche, tapez 'Grafana' et selectionnez la tuile 'Grafana'
- cliquez sur 'Create'
- selectionnez la vue 'YAML' pour voir la configuration par defaut de l'instance Grafana qui sera créée. Notez le login/password
- completez le yaml pour ajouter les infos sur la route et le ServiceAccount comme ci-dessous, en modifiant dans le host 'test-anf' par le nom de votre projet : 
  ```yaml
  apiVersion: grafana.integreatly.org/v1beta1
  kind: Grafana
  metadata:
    labels:
      dashboards: grafana-a
      folders: grafana-a
    name: grafana-a
    namespace: test-anf
  spec:
    config:
      auth:
        disable_login_form: 'false'
      log:
        mode: console
      security:
        admin_password: start
        admin_user: root
    route:
      spec:
        host: grafana-test-anf.apps.anf.math.cnrs.fr
        path: /
        tls:
          insecureEdgeTerminationPolicy: Redirect
          termination: edge
        to:
          kind: Service
          name: grafana-a-service
          weight: 100
        wildcardPolicy: None
    serviceAccount:
      automountServiceAccountToken: true
  ```
  Le serviceAccount est comme un compte utilisateur interne à votre projet Openshift, il permettra de requeter le prometheus depuis Grafana en utilisant.
- cliquez sur 'Create' pour lancer la création de Grafana

Au bout de quelques secondes, vous avez acces à votre instance Grafana à l'URL renseignée dans la partie 'host' de la route.





### Configuration de la DataSource

Les DataSources permettent d'indiquer à Grafana où lire les metrics. 
Nous allons configurer comme datasource le Prometheus du cluster.

#### S'authentifier aupres du Prometheus

 Pour interroger le Prometheus, il faut etre authentifié. Il serait possible d'utiliser votre compte utilisateur mais il est preferable d'utiliser un compte machine (un serviceAccount) dédié à cela. L'installation de Grafana a créé un serviceAccount automatiquement. Vous pouvez le voir en vue 'Developer' :
- cliquez sur Search
- dans le menu 'Resources', tapez `ServiceAccount`
- vous devriez voir apparaitre l'ensemble des serviceAccounts de votre projet, dont 'grafana-a-sa'.

Lors de la création de ce serviceAccount, un token a été généré automatiquement. C'est ce token que nous allons utiliser pour nous authentifier aupres du Prometheus. Si vous regardez le 'yaml' du serviceAccount 'grafana-a-sa', vous pourrez voir qu'il y a un bloc de ligne comme ci-dessous : 
```yaml
secrets:
  - name: grafana-a-sa-dockercfg-zc7n2
```
Ce bloc indique que le serviceAccount possede un **secret**. Allons voir ce secret en vue 'Developer' :
- cliquez sur Search
- dans le menu 'Resources', tapez `Secret`
- vous devriez voir le secret en question, par exemple 'grafana-a-sa-token-7xw4s'

Ce secret possede une clé **token** qui a pour valeur le token de notre **serviceAccount**. C'est ce token que nous allons utiliser pour l'authentification.

#### Donner les droits au serviceAccount

Nous allons donner les droits de requeter Prometheus au serviceAccount, pour cela en mode 'Developer' : 
- allez dans le menu **Project**
- allez dans l'onget **Project access**
- ajoutez un access en cliquant sur 'Add access' et renseigner les informations : 
  - type: ServiceAccount
  - name: le nom de votre serviceAccount qui doit etre `grafana-a-sa`
  - role: view
  - project: votre projet 
- cliquez sur **Save**

#### Creation de la datasource 

Depuis l'interface web de votre instance grafana, allez dans le menu **Connections** puis **Data sources** : 
- cliquez sur 'Add data sources'
- selectionnez le type 'Prometheus' puis renseignez les valeurs ci-dessous :
  - Dans la partie **Connection**
    - Prometheus server URL : `https://thanos-querier.openshift-monitoring.svc.cluster.local:9092`
  - Dans la partie **Authentication**
    - Authentication methods: `No Authentication`
  - dans la partie **TLS settings**
    - cochez la case 'Skip TLS certificate validation'
  - Dans la partie **HTTP headers**, ajouter un entete HTTP : 
    - Header: `Authorization`
    - Value: `Bearer <token serviceAccount>` en replacement `<token serviceAccount>` par le token du serviceAccount récupéré précédemment 
  - Dans la partie **Other**
    - Custom query parameters: `namespace=test-anf` (remplacez test-anf par le nom de votre projet)
    - HTTP method: GET
  
Cliquez sur **Save & test**. Votre instance Grafana est maintenant configurée pour interroger le Prometheus.

### Création d'un dashboard

Nous pouvons maintenant commencer à créer notre premier dashboard. 
Dans le menu de Grafana, cliquez sur 'Dashboard', puis 'Create Dashboard'. 
Vous avez ici plusieurs choix : 
 - **Add visualization** : vous permet de creer votre dashboard entierement à la main
 - **Add a library panel** : permet d'inclure des graphiques qui proviennent d'autres dashboard
 - **Import a dashboard** : permet d'importer un dashboard (au format JSON). De nombreux dashboards sont disponibles en telechargement, par exemple sur le site https://grafana.com/grafana/dashboards/

Cliquez sur **Add visualization** pour creer votre dashboard. Apres avoir selectionné votre Datasource, vous arrivez sur la page vous permettant de creer votre premier graphique.

- le cadre de droite permet de configurer le graphique. Vous pouvez choisir le type de graphe que vous voulez (time series, bar chart, etc.), donner un titre, etc.
- le cadre en bas vous permet de creer votre requete de metrics. 
- le cadre juste au dessus est le resultat.
Vous pouvez voir ci-dessous un exemple : 
![Vue Grafana](grafana1.png "Panel Grafana")

Cliquez sur *Apply* pour creer le graphique. 
Vous pouvez maintenant ajouter n'importe quel graphique que vous souhaitez afficher sur votre dashboard. 

### Sauvegarder le dashboard

Lorsque vous avez fini de creer le dashboard, il faut penser à le sauvegarder. Vous pouvez cliquer sur le bouton "Save dashboard" de Grafana mais cela ne suffira pas. Le pod Grafana qui a été créé ne possede pas de volume persistent et donc ne stocke rien de facon perenne. Un redémarrage du pod entrainera une perte des modifications que vous avez réalisées pour ajouter votre datasource et creer votre dashboard. Il serait possible d'ajouter un volume persistant à notre Grafana pour conserver les modifications, mais il y a mieux !

### Création des resources avec l'opérateur Grafana

Pour garder l'esprit GitOps, vous pouvez configurer vos *DataSource* et *Dashboard* grace à des ressources Kubernetes fournies par l'opérateur Grafana. Vous avez à votre disposition deux CRDs (Custom Resource Definition) Kubernetes : GrafanaDashboard et GrafanaDatasource. 


#### CRD GrafanaDatasource

Ce type de resource Kubernetes permet de créer un datasource pour Grafana. Pour creer cette ressource :
- en mode 'Developer', cliquez sur le bouton '+Add' dans le menu à gauche
- dans le 'Developer Catalog', choisissez 'All services'
- dans le champ recherche, tapez 'datasource' et selectionnez la tuile 'GrafanaDatasource'
- cliquez sur 'Create'
- selectionnez la vue 'YAML' pour voir la configuration par defaut puis completer le yaml comme ci-dessous.

```yaml
kind: GrafanaDatasource
apiVersion: grafana.integreatly.org/v1beta1
metadata:
  name: grafanadatasource-prometheus
  namespace: test-anf
spec:
  datasource:
    access: proxy
    isDefault: true
    jsonData:
      customQueryParameters: namespace=test-anf
      httpHeaderName1: Authorization
      httpMethod: GET
      timeInterval: 5s
      tlsSkipVerify: true
    secureJsonData:
      httpHeaderValue1: 'Bearer ${token}'
    name: prometheus
    type: prometheus
    url: 'https://thanos-querier.openshift-monitoring.svc.cluster.local:9092'
  instanceSelector:
    matchLabels:
      dashboards: grafana-a
  plugins:
    - name: grafana-clock-panel
      version: 1.3.0
  valuesFrom:
    - targetPath: secureJsonData.httpHeaderValue1
      valueFrom:
        secretKeyRef:
          key: token
          name: grafana-a-sa-token-7xw4s
```



#### CRD dashboard

Ce type de resource Kubernetes permet de créer un dashboard. 
Commencez par récuperer votre dashboard au format JSON : 
- sur grafana, lorsque vous etes sur votre dashboard, cliquez sur le bouton "dashboard settings" (la roue dentée) en haut à droite.
- allez dans l'onglet *JSON Model*

Vous pouvez maintenant creer votre ressource Kubernetes GrafanaDashboard :
- en mode 'Developer', cliquez sur le bouton '+Add' dans le menu à gauche
- dans le 'Developer Catalog', choisissez 'All services'
- dans le champ recherche, tapez 'dashboard' et selectionnez la tuile 'GrafanaDashboard'
- cliquez sur 'Create'


```yaml
apiVersion: grafana.integreatly.org/v1beta1
kind: GrafanaDashboard
metadata:
  name: grr-dashboard
  namespace: test-anf
  labels:
    app.kubernetes.io/instance: grafana
spec:
  datasources:
    - datasourceName: prometheus
      inputName: DS_PROMETHEUS
  instanceSelector:
    matchLabels:
      dashboards: grafana
  resyncPeriod: 5m
  json: |
    {
      "__inputs": [
        {
          "name": "DS_PROMETHEUS",
          "label": "prometheus",
          "description": "",
          "type": "datasource",
          "pluginId": "prometheus",
          "pluginName": "Prometheus"
        }
    [...]
```
