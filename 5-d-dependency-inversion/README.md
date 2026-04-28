# Principe SOLID — D : Dependency Inversion Principle (DIP)
### Inversion de Dépendance — Fiche de cours Bac+4 Informatique

---

## Table des matières

1. [Définition](#1-définition)
2. [Les deux règles fondamentales](#2-les-deux-règles-fondamentales)
3. [Analogie concrète](#3-analogie-concrète)
4. [Mauvais exemple — Sans DIP](#4-mauvais-exemple--sans-dip)
5. [Bon exemple — Avec DIP](#5-bon-exemple--avec-dip)
6. [Diagrammes UML textuels](#6-diagrammes-uml-textuels)
7. [Schéma logique des dépendances](#7-schéma-logique-des-dépendances)
8. [Comparaison avant / après DIP](#8-comparaison-avant--après-dip)
9. [À retenir — Version courte](#9-à-retenir--version-courte)
10. [Conclusion](#10-conclusion)

---

## 1. Définition

Le **Dependency Inversion Principle (DIP)** est le cinquième principe SOLID, formulé par Robert C. Martin (*Uncle Bob*).

> **"Les modules de haut niveau ne doivent pas dépendre des modules de bas niveau. Les deux doivent dépendre d'abstractions. Les abstractions ne doivent pas dépendre des détails. Ce sont les détails qui doivent dépendre des abstractions."**
>
> — Robert C. Martin, *Agile Software Development*, 2003

En pratique, DIP signifie que l'on **inverse le sens habituel de la dépendance** : plutôt qu'un module métier dépende directement d'une implémentation concrète, il dépend d'une interface (contrat abstrait), et c'est l'implémentation concrète qui se conforme à ce contrat.

<p align="right"><em>— Page 1 / 9 —</em></p>
<div style="page-break-after: always;"></div>

## 2. Les deux règles fondamentales

### Règle 1 — Les modules de haut niveau ne doivent pas dépendre des modules de bas niveau

| Terme | Définition |
|---|---|
| **Module de haut niveau** | Contient la logique métier centrale. Orchestre les traitements. Ex : `Conducteur`, `GestionnaireFlotte` |
| **Module de bas niveau** | Fournit des fonctionnalités techniques ou concrètes. Ex : `Voiture`, `Moto`, `CamionElectrique` |

Sans DIP, le module de haut niveau **importe directement** le module de bas niveau. Si l'implémentation bas niveau change (ex : on passe d'une `Voiture` thermique à une `VoitureElectrique`), le code métier doit être modifié — ce qui est une violation du **Open/Closed Principle** et crée un **couplage fort**.

Avec DIP, les deux dépendent d'une **abstraction commune** (interface, classe abstraite) : le module de haut niveau programme contre le contrat, pas contre la réalisation.

---

### Règle 2 — Les abstractions ne doivent pas dépendre des détails ; ce sont les détails qui dépendent des abstractions

L'interface (abstraction) est définie selon les **besoins du module de haut niveau**, non selon les contraintes du module de bas niveau.

**Exemple concret :**
- Mauvais : l'interface `IVehicule` expose une méthode `injecter_carburant_diesel(litres: float)` — c'est un détail d'implémentation physique qui remonte dans l'abstraction.
- Bon : l'interface expose `demarrer()`, `accelerer(vitesse)`, `freiner()` — ce sont des termes métier, indépendants du type de motorisation.

<p align="right"><em>— Page 2 / 9 —</em></p>
<div style="page-break-after: always;"></div>

## 3. Analogie concrète

Imaginez un **GPS de navigation**.

- Le GPS (module de haut niveau) donne des instructions : *"Tournez à gauche dans 200m"*, *"Accélérez"*, *"Arrêtez-vous"*.
- Il ne sait pas si le conducteur est dans une **voiture**, sur une **moto** ou au volant d'un **camion électrique**.
- Chaque véhicule (module de bas niveau) doit savoir interpréter ces instructions — c'est le **contrat commun** (abstraction : `IVehicule`).
- Si demain un nouveau type de véhicule apparaît (scooter autonome, drone terrestre), le GPS fonctionne sans modification.

**Traduit en code :**

```
GPS / Conducteur          = Conducteur        (logique métier)
Contrat de conduite       = IVehicule         (interface / abstraction)
Voiture / Moto / Camion   = implémentations concrètes
```

On ne reconstruit pas le GPS à chaque nouveau véhicule. C'est exactement le bénéfice de DIP.

---

## 4. Mauvais exemple — Sans DIP

### Code complet

```python
# === BAS NIVEAU : implémentation concrète ===

class Voiture:
    def demarrer_moteur_essence(self) -> None:
        print("[VOITURE] Moteur essence démarré.")

    def appuyer_accelerateur(self, vitesse: int) -> None:
        print(f"[VOITURE] Accélération à {vitesse} km/h.")

    def appuyer_frein(self) -> None:
        print("[VOITURE] Freinage mécanique.")


# === HAUT NIVEAU : logique métier ===

class Conducteur:
    def __init__(self):
        # Dépendance directe sur une implémentation concrète
        self.vehicule = Voiture()

    def effectuer_trajet(self) -> None:
        self.vehicule.demarrer_moteur_essence()
        self.vehicule.appuyer_accelerateur(90)
        self.vehicule.appuyer_frein()


# === UTILISATION ===

conducteur = Conducteur()
conducteur.effectuer_trajet()
```

<p align="right"><em>— Page 3 / 9 —</em></p>
<div style="page-break-after: always;"></div>

### Problèmes identifiés

#### Couplage fort
`Conducteur` instancie lui-même `Voiture` avec `self.vehicule = Voiture()`. Les deux classes sont **liées à la compilation** : changer l'une implique de modifier l'autre.

Les noms de méthodes (`demarrer_moteur_essence`, `appuyer_accelerateur`) sont propres à la `Voiture`. Si demain on veut une `Moto`, il faut **réécrire** `Conducteur`.

#### Rigidité architecturale
Si le parc automobile évolue et qu'on doit gérer une `Moto`, un `CamionElectrique` ou une `Trottinette`, il faut :
1. Créer chaque nouvelle classe véhicule.
2. **Modifier** `Conducteur` pour choisir quel véhicule utiliser.
3. Répéter cette opération à chaque nouveau véhicule — violation directe du **Open/Closed Principle**.

#### Impact sur les tests unitaires
Il est **impossible** de tester `Conducteur` en isolation. Tout test instanciera réellement une `Voiture`. On ne peut pas substituer un faux objet (mock) sans réécrire la classe.

```python
# Impossible à mocker proprement :
def test_trajet():
    conducteur = Conducteur()
    # Voiture est instanciée à l'intérieur — on ne peut pas l'intercepter
    conducteur.effectuer_trajet()
    # Comment vérifier que le véhicule a bien démarré, sans dépendre de Voiture ?
```

#### Résumé des violations

| Problème | Description |
|---|---|
| Couplage fort | `Conducteur` connaît et instancie `Voiture` directement |
| Pas extensible | Ajouter un véhicule = modifier `Conducteur` |
| Non testable | Impossible d'injecter un mock sans refactoring |
| Fragile | Un changement dans `Voiture` (ex : renommer une méthode) casse `Conducteur` |

<p align="right"><em>— Page 4 / 9 —</em></p>
<div style="page-break-after: always;"></div>

## 5. Bon exemple — Avec DIP

### Principe appliqué

1. On définit une **interface** (`IVehicule`) qui représente le contrat de tout véhicule.
2. `Conducteur` dépend de `IVehicule`, jamais d'une classe concrète.
3. Les implémentations concrètes (`Voiture`, `Moto`, `CamionElectrique`) **respectent** ce contrat.
4. La dépendance concrète est **injectée de l'extérieur** (injection de dépendance).

---

### Code complet

```python
from abc import ABC, abstractmethod


# =============================================
# ABSTRACTION — Contrat que tout véhicule
# doit respecter (défini selon les besoins
# du module de HAUT niveau : le Conducteur)
# =============================================

class IVehicule(ABC):
    @abstractmethod
    def demarrer(self) -> None:
        """Démarre le véhicule, quel que soit son mode de propulsion."""
        ...

    @abstractmethod
    def accelerer(self, vitesse: int) -> None:
        """Accélère jusqu'à la vitesse cible en km/h."""
        ...

    @abstractmethod
    def freiner(self) -> None:
        """Ralentit et stoppe le véhicule."""
        ...


# =============================================
# BAS NIVEAU — Implémentations concrètes
# Elles dépendent de IVehicule, pas l'inverse
# =============================================

class Voiture(IVehicule):
    def demarrer(self) -> None:
        print("[VOITURE]  Moteur essence démarré.")

    def accelerer(self, vitesse: int) -> None:
        print(f"[VOITURE]  Accélération à {vitesse} km/h.")

    def freiner(self) -> None:
        print("[VOITURE]  Freinage mécanique appliqué.")


class Moto(IVehicule):
    def demarrer(self) -> None:
        print("[MOTO]     Démarrage du moteur 2 roues.")

    def accelerer(self, vitesse: int) -> None:
        print(f"[MOTO]     Accélération à {vitesse} km/h.")

    def freiner(self) -> None:
        print("[MOTO]     Frein avant et arrière activés.")


class CamionElectrique(IVehicule):
    def demarrer(self) -> None:
        print("[CAMION E] Moteur électrique silencieux activé.")

    def accelerer(self, vitesse: int) -> None:
        print(f"[CAMION E] Accélération progressive à {vitesse} km/h.")

    def freiner(self) -> None:
        print("[CAMION E] Freinage régénératif enclenché.")

<p align="right"><em>— Page 5 / 9 —</em></p>
<div style="page-break-after: always;"></div>

# =============================================
# HAUT NIVEAU — Logique métier
# Dépend uniquement de l'abstraction IVehicule
# =============================================

class Conducteur:
    def __init__(self, vehicule: IVehicule) -> None:
        # Injection de dépendance : le véhicule est fourni de l'extérieur
        self._vehicule = vehicule

    def effectuer_trajet(self) -> None:
        self._vehicule.demarrer()
        self._vehicule.accelerer(90)
        self._vehicule.freiner()


# =============================================
# UTILISATION — On choisit l'implémentation
# au moment de la composition (main, factory,
# DI container, fichier de config...)
# =============================================

if __name__ == "__main__":
    print("=== Trajet en voiture ===")
    Conducteur(vehicule=Voiture()).effectuer_trajet()

    print("\n=== Trajet en moto ===")
    Conducteur(vehicule=Moto()).effectuer_trajet()

    print("\n=== Trajet en camion électrique ===")
    Conducteur(vehicule=CamionElectrique()).effectuer_trajet()
```

**Sortie :**
```
=== Trajet en voiture ===
[VOITURE]  Moteur essence démarré.
[VOITURE]  Accélération à 90 km/h.
[VOITURE]  Freinage mécanique appliqué.

=== Trajet en moto ===
[MOTO]     Démarrage du moteur 2 roues.
[MOTO]     Accélération à 90 km/h.
[MOTO]     Frein avant et arrière activés.

=== Trajet en camion électrique ===
[CAMION E] Moteur électrique silencieux activé.
[CAMION E] Accélération progressive à 90 km/h.
[CAMION E] Freinage régénératif enclenché.
```

---

### Testabilité — Mock sans dépendance externe

```python
# test_conducteur.py
from unittest.mock import MagicMock
from vehicule import Conducteur, IVehicule


def test_effectuer_trajet_appelle_les_trois_methodes():
    # On crée un faux IVehicule — aucun véhicule réel instancié
    mock_vehicule = MagicMock(spec=IVehicule)

    conducteur = Conducteur(vehicule=mock_vehicule)
    conducteur.effectuer_trajet()

    # On vérifie que les trois étapes ont bien été appelées
    mock_vehicule.demarrer.assert_called_once()
    mock_vehicule.accelerer.assert_called_once_with(90)
    mock_vehicule.freiner.assert_called_once()
```

Le test est **pur, rapide, déterministe** — il ne démarre aucun moteur réel.

<p align="right"><em>— Page 6 / 9 —</em></p>
<div style="page-break-after: always;"></div>

### Explication détaillée

| Élément | Rôle | Dépend de |
|---|---|---|
| `IVehicule` | Contrat abstrait | Rien (c'est la base) |
| `Voiture` | Détail d'implémentation | `IVehicule` |
| `Moto` | Détail d'implémentation | `IVehicule` |
| `CamionElectrique` | Détail d'implémentation | `IVehicule` |
| `Conducteur` | Logique métier | `IVehicule` uniquement |

`Conducteur` **ne connaît pas** `Voiture`. Il sait uniquement qu'il peut appeler `demarrer()`, `accelerer()` et `freiner()`. Quel véhicule répond ? C'est décidé **en dehors** du conducteur, au moment de la composition de l'application.

---

## 6. Diagrammes UML textuels

### Architecture SANS DIP (couplage fort)

```
┌──────────────────────┐         ┌──────────────────────┐
│      Conducteur      │────────▶│       Voiture        │
│  (module haut niv.)  │ dépend  │  (module bas niveau) │
│                      │ direct  │                      │
└──────────────────────┘         └──────────────────────┘

Problème : flèche de dépendance entre deux modules concrets.
Conducteur ne peut piloter QUE Voiture.
```

---

### Architecture AVEC DIP (couplage faible via abstraction)

```
┌──────────────────────┐         ┌──────────────────────┐
│      Conducteur      │────────▶│    <<interface>>     │
│  (module haut niv.)  │ dépend  │      IVehicule       │
│                      │ de      │  + demarrer()        │
└──────────────────────┘         │  + accelerer()       │
                                 │  + freiner()         │
                                 └──────────┬───────────┘
                                            │ implémente
                             ┌──────────────┼──────────────┐
                             ▼              ▼              ▼
                     ┌────────────┐ ┌────────────┐ ┌──────────────────┐
                     │  Voiture   │ │    Moto    │ │ CamionElectrique │
                     └────────────┘ └────────────┘ └──────────────────┘

Les flèches vont VERS l'abstraction, jamais entre concrets.
Conducteur peut piloter n'importe quel véhicule conforme au contrat.
```

<p align="right"><em>— Page 7 / 9 —</em></p>
<div style="page-break-after: always;"></div>

## 7. Schéma logique des dépendances

```
MAUVAIS (couplage direct) :

  Conducteur  ──────────────────────▶  Voiture
   (haut)                               (bas)


BON (inversion via abstraction) :

  Conducteur  ──────▶  IVehicule  ◀──  Voiture
   (haut)              (abstrait)       (bas)
                            ▲
                            │
                           Moto
                            ▲
                            │
                    CamionElectrique

La dépendance est inversée :
  - Conducteur ne connaît plus Voiture
  - Voiture se soumet au contrat IVehicule
  - On peut ajouter n'importe quel véhicule sans toucher Conducteur
```

---

## 8. Comparaison avant / après DIP

| Critère | Sans DIP | Avec DIP |
|---|---|---|
| **Couplage** | Fort — `Conducteur` lié à `Voiture` | Faible — lié via `IVehicule` |
| **Extensibilité** | Ajouter un véhicule = modifier `Conducteur` | Ajouter un véhicule = créer une nouvelle classe |
| **Testabilité** | Impossible d'injecter un mock | Mock trivial via injection |
| **Respect OCP** | Violé — modification nécessaire | Respecté — extension sans modification |
| **Réutilisabilité** | `Conducteur` inutilisable sans `Voiture` | Réutilisable avec n'importe quel `IVehicule` |
| **Lisibilité** | Dépendances cachées dans le constructeur | Dépendances explicites et typées |
| **Déploiement** | Changement en cascade | Changements isolés par véhicule |

<p align="right"><em>— Page 8 / 9 —</em></p>
<div style="page-break-after: always;"></div>

## 9. À retenir — Version courte

> **DIP en 3 points :**
>
> 1. Les classes de haut niveau (`Conducteur`) dépendent d'**interfaces** (`IVehicule`), jamais de classes concrètes.
> 2. Les classes concrètes (`Voiture`, `Moto`, `CamionElectrique`) **implémentent** ces interfaces.
> 3. La dépendance concrète est **injectée de l'extérieur** (injection de dépendance).

> **Le mot-clé : Abstraction.**
> Programme toujours contre une interface, jamais contre une implémentation.

---

## 10. Conclusion

Le Dependency Inversion Principle est l'un des principes les plus structurants en architecture logicielle moderne. Son application systématique a des effets concrets et mesurables dans les projets professionnels :

**Maintenance facilitée** : modifier ou remplacer un type de véhicule (passer d'un moteur thermique à électrique, ajouter un véhicule autonome) ne nécessite d'écrire qu'une nouvelle implémentation de `IVehicule` — le code métier (`Conducteur`) reste intact.

**Tests unitaires fiables** : l'injection de dépendance permet de substituer n'importe quelle implémentation par un mock, rendant les tests rapides, déterministes et indépendants des dépendances réelles (matériel, réseau, base de données).

**Architecture évolutive** : DIP est le fondement de nombreux patterns avancés — *Strategy Pattern*, *Repository Pattern*, *Ports & Adapters (Hexagonal Architecture)* — qui reposent tous sur la séparation entre contrat abstrait et implémentation concrète.

**Réutilisabilité** : un module de haut niveau codé contre une interface peut être réutilisé dans un contexte totalement différent simplement en injectant une nouvelle implémentation conforme au contrat.

En résumé, DIP n'est pas un luxe théorique. C'est une discipline de conception qui détermine la capacité d'une base de code à évoluer sans se dégrader — ce qui est précisément la définition d'un logiciel bien conçu.

---

*Fiche rédigée au niveau Bac+4 Informatique — Principes SOLID, Architecture Logicielle*
*Référence : Robert C. Martin — "Agile Software Development: Principles, Patterns, and Practices" (2003)*

<p align="right"><em>— Page 9 / 9 —</em></p>
