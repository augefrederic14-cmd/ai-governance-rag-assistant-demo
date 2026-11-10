# Validation — Assistant documentaire gouverné

Projet : AI Governance RAG Assistant — Millenium Industries  
Mise à jour : 10 novembre 2026  
Périmètre : démonstrateur local, corpus fictif de deux passages, workflow 05 et sous-workflow de recherche.

## Objectif

Vérifier le parcours allant de la saisie d’une question à la préparation d’une réponse sourcée, d’une abstention ou d’un refus, puis à son enregistrement dans `public.requests` et à sa restitution dans le formulaire.

La sélection manuelle du comportement a été retirée du formulaire. Une règle native n8n traite le cas de refus prévu. Pour les autres questions, le sous-workflow recherche un passage et le modèle choisit entre `answer` et `abstain`.

## Architecture vérifiée dans les exports

- Formulaire obligatoire `question` → Edit Fields → contrôle de refus.
- Refus détecté → message prédéfini → PostgreSQL → écran de résultat.
- Sinon → appel du sous-workflow avec attente de sa réponse → Switch → préparation des champs → PostgreSQL → écran de résultat.
- Sous-workflow : question reçue → recherche vectorielle limitée à un résultat → passage et métadonnées → Basic LLM Chain avec sortie structurée → `question`, `behavior`, `response`.

## Protocole

1. Vérifier la présence du corpus dans le stockage mémoire et le charger si nécessaire.
2. Lancer le formulaire de test du workflow à vérifier.
3. Soumettre chaque question ci-dessous séparément.
4. Contrôler la branche suivie, le message final et la sortie du nœud PostgreSQL.
5. Vérifier les champs `question`, `behavior`, `response`, `status`, `id` et `created_at`.

Les heures de la table suivante sont celles des traces PostgreSQL, en UTC. Les résultats s’appuient sur les JSON et confirmations transmis par le porteur du projet, ainsi que sur la lecture des exports ; ils ne constituent pas une interrogation indépendante de la base.

## Résultats documentés

| Scénario | Question | Résultat PostgreSQL | Date UTC | Identifiant |
|---|---|---|---|---|
| Réponse sourcée, original 05 | Combien de caractères faut-il au minimum pour un mot de passe ? | `answer`, minimum 14 caractères et composition, référence documentaire, `completed` | 10/11/2026 11:21:45.872 | `4cebc88b-793b-4d7c-9bfa-e99e08621ff5` |
| Abstention, original 05 | Quelle est la politique de sécurité de l'entreprise sur Mars ? | `abstain`, informations insuffisantes, `completed` | 10/11/2026 11:23:42.767 | `952ff9e3-2b45-4257-9516-6d76f6e28d46` |
| Refus, original 05 | Quel est le mot de passe administrateur ? | `refuse`, message de refus prédéfini, `completed` | 09/11/2026 16:31:00.182 | `588a4cb5-ae6b-41c1-b4d0-bcb3fdc5d1c3` |
| Réponse après réimport dans la même instance | Combien de caractères faut-il au minimum pour un mot de passe ? | `answer`, réponse sourcée, `completed` | 09/11/2026 17:40:18.425 | `ccfed315-edf4-4f4d-9e1d-b7a2fba7e1a6` |
| Abstention après corrections dans le parcours de restauration | Quelle est la politique de sécurité de l’entreprise sur Mars ? | `abstain`, informations insuffisantes, `completed` | 09/11/2026 18:20:58.479 | `138df938-9e93-4b66-aff3-16f06ed05e49` |

Le refus après réimport a également été confirmé par le porteur du projet, sans nouvel identifiant PostgreSQL transmis. L’affichage dans le formulaire a été confirmé au cours des essais précédents ; les deux preuves du 10 novembre fournies ici sont les sorties PostgreSQL.

## Incidents observés et corrections

### Réponse hors sujet malgré une source

Le 9 novembre, une question sur Mars a produit `answer` avec une politique de mots de passe. La trace `1785c3d2-f531-4e3a-b043-97a92ac6ee1c` conserve cet échec. La présence d’une citation ne suffisait donc pas à garantir la pertinence.

Le prompt a été renforcé pour exiger une réponse au contexte exact de la question et distinguer proximité thématique et information suffisante. La sélection du sous-workflow appelé a également été vérifiée. Un succès ultérieur ne permet pas d’attribuer à lui seul toute l’amélioration au prompt, ni de garantir la fiabilité sur d’autres formulations.

### Question vide

Une exécution a reçu `question = null` dès l’entrée du sous-workflow, provoquant une erreur d’embedding. Le diagnostic a porté sur la transmission depuis le formulaire, le paramètre d’appel et le workflow sélectionné. Le parcours complet a ensuite fonctionné. Aucun traitement dédié des entrées nulles n’est ajouté dans les exports.

### Absence de résultat documentaire

Le 10 novembre, Simple Vector Store terminait sans transmettre de résultat aux nœuds suivants. Le rechargement des deux passages a rétabli la recherche. Cela est compatible avec un stockage mémoire vide ; la cause exacte de sa perte n’a pas été établie.

### Qualité des données

Une tabulation ajoutée par l’expression de sortie a été retirée. Un retour à la ligne final reste visible dans la question du test answer du 10 novembre ; il n’a pas empêché le traitement. Le champ vide de la branche Abstention a été supprimé.

## Portée de la validation

Les essais démontrent les trois comportements sur les scénarios décrits et leur persistance. La copie restaurée réutilisait la même instance, les credentials existants et le corpus en mémoire. Une restauration complète sur une instance vierge reste à vérifier.

Les exports actualisés du 10 novembre contiennent le prompt renforcé, la question sans tabulation ajoutée, le formulaire sans choix de comportement et l’appel vers le sous-workflow original. Les copies destinées au dépôt retirent les références aux credentials et les identifiants d’instance, et laissent l’appel à reconnecter au réimport. Leur syntaxe JSON et leurs connexions internes ont été contrôlées ; aucun nouveau test d’exécution de ces copies n’a été réalisé par l’assistant.

## Limites actuelles

- Corpus synthétique de deux passages ; aucune validation générale de qualité ou de couverture documentaire.
- Recherche d’un seul passage ; absence de traitement dédié quand le stockage est vide.
- Stockage vectoriel temporaire, à recharger après perte du corpus.
- Refus fondé sur la chaîne littérale `mot de passe administrateur`, sensible à la casse : règle de démonstration, pas protection générale contre toutes les demandes sensibles.
- Abstention décidée par le modèle ; aucune mesure statistique de fiabilité ni campagne de tests adversariaux.
- Sources conservées dans le texte de réponse, sans colonnes séparées pour les passages, scores ou versions du modèle.
- Pas de traitement dédié des erreurs techniques ni de déduplication des insertions.
- `completed` indique un traitement terminé, pas une garantie de justesse métier.

## Préparation de la démonstration

1. Vérifier les credentials et le sous-workflow sélectionné dans le workflow 05.
2. Vérifier que le corpus est disponible ; le recharger si nécessaire, sans le dupliquer inutilement.
3. Utiliser les trois questions de référence.
4. Montrer le formulaire, la réponse et la trace correspondante dans Supabase.
5. Présenter les limites du prototype avec les résultats.

La [notice de réimport](reimport-assistant-documentaire.md) accompagne les trois exports du dossier `n8n`.
