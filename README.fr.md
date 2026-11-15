# 🎄 Calendrier de l'Avent — guide de personnalisation

Ce calendrier est **un seul fichier `index.html` autonome** : HTML, CSS, JavaScript et toutes les images (encodées en base64) sont dans le même fichier. Aucune dépendance externe, aucun serveur : on peut l'ouvrir en local, l'héberger n'importe où (GitHub Pages, intranet…) ou le publier comme artefact.

Pour l'adapter à un autre calendrier (autre thème, autre année, autres contenus), on modifie quatre choses :

1. les **textes** des 24 cases,
2. les **images** (tour de départ, image révélée, décors),
3. la **date / la clé de sauvegarde**,
4. optionnellement le **comportement** (ordre des cases, animations, langues).

---

## 1. Principe de fonctionnement

- Une image de fond (`tower`) est répétée 6 fois : elle forme une grille de **6 colonnes × 4 lignes = 24 portes**.
- Chaque porte porte un numéro de jour. Quand on l'ouvre, la **partie correspondante** d'une grande image (la « scène ») apparaît par-dessus la porte. Une fois les 24 cases ouvertes, la scène est entièrement visible.
- Cliquer sur une case ouverte (ou venant de s'ouvrir) affiche une **fenêtre** avec le titre, le texte, un lien vers la source, et un bouton de copie si la case contient du code.
- Les cases ouvertes sont mémorisées dans le navigateur (`localStorage`), la langue dans un **cookie**.
- Une case ne s'ouvre que **le jour correspondant** (du 1er au 24 décembre de l'année configurée).

---

## 2. Les contenus (textes des 24 cases)

Dans le `<script>`, repérer les tableaux :

```js
const TIPS_FR = [ {tag, title, text, src, url, code?}, ... ];   // 24 entrées
const TIPS_EN = [ {tag, title, text, src}, ... ];               // 24 entrées
const TIPS_URL = TIPS_FR.map(x => x.url);                       // les URL viennent du français
```

L'entrée n°1 du tableau correspond au **jour 1**, la n°24 au jour 24 (l'ordre du tableau = l'ordre des jours, pas la position sur la grille).

| Champ   | Où | Rôle |
|---------|----|------|
| `tag`   | FR + EN | Étiquette affichée à côté du « Jour N » (ici `année · catégorie`). |
| `title` | FR + EN | Titre de la fenêtre. |
| `text`  | FR + EN | Texte principal (texte brut, pas de HTML). |
| `src`   | FR + EN | Libellé du bouton de source (ex. `Wikipédia — CICS`). |
| `url`   | FR seulement | Lien du bouton de source, ouvert dans un nouvel onglet. |
| `code`  | FR seulement, facultatif | Si présent, affiche un bloc de code et un bouton « Copier le code ». |

Conseils :
- Utiliser des liens **vérifiés**, idéalement des sources primaires. Le lien est aussi visible en infobulle au survol du bouton.
- Si vous n'avez qu'une langue, supprimez `TIPS_EN` et la logique de langue (voir §6), ou dupliquez simplement le français.
- Dans ce calendrier la case 24 est une carte de clôture (« Joyeux Noël ») : c'est celle qui déclenche l'animation du traîneau (voir §7).

---

## 3. Les images

Toutes les images sont des **data URI** (`data:image/…;base64,…`) dans le fichier. Voici l'inventaire, dans l'ordre d'apparition, pour les retrouver :

| # | Format | Rôle | Où la repérer |
|---|--------|------|---------------|
| 0 | WebP | Panneau de titre (bois, guirlande, nœud) | `.panel{ background:url(...) }` |
| 1 | JPEG | **Tour de départ** (le « mainframe » répété 6 fois) | `.board{ background:... url(...) }` |
| 2–4 | WebP | Neige du haut : extrémité gauche / motif répétable / extrémité droite | `.snowcap .sl`, `.sm`, `.sr` |
| 5 | GIF | Père Noël en traîneau (animation) | `.sleigh i, .patrol i` |
| 6 | JPEG | **Scène révélée** (image finale) | `.tile{ background-image:url(...) }` |
| 7 | WebP | Cadre de la fenêtre d'anecdote | `.modal::before{ border-image:url(...) }` |

### 3.1 Changer la scène révélée (image finale)

C'est la personnalisation la plus fréquente.

- **Format idéal : 3:2** (ex. 1536×1024). Le plateau est configuré en `aspect-ratio: 3/2` (voir `.board`). Une image d'un autre ratio sera étirée : soit on la recadre au ratio 3:2, soit on modifie `aspect-ratio` du `.board` (et on accepte que la tour de départ soit étirée en proportion).
- Remplacer l'URI n°6 par la nouvelle image (JPEG conseillé, qualité 85–90).
- La scène est découpée en 6×4 tuiles via `background-size:600% 400%`. Rien d'autre à régler.

Script Python pour remplacer la n-ième image d'un fichier :

```python
import re, base64
s = open('index.html', encoding='utf-8').read()
new = 'data:image/jpeg;base64,' + base64.b64encode(open('scene.jpg','rb').read()).decode()
matches = list(re.finditer(r'data:image/\w+;base64,[A-Za-z0-9+/=]+', s))
m = matches[6]                      # n° dans le tableau ci-dessus
s = s[:m.start()] + new + s[m.end():]
open('index.html', 'w', encoding='utf-8').write(s)
```

### 3.2 Changer la tour de départ (l'image qui cache la scène)

- L'image n°1 est **une seule tour** (ici 254×790 px) répétée 6 fois horizontalement : `repeat-x 0 0 / calc(100% / 6) 100%`.
- Elle doit contenir **4 panneaux verticaux** (un par ligne). Si votre image a une autre géométrie, ajustez les constantes JS :

```js
const TW=254, TH=790;          // largeur / hauteur de l'image de la tour (en px)
const PX=29,  PW=183, PH=184;  // position X, largeur et hauteur d'un panneau (en px)
const PYS=[6,203,399,595];     // position Y de chacun des 4 panneaux
```

  Ces valeurs servent à placer les portes cliquables **exactement** sur les panneaux (elles sont converties en pourcentages). Mesurez-les dans un éditeur d'image sur votre propre tour.
- Autre approche : une image de fond unique (sans répétition) — remplacer `repeat-x … / calc(100% / 6) 100%` par `no-repeat center / 100% 100%` et recalculer les positions.

### 3.3 Changer les décors (titre, neige, cadre de fenêtre)

- **Panneau de titre** (n°0) : ratio d'origine 1400×406 (`aspect-ratio:1400/406`). Le texte est centré dans la zone claire via le `padding` du `.panel` (en `cqw`, proportionnel à la largeur). Si vous changez l'image, ajustez `padding:10.5cqw 9cqw 4.6cqw 10cqw` (haut, droite, bas, gauche) pour que le titre reste dans la zone utile.
- **Neige** (n°2–4) : trois morceaux de **même hauteur** et de même épaisseur de bande aux raccords. Le motif du milieu est répété (`repeat-x`). Les ratios sont écrits dans le CSS (`aspect-ratio:1399/684` et `1075/684`) : à mettre à jour si vous changez les images. Position et hauteur : `.snowcap{ top:-3.8cqw; height:12cqw; left/right:-2.2cqw }`.
- **Cadre de la fenêtre** (n°7) : utilisé en `border-image` découpé en 3 bandes (haut 330 px, milieu étirable, bas 350 px dans l'image d'origine). Si vous changez l'image, mettez à jour les valeurs `330` / `350` dans `.modal::before` ainsi que le `padding` du `.modal` pour garder le texte dans la zone claire.
- Pour **supprimer** un décor : supprimer l'élément correspondant (`.snowcap`, `.panel`…) et le CSS associé.

Astuce : pour toute image avec fond à rendre transparent, détourez-la (PNG/WebP avec canal alpha) avant de l'encoder ; les décors actuels sont des WebP transparents.

---

## 4. Dates et sauvegarde

```js
const YEAR = 2026;                  // année du calendrier
const KEY  = "avent-dino-" + YEAR;  // clé de sauvegarde dans localStorage
```

- Les cases ne s'ouvrent qu'à partir du **1er décembre de `YEAR`** : la case N s'ouvre le N décembre. Après le 25 décembre, toutes sont ouvrables.
- Pour un autre mois ou un autre rythme, modifier la fonction `currentDay()` (elle renvoie le dernier jour ouvrable, de 0 à 24) :

```js
function currentDay(){
  const now = new Date();
  if (now < new Date(YEAR, 11, 1))  return 0;   // 11 = décembre (les mois commencent à 0)
  if (now >= new Date(YEAR, 11, 25)) return 24;
  return Math.min(24, now.getDate());
}
```

- Si vous gardez le même fichier d'une année à l'autre, il suffit de changer `YEAR` : la clé de sauvegarde change aussi, donc les cases repartent fermées.
- Le message « Patience ! La case N s'ouvrira le N décembre » utilise `STR.fr.month` / `STR.en.month`.

---

## 5. Disposition des cases

```js
const ORDER = [7,19,2,14,23,10, 17,4,12,24,1,21, 9,15,6,20,13,3, 22,11,18,5,16,8];
const COLS  = 6;
```

- `ORDER` liste les numéros de jour **ligne par ligne**, de gauche à droite (4 lignes de 6). Mélangez comme vous voulez : chaque numéro de 1 à 24 doit apparaître **une seule fois**.
- Pour une autre grille (ex. 4×6), changer `COLS`, le nombre de lignes (`row/4`, `25%`, `row/3` dans la création des tuiles et dans `.tile`), `PYS` et `background-size:600% 400%`.

---

## 6. Langues et textes de l'interface

Les libellés de l'interface sont dans l'objet `STR` :

```js
const STR = {
  fr: { title, h1, sub, count, reset, foot, day, src, month, wait, testOn, testOff, resetT, copy, copied, copyFail, open, again, close, test },
  en: { … }
};
```

- `h1` : titre du calendrier ; `sub` : sous-titre ; `foot` : pied de page ; `day` : préfixe (« Jour » / « Day ») ; `src` : préfixe du bouton de source (vide par défaut → seul le nom de la source est affiché).
- **Ajouter une langue** : (1) ajouter une clé dans `STR` et un tableau `TIPS_XX`, (2) adapter la fonction `tips(day)` pour la renvoyer, (3) ajouter un bouton dans le bloc `<span class="lang">` et son `onclick`, (4) compléter `applyLang()`.
- **Mémorisation** : la langue est stockée dans le cookie `avent-lang` (1 an) avec un repli sur `localStorage`. Au premier passage, la langue du navigateur est utilisée (français sinon anglais).
- **Supprimer le choix de langue** : retirer le bloc `.lang`, forcer `LANG="fr"` et supprimer la lecture du cookie.

---

## 6 bis. Apparence générale (CSS)

Variables de couleur en haut de la feuille de style :

```css
:root{ --night:#0b1a45; --night2:#16307a; --gold:#ffd24a; --red:#c9202e; --green:#1f7a3d; --snow:#f4f8ff; }
```

- Fond de page : `body{ background: radial-gradient(...) }`.
- Cadre autour du plateau : `.frame{ border:6px solid #7a4a1d; box-shadow:0 0 0 3px var(--gold) … }`.
- Pastille numérotée des portes : `.door .n` (taille `font`, `width`, couleurs).
- Bouton de source : `.modal .srcbtn` (couleur `#1f7a3d`, ombre en relief).
- Titre : `h1{ color:#8f1420; font-size:min(6cqw,3rem) }` ; sous-titre : `.sub`.
- Neige qui tombe : le bloc `/* neige */` en bas du script (nombre de flocons `40`, durées, tailles). Désactivée automatiquement si l'appareil demande de réduire les animations.

---

## 7. Animations et interactions

### Traîneau du Père Noël (après l'ouverture de la case 24)

```css
.patrol{ width:min(58vw,420px); animation:patrolX 17s linear infinite; z-index:30; }
@keyframes patrolWave{ … }      /* balancement vertical (sinusoïdal) */
```

- **Déclencheur** : la case `24` ouverte (`opened.has(24)` dans `refresh()`). Pour un autre déclencheur, changer ce test (ex. une autre case, ou toutes les cases ouvertes).
- **Taille** : `width` de `.patrol`. **Vitesse** : durée `17s` (plus petit = plus rapide).
- **Balancement** : `patrolWave` (amplitude `7vh`, inclinaison `-6deg → 8deg`, durée `3.2s`).
- **Hasard** : à chaque nouveau passage, la fonction `randPatrol()` tire une hauteur de départ et une hauteur d'arrivée (entre 2 % et 78 % de la hauteur de l'écran). Le sens est toujours de gauche à droite. Modifier les bornes `lo` / `hi`.
- **Image** : remplacer le GIF (n°5). Ajuster `aspect-ratio:724/241` au ratio du nouveau GIF.
- **Désactiver** : supprimer l'élément `<div class="patrol">` et la ligne `$("patrol").classList.toggle(...)` dans `refresh()`.
- Reste dans le fichier un élément `.sleigh` et la fonction `flySleigh()` d'une ancienne version (vol en diagonale unique) ; ils ne sont plus appelés et peuvent être supprimés sans effet.

### Mode test (caché)

Un bouton invisible est placé sur le **cadeau vert** du panneau de titre. Un clic active/désactive le mode test : toutes les cases deviennent ouvrables sans attendre la date, avec un halo doré autour du cadeau. Le bouton « Réinitialiser » efface les cases ouvertes (et arrête le traîneau).

- Position du bouton : `.giftbtn{ left:10.5%; top:75%; width:8%; height:18% }` (en pourcentage du panneau). **À recaler** si vous changez le panneau de titre.
- Pour le désactiver complètement : supprimer le `<button class="giftbtn">` et le gestionnaire `$("testBtn").onclick`.

### Fenêtre d'anecdote

- Taille : `--W:min(760px,94vw)` dans `.modal` (largeur maximale).
- Fermeture : croix, clic en dehors, touche Échap.
- Bouton de copie du code : n'apparaît que si la case a un champ `code`.

---

## 8. Publier / héberger

- **En local** : double-cliquer sur `index.html`.
- **Hébergement statique** (GitHub Pages, Netlify, serveur web) : déposer `index.html` tel quel.
- **Cookies** : le cookie de langue peut être ignoré si la page est affichée dans un cadre qui bloque les cookies ; un repli sur `localStorage` est prévu, et à défaut la langue du navigateur est utilisée.
- **Poids** : le fichier pèse environ 2 Mo à cause des images. Pour l'alléger, recompressez les images (WebP qualité ~80, JPEG qualité ~80, taille d'affichage réaliste).

---

## 9. Check-list pour créer un nouveau calendrier

1. Dupliquer `index.html` sous un autre nom.
2. Changer `YEAR` et, si besoin, `KEY`.
3. Remplacer les 24 entrées de `TIPS_FR` (et `TIPS_EN`) : `tag`, `title`, `text`, `src`, `url`.
4. Changer les textes de `STR` (`title`, `h1`, `sub`, `foot`).
5. Remplacer la **scène** (n°6, ratio 3:2) et la **tour** (n°1) ; ajuster `TW/TH/PX/PW/PH/PYS` si la tour change.
6. (Optionnel) Remplacer les décors : panneau de titre, neige, cadre de fenêtre, GIF du traîneau.
7. Recaler le bouton caché du mode test (`.giftbtn`) sur le nouvel élément du panneau de titre.
8. Tester : activer le mode test, ouvrir toutes les cases, vérifier les liens de source, basculer la langue, redimensionner la fenêtre (mobile et ordinateur).
9. Réinitialiser le calendrier avant publication, puis désactiver le mode test.

---

## 10. Licence

- **Code** (HTML, CSS, JavaScript) : licence **MIT** — voir le fichier `LICENSE`.
- **Contenus créés par l'auteur** (textes des cases, illustrations originales) : licence **CC BY 4.0** — voir le fichier `LICENSE-CONTENT`. Vous pouvez les réutiliser et les modifier, y compris commercialement, à condition de citer l'auteur (« zarafa-dev-io ») et d'indiquer les modifications.
- **Marques et visuels IBM** : l'image du z17 (la tour mainframe), le logo IBM et le logo « Z » appartiennent à IBM. IBM et z17 sont des marques d'IBM. Ce projet est indépendant et n'est ni affilié à IBM, ni approuvé ou sponsorisé par IBM ; ces éléments ne sont pas couverts par les licences ci-dessus.
- **Éléments non couverts** : tout élément appartenant à un tiers (par exemple la photo de la tour mainframe IBM et les marques IBM, ou toute image / animation dont vous ne détenez pas les droits) n'est **pas** couvert par ces licences. Remplacez-les par vos propres visuels avant de redistribuer le calendrier.

Copyright (c) 2026 zarafa-dev-io
