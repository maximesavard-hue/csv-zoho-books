# CLAUDE.md — Convertisseur CSV Banques → Zoho Books

## But du projet
Page web statique (`index.html`, HTML/CSS/JS vanilla, aucune dépendance/build) qui convertit
l'export CSV d'une institution financière en un CSV standardisé importable dans Zoho Books
(colonnes : `Date, Description, Debit, Credit, Solde`). Tout tourne côté client dans le navigateur.

## Ligne directrice de travail
- Toujours faire un `git pull` (ou vérifier l'état du dépôt distant) avant de commencer une tâche.
- Avant toute modification, lire `DECISIONS.md` pour connaître le contexte et les choix déjà faits.
- Après une décision structurante (nouveau format, changement de comportement, choix de design),
  ajouter une entrée dans `DECISIONS.md` — voir ce fichier pour le format.
- Garder le projet en un seul fichier statique sans dépendance, sauf si l'utilisateur demande
  explicitement d'introduire un build/framework.
- Ne pas ajouter de complexité (validation, gestion d'erreurs, abstractions) non requise par une
  banque réellement supportée.

## Architecture (index.html)
- `BANKS` : objet de config par banque. Chaque entrée définit :
  - `hasHeader` / `skipFirstRow` : présence d'une ligne d'en-tête à ignorer
  - `typeCol` : colonne indiquant le type de compte (si le CSV mélange plusieurs comptes/cartes)
  - `dateCol`, `dateFormat` (`YMD` ou `MDY`)
  - `descCol`, `debitCol`, `creditCol`, `balanceCol`
  - `needsAccountFilter` : true si un CSV peut contenir plusieurs comptes à distinguer
- `parseCSV` : parseur CSV maison (gère les guillemets basiques seulement).
- `buildOutput` : applique le filtre de compte (si requis) et mappe vers les colonnes de sortie.
- `renderAccountBtns` : génère les boutons de sélection de compte/carte quand `needsAccountFilter`
  est vrai ; les labels FR (ex. `EOP` → "Chèques") sont codés en dur — à étendre par banque.
- `getLastTransactionMonthYear` / `downloadCSV` : calculent le préfixe `mmaa` et le nom du
  fichier exporté (ex. `0824Desjardins_EOP_ZohoBooks.csv`).

## Procédure générique pour ajouter une nouvelle banque/carte
1. Obtenir un exemple réel d'export CSV (avec des données anonymisées si besoin) — ne jamais
   deviner le format sans exemple.
2. Déterminer : présence d'en-tête, ordre des colonnes, format de date, si débit/crédit sont
   dans des colonnes séparées ou une seule colonne signée (fréquent pour les cartes de crédit —
   attention au sens du signe : une charge peut être positive ou négative selon l'émetteur).
3. Ajouter une entrée dans `BANKS` avec un nom de clé clair (ex. `triangle`).
4. Si le CSV peut contenir plusieurs comptes/cartes dans un seul fichier, mettre
   `needsAccountFilter: true` et étendre les labels dans `renderAccountBtns`.
5. Ajouter le bouton correspondant dans le HTML (section "Étape 1 — Sélectionner la banque").
6. Tester avec le vrai fichier : vérifier l'aperçu (5 premières lignes), le total de transactions,
   le nom du fichier exporté, et idéalement une petite importation test dans Zoho Books.
7. Documenter le choix (format de date, signe débit/crédit, particularités) dans `DECISIONS.md`.

## État actuel des banques supportées
- Desjardins : CSV multi-comptes (EOP/ET2/MC2), sans en-tête, dates `YMD`.
- TD : un CSV par compte, sans en-tête, dates `MDY`.
- Triangle (Mastercard) : en-tête détecté dynamiquement (`headerMarker: 'REF'`, le nombre de
  lignes de préambule varie d'un fichier à l'autre), montant signé unique (`signedAmountCol`)
  plutôt que Debit/Credit séparés, date = date de la transaction (pas date d'inscription — voir
  `DECISIONS.md`), pas de colonne solde par transaction.
