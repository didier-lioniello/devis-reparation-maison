# 02 · Devis réparation maison par photo (n8n + IA Vision)

## Objectif
Le client photographie le problème (fuite, carrelage cassé, mur fissuré, porte qui frotte…), clique sur « Envoyer » et reçoit **deux budgets** :
- **Avec l'entreprise** : main-d'œuvre, matériel et déplacement, avec la TVA (TTC).
- **En le faisant lui-même** : liste du matériel avec les prix, temps estimé, difficulté et guide étape par étape.

## Schéma

```mermaid
flowchart TD
    A[Client : photos + courte description<br/>formulaire web / WhatsApp] --> B{Photos exploitables ?}
    B -- Non --> B2[Demande de photos<br/>plus nettes ou plus larges]
    B2 --> A
    B -- Oui --> C[IA Vision : type de réparation,<br/>pièce, surface, gravité, confiance]
    C --> S{Risque sécurité ?<br/>gaz, électricité tableau,<br/>structure, toiture}
    S -- Oui --> P[Pro obligatoire<br/>pas d'option DIY]
    S -- Non --> D{Choix du client}
    P --> E
    D -- Avec l'entreprise --> E[Chiffrage pro via la grille<br/>MO × taux + matériel + déplacement]
    E --> T[TVA : 10 % rénovation logement<br/>de plus de 2 ans, sinon 20 %]
    T --> V[Validation de l'artisan en 1 clic]
    V --> R1[Devis indicatif TTC<br/>+ lien de RDV]
    D -- Je le fais moi-même --> F[Liste du matériel<br/>prix catalogue TTC]
    F --> G[Guide pas à pas + outils<br/>temps et difficulté]
    G --> R2[Budget DIY + guide<br/>+ option « finalement je veux un pro »]
    R2 -.-> E
    R1 --> L[Stockage CRM / Airtable]
    R2 --> L
```

## Explication étape par étape
1. **Envoi** : le client remplit un formulaire simple (photos et une phrase), envoyé à un webhook n8n.
2. **Contrôle des photos** : si elles sont floues ou trop serrées, un message automatique demande d'autres photos.
3. **Analyse IA** : le modèle renvoie un JSON (catégorie : plomberie, électricité, peinture, menuiserie ou maçonnerie ; élément touché ; surface estimée ; gravité ; confiance). **L'IA ne fixe aucun prix.**
4. **Filtre sécurité** : gaz, tableau électrique, structure et toiture imposent un professionnel. L'option « faire soi-même » n'est pas proposée.
5. **Avec l'entreprise** : un nœud Code applique la grille de l'artisan (taux horaire, temps par type d'intervention, matériel, déplacement).
6. **TVA** : 10 % pour des travaux d'amélioration ou d'entretien dans un logement de plus de 2 ans (attestation du client), 20 % sinon. Le taux de 5,5 % s'applique à la rénovation énergétique.
7. **Validation de l'artisan** : il approuve ou ajuste en un clic, puis le devis indicatif TTC part avec un lien de rendez-vous.
8. **Faire soi-même** : liste du matériel avec les prix (catalogue maintenu dans Google Sheets ou API d'enseigne), outils nécessaires, temps, difficulté sur 5 et tutoriel généré par l'IA.
9. **Retour possible** : un bouton « finalement je préfère un pro » renvoie vers le chiffrage de l'entreprise, ce qui permet de récupérer le lead.
10. **Suivi** : toutes les demandes sont stockées pour mesurer la conversion.

## Nœuds n8n principaux
`Webhook` → `IF` → `OpenAI` (vision) → `Code` (parse + règles sécurité) → `Switch` (pro / DIY) → `Google Sheets` (grille et catalogue) → `Wait` (validation) → `HTML to PDF` → `Email/WhatsApp` → `Airtable`.

## Garde-fous
- Les prix viennent de la grille de l'artisan et du catalogue, jamais de l'IA.
- Le prix est donné en fourchette avec la mention « estimation indicative, sous réserve de visite ».
- Le DIY est bloqué pour les interventions dangereuses ou réglementées.
- RGPD : consentement au stockage des photos et durée de conservation définie.
