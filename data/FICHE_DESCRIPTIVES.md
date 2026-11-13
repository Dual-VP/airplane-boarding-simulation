# Fiches descriptives des jeux de données — Plans de cabine

## Jeu de données 1 : Plan de cabine Airbus A320

| Champ | Détail |
|---|---|
| **Nom** | `A320_cabin.csv` (grille numérique) / `A320_cabin_labels.csv` (identifiants sièges) |
| **Source** | [SeatMaps.com — Air France A320](https://seatmaps.com/fr/airlines/af-air-france/airbus-a320/#87682805257e619d49b8e0dfdc14affa) |
| **Avion** | Airbus A320-200, configuration Air France V.2 « Métro » |
| **Nombre de sièges** | **178** |
| **Disposition** | Monocouloir, 3–3 (ABC \| couloir \| DEF) |
| **Licence** | Données ouvertes (constructeur / SeatGuru) |
| **Format** | CSV tabulaire, 41 lignes × 9 colonnes |
| **Résolution** | 1 cellule = 1 patch NetLogo = 0,5 m |
| **Rôle** | Initialiser l'environnement spatial et attribuer les cibles des agents |

### Codage des cellules (grille numérique)

| Valeur | Signification | Couleur suggérée (NetLogo) |
|:---:|---|---|
| `0` | Mur / fuselage (impraticable) | Gris foncé |
| `1` | Allée (corridor praticable) | Noir |
| `2` | Siège (destination passager) | Bleu |
| `3` | Porte / entrée (point d'apparition) | Vert |

### Configuration détaillée des rangées

| Rangée(s) | Sièges | Total | Notes |
|:---:|---|:---:|---|
| 1 | D, E, F | 3 | Cloison avant (lavabo/placard côté gauche) |
| 2–10 | A, B, C, D, E, F | 54 | Standard économique |
| 11 | A, B, C, D, E, F | 6 | Devant sortie de secours, sièges non inclinables |
| 12 | A, B, C, D, E, F | 6 | **Sortie de secours 1** (extra legroom) |
| 13 | — | 0 | Rangée absente (superstition) |
| 14 | A, B, C, D, E, F | 6 | **Sortie de secours 2** (extra legroom) |
| 15–30 | A, B, C, D, E, F | 96 | Standard économique |
| 31 | A, B, D, E, F | 5 | Siège C absent |
| 32 | A, B | 2 | Rangée tronquée (galley/toilettes arrière) |
| **Total** | | **178** | |

### Structure de la grille (coupe transversale)

```
Col :  0    1    2    3    4    5    6    7    8
       █    A    B    C    ·    D    E    F    █
     Mur  Hublot Milieu Couloir Allée Couloir Milieu Hublot Mur
```

---

## Jeu de données 2 : Plan de cabine Boeing 737-400

| Champ | Détail |
|---|---|
| **Nom** | `B737_cabin.csv` (grille numérique) / `B737_cabin_labels.csv` (identifiants sièges) |
| **Source** | [SeatMaps.com — Aero Contractors B737 Classic](https://seatmaps.com/fr/airlines/n2-aero-contractors/boeing-737-classic/) |
| **Avion** | Boeing 737-400, configuration Aero Contractors |
| **Nombre de sièges** | **144** (12 Business + 132 Économique) |
| **Disposition** | Monocouloir, Business 2–2 (AC \| DF) puis Économique 3–3 (ABC \| DEF) |
| **Licence** | Données ouvertes |
| **Format** | CSV tabulaire, 35 lignes × 9 colonnes |
| **Résolution** | 1 cellule = 1 patch NetLogo = 0,5 m |
| **Rôle** | Initialiser l'environnement spatial (configuration alternative) |

### Configuration détaillée des rangées

| Rangée(s) | Classe | Sièges | Total | Notes |
|:---:|---|---|:---:|---|
| 1–3 | Business | A, C, D, F | 12 | Config 2–2 (B et E absents), pitch 91–94 cm |
| 4 | Économique | A, B, C, D, E, F | 6 | Cloison (extra legroom) |
| 5–10 | Économique | A, B, C, D, E, F | 36 | Standard |
| 10 | Économique | A, B, C, D, E, F | — | Non inclinable (devant sortie) |
| 11 | Économique | A, B, C, D, E, F | 6 | **Sortie de secours 1** (extra legroom) |
| 12 | Économique | A, B, C, D, E, F | 6 | **Sortie de secours 2** (extra legroom) |
| 13–23 | Économique | A, B, C, D, E, F | 66 | Standard |
| 24 | — | — | 0 | Rangée absente |
| 25–26 | Économique | A, B, C, D, E, F | 12 | Dernières rangées (26 = non inclinable) |
| **Total** | | | **144** | |

---

## Utilisation dans NetLogo

### Chargement du CSV

```netlogo
extensions [csv]

patches-own [
  patch-type    ;; "wall", "aisle", "seat", "door"
  seat-id       ;; ex: "12A", "5F" (vide si non-siège)
  seat-row      ;; numéro de rangée (0 si non-siège)
  seat-letter   ;; lettre du siège "A"-"F" (vide si non-siège)
]

to load-cabin [filename]
  let raw csv:from-file filename
  let grid-height length raw
  let grid-width length (item 0 raw)
  
  ;; Redimensionner le monde : x ∈ [0, grid-width-1], y ∈ [0, grid-height-1]
  resize-world 0 (grid-width - 1) 0 (grid-height - 1)
  
  let y-row 0
  foreach raw [ row-data ->
    let x-col 0
    foreach row-data [ cell ->
      ask patch x-col (grid-height - 1 - y-row) [
        (ifelse
          cell = 0 [ set patch-type "wall"  set pcolor gray ]
          cell = 1 [ set patch-type "aisle" set pcolor black ]
          cell = 2 [ set patch-type "seat"  set pcolor blue ]
          cell = 3 [ set patch-type "door"  set pcolor green ]
        )
      ]
      set x-col x-col + 1
    ]
    set y-row y-row + 1
  ]
end
```

### Chargement des identifiants de sièges

```netlogo
to load-seat-labels [filename]
  let raw csv:from-file filename
  let grid-height length raw
  
  let y-row 0
  foreach raw [ row-data ->
    let x-col 0
    foreach row-data [ cell ->
      ask patch x-col (grid-height - 1 - y-row) [
        if patch-type = "seat" [
          set seat-id cell
          ;; Extraire le numéro de rangée et la lettre
          set seat-row read-from-string (substring cell 0 (length cell - 1))
          set seat-letter last cell
        ]
      ]
      set x-col x-col + 1
    ]
    set y-row y-row + 1
  ]
end
```
