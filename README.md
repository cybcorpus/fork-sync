# fork-sync

Un seul workflow pour maintenir à jour **tous les forks** de l'organisation.

Chaque entrée de `forks.json` est fusionnée avec son amont via
`POST /repos/{org}/{fork}/merge-upstream`. Ajouter un fork = ajouter
une entrée dans `forks.json` : aucun fichier à déposer dans le fork.

## Fonctionnement

| Élément | Valeur |
|---|---|
| Fréquence | chaque lundi 05:00 (Europe/Paris) |
| Manuel | Actions → Synchronisation des forks → Run workflow |
| Fork unique | champ `fork`, ex. `lynis` |
| Parallélisme | 6 forks à la fois |

## Résultats

| Situation | Effet |
|---|---|
| `200` | fork à jour, ou fusion effectuée |
| `409` conflit | `warning` — le fork a des commits locaux, **rien n'est écrasé** |
| `403` / `404` | `warning` — amont supprimé, renommé, ou jeton sans droit |

Le job par fork n'échoue que sur une erreur inattendue, pour ne pas masquer
les autres forks par un `fail-fast`.

## Secret

`SYNC_TOKEN` (Secrets → Actions, **au niveau du dépôt**) : PAT classic d'un
compte membre de l'organisation, scope `repo` + `workflow`.
Sans lui, l'API `merge-upstream` répond `403`.

Un secret d'organisation ne suffit pas si son accès est restreint aux dépôts
publics : le renseigner sur le dépôt évite cette ambiguïté.

## Origine des forks

- dépôts individuels : `cheatsheet_typst`, `obsidian_prompts`, `quartz`,
  `Open-Source-Security-Guide`, `lynis`, `Linux-Server-Hardening-Secure-Configuration`,
  `CIS-PDF-to-Excel`, `ansible`, `harness` ;
- **organisation `ansible-lockdown`** : GitHub ne permet pas de forker une
  organisation. Les dépôts actifs ont été forkés un par un — les dépôts
  archivés sont exclus, un fork archivé ne pouvant plus être mis à jour.

## Miroirs d'amonts non GitHub

`mirrors.json` couvre les amonts que GitHub ne peut pas forker (Bitbucket,
GitLab). `sync-mirrors.yml` clone l'amont et force-pousse le reflet : ces
dépôts ne reçoivent aucun commit local.

| Miroir | Amont |
|---|---|
| `cis-controls-v8.1_oscal` | `bitbucket.org/cis-it-workspace/cis-controls-v8.1_oscal` |

Ajouter un miroir = une entrée dans `mirrors.json`, aucun fichier à déposer
dans le dépôt cible.
