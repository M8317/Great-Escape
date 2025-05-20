# Bot *Great Escape* – README (FR)

## 1. Objet du projet

Ce dépôt contient un bot CodinGame pour **Great Escape** (variante 1 v 1 de *Quoridor*). L’IA a atteint la **48ᵉ place sur 1 283** au classement public. Le programme conjugue calcul de chemins exact, recherche adversariale en profondeur limitée et strict respect de la contrainte temps réelle (\~ 95 ms par tour).

## 2. Algorithmes et modèles utilisés

| Composant                        | Rôle dans le bot                                                                                                 | Pourquoi ce choix                                                        |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| **BFS (Breadth‑First Search)**   | Calcule la distance exacte d’un pion à sa ligne d’arrivée ; vérifie qu’un nouveau mur laisse un chemin à chacun. | Grille à coût uniforme → BFS optimal, < 0,05 ms/appel.                   |
| **Minimax + élagage Alpha–Bêta** | Explore l’arbre des coups jusqu’à **profondeur 5** pour choisir l’action.                                        | Recherche adversariale fiable ; Alpha–Bêta élimine \~ 90 % des branches. |
| **Table de transposition**       | Mémorise \<hash d’état, profondeur> → (score, meilleur coup).                                                    | Évite les ré‑évaluations, –25 % de nœuds en milieu de partie.            |

## 3. Vue d’ensemble de l’architecture

```text
Entrée CG → Parseur
              │
              ▼
   Représentation d’état (mutable)
              │
              ▼
     Générateur de coups (mouvements + murs)
              │
              ▼
   Recherche Minimax / Alpha–Bêta (+ table)
              │
              ▼
   Coup sélectionné → Sortie (⩽ 95 ms)
```

## 4. Parcours du code

### 4.1 Paramètres clés

* `TIME_LIMIT = 0.095` s : marge de sécurité.<br>
* `MAX_DEPTH = 5` : profondeur maximale avec *iterative deepening*.<br>
* `WALLS_TO_TRY = 4` : on ne teste que 4 murs « prometteurs » autour de l’adversaire.

### 4.2 Structure `state`

```python
{
  "positions": [(x0, y0), (x1, y1)],  # pions
  "walls_h": set[(x, y)],             # segments horizontaux
  "walls_v": set[(x, y)],             # segments verticaux
  "walls_left": [w0, w1],             # murs restants
  "current_player": 0 | 1,
  "turn_number": int
}
```

L’état est **modifié sur place** ; `apply_move()` / `undo_move()` assurent le retour arrière en O(1).

### 4.3 Génération des coups

1. Déplacement : toutes les cases voisines libres via `get_neighbors()`.
2. Mur : ≤ 4 positions dans une fenêtre 5 × 5 centrée sur l’adversaire, filtrées par `is_valid_wall()`.

### 4.4 Fonction d’évaluation

```python
score = 10*(d_opp - d_me) + 2*(w_me - w_opp) - 0.1*turn
```

*Distances* obtenues par BFS ; coefficients ajustés par grille sur 1 000 parties auto‑jouées.

### 4.5 Moteur de recherche

`alpha_beta()` réalise l’approfondissement itératif :

* vérification du temps à chaque appel récursif ;
* consultation de la table avant d’explorer les enfants ;
* coupures Alpha–Bêta pour arrêter tôt.

## 5. Performances

|                                       Profondeur | Nœuds moyens | Temps moyen/tour |
| -----------------------------------------------: | -----------: | ---------------: |
|                                                3 |          800 |             3 ms |
|                                                4 |        1 900 |            12 ms |
|                                                5 |        3 300 |            38 ms |
| Même en situation extrême, le bot reste < 95 ms. |              |                  |

## 6. Lancer localement

```bash
python bot.py < entree.txt > sortie.txt
```

`entree.txt` doit suivre le protocole tour‑par‑tour de CodinGame. Dans l’IDE CG, il suffit d’uploader le fichier.

## 7. Pistes d’amélioration

* **Monte‑Carlo Tree Search** guidé par un réseau léger pour explorer plus profondément.

