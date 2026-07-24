# La Cave — Chartreuse

Inventaire d'une collection de Chartreuse. Page statique, sans dépendance ni serveur :
un fichier HTML, un fichier de données, quelques icônes.

La cave vit dans **`cave.json`, à la racine de la branche `main` du dépôt**. Tous les
appareils lisent et écrivent ce même fichier : la cave est la même partout.

## Les deux onglets

- **Cave** — le spectre des cuvées (chaque segment proportionnel au nombre de bouteilles,
  toucher pour filtrer), la répartition par millésime et par cuvée, la liste complète avec
  recherche et tri. Toucher une bouteille ouvre sa fiche : modification ou suppression
  (le bouton **Supprimer** demande une confirmation avant d'agir).
- **Saisie** — le formulaire d'ajout, les dernières bouteilles saisies, et les réglages
  du dépôt.

## Mettre en ligne (5 min)

1. Créer un dépôt GitHub, par exemple `cave`.
2. Y déposer `index.html`, `cave.json`, `manifest.json`, `favicon.png`, `icon-180.png`,
   `icon-192.png`, `icon-512.png`, `icon-512-maskable.png`
   (bouton **Add file → Upload files**, glisser les fichiers, **Commit changes**).
3. Onglet **Settings → Pages** : *Source* = **Deploy from a branch**, branche `main`,
   dossier `/ (root)`. **Save**.
4. Au bout d'une minute, `https://<compte>.github.io/cave/` répond.

Sur le téléphone : ouvrir cette adresse, puis **Partager → Sur l'écran d'accueil** (iOS)
ou **⋮ → Ajouter à l'écran d'accueil** (Android). L'icône est la photo des deux bouteilles.

## Écrire dans le dépôt : le jeton

En lecture, rien à configurer : l'appli déduit le compte et le dépôt de l'adresse
`https://<compte>.github.io/<dépôt>/` et lit `cave.json` directement. **Un appareil sans
jeton consulte la cave mais ne peut pas la modifier.**

Pour saisir depuis un appareil, il lui faut un jeton GitHub :

1. **GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens
   → Generate new token**.
2. *Repository access* : **Only select repositories** → le dépôt `cave`.
3. *Repository permissions* : **Contents → Read and write**. Rien d'autre.
4. Choisir une date d'expiration, générer, copier le jeton.
5. Dans l'appli : **Saisie → Réglages du dépôt et jeton d'accès**, coller le jeton,
   **Enregistrer les réglages**.

Le jeton reste dans le stockage du navigateur de cet appareil. **Il ne doit jamais être
écrit dans un fichier du dépôt** — le dépôt est public. Un jeton limité à ce seul dépôt et
à la permission *Contents* ne donne accès à rien d'autre ; en cas de doute, le révoquer
depuis GitHub et en générer un autre. Le bouton **Oublier le jeton** l'efface de l'appareil.

## Comment la synchronisation se comporte

- Chaque ajout, modification ou suppression est écrit dans `cave.json` environ deux secondes
  plus tard, en un commit. Le voyant en haut de l'onglet Saisie indique l'état :
  vert (à jour), ambre (enregistrement en attente), rouge (échec).
- Sans réseau, les modifications restent sur l'appareil et repartent au prochain
  enregistrement réussi. **Enregistrer maintenant** force l'envoi.
- Si le fichier a changé entre-temps depuis un autre appareil, GitHub refuse l'écriture et
  l'appli le signale : **Relire le dépôt** reprend la version publiée — au prix des
  modifications locales non enregistrées.
- **Relire le dépôt** est aussi le moyen de récupérer une saisie faite ailleurs.

## Format d'une bouteille

```json
{
  "id": "b001",
  "cuvee": "centenaire",
  "volume": 0.7,
  "millesime": "2021",
  "prix": 90,
  "provenance": "",
  "commentaire": "",
  "statut": "cave"
}
```

- `cuvee` : clé du catalogue. Les cuvées d'origine sont définies dans `index.html`
  (`CUVEES_BASE`) ; celles ajoutées depuis l'appli sont stockées dans la section `cuvees`
  de `cave.json`, avec leur nom, leur couleur et leur rang.
- `volume` : **en litres** dans le fichier, saisi **en centilitres** dans l'appli
  (3, 20, 35, 50, 70, 100, 150, 300 cl en raccourcis, ou n'importe quelle valeur).
  `format` (facultatif) remplace le libellé calculé, par ex. `"coffret 6 × 3 cl"`.
- `millesime` : une année (`"2021"`) ou une fourchette (`"1972-1978"`). Les fourchettes
  sont classées d'après leur première année et forment leur propre ligne dans les
  statistiques. `null` pour une bouteille sans date.
- `prix` : prix d'achat, ou `null`. Une bouteille sans prix est considérée comme offerte et
  s'affiche ainsi dans la liste.
- `statut` : `cave`, `bue` ou `offerte`. Seules les bouteilles `cave` comptent dans le
  spectre, les millésimes et les totaux ; les autres restent dans la liste, barrées.

## Les couleurs

Chaque cuvée porte la teinte de sa liqueur :

| Famille | Cuvées |
|---|---|
| Verts | Élixir Végétal, V.E.P. Verte, 1605, Chartreuse Verte |
| Mélanges vert-jaune | Cuvée des M.O.F., 9e Centenaire, La Tau |
| Jaunes et ambres | Génépi, Chartreuse Jaune, Reine des Liqueurs, V.E.P. Jaune, Tarragone |
| Bleu | Coffret dégustation |

Le spectre est ordonné du vert le plus sombre à l'ambre le plus profond, le bleu à la fin.

## Changer la couleur d'une cuvée

Onglet **Saisie**, section **Couleurs des cuvées** : toutes les cuvées du catalogue y sont
listées dans l'ordre du spectre, avec le nombre de bouteilles de chacune. Toucher une
pastille ouvre le sélecteur de couleur du téléphone ; le spectre et la liste se mettent à
jour immédiatement, et la nouvelle teinte part dans `cave.json` comme le reste — donc elle
vaut pour tous les appareils, définitivement.

Une recoloration ne déplace pas la cuvée dans le spectre : sa position reste celle du
catalogue. Le lien **rétablir**, en tête de section, rend leurs teintes d'origine aux cuvées
de Chartreuse sans toucher aux cuvées que vous avez créées.

Techniquement, la section `cuvees` de `cave.json` sert aux deux : une clé connue
(`centenaire`, `jaune`…) y est une **recoloration** qui prend le pas sur le catalogue
d'`index.html` ; une clé inconnue (`perso-…`) est une **cuvée que vous avez créée**. Les
premières sont conservées telles quelles, les secondes disparaissent d'elles-mêmes quand
plus aucune bouteille ne les utilise.

## Ajouter une cuvée depuis l'appli

Dans le formulaire de saisie, choisir **＋ Nouvelle cuvée…**, lui donner un nom et une
couleur — au nuancier ou à la pipette. La cuvée prend automatiquement sa place dans le
spectre d'après la teinte choisie : une couleur verte la place tôt, un ambre vers la fin,
un bleu ou un violet tout à la fin. Sa couleur se corrige ensuite depuis la section
**Couleurs des cuvées**, comme les autres. Une cuvée dont plus aucune bouteille ne dépend
disparaît d'elle-même du fichier.

## Saisie en série

Le champ **Quantité** crée d'un coup plusieurs bouteilles identiques — pratique pour six
9e Centenaire du même millésime achetées ensemble. Le formulaire garde la cuvée et la
contenance après l'ajout, seul le commentaire est vidé.

## Le dépôt est public

La liste et les prix d'achat sont donc lisibles par tout le monde et indexables. Pour
revenir en arrière : passer le dépôt en privé et l'héberger sur Cloudflare Pages ou Netlify
(gratuits sur dépôt privé), ou retirer les prix du `cave.json` publié.

## Détails techniques

- Aucun framework, aucun outil de build. `index.html` se modifie directement.
- Écriture via l'API GitHub *Contents* (`PUT`), avec le `sha` du fichier pour détecter les
  écritures concurrentes. Lecture par la même API, ou à défaut par `cave.json` servi par
  Pages, ou à défaut par la copie de secours embarquée dans `index.html`.
- Deux polices chargées depuis Google Fonts (Marcellus, Karla) ; sans réseau, l'appli
  bascule sur les polices système sans casser la mise en page.
- Icônes générées à partir de la photo des deux bouteilles ; `icon-512-maskable.png` laisse
  la marge de sécurité exigée par Android pour les icônes rognées en rond.
- Testé de 320 à 1280 px de large. Navigation clavier, contrastes vérifiés, animations
  désactivées si le système le demande.
