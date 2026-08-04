# Export du CV en PDF

Le CV est écrit en HTML (`index.html` — nommé ainsi pour être servi
à la racine par GitHub Pages). Le PDF est un artefact généré :
on ne le modifie jamais à la main, on régénère.

## Commande

Depuis `/Users/targetmobile/Documents/work/perso/cv/cv` (le CV vit dans le
sous-dossier `cv/` du dépôt, servi sur https://mihajah.github.io/cv/) :

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless \
  --disable-gpu \
  --no-pdf-header-footer \
  --print-to-pdf=cv-Mihaja-Herinjaka-Rasolo.pdf \
  --virtual-time-budget=8000 \
  index.html
```

Le fichier attendu fait ~330 Ko, **1 page A4** (595 × 842 pt).

### Détail des options

| Option | Rôle |
|---|---|
| `--headless` | pas de fenêtre, Chrome imprime et rend la main |
| `--disable-gpu` | évite des avertissements GPU sur macOS ; sans effet sur le rendu |
| `--no-pdf-header-footer` | **indispensable** — sinon Chrome ajoute l'URL et la date en marge |
| `--virtual-time-budget=8000` | laisse 8 s pour charger les polices Google Fonts avant l'impression |
| `--print-to-pdf=…` | fichier de sortie |

Les erreurs `CVDisplayLinkCreateWithCGDisplay failed` ou
`externally_managed_app_manager` dans la sortie sont du bruit Chrome sur macOS :
si la ligne `… bytes written to file` apparaît, l'export est bon.

## Alternative manuelle

Chrome → **Imprimer** → **Enregistrer au format PDF**, avec :

- Marges : **Aucune** (le gabarit A4 est déjà dans le CSS via `@page`)
- **Graphiques d'arrière-plan : activé** — sinon le fond papier, l'encadré IA et
  les puces de stack technique s'impriment en blanc

## Vérifications après export

```bash
# nombre de pages — doit renvoyer 1
python3 -c "d=open('cv-Mihaja-Herinjaka-Rasolo.pdf','rb').read();print(d.count(b'/Type /Page')-d.count(b'/Type /Pages'))"

# le texte est bien du texte (et non une image) — important pour les ATS
/opt/local/bin/gs -q -dNOPAUSE -dBATCH -sDEVICE=txtwrite -sOutputFile=- cv-Mihaja-Herinjaka-Rasolo.pdf | head

# aperçu visuel d'une page en PNG
/opt/local/bin/gs -q -dNOPAUSE -dBATCH -sDEVICE=png16m -r120 \
  -dFirstPage=1 -dLastPage=1 -sOutputFile=/tmp/cv-p1.png cv-Mihaja-Herinjaka-Rasolo.pdf
```

Le PDF doit conserver : texte sélectionnable avec accents corrects, liens
cliquables (`mailto:` + LinkedIn), polices embarquées en sous-ensembles.

## Piège à connaître : la mise en page tient sur une page *au pixel près*

La typographie est calibrée pour remplir exactement une page A4. Ajouter une
puce ou une expérience fait passer à deux pages — et Chrome **ne coupe pas** la
grille à deux colonnes : il la déplace en entier, donc la page 1 se retrouve
vide et tout le contenu part en page 2.

Ce n'est pas un bug. Deux issues :

1. resserrer l'échelle typographique (`.profil p`, `ul.bullets li`,
   `.job + .job`, `section + section` dans le `<style>`) ;
2. assumer une vraie mise en page sur deux pages.

Toujours vérifier le nombre de pages après une modification de contenu.

## Fond papier sur toute la page

En impression, `.sheet` porte `min-height:295mm` et la couleur papier est aussi
posée sur `html, body`. Sans cela, le fond s'arrête là où le contenu se termine
et une bande blanche apparaît en bas de page. Ne pas remettre `min-height:0`
dans le bloc `@media print`.

## Note sur les polices

Le HTML charge Newsreader, IBM Plex Sans et IBM Plex Mono depuis Google Fonts :
**l'export nécessite une connexion internet**. Hors ligne, Chrome retombe sur
Georgia / Helvetica et le rendu change. Pour rendre le fichier autonome, il
faudrait embarquer les polices en `@font-face` avec des URI `data:` base64.
