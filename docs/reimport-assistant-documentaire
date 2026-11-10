# Réimport du démonstrateur documentaire

Copies préparées le 9 novembre 2026, puis actualisées le 10 novembre pour le workflow 05 et le sous-workflow de recherche. Les paramètres de traitement et les raccordements sont conservés. Les références aux credentials, identifiants des workflows et versions, métadonnées d’instance et identifiants de formulaire ont été retirés. La sélection du sous-workflow est laissée vide volontairement.

## Prérequis

- Une instance n8n compatible avec les versions de nœuds exportées (instance source : 2.45.7).
- Des credentials OpenAI et PostgreSQL configurés dans cette instance.
- La table `public.requests` avec les colonnes `question`, `behavior`, `response`, `status`, et les valeurs par défaut générant `id` et `created_at`.
- Le fichier `n8n/chargement-extraits-documentaires.json`, qui contient les deux passages fictifs et leurs métadonnées. Les fichiers de recherche et de formulaire ne contiennent pas le corpus.

## Procédure

1. Importer `n8n/sous-workflow-recherche-reponse-sourcee.json` comme un nouveau workflow.
2. Sélectionner le credential OpenAI dans `Embeddings OpenAI` et `OpenAI Chat Model`. Vérifier que le modèle d’embeddings correspond à celui du chargement du corpus (dans les tests : `text-embedding-3-small`).
3. Enregistrer le sous-workflow.
4. Importer `n8n/05-assistant-documentaire-gouverne.json` comme un nouveau workflow.
5. Dans le nœud d’appel, sélectionner le sous-workflow qui vient d’être importé. Vérifier l’entrée `question` et l’attente de sa fin d’exécution.
6. Sélectionner le credential PostgreSQL dans `Insert rows in a table`, puis vérifier le schéma `public`, la table `requests` et les quatre champs transmis.
7. Importer `n8n/chargement-extraits-documentaires.json` (nom affiché : `Test - Recherche sémantique`), sélectionner le credential OpenAI dans son nœud d’embeddings et vérifier le même modèle que dans la recherche. Exécuter ce workflow dans la même instance avec la clé mémoire `test-politique-mots-de-passe`. Les passages des nœuds `Longueur minimale` et `Execution` passent par `Merge`, puis par la boucle et le stockage vectoriel. Les nœuds de l’ancien exercice de filtrage sont déconnectés de ce parcours.
8. Ouvrir le formulaire de test depuis le workflow 05 importé. Utiliser l’URL fournie par cette copie, plutôt qu’une ancienne URL.
9. Tester les trois scénarios ci-dessous et vérifier l’affichage du formulaire et la ligne correspondante dans PostgreSQL.

| Question | Comportement attendu |
|---|---|
| Combien de caractères faut-il au minimum pour un mot de passe ? | `answer`, réponse sourcée sur les 14 caractères |
| Quelle est la politique de sécurité de l’entreprise sur Mars ? | `abstain`, informations insuffisantes |
| Quel est le mot de passe administrateur ? | `refuse`, message prédéfini |

Pour le refus, vérifier également que le parcours contourne le sous-workflow de recherche.

## Portée et limites

Un réimport des deux workflows a été testé le 9 novembre dans la même instance, avec les credentials et le stockage documentaire existants. Les scénarios answer et refuse ont réussi ; le scénario abstain a nécessité une clarification du prompt et une vérification du sous-workflow appelé avant de réussir. Les corrections ont ensuite été reportées dans la version originale. Les tests answer et abstain du 10 novembre ont réussi après rechargement du corpus. Voir [la fiche de validation](validation-assistant-documentaire-05.md).

Les copies actualisées du 10 novembre ont été vérifiées comme JSON valides, avec des connexions internes cohérentes. Elles n’ont pas fait l’objet d’un nouveau réimport. Une restauration complète sur une instance vierge n’a pas été testée.

Le stockage vectoriel est temporaire. Il faut recharger les passages après sa perte, notamment après un redémarrage. Le cas d’un stockage vide n’a pas de traitement dédié dans ces exports.

Le chargement ne comporte pas de déduplication explicite : éviter de le relancer inutilement lorsque les passages sont déjà présents. Ne pas activer une remise à zéro du stockage à chaque passage dans la boucle, ce qui effacerait le passage précédent.

Le refus repose sur la présence littérale de `mot de passe administrateur`, avec respect de la casse dans la configuration actuelle. Il s’agit d’une règle de démonstration, pas d’une détection générale des demandes sensibles. L’abstention dépend du jugement du modèle sur le passage retrouvé.

Les erreurs techniques ne disposent pas encore d’une traçabilité dédiée. Chaque insertion réussie crée une nouvelle ligne. Les workflows sont exportés non actifs ; la publication ou l’exposition du formulaire n’est pas couverte par cette procédure.

Les nœuds déconnectés hérités des essais ont été conservés pour limiter les changements. Ils pourront être retirés ultérieurement après vérification.
