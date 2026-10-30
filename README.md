# Main levée

Dessiner dans l'air avec sa main, ou cadrer une zone de l'image avec ses doigts pour la transformer, devant sa webcam, directement dans le navigateur. Le projet s'inspire de la [démo de dessin de Yoha](https://handtracking-io.github.io/yoha/draw_demo/) et de [FingerFrame](https://fingerframe.app/), mais repose sur [MediaPipe Hand Landmarker](https://ai.google.dev/edge/mediapipe/solutions/vision/hand_landmarker), une bibliothèque maintenue par Google.

La vidéo reste dans le navigateur. Seuls la bibliothèque (jsDelivr) et le modèle (Google) sont téléchargés au démarrage.

## Les deux modes

On passe de l'un à l'autre avec les boutons « Dessin » et « Cadre » de la barre d'outils, ou avec la touche `M`.

### Dessin

| Geste | Effet |
| --- | --- |
| Main ouverte | déplace le curseur |
| Pince (pouce contre index) | dessine |
| Poing maintenu une seconde | efface tout |

### Cadre

| Geste | Effet |
| --- | --- |
| Pouce et index bien écartés, sur les deux mains | les quatre bouts de doigts sont les coins d'un cadre ; l'image y prend le style choisi |
| Bouger les doigts | le cadre se déforme avec eux : trapèze, losange, n'importe quel quadrilatère |
| Clic des deux mains à la fois (pouce contre index) pendant que le cadre est ouvert | passe au style suivant (au plus un changement toutes les 0,7 seconde) |
| Poing d'une main maintenu une seconde | efface le dessin, comme en mode Dessin ; un arc rouge se remplit autour du poing |
| Baisser les mains | le cadre reste 0,4 seconde en place, puis s'efface en fondu |

Les côtés du cadre sont des pointillés qui défilent d'un doigt à l'autre, avec un point qui respire sur chaque doigt. Les coins sont toujours reliés dans le même ordre : index gauche, index droit, pouce droit, pouce gauche. Le cadre le plus naturel se fait donc les index en haut et les pouces en bas. Le cadre garde sa place dès qu'une main est refermée, le temps que l'autre la rejoigne : on peut cliquer une main après l'autre, et la pastille rappelle « clique aussi avec l'autre main ». Il faut rouvrir les doigts pour cliquer de nouveau. La pastille en haut à droite indique ce que la caméra voit : « Mains : 1/2 », « écarte pouce et index » ou le style en cours.

### Pourquoi le cadre est fluide

Ces réglages viennent de [fingerframe.app](https://fingerframe.app/). Le site n'a pas de dépôt public, mais son code est servi au navigateur ; ses idées ont été réécrites ici.

- **Lissage adaptatif** : quand les doigts bougent peu, le cadre ne rattrape que 35 % de l'écart à chaque image, ce qui gomme les tremblements. Dès qu'ils bougent vite, il en rattrape jusqu'à 85 %, ce qui évite le retard.
- **Affichage à chaque rafraîchissement de l'écran** (60 ou 120 images par seconde) : entre deux images de la caméra, le cadre continue de glisser vers les doigts au lieu de sauter.
- **Deux seuils** : il faut écarter le pouce et l'index d'au moins 0,75 fois la taille de la paume pour ouvrir le cadre, mais 0,35 suffit pour le garder.
- **Maintien et fondu** : le cadre ne disparaît pas au premier raté de la détection.
- **Détection allégée** : MediaPipe analyse une copie de l'image réduite à 640 pixels de large, au plus 30 fois par seconde en mode Cadre, avec des seuils de confiance bas pour perdre les mains moins souvent.

### Les 21 styles

Cartoon, Pixel, Croquis, Noir, Briques, Manga, Néon, Aquarelle, Pâte à modeler, Glitch, VHS, Lavis, Jade, Papier découpé, Feuille d'or, Porcelaine, Pop Art, Peinture, Anime, Film 3D, Cyberpunk.

C'est la panoplie de [fingerframe.app](https://fingerframe.app/) :

- **Ses 18 filtres** sont les 18 premiers styles, sous des noms français. Par exemple, Toon devient Cartoon, Van Gogh devient Peinture et Lego devient Briques.
- **Ses styles IA** (Anime, 3D Movie, Cyberpunk) sont calculés là-bas par un service payant, dans le cloud. Ici, ils sont refaits en simple traitement d'image dans le navigateur, gratuitement et sans réseau.
  - **Anime** : trois tons à plat et un trait d'encre épais.
  - **Film 3D** : surfaces lissées, lumière chaude et halo doux.
  - **Cyberpunk** : palette néon rose et cyan, contours lumineux et pluie en biais.
- **Son quatrième style IA, Lego**, correspond à Briques.

On choisit le style dans la liste « Style du cadre », avec la touche `F` ou en cliquant des deux mains. Sur un Mac récent, le plus lourd (VHS) prend environ 15 ms par image, soit la moitié du temps disponible à 30 images par seconde.

## Photo et vidéo

- **Photo** (touche `P`) enregistre `main-levee-photo.png` : la vidéo, le cadre stylisé et le dessin, comme à l'écran.
- **Filmer** (touche `R`) enregistre la même image en vidéo. Le bouton devient « Arrêter » et une pastille rouge compte le temps. Au second clic, le fichier `main-levee-video.mp4` (ou `.webm` selon le navigateur) est téléchargé.

## Raccourcis

| Touche | Effet |
| --- | --- |
| `M` | change de mode |
| `F` | style suivant |
| `P` | photo |
| `R` | démarre ou arrête la vidéo |
| `Cmd+Z` / `Ctrl+Z` | annule le dernier trait |

Les raccourcis sont ignorés quand une liste ou un réglage a le focus.

## Lancer le site

Le navigateur n'autorise la caméra que sur `https://` ou sur `localhost`. Il faut donc servir le dossier plutôt qu'ouvrir le fichier directement :

```bash
git clone https://github.com/FLP-0/main-leve.git
cd main-leve
python3 -m http.server 8000
```

Ouvre ensuite <http://localhost:8000> dans Chrome, puis clique sur « Activer la caméra ». Arrête le serveur avec `Ctrl+C`.

## Comment ça marche

Tout est dans `index.html`, découpé en sections commentées :

1. **Réglages** : seuils des gestes, lissage du curseur et du cadre, durées du fondu, couleurs. C'est là qu'on ajuste le comportement.
2. **Chargement du modèle** : MediaPipe repère 21 points sur chaque main, jusqu'à deux mains, sur chaque image de la caméra.
3. **Gestes** : de simples calculs de distances et d'angles entre ces points.
   - Pince : l'écart pouce-index, divisé par la taille de la paume, passe sous un seuil.
   - Poing : le bout de chaque doigt est plus près du poignet que son articulation du milieu.
   - Main ouverte pour le cadre : l'écart entre le bout du pouce et le bout de l'index, mesuré comme la pince. Les coins du cadre sont les bouts de ces deux doigts, sur chaque main.
4. **Boucle principale** : à chaque rafraîchissement de l'écran, une détection si la caméra a donné une nouvelle image, puis le dessin ou le cadre selon le mode.
5. **Styles du cadre** : le rectangle qui contient le cadre est copié dans une petite image de travail, transformé, agrandi, puis découpé selon les traits qui relient les doigts.
6. **Dessin** : chaque trait garde ses points, ce qui permet d'annuler, de redimensionner et d'exporter.

Les points de la main sont numérotés de 0 à 20 : 0 pour le poignet, 4 pour le bout du pouce, 8 pour le bout de l'index, 12 pour le majeur, 16 pour l'annulaire et 20 pour l'auriculaire.

### Ajouter un style

Un style est une entrée du tableau `STYLES`. Seul `name` est obligatoire :

| Champ | Rôle |
| --- | --- |
| `name` | nom affiché dans la liste et sur le cadre |
| `width` | largeur de l'image de travail, en pixels ; un nombre, ou une fonction de la largeur du cadre à l'écran. Plus petit : plus rapide et plus pixelisé |
| `css` | filtre CSS appliqué en copiant la vidéo, par exemple `'sepia(1) contrast(1.2)'` |
| `pixels(image, now)` | modifie directement les pixels de l'image de travail (`ImageData`) |
| `after(ctx, w, h, now)` | dessine par-dessus l'image de travail |
| `decorate(ctx, x, y, w, h, cols, rows)` | dessine sur l'écran, à pleine résolution, après l'agrandissement (les tenons des Briques, par exemple) |
| `smooth` | `false` pour garder des pixels nets à l'agrandissement |

Le style le plus simple tient en une ligne :

```js
{ name: 'Sépia', width: 360, css: 'sepia(1) contrast(1.2)' },
```

### Essayer depuis la console

La page expose `window.mainLevee` pour expérimenter sans caméra ni gestes :

```js
mainLevee.setMode('frame');   // ou 'draw'
mainLevee.setStyle(4);        // le 5e style, Briques
mainLevee.STYLES.map(s => s.name);
```

## Idées pour aller plus loin

- Une gomme : un autre geste, par exemple deux doigts levés.
- Changer de couleur en pointant une pastille avec le doigt.
- Plusieurs cadres à la fois, chacun avec son style.
- Un cadre à six coins : ajouter le creux entre le pouce et l'index de chaque main (points 2 et 5).
- Mettre le site en ligne (GitHub Pages sert en `https://`, donc la caméra y fonctionne).
