# cub3D

![Language](https://img.shields.io/badge/langage-C-blue) ![Graphics](https://img.shields.io/badge/graphique-MiniLibX%20%7C%20raycasting-purple)

Un moteur de raycasting dans le style de Wolfenstein 3D : un labyrinthe 3D explorable rendu depuis une grille 2D, un rayon par colonne d'écran, traversée DDA, tranches de mur mises à l'échelle selon la distance. Tourne en 1280×720 avec murs texturés, minimap, sprites et visée souris.

## Pipeline de raycasting

```mermaid
flowchart TD
    A["fichier de scène .cub"] --> B["PARSER<br/>textures NO/SO/WE/EA ·<br/>couleurs F/C · grille de map"]
    B --> C["VALIDATEUR<br/>map fermée par des murs ·<br/>un seul joueur · pas de trous ·<br/>chars & couleurs valides"]
    C --> D["init joueur<br/>position + direction de vue<br/>depuis N/S/E/W"]
    D --> E["BOUCLE DE RENDU — pour chacune des 1280 colonnes"]
    subgraph E["BOUCLE DE RENDU — pour chacune des 1280 colonnes"]
        F["lancer de rayon<br/>(dir = joueur + offset du plan caméra)"] --> G["traversée DDA de la grille<br/>case par case jusqu'au mur"]
        G --> H["distance perpendiculaire<br/>(anti fisheye)"]
        H --> I["hauteur de tranche = H / dist<br/>choix texture NO/SO/WE/EA<br/>échantillonnage de la colonne"]
    end
    E --> J["remplissage sol & plafond (F/C)"]
    J --> K["sprites & minimap en overlay"]
    K --> L["mlx_put_image → frame"]
    M["hooks d'entrée<br/>WASD · ← → · souris · ESC"] --> N["déplacement avec collision<br/>rotation de la vue"]
    N --> E
```

## Le format de scène `.cub`

```
NO ./textures/north.xpm      SO ./textures/south.xpm
WE ./textures/west.xpm       EA ./textures/east.xpm
F 220,100,0                  C 225,30,0

111111
1000N1
100011
111111
```

Le parser rejette tout ce qui est invalide avec une ligne `Error` : le dépôt embarque plus de 20 maps pièges (`maps/test_*.cub`) couvrant textures dupliquées, débordements de couleur, murs ouverts, joueur manquant, caractères parasites, lignes vides dans la map.

## Contrôles

| Entrée | Action |
|:---:|---|
| `W` `A` `S` `D` | avancer / strafer |
| `←` `→` ou souris | tourner la caméra |
| `ESC` / croix de la fenêtre | sortie propre |

## Structure

```
cub3d/
├── srcs/
│   ├── parsing/       # lecture .cub, grille, textures, couleurs, sprites
│   ├── check/         # vérification des arguments & extensions
│   ├── player/        # position de spawn & vecteurs de direction
│   ├── raycasting/    # préparation des rayons & DDA
│   ├── rendering/     # dessin des frames, textures, sprites, minimap
│   ├── events/        # hooks clavier & souris
│   └── cleanup/       # libération centralisée des ressources
├── includes/          # cub3d.h (1280×720, FOV, vitesses)
├── maps/              # 1 map valide + 20 maps pièges invalides
├── textures/          # textures de murs XPM
├── minilibx-linux/
└── libft/
```

## Compilation & lancement

```bash
sudo apt install libx11-dev libxext-dev
make
./cub3D maps/valid.cub
```
