# Skill — `drawio-diagrams`

Skill [Claude Code](https://docs.claude.com/claude-code) pour produire des fichiers
**`.drawio` / [diagrams.net](https://www.drawio.com/)** (mxGraph XML) **à la main** : schémas
d'architecture / infra, flowcharts, network maps, org charts, boîtes-et-flèches. Le résultat
est un fichier **éditable et ouvrable** dans diagrams.net ou l'extension VS Code draw.io —
pas une image exportée.

## Installation

Clone le repo directement comme dossier de skill :

```bash
git clone https://github.com/Killian-Aidalinfo/drawio-diagrams.git ~/.claude/skills/drawio-diagrams
```

Une fois installé, Claude charge ce skill automatiquement dès qu'une tâche demande de créer,
éditer ou versionner un diagramme au format `.drawio`.

## Quand l'utiliser

- Produire un schéma d'architecture / infrastructure éditable (`.drawio`)
- Dessiner un flowchart, une carte réseau, un org chart, un diagramme boîtes-et-flèches
- Éditer / versionner un diagramme draw.io existant sans casser le format
- Obtenir un fichier ouvrable dans diagrams.net plutôt qu'une image figée

## Prompt à donner à Claude

```text
Utilise le skill drawio-diagrams (lis SKILL.md avant d'agir).

[Décris ton diagramme, par exemple :]
Fais-moi un schéma .drawio de l'archi <X> : <composants> reliés par <flux>.
```

## Pré-requis

Aucun. Le seul outil utilisé est `python3` (présent partout) pour valider le XML :

```bash
python3 -c "import xml.etree.ElementTree as ET; ET.parse('diagram.drawio'); print('XML OK')"
```

## Contenu

| Fichier | Rôle |
|---|---|
| `SKILL.md` | Référence mxGraph : modèle de cellule, échappement HTML, squelette, blocs copier-coller, table de styles, workflow, common mistakes |
| `template.drawio` | Squelette valide prêt à copier (titre + zone swimlane + 2 nodes + 1 edge) |

## Licence

MIT — voir [`LICENSE`](LICENSE).
