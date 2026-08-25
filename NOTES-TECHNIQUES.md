# Villa Namasté — Notes techniques

Carnet de bord du système d'automatisation.
Dernière mise à jour : 25 août 2026

---

## Architecture

| Composant | Rôle |
|---|---|
| GitHub Pages | Site vitrine `stormy30.github.io/villa-namaste` |
| Airtable — base "Réservations Villa" | Tables : Réservations Villa, Disponibilités, Demandes de contact |
| Make.com | 6 scénarios d'orchestration |

---

## Scénarios Make

### Actifs (webhooks — coût nul au repos)
- **Site Web — Formulaire de Contact** : reçoit les demandes du site → Airtable + email
- **Site Web — Lecture Disponibilités (calendrier)** : alimente le calendrier du site en JSON

### En pause depuis le 25/08/2026 (quota Airtable dépassé)
- **Sync Disponibilités — iCal Booking & Airbnb** : 1×/jour (1440 min)
- **Alerte Ménage — Départs J+1**
- **Email Bienvenue — Séjours Piscine (>4 nuits)**
- **Import Réservations — CSV Gmail (Booking.com)** : déclenchement manuel (Run once)

### Archives
- **Integration HTTP (copy)ANCIEN** : sauvegarde d'avant la correction du 24/07/2026

---

## ⚠️ Règles à ne jamais enfreindre

### Fréquence de la sync iCal : minimum 1440 min
Juillet 2026 : réglée à 5 min par erreur, avec retraitement complet du dataset
à chaque passage → **18 783 crédits Make en une semaine**.

### Quota Airtable : 1 000 appels API/mois (plan Free)
Chaque **Create / Update / Delete** = 1 appel **par enregistrement**.
Un *Search* qui remonte 100 lignes = 1 seul appel.

Août 2026 : **4 207 appels** consommés → API bloquée jusqu'au 1er septembre.
Double cause :
1. Bug Booking.com — 60 dates aléatoires bloquées sans tarif, remontées dans l'iCal
   (corrigé côté Booking en paramétrant 365 jours disponibles et tarifés)
2. Le scénario de sync **supprime et recrée tout** à chaque passage

### Import Réservations — CSV Gmail
Le module *Watch emails* doit garder :
- le filtre `is:unread`
- **Mark as read = Yes**

Sans ça, il rescanne tout l'historique Gmail à chaque exécution.
Le Message ID du module pièces jointes doit rester **lié dynamiquement**
au module précédent, jamais figé en dur.

---

## 🔧 Chantier ouvert

**Revoir la logique de la sync iCal.**
Aujourd'hui : rase et reconstruit la table à chaque passage (~220 opérations).
Objectif : comparer l'iCal à l'existant et ne toucher que les différences.
Cible : moins de 10 appels Airtable par jour.

Piste complémentaire : filtrer les séjours dont la date de fin est passée —
inutile de traiter l'historique.

---

## Repli du calendrier (site)

`index.html` contient une constante `DISPO_SECOURS` avec des dates figées.
Si le webhook de lecture échoue, le calendrier affiche ces dates et ajoute
« · mise à jour manuelle » à côté du nom du mois.

**Penser à mettre ces dates à jour si l'indisponibilité se prolonge.**

Dates actuellement figées :
- 23/08/2026 → 30/08/2026
- 02/09/2026 → 11/09/2026

---

## Reprise du 1er septembre 2026

1. Vérifier la réinitialisation du quota (workspace settings → billing)
2. Réactiver Alerte Ménage et Email Bienvenue (peu coûteux)
3. **Ne remettre la sync en route qu'après avoir revu sa logique**
4. Surveiller la consommation Airtable pendant les premiers jours
