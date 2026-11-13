# Simulation d’embarquement d’un avion commercial

## Question et indicateurs
* **Question mesurable :** Quelle stratégie d'embarquement (aléatoire, par zones arrière-avant, par hublot-puis-couloir) minimise le temps total d'embarquement d'un avion monocouloir de 178 places standard (classe économique) ?
* **Indicateur :** Le temps total d'embarquement, calculé via le nombre de ticks nécessaires pour que tous les passagers soient à leur place.
* **Échelle :** Spatiale de l'ordre du demi-mètre (pour isoler un siège ou un passager) et temporelle de l'ordre de la seconde.

## Modèle ODD (Agents, Environnement, Comportements)
* **Agents :** Les passagers, définis par leur numéro de siège attribué et leur encombrement (bagage ou non).
* **Environnement :** Une grille représentant la cabine, séparant les patchs "allée" des patchs "sièges".
* **Comportements :** Avancer dans l'allée, patienter si le patch frontal est occupé (interférence), s'arrêter pour ranger un bagage, et s'asseoir.
* **Paramètres :** Stratégie d'appel à la porte, pourcentage de remplissage de l'avion, proportion de passagers avec bagages cabines, nombre d’allées (1 ou 2), nombre d’entrées-sorties (1 avant ou 1 avant + 1 arrière), nombre de sièges, nombre de sièges par rangée. Choix de configuration d’avion prédéfini (A320, Boeing 737, Custom).

## Données et fiches descriptives

### Jeu de données 1 : Plan de cabine
* **Nom :** Configuration cabine A320, 2ᵉ étage, 178 places économiques.
* **Source :** Données constructeurs / SeatGuru.
* **Licence :** Données ouvertes.
* **Format et résolution :** Fichier CSV tabulaire, résolution où 1 cellule correspond à un espace siège/allée.
* **Rôle :** Initialiser l'environnement spatial et attribuer les cibles des agents.

### Jeu de données 2 : Paramètres d'installation
* **Nom :** Temps de rangement et déplacement.
* **Source :** Littérature scientifique sur l'embarquement.
* **Format et résolution :** Valeurs tabulaires (CSV) ou constantes de distribution.
* **Rôle :** Paramétrer la durée de blocage de l'allée et calibrer le modèle.

## Rôles spécifiques des données
* **Initialiser :** Le plan CSV positionne les sièges et le point d'apparition des passagers.
* **Paramétrer :** La littérature définit les distributions de vitesse et les temps d'arrêt.
* **Calibrer :** Les temps d'embarquement moyens d'une compagnie aérienne classique ajustent le modèle de base.
* **Valider :** Un jeu de données indépendant (ex. expérimentations filmées ou autre compagnie) vérifie les résultats obtenus sans réajuster les paramètres.

## Échelles spatiales et temporelles
* **1 patch = 0,5 mètre :** Correspond approximativement à l'encombrement d'un passager debout ou à la largeur d'un siège, minimisant la perte d'information.
* **1 tick = 1 seconde :** Résolution permettant de modéliser finement les bouchons dans l'allée et les délais de rangement.


## Évaluation et critères de réussite
* **Indicateurs de sortie :** Temps final d'embarquement, et nombre d'interférences (temps cumulé passé à attendre).
* **Comparaison :** Test de Kolmogorov-Smirnov pour comparer la distribution des temps simulés aux observations, ou corrélation de rang (Spearman) pour classer l'efficacité des stratégies.
* **Critère de réussite :** Le modèle doit classer les stratégies dans le même ordre d'efficacité que la littérature avec une erreur quadratique moyenne (RMSE) acceptable sur le scénario de référence.


## Plan d’expériences (BehaviorSpace)
* **Configurations :** Croisement de 3 stratégies d'embarquement avec 3 niveaux de remplissage (ex. 50%, 75%, 100%) et 2 niveaux de volume de bagages.
* **Répétitions :** Minimum 20 répétitions par configuration avec des graines aléatoires (`random-seed`) différentes pour tenir compte de l'ordre d'arrivée stochastique à la porte.
* **Simulations :** $3 \times 3 \times 2 \times 20 = 360$ exécutions, avec les indicateurs mesurés uniquement à la fin (état final).


## Hypothèses et limites
* **Hypothèses :** Les passagers se dirigent sans erreur vers leur siège, ne remontent jamais l'allée à contresens, et les groupes voyageant ensemble (familles) ne sont pas modélisés, ce qui accélère artificiellement l'installation.
* **Limites :** Les résultats dépendent fortement de la distribution des temps de rangement des bagages, paramètre très variable en réalité. En l'absence de données terrain privées pour valider, le modèle dépend d'une validation croisée avec la littérature existante, limitant sa validité externe.

---

## Architecture et Modélisation multi-agents (NetLogo)

Le modèle est découpé de manière modulaire via des fichiers `.nls` inclus :

* **`model/airplane_boarding.nlogox`** : Modèle principal (interface graphique, contrôleur `setup` / `go`, variables globales, widgets et expériences BehaviorSpace).
* **`model/environment.nls`** : Chargement des plans cabines depuis les CSV (`A320` et `B737`), gestion de la grille, rendu visuel et étiquetage.
* **`model/passengers.nls`** : Modélisation des agents passagers (`breed [ passengers passenger ]`), attributs, génération selon taux d'occupation et bagages, déplacements dans l'allée, arrêts de rangement et installation aux sièges.
* **`model/strategies.nls`** : Architecture modulaire et extensible pour l'ordonnancement de la file d'embarquement.

### Stratégies d'embarquement implémentées
1. **Aléatoire (`random`)** : Les passagers se présentent à la porte dans un ordre aléatoire, sans consigne de rangée ni de siège.
2. **Avant vers Arrière (`front-to-back`)** : Priorité aux rangées avant (1, 2, ...). Les passagers de l'avant montent en premier, puis ceux de l'arrière.
3. **Arrière vers Avant (`back-to-front`)** : Priorité aux rangées arrière (32, 31, ...). Les passagers du fond montent en premier, puis ceux de l'avant.
4. **WilMA (`wilma`)** : *Window-Middle-Aisle*. Priorité aux hublots (A, F), puis milieu (B, E), puis couloirs (C, D).