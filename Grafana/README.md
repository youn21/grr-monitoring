# TP Grafana

[TOC]

## Observer les metrics depuis PLMShift

Openshift/OKD propose un menu permettant de faire de consulter les données de monitoring (metrics) et d'en faire des graphiques. 
En mode *Developer*, rendez vous dans le menu Observe. Plusieurs onglets sont disponibles. Nous allons nous interesser aux deux premiers : 
### Dashboard
Cet onglet propose une série de dashboard prédéfinis. Ils permettent d'avoir des informations sur les ressources (CPU, mémoire, réseau) consommées par votre projet actuel. 

>:bulb:
>En cliquant sur le bouton **inspect** d'un graphique puis **Show PromQL** vous pouvez voir la requete utilisée. Cela peut etre utile pour créer vos propres requetes !


### Metrics
Cet onglet vous donne plus de liberté et permet d'afficher une graphe d'une metrics simple ou d'une requete plus complexes. 
Vous avez acces à des metrics standards (CPU Usage, Memory Usage, etc.)
En sélectionnant le champ **Custom query**, vous pouvez interroger n'importe quelle metrics disponibles dans votre projet. 

>:bulb:
>Le champ **Custom query** fait une auto-complétion des metrics que vous rentrez, vous pouvez par exemple taper "cpu" pour avoir l'ensemble des metrics qui contiennent le mot "cpu". 


## Visualisation des metrics depuis Grafana

### Configuration de la DataSource

### Création/import d'un dashboard

### Création des resources avec l'opérateur Grafana

Pour garder l'esprit GitOps, vous pouvez configurer vos *DataSource* et *Dashboard* grace à des ressources Kubernetes fournies par l'opérateur Grafana. Vous avez à votre disposition deux CRDs (Custom Resource Definition) Kubernetes : GrafanaDatasource et GrafanaDashboard.

#### CRD GrafanaDatasource

```yaml
apiVersion: grafana.integreatly.org/v1beta1
kind: GrafanaDatasource
metadata:
  name: prometheus-datasource
  namespace: grafana
  labels:
    app.kubernetes.io/instance: grafana
spec:
  datasource:
    basicAuthUser: '${user}'
    access: proxy
    editable: true
    secureJsonData:
      basicAuthPassword: '${password}'
    name: Prometheus
    url: 'https://thanos-querier.openshift-monitoring.svc.cluster.local:9091'
    jsonData:
      timeInterval: 5s
      tlsSkipVerify: true
    basicAuth: true
    isDefault: false
    type: prometheus
  instanceSelector:
    matchLabels:
      dashboards: grafana
  resyncPeriod: 5m
  valuesFrom:
    - targetPath: basicAuthUser
      valueFrom:
        secretKeyRef:
          key: user
          name: victoriametrics-credentials
    - targetPath: secureJsonData.basicAuthPassword
      valueFrom:
        secretKeyRef:
          key: password
          name: victoriametrics-credentials
```
#### CRD dashboard

```yaml
apiVersion: grafana.integreatly.org/v1beta1
kind: GrafanaDashboard
metadata:
  name: plmlatex-dashboard
  namespace: grafana
  labels:
    app.kubernetes.io/instance: grafana
spec:
  datasources:
    - datasourceName: plmapps
      inputName: DS_METRICS.VIRTUALDATA
  instanceSelector:
    matchLabels:
      dashboards: grafana
  resyncPeriod: 5m
  url: 'https://plmlab.math.cnrs.fr/monitoring-plm/grafana/-/raw/main/grafana-dashboards/PLMlatex.json'
```