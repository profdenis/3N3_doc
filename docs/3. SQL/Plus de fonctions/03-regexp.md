# Guide des Expressions Régulières (Regex) pour les Bases de Données

- [PostgreSQL Doc](https://www.postgresql.org/docs/current/functions-matching.html)
- [RegEx101](https://regex101.com) (choisir POSIX ERE)

## 1. Le Pattern Matching (Recherche de motifs)

Avant de plonger dans les expressions régulières, il est crucial de comprendre le concept de **Pattern Matching**.

Le *pattern matching* est une technique qui consiste à chercher une séquence de caractères spécifique (un "motif") à
l'intérieur d'un texte. Dans une base de données, on ne cherche pas seulement une correspondance exacte (ex:
`nom = 'Dupont'`), mais souvent des correspondances partielles (ex: "tous les noms qui commencent par D").

### L'opérateur `LIKE` et `ILIKE` (Standard SQL)

En SQL (et particulièrement dans PostgreSQL), on utilise d'abord des opérateurs simples pour le pattern matching avant
d'utiliser les Regex.

#### L'opérateur `LIKE` (Sensible à la casse)

Il utilise deux caractères spéciaux (wildcards) :

* `%` : Représente **zéro, un ou plusieurs** caractères.
* `_` : Représente **un seul** caractère.

**Exemples :**

* `WHERE nom LIKE 'D%'` $\rightarrow$ Tous les noms commençant par 'D' (*Dupont, Durand, D*).
* `WHERE nom LIKE '%son'` $\rightarrow$ Tous les noms finissant par 'son' (*Harrison, Wilson*).
* `WHERE code LIKE 'A_Z'` $\rightarrow$ Un code de 3 lettres commençant par A et finissant par Z (*ABZ, A1Z, A-Z*).

#### L'opérateur `ILIKE` (Insensible à la casse)

PostgreSQL propose `ILIKE`, qui fonctionne exactement comme `LIKE` mais ignore la différence entre majuscules et
minuscules.

**Exemple :**

* `WHERE nom ILIKE 'dupont'` $\rightarrow$ Trouvera 'Dupont', 'DUPONT', 'duPont', etc.

---

## 2. Les Expressions Régulières (Regex) en Général

Le `LIKE` est limité. Si vous voulez chercher "un numéro de téléphone qui commence par 06 ou 07" ou "une adresse email
valide", le `LIKE` devient vite illisible. C'est là qu'interviennent les **Expressions Régulières (Regex)**.

Une Regex est un langage formel très puissant qui définit un modèle de recherche complexe.

### La syntaxe de base (Les métacaractères)

| Symbole | Nom                   | Signification                                             | Exemple                                                            |
|:--------|:----------------------|:----------------------------------------------------------|:-------------------------------------------------------------------|
| `.`     | Point                 | N'importe quel caractère unique (sauf saut de ligne)      | `c.t` $\rightarrow$ cat, cot, c8t                                  |
| `^`     | Accent circonflexe    | Début de la chaîne                                        | `^A` $\rightarrow$ commence par A                                  |
| `$`     | Dollar                | Fin de la chaîne                                          | `z$` $\rightarrow$ finit par z                                     |
| `|`    | Pipe                   | **OU** logique (choix entre plusieurs motifs)             | `chat|chien` $\rightarrow$ cherche l'un ou l'autre                 |
| `()`    | Parenthèses           | **Groupe** de caractères (permet de lier des éléments)    | `(abc)+` $\rightarrow$ cherche "abcabc..."                         |
| `[]`    | Crochets              | **Classe** de caractères (un seul élément parmi la liste) | `[aeiou]` $\rightarrow$ une voyelle                                |
| `{n}`   | Accolades (fixe)      | Répétition **exactement $n$ fois**                        | `\d{3}` $\rightarrow$ exactement 3 chiffres                        |
| `{n,m}` | Accolades (plage)     | Répétition **entre $n$ et $m$ fois**                      | `\d{2,4}` $\rightarrow$ de 2 à 4 chiffres                          |
| `*`     | Astérisque            | **0 ou plusieurs** fois l'élément précédent               | `ab*` $\rightarrow$ a, ab, abb...                                  |
| `+`     | Plus                  | **1 ou plusieurs** fois l'élément précédent               | `ab+` $\rightarrow$ ab, abb... (mais pas "a")                      |
| `?`     | Point d'interrogation | **0 ou 1 fois** (rend l'élément optionnel)                | `chais?e` $\rightarrow$ chaise ou chise                            |
| `\`     | Backslash             | **Échappement** (pour un caractère spécial ou un code)    | `\.` $\rightarrow$ un vrai point (pas le symbole "n'importe quoi") |
| `\d`    | Séquence              | Un **chiffre** (digit)                                    | `\d\d` $\rightarrow$ deux chiffres                                 |
| `\w`    | Séquence              | Un **caractère de mot** (lettre, chiffre, _)              |                                                                    |
| `\s`    | Séquence              | Un **espace** blanc (espace, tabulation)                  |                                                                    |

---

## 3. Les Regex dans PostgreSQL

PostgreSQL utilise l'implémentation des expressions régulières de type **POSIX**. Contrairement au `LIKE`, on utilise
des opérateurs spécifiques.

### Les opérateurs de comparaison

* `~` : Correspondance (match) avec respect de la casse.
* `~*` : Correspondance avec **insensibilité** à la casse.
* `!~` : Ne correspond **pas** (respect de la casse).
* `!~*` : Ne correspond **pas** (insensibilité à la casse).

---

### A. Niveau Simple : Utilisation basique

On utilise ici les opérateurs pour filtrer des données avec des motifs simples.

**Exemple 1 : Trouver des noms commençant par 'S' ou 'T' (insensible à la casse)**

```sql
SELECT nom
FROM clients
WHERE nom ~* '^[st]';
-- ^ : début de chaîne
-- [st] : soit s, soit t
```

**Exemple 2 : Trouver des codes produits qui sont composés d'exactement 3 chiffres**

```sql
SELECT code_produit
FROM produits
WHERE code_produit ~ '^\d{3}$';
-- ^ : début
-- \d{3} : exactement 3 chiffres
-- $ : fin
```

---

### B. Niveau Complexe : Combinaisons et classes

Ici, on commence à utiliser des parenthèses pour grouper des éléments et des quantificateurs plus précis.

**Exemple 3 : Valider un format de date simple (JJ/MM/AAAA)**
On veut vérifier que la date ressemble à `01/01/2023`.

```sql
SELECT date_commande
FROM commandes
WHERE date_commande ~ '^\d{2}/\d{2}/\d{4}$';
```

**Exemple 4 : Extraire des domaines d'emails**
On cherche tous les utilisateurs ayant une adresse email finissant par `.com` ou `.fr`.

```sql
SELECT email
FROM utilisateurs
WHERE email ~ '@.*\.(com|fr)$';
-- @ : contient un arobase
-- .* : n'importe quoi
-- \. : un point (on met un backslash car le point seul est un métacaractère)
-- (com|fr) : soit com, soit fr
-- $ : à la fin de la chaîne
```

---

### C. Niveau Avancé (Optionnel pour les étudiants)

Pour les besoins de l'analyse de données, PostgreSQL propose des fonctions puissantes pour manipuler les résultats des
Regex.

#### 1. `regexp_matches()` : Extraction de données

Cette fonction permet de "capturer" une partie du texte.

```sql
-- On veut extraire uniquement le prénom d'une colonne 'nom_complet' (format "Prénom Nom")
SELECT regexp_matches('Jean Dupont', '^(\w+) \w+$');
-- Retourne : 'Jean'
```

#### 2. `regexp_replace()` : Modification de texte

Permet de nettoyer une colonne en remplaçant un motif par autre chose.

```sql
-- On veut enlever tous les tirets d'un numéro de téléphone
SELECT regexp_replace('06-12-34-56-78', '-', '');
-- Retourne : '0612345678'
```

#### 3. Les Groupes de capture (Backreferences)

On peut utiliser les parenthèses pour isoler des morceaux et les réutiliser.

```sql
-- Inverser le nom et le prénom (Format: "Prénom Nom" -> "Nom, Prénom")
SELECT regexp_replace('Jean Dupont', '^(\w+) (\w+)$', '\2, \1');
-- \1 est le premier groupe, \2 le deuxième.
-- Retourne : 'Dupont, Jean'
```

# Exercices : Maîtriser les Expressions Régulières

**Consignes :** Pour chaque exercice, déterminez la **Regex** (le motif) qui permettrait de valider ou de trouver la
"Donnée cible" dans une chaîne de texte.

---

## Section A : Niveau Simple (Fondamentaux)

*Objectif : Utiliser les ancres (`^`, `$`), les classes de caractères (`[]`, `\d`) et les quantificateurs de base.*

**Exercice 1 : Le code produit**

* **Contexte :** Vous devez identifier des codes produits qui commencent obligatoirement par la lettre 'P' (majuscule)
  suivie d'exactement 3 chiffres.
* **Donnée cible :** `P123`
* **Votre Regex :** ____________________

**Exercice 2 : L'identifiant utilisateur**

* **Contexte :** Un système demande un identifiant composé de seulement 5 lettres minuscules (pas de chiffres, pas de
  majuscules).
* **Donnée cible :** `abcde`
* **Votre Regex :** ____________________

**Exercice 3 : La catégorie de prix**

* **Contexte :** Vous cherchez des cellules qui ne contiennent qu'un seul caractère, et ce caractère doit être un
  chiffre.
* **Donnée cible :** `5`
* **Votre Regex :** ____________________

**Exercice 4 : Le mot de passe minimaliste**

* **Contexte :** Un champ de test accepte soit le mot "Oui", soit le mot "Non" (attention à la casse : il doit être
  identique).
* **Donnée cible :** `Oui`
* **Votre Regex :** ____________________

---

## Section B : Niveau Complexe (Combinaisons et Classes)

*Objectif : Utiliser les parenthèses `()`, le "OU" `|`, les plages `{n,m}`, et l'échappement `\`.*

**Exercice 5 : Le format de téléphone simplifié**

* **Contexte :** Vous voulez repérer des numéros de téléphone formatés avec des tirets, composés de deux groupes de deux
  chiffres, séparés par un tiret (ex: 12-34).
* **Donnée cible :** `12-34`
* **Votre Regex :** ____________________

**Exercice 6 : La validation de prix européen**

* **Contexte :** Un prix peut être écrit avec un point ou une virgule pour séparer les décimales (ex: `19.99` ou
  `19,99`). On veut capturer le montant avec deux chiffres après la virgule/point.
* **Donnée cible :** `19,99`
* **Votre Regex :** ____________________

**Exercice 7 : L'extension de fichier**

* **Contexte :** Vous cherchez des fichiers qui finissent soit par `.jpg`, soit par `.png`, soit par `.gif`.
* **Donnée cible :** `image.png`
* **Votre Regex :** ____________________

**Exercice 8 : L'identifiant alphanumérique complexe**

* **Contexte :** Un code d'inventaire est composé de 2 lettres suivies d'un tiret, puis de 3 à 5 chiffres.
* **Donnée cible :** `AB-12345`
* **Votre Regex :** ____________________

---

## Section C : Niveau Avancé (Fonctions de manipulation)

*Objectif : Préparer l'utilisation de `regexp_replace` et `regexp_matches`.*

**Exercice 9 : Le nettoyage de texte (Substitution)**

* **Contexte :** Vous avez une colonne `adresse_client` qui contient des espaces inutiles entre les mots. Vous voulez
  utiliser `regexp_replace` pour remplacer chaque espace par un tiret `-`.
* **Donnée cible :** `Rue de la Paix` $\rightarrow$ `Rue-de-la-Paix`
* **Votre Regex (le motif à chercher) :** ____________________

**Exercice 10 : L'extraction de données (Capture)**

* **Contexte :** Vous avez une colonne `description_produit` qui contient : "Réf: 9982 - Produit Bleu". Vous voulez
  extraire uniquement le numéro de référence (les chiffres) en utilisant des groupes de capture.
* **Donnée cible :** `9982`
* **Votre Regex (incluant les parenthèses pour la capture) :** ____________________

---
---

# Correction

**Section A :**

1. `^P\d{3}$`
2. `^[a-z]{5}$`
3. `^\d$`
4. `^(Oui|Non)$`

**Section B :**

5. `^\d{2}-\d{2}$`
6. `^\d+[\.,]\d{2}$`
7. `\.(jpg|png|gif)$`
8. `^[A-Z]{2}-\d{3,5}$`

**Section C :**

9. ` ` (un espace) ou `\s`
10. `Réf: (\d+)` (Le groupe de capture est `(\d+)`)




-------

??? info "Utilisation de l'IA"
    Page rédigée en partie avec l'aide d'un assistant IA. L'IA a été utilisée pour générer des
    explications, des exemples et/ou des suggestions de structure. Toutes les informations ont
    été vérifiées, éditées et complétées par l'auteur.