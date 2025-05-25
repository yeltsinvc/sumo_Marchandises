# Tutoriel : SUMO pour la Modélisation du Transport de Marchandises

*Date : 25 mai 2025*

## 1. Introduction à SUMO et au Transport de Marchandises

### 1.1 Qu'est‑ce que SUMO ?

SUMO (Simulation of Urban MObility) est une suite de simulation de trafic **open‑source**, portable et **microscopique**. Elle est conçue pour simuler de grands réseaux routiers. SUMO permet de modéliser :

* le comportement individuel de chaque véhicule (microscopique),
* les interactions entre véhicules,
* l'impact des systèmes de gestion du trafic.

Initialement axé sur la mobilité urbaine des passagers, SUMO offre également des fonctionnalités robustes pour la **modélisation du transport de marchandises**. Sa flexibilité permet de :

* définir des types de véhicules spécifiques (camions de différentes tailles, capacités),
* créer des itinéraires complexes incluant des points d'arrêt pour chargement/déchargement,
* analyser des indicateurs clés pour la logistique.

### 1.2 Pourquoi utiliser SUMO pour le transport de marchandises ?

La modélisation du transport de marchandises est cruciale pour :

* Évaluer l'impact des flux de fret sur la congestion routière et l'environnement.
* Optimiser les itinéraires logistiques et les schémas de livraison.
* Tester l'efficacité de nouvelles politiques de transport (ex : zones à faibles émissions, restrictions d'accès pour les poids lourds).
* Planifier les infrastructures logistiques (ex : dépôts, plateformes multimodales).

SUMO est un outil précieux dans ce contexte car il permet de :

* **Définir des véhicules hétérogènes** : camions de différentes tailles, poids, capacités, motorisations.
* **Modéliser des comportements spécifiques** : temps d'arrêt pour opérations logistiques, respect des limitations de vitesse spécifiques aux poids lourds.
* **Analyser des indicateurs pertinents** : temps de parcours, consommation de carburant (via les émissions), retards, longueurs de files d'attente.
* **Intégrer des données externes** : réseaux routiers réels (ex : *OpenStreetMap*), plans de demande de transport.
* **Interagir dynamiquement** avec la simulation via l'interface *TraCI* (Traffic Control Interface), pour simuler des systèmes de gestion de flotte en temps réel.

### 1.3 Objectifs de ce tutoriel

Ce tutoriel a pour objectif de vous guider à travers les étapes de création et d'analyse de simulations de transport de marchandises avec SUMO. Nous aborderons :

1. La création de réseaux routiers simples.
2. La définition de véhicules de transport de marchandises et de leurs itinéraires.
3. La modélisation des opérations logistiques (arrêts, chargement/déchargement).
4. La collecte et l'interprétation des résultats de simulation.

Chaque section est accompagnée d'exercices pratiques pour mettre en application les concepts présentés. Nous nous concentrerons sur l'aspect **pratique** et le **pourquoi** de chaque configuration.

---

## 2. Prérequis et Installation

### 2.1 Installation de SUMO

Avant de commencer, assurez‑vous d'avoir une version fonctionnelle de SUMO installée sur votre système (Windows, Linux ou macOS).

* **Site officiel** : [https://sumo.dlr.de/docs/Downloads.html](https://sumo.dlr.de/docs/Downloads.html)
* **Documentation d'installation** : [https://sumo.dlr.de/docs/Installing/index.html](https://sumo.dlr.de/docs/Installing/index.html)

Nous vous recommandons d'installer la dernière version stable. Après l'installation, vérifiez que les outils en ligne de commande de SUMO (comme `sumo`, `sumo-gui`, `netconvert`, `jtrrouter`) sont accessibles depuis votre terminal (généralement en ajoutant le répertoire **bin** de SUMO à votre variable d'environnement `PATH`).

### 2.2 Structure des fichiers SUMO

Une simulation SUMO typique repose sur plusieurs fichiers **XML** :

| Fichier    | Rôle                                                                                 | Comment l'obtenir                                                  |
| ---------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------------------ |
| `.net.xml` | Décrit la topologie du réseau routier (nœuds, arêtes, feux de signalisation).        | Généré via \`\` à partir de `.nod.xml` & `.edg.xml`, ou import OSM |
| `.rou.xml` | Définit les véhicules, leurs types, leurs itinéraires et leurs heures de départ.     | Manuel / outils (DUAROUTER, jtrrouter)                             |
| `.add.xml` | Éléments supplémentaires : détecteurs, zones de stationnement, arrêts, polygones…    | Optionnel                                                          |
| `.sumocfg` | Fichier principal liant tous les autres et définissant les paramètres de simulation. | Manuel                                                             |

Nous créerons ces fichiers pas à pas dans les exercices suivants.

---

## 3. Concepts Clés pour la Modélisation du Fret dans SUMO

### 3.1 Le réseau routier (`.net.xml`)

Le réseau est la base de toute simulation. Pour le fret, certains aspects sont particulièrement importants :

* **Types de routes** : autoroutes, routes nationales, routes urbaines, avec leurs limitations de vitesse respectives.
* **Restrictions d'accès** : certaines routes peuvent être interdites aux poids lourds ou avoir des restrictions de poids/hauteur.
* **Nombre de voies** : impacte la capacité et la possibilité de dépassement.
* **Géométrie des intersections** : essentielle pour modéliser correctement les manœuvres des longs véhicules.

### 3.2 Les types de véhicules (`<vType>`)

SUMO permet de définir des types de véhicules avec des caractéristiques précises. Pour le fret, on s'intéressera notamment à :

* `id` : Identifiant unique (ex : `camion_standard`, `semi_remorque`).
* `vehicleClass` : Catégorie du véhicule (`truck` pour les poids lourds).
* `length`, `width`, `height` : Dimensions en mètres.
* `maxSpeed` : Vitesse maximale en m/s.
* `accel`, `decel` : Capacités d'accélération et de décélération.
* `emissionClass` : Classe d'émission (ex : `HBEFA3/HDV`).
* `color` : Pour une identification visuelle facile dans **sumo-gui**.

Ces paramètres influencent directement le comportement du véhicule dans la simulation.

### 3.3 Itinéraires et flux de véhicules (`<vehicle>`, `<flow>`)

Un véhicule individuel est défini par `<vehicle>` qui inclut son type, son heure de départ et son itinéraire. L'itinéraire peut être :

* **Explicite** : séquence d'arêtes (`<route edges="edge1 edge2 …"/>`).
* **Calculé** par SUMO si l'origine (`from`) et la destination (`to`) sont fournies.

Pour générer un grand nombre de véhicules, on utilise souvent `<flow>` :

* `id` : Identifiant du flux.
* `type` : Référence à un `<vType>`.
* `begin`, `end` : Période de génération (s).
* `vehsPerHour` ou `period` : Taux ou intervalle.
* `from`, `to` ou `route` : Origine/destination ou itinéraire explicite.

### 3.4 Arrêts et stationnement (`<stop>`, `<parkingArea>`)

Pour modéliser les opérations logistiques (chargement, déchargement, pauses), on utilise `<stop>` :

* `lane` : Voie (format : `edgeID_laneIndex`).
* `duration` : Durée de l'arrêt (s).
* `parking="true"` : Le véhicule utilise une place de stationnement.
* `startPos` & `endPos`, `triggered`, `until` pour des contrôles avancés.

Des zones dédiées `<parkingArea>` peuvent être définies dans un fichier additionnel, avec capacité limitée.

---

## 4. Exercice 1 : Première Simulation – Un Camion, Un Trajet Simple

**Objectif** : Simuler un unique camion se déplaçant d'un dépôt (A) vers un client (B) sur un réseau très simple.

### 4.1 Création du réseau

Réseau minimaliste : deux nœuds et une arête (aller) + une sortie.

#### 4.1.1 `freight.nod.xml`

```xml
<nodes>
    <node id="depot"        x="0.0"   y="0.0" type="priority"/>
    <node id="client"       x="500.0" y="0.0" type="priority"/>
    <node id="exit_client"  x="550.0" y="0.0" type="traffic_light"/>
</nodes>
```

#### 4.1.2 `freight.edg.xml`

```xml
<edges>
    <edge id="route_depot_client" from="depot"  to="client"      priority="1" numLanes="1" speed="13.89"/>
    <edge id="route_client_exit"  from="client" to="exit_client" priority="1" numLanes="1" speed="8.33"/>
</edges>
```

#### 4.1.3 Conversion en `.net.xml`

```bash
netconvert --node-files=freight.nod.xml \
           --edge-files=freight.edg.xml \
           --output-file=freight.net.xml
```

### 4.2 Véhicule et itinéraire

#### 4.2.1 `freight.rou.xml`

```xml
<routes>
    <vType id="camion_standard" accel="1.0" decel="4.5" length="12.0"
           minGap="2.5" maxSpeed="22.22" vehicleClass="truck" color="0,0,1"/>

    <vehicle id="camion1" type="camion_standard" depart="0" departLane="0" departSpeed="max">
        <route edges="route_depot_client route_client_exit"/>
    </vehicle>
</routes>
```

### 4.3 Configuration de la simulation

#### 4.3.1 `freight.sumocfg`

```xml
<configuration>
    <input>
        <net-file   value="freight.net.xml"/>
        <route-files value="freight.rou.xml"/>
    </input>

    <time>
        <begin value="0"/>
        <end  value="100"/>
    </time>

    <processing>
        <ignore-route-errors value="true"/>
    </processing>

    <gui_only>
        <start             value="true"/>
        <tracker-interval  value="0.1"/>
    </gui_only>
</configuration>
```

### 4.4 Lancement et visualisation

```bash
sumo-gui -c freight.sumocfg
```

Vous devriez voir le camion bleu partir de **depot** → **client** → **exit\_client**.

*Pourquoi ces étapes ?*

* **Séparation des préoccupations** : `.nod.xml` & `.edg.xml` = structure, `.rou.xml` = demande.
* \`\` : valide la géométrie et crée un réseau optimisé.
* \`\` : réutilisable, rend les fichiers plus lisibles.
* **Fichier de configuration** : chef d'orchestre.

---

## 5. Exercice 2 : Multiples Camions et Itinéraires

**Objectif** : Simuler plusieurs camions (types différents) avec des flux.

Nous réutilisons `freight.net.xml`.

### 5.2 Fichier des routes `freight_multi.rou.xml`

```xml
<routes>
    <vType id="camion_standard" accel="1.0" decel="4.5" length="12.0"
           minGap="2.5" maxSpeed="22.22" vehicleClass="truck" color="0,0,1"/>

    <vType id="fourgonnette" accel="1.5" decel="5.0" length="7.0"
           minGap="2.0" maxSpeed="25.0" vehicleClass="truck"
           guiShape="delivery" color="0,1,0"/>

    <route id="route_depot_client1" edges="route_depot_client route_client_exit"/>

    <flow id="flux_camions_std" type="camion_standard" route="route_depot_client1"
          begin="0" end="300" period="60"/>

    <flow id="flux_fourgonnettes" type="fourgonnette"
          begin="10" end="300" vehsPerHour="30" departLane="0" departSpeed="max">
        <route edges="route_depot_client"/>
    </flow>

    <vehicle id="camion_special" type="camion_standard" depart="50" color="1,0,0">
        <route edges="route_depot_client"/>
    </vehicle>
</routes>
```

### 5.3 Mise à jour du `.sumocfg`

```xml
<input>
    <net-file   value="freight.net.xml"/>
    <route-files value="freight_multi.rou.xml"/>
</input>
<time>
    <begin value="0"/>
    <end  value="600"/>
</time>
```

### 5.4 Lancement

```bash
sumo-gui -c freight_multi.sumocfg
```

*Pourquoi utiliser des flux ?*

* **Gestion de la demande** continue.
* **Modèles agrégés** : moins de véhicules individuels à définir.
* **Variabilité** : SUMO ajoute une stochasticité.

---

## 6. Exercice 3 : Modélisation des Dépôts et des Temps d'Opération

### 6.1 Ajout de `<stop>`

```xml
<routes>
    <vType id="camion_standard" accel="1.0" decel="4.5" length="12.0"
           minGap="2.5" maxSpeed="22.22" vehicleClass="truck" color="0,0,1"/>

    <vType id="fourgonnette" accel="1.5" decel="5.0" length="7.0" minGap="2.0"
           maxSpeed="25.0" vehicleClass="truck" guiShape="delivery" color="0,1,0"/>

    <route id="route_depot_client1" edges="route_depot_client route_client_exit"/>

    <flow id="flux_camions_std" type="camion_standard" route="route_depot_client1"
          begin="0" end="300" period="60">
        <stop lane="route_depot_client_0" duration="120" parking="true"/>
    </flow>

    <flow id="flux_fourgonnettes" type="fourgonnette" begin="10" end="300"
          vehsPerHour="30" departLane="0" departSpeed="max">
        <route edges="route_depot_client"/>
        <stop lane="route_depot_client_0" duration="60" parking="true" endPos="-5"/>
    </flow>

    <vehicle id="camion_special" type="camion_standard" depart="50" color="1,0,0">
        <route edges="route_depot_client route_client_exit"/>
        <stop lane="route_depot_client_0" duration="90" parking="true" startPos="50" endPos="70"/>
        <stop lane="route_client_exit_0" duration="30" parking="true"/>
    </vehicle>
</routes>
```

### 6.2 Zones de stationnement `freight.add.xml`

```xml
<additional>
    <!-- Zona de carga del cliente : 3 plazas en batería -->
    <parkingArea id="pa_client_loading_dock"
                 lane="route_depot_client_0"
                 startPos="100" endPos="150"
                 roadsideCapacity="3"/>

    <!-- Parking del depósito principal : 5 plazas en batería -->
    <parkingArea id="pa_depot_main"
                 lane="route_depot_client_0"
                 startPos="10" endPos="50"
                 roadsideCapacity="5"/>
</additional>

```

Ajoutez ce fichier via `<additional-files>` dans le `.sumocfg`.
```xml
<configuration>
    <input>
        <net-file   value="freight.net.xml"/>
        <route-files value="freight_multi.rou.xml"/>
        <additional-files value ="freight.add.xml"/> 
    </input>
    <time>
        <begin value="0"/>
        <end  value="600"/>
    </time>
        <processing>
        <ignore-route-errors value="true"/>
    </processing>

    <gui_only>
        <start             value="true"/>
        <tracker-interval  value="0.1"/>
    </gui_only>
</configuration>
```

*Pourquoi modéliser arrêts & stationnement ?*

* **Réalisme des temps de trajet**.
* **Impact sur la congestion** si véhicules bloquent la voie.
* **Dimensionnement des infrastructures**.
* **Évaluation de scénarios** (nouvelles zones, durées, politiques).

---

