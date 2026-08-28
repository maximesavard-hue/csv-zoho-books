# DECISIONS.md — Journal des décisions

Journal des choix structurants du projet, pour garder une ligne directrice cohérente d'une
session Claude Code à l'autre. Ajouter une entrée après toute décision qui affecte le format,
le comportement ou l'architecture — pas besoin de logger les détails déjà couverts par git log.

Format d'entrée :
```
## AAAA-MM-JJ — Titre court
Contexte : pourquoi ce besoin/problème s'est posé.
Décision : ce qui a été choisi.
Raison : pourquoi ce choix plutôt qu'un autre.
```

---

## 2026-06-29 — Choix de l'architecture (reconstruit du commit initial)
Contexte : besoin de convertir les exports CSV de Desjardins et TD vers un format standard
importable dans Zoho Books.
Décision : page HTML unique, tout en JS vanilla inline, sans build ni dépendance. Config par
banque dans un objet `BANKS` plutôt qu'un système de plugins.
Raison : projet personnel à usage restreint (2 banques), simplicité et zéro maintenance
d'outillage priorisées sur l'extensibilité.

## 2026-07-30 — Préfixe mmaa dans le nom de fichier exporté
Contexte : besoin de distinguer facilement les fichiers exportés par mois/année lors de
l'import dans Zoho Books.
Décision : le nom de fichier téléchargé est préfixé par `mmaa` (mois+année de la dernière
transaction du lot), suivi du nom de la banque/compte, ex. `0824Desjardins_EOP_ZohoBooks.csv`.
Raison : facilite le tri et l'identification des fichiers dans l'explorateur de fichiers.

## 2026-08-28 — Ajout de CLAUDE.md et DECISIONS.md
Contexte : ajout prévu d'une nouvelle banque (Triangle Mastercard) ; besoin de garder une
continuité de contexte entre les sessions Claude Code.
Décision : création de `CLAUDE.md` (instructions génériques et ligne directrice de travail) et
de `DECISIONS.md` (journal des décisions structurantes, à mettre à jour à chaque changement
important).
Raison : le projet n'a ni README ni documentation ; tout le contexte vivait uniquement dans le
code, ce qui rend difficile la reprise du travail d'une session à l'autre.

## 2026-08-28 — Ajout du support Triangle Mastercard
Contexte : ajout d'une nouvelle carte de crédit (Mastercard Triangle/Canadian Tire). Analyse de
plusieurs exports CSV et des relevés PDF correspondants (dossier
`Z:\General\Finance\Releve_banquaire\MasterCard Triangle - Perso`).
Décision :
- Format CSV Triangle : colonnes `REF, DATE DE LA TRANSACTION, DATE D'INSCRIPTION, CATEGORIE,
  DESCRIPTION, Catégorie(FR), MONTANT`. `CATEGORIE` vaut `PURCHASE`/`PAYMENT`/`RETURN`.
  `MONTANT` est un montant signé unique (positif = achat, négatif = paiement ou retour) —
  confirmé conforme au PDF (section "Débits totaux" = achats positifs, "Crédits totaux" =
  paiements/retours négatifs).
- Le nombre de lignes de préambule avant l'en-tête varie selon le fichier (0 à 3 lignes). Ajout
  d'un mécanisme générique `headerMarker` : au lieu de sauter un nombre fixe de lignes
  (`skipFirstRow`), on cherche la ligne dont la première colonne égale `headerMarker` (`'REF'`
  pour Triangle) et on prend tout ce qui suit comme données.
- Ajout d'un mécanisme générique `signedAmountCol` dans `BANKS` : quand une banque n'a pas de
  colonnes Debit/Credit séparées mais un seul montant signé, positif → Debit, négatif → Credit
  (montant absolu).
- Date utilisée dans la colonne "Date" du CSV Zoho : **date de la transaction** (pas date
  d'inscription), pour que la date corresponde à celle visible sur les reçus/photos de reçus que
  l'utilisateur associe manuellement aux transactions dans Zoho Books.
- Pas de `needsAccountFilter` pour Triangle : un seul compte/carte par fichier.
- Pas de colonne solde par transaction dans le CSV Triangle (seulement un solde global en
  préambule) → colonne `Solde` laissée vide pour cette banque.
Raison : reproduire fidèlement le relevé bancaire officiel (PDF) est la priorité absolue, car les
fichiers sont ensuite importés dans Zoho Books et associés à des reçus photo — toute divergence
de date ou de montant casserait ce rapprochement.
