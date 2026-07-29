# Analyse SQL des courses Uber

Analyse d'une base de données relationnelle de type Uber (8 tables, ~10 000 courses) réalisée en SQL (MySQL), avec pour objectif d'identifier les leviers de revenu et les points de friction opérationnels de la plateforme.

## Contexte

Ce projet simule le travail d'un Data Analyst chargé d'exploiter les données d'exploitation d'une plateforme de VTC afin d'en tirer des enseignements business exploitables par les équipes opérationnelles et marketing.

## Objectifs

- Mesurer la performance globale de l'activité (volume, revenu, panier moyen)
- Identifier les patterns temporels et géographiques de la demande
- Quantifier et comprendre les annulations
- Proposer des recommandations business actionnables

## Problématique

> Quels sont les leviers de revenu et les points de friction dans l'activité Uber, et quelles actions concrètes en tirer ?

## Données utilisées

Base MySQL relationnelle composée de 8 tables :

| Table | Contenu |
|---|---|
| `trips` | Table centrale : détail de chaque course |
| `users` | Comptes utilisateurs (clients + chauffeurs) |
| `drivers` | Informations véhicule/chauffeur |
| `riders` | Informations client |
| `locations` | Zones de prise en charge / dépose |
| `payments` | Paiements liés aux courses |
| `reviews` | Avis post-course |
| `cancellations` | Détail des annulations |

Le dataset source (20 000 courses) a été réduit à ~10 000 courses par **échantillonnage stratifié** (mois × statut) afin de préserver fidèlement les distributions temporelles et catégorielles d'origine.

## Méthodologie

1. Nettoyage et réduction du dataset (échantillonnage stratifié, cascade sur les tables liées)
2. Création d'une vue métier `trips_full` (jointure courses + zones + véhicules) pour des analyses lisibles sans complexifier les requêtes
3. Analyses SQL progressives : agrégats globaux → segmentation → analyse temporelle
4. Interprétation business systématique de chaque résultat

## Installation

```bash
mysql -u root -p < uber_reduced.sql
```

Le script crée automatiquement la base `Uber`, ses tables et la vue `trips_full`.

## Requêtes principales

```sql
-- Revenu total et panier moyen
SELECT ROUND(SUM(total_fare),2) AS revenu_total,
       ROUND(AVG(total_fare),2) AS panier_moyen
FROM trips_full
WHERE status = 'completed';

-- Revenu par jour de la semaine
SELECT DAYNAME(requested_at) AS jour, COUNT(*) AS nb,
       ROUND(SUM(total_fare),2) AS revenu,
       ROUND(AVG(surge_multiplier),2) AS surge_moyen
FROM trips_full
WHERE status = 'completed'
GROUP BY jour
ORDER BY revenu DESC;

-- Taux et causes d'annulation
SELECT cancelled_by, COUNT(*) AS nb
FROM cancellations
GROUP BY cancelled_by;
```

## Résultats clés

- **301 992 $** de revenu total, panier moyen de **35,90 $**
- Le **vendredi** génère **+15% de revenu** par rapport au 2ᵉ meilleur jour, porté par un surge moyen de **1.48** contre **1.02** le dimanche
- Taux d'annulation de **14,8%**, dont **69%** à l'initiative du client (attente trop longue, changement d'avis)
- Demande **géographiquement homogène** sur les 40 zones : pas de hotspot dominant

## Limites

- Dataset échantillonné et partiellement synthétique : pas de saisonnalité ni de rush hour marqué détecté ici, à valider sur des données réelles
- Pas de colonne "catégorie de course" (UberX/XL) dans le schéma source ; l'analyse par type de trajet s'appuie sur le moyen de paiement

## Recommandations

1. Renforcer l'offre de chauffeurs le vendredi pour capter le pic de demande lié au surge
2. Réduire les annulations liées à l'attente en améliorant la précision de l'ETA affiché au client
3. Privilégier une stratégie de rétention transverse plutôt qu'un ciblage marketing par zone

## Stack technique

`MySQL` · `SQL`

---
