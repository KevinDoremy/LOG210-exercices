
#CU01 - Noter une réservation
Le propriétaire d’un petit hôtel désire se doter d’une application pour gérer les réservations et les séjours de ses clients. Il ne veut pas que l’application s’occupe de la facturation. La figure 1 illustre les tâches accomplies par un commis à la réception. Vous devez analyser et concevoir une application permettant de noter les réservations, démarrer les séjours et transférer les séjours de chambre.
## Diagramme de cas d'utilisation

```plantuml
@startuml
left to right direction
package "Application de gestion hotelière" {
 usecase "CU01-Noter une reservartion d'une seule chambre" as N
 usecase "CU02-Noter une reservartion de plusieurs séjour" as S
 usecase "CU03-Transférer un séjour de chambre" as T
 
}
Commis --> N
Commis --> S
Commis --> T

@enduml
```

## CU01-Cas d'utilisation «Noter une réservation»
**Pré(s) condition(s) :**
•	Le commis est authentifié.
**Post condition(s) :**
•	Une réservation est inscrite dans le système.
**Acteur principal:**
•	Le commis à la réception (commis).

**Scénario principal**
1.	Un client appelle à l'hôtel pour placer une réservation.
2.	Le commis démarre une nouvelle réservation.
3.	Le commis entre:
i.	La date d'arrivée;
ii.	La date de départ;
iii.	Le nom de la catégorie de chambre;
iv.	La quantité de chambres.
6.	Le système affiche toutes les informations entrées.
7.	Le commis valide les informations auprès du client et confirme la réservation à l'aide du nom et du numéro de téléphone du client.
8.	Le système enregistre la réservation et affiche le numéro de confirmation.
9.	Le commis communique le numéro de confirmation au client.

**Scénarios alternatifs**
*. En tout temps, le commis annule la réservation.
1.	Le système supprime la réservation.
2.	Fin du cas d'utilisation.

3a. Il n'y a pas assez de chambres de la catégorie choisie pour la période entrée.
1.	Le système affiche un message d'avertissement.
2.	Le système affiche les disponibilités de toutes les catégories pour la période entrée.
3.	Le commis communique les disponibilités au client.
4.	Retour à 3.

#### CU01-Réservation simple
Jean Hernandez appelle à l'hôtel pour réserver une chambre.
Le commis entrera:
Le 2011-06-01 comme date d'arrivée
Le 2011-06-02 comme date de départ
"Luxe" comme catégorie de chambre
1 pour la quantité
Le commis terminera ensuite la réservation en entrant le nom du client et communiquera le numéro de confirmation à monsieur Hernandez.

## Interfaces usagé
```plantuml
@startuml
(*) --> "
{{
salt
{
Menu Principal
~~
[Noter une réservation]
}
}}
" as main

main -right-> "
{{
salt
{+
  Noter une réservation
  --
  Date d'arrivé | "2020-03-03"
  Date de départ | "2020-03-06"
  Catégorie | ^Categorie^ 
   [Cancel] | [Réserver]
}
}}
" as reserver

reserver -right-> "
{{
salt
{+
Menu Principal
~~
Confirmation de votre ?
--
  No confirmatino: | ?
  Date d'arrivé : |  2020-03-03
  Date de départ : | 2020-03-06
  Catégorie: | categorie1
==
[Noter une réservation]
  --}
}}
" as mainReservationConfirme

@enduml
```

## CU01-Modèle du domaine

```plantuml
@startuml
class Hotel <<Conteneur, Object physique>>
class Categorie <<Description d'entité>> {
  nom: String
}
class Reservation <<Transaction>>{
  dateArrive: date
  dateDepart: date
}
class Commis <<Role>>
class Chambre <<Objet physique, Contenu dans l'hotel, Produit d'une transaction>>
class Client <<Role>>{
  string nom
}

Hotel "1" -- "*" Chambre: possède
Chambre "*" -- "1" Categorie: appartient
Client "1" -- "1" Reservation: demande
Commis "1" -- "*" Reservation: cree
Hotel "1" -- "*" Commis : Emploie
Reservation "*" -- "1" Chambre : est reservé par <
@enduml
```

## Diagramme de séquence système

```plantuml
@startuml
title: Noter une réservation
skinparam style strictuml
Actor ":Commis" as Commis
Participant ":Systeme" as Systeme
Commis -> Systeme: demarrerReservation()
Commis <<-- Systeme: demander dateArrive,dateDepart, categorie, [nomCategorie] 
Commis -> Systeme: reserverChambre(dateArrive:string, dateDepart:string, nomCategorie: String)
Commis <<-- Systeme: confirmation?, menu principale
@enduml
```

## Contrat
**demarrerReservation()**
  - aucune

  **reserverChambre(dateArrive:string, dateDepart:string, nomCategorie: String)**
  - Précondition:
    - c:Commis est authentifier

  - Postcondition
    - Une instance r:Reservation a été créé
    - Une association a été créée entre r:Reservation et Chambre sur la base de correspondance avec categorie.nom == nomCategorie
    - Une association a été cree entre c:Commis et r:Reservation
    - Une instance cl:Client a été créée (? Nom, Prénom, Courriel)
    - Une association entre r:Reservation et cl:Client a été créée
    - r.dateArrivé est devenu dateArrivé
    - r.dateDepart est devenu dateDepart

## RDCU

### demarrerReservation()
```plantuml
@startuml
skinparam style strictuml
Participant ":ControleurReservation" as S
Participant "mc:Map<nom:String,categorie:Categorie>" as MC

note right  of MC: Visible par le \ncontroleur puisque \ntout les hotels utilisent \nles mêmes catégories
note left of S:Controleur de session
note right of S: use mc.keys() to get array of category
 -> S: [nomCategorie] = demarrerReservation()

@enduml
```

### reserverChambre
```plantuml
@startuml
skinparam style strictuml
Participant ":ControleurReservation" as S
Participant "c:Commis" as C
Participant "cl:Client" as CL
Participant "mc:Map<nom:String,categorie:Categorie>" as MC
Participant "cat:Categorie" as CAT
Participant "ch:Chambre" as CH
Participant "r:Reservation" as R

note right of C: Visible par le \ncontroleur\npuisque c'est une \nprécondition
note right  of MC: Visible par le \ncontroleur puisque \ntout les hotels utilisent \nles mêmes catégories
note left of S:Controleur de session
 -> S: reserverChambre(\ndateArrive:String, \ndateDepart:String, \ncategorie: String)
 note right of S: Createur
 S -> CL**: cl = Create(?)
 note right of S: Expert en information
S -> MC: cat = get(nom: String)
 note right of S: Expert en information
S -> CAT: ch = getChambreLibre(\ndateArrivé: String, \ndateDepart:String)
note right of S: Createur, cohésion et couplage
S -> R**: r=Create(\nch:Chambre, \nc:Commis, \ncl:Client, \ndateArrive:String, dateDepart: String)

@enduml
```



## Diagramme de classe

```plantuml
@startuml
class ControleurReservation{
  demarrerReservation()
  reserverChambre(dateArrive:string, dateDepart:string, categorie: String)
}
class "Map<nom:String,categorie:Categorie>" as MC {
  get(nom:String): Categorie
}

class Hotel <<Conteneur, Object physique>>
class Categorie <<Description d'entité>> {
  nom: String
  getChambreLibre(dateArrivé: String,dateDepart:String): Chambre
}
class Reservation <<Transaction>>{
  dateArrive: date
  dateDepart: date
  Reservation(ch:Chambre,c:Commis,cl:Client,dateArrive:String, dateDepart: String)
}
class Commis <<Role>>
class Chambre <<Objet physique, Contenu dans l'hotel, Produit d'une transaction>>
class Client <<Role>> {
  Client(?)
}

Hotel  *--> "*" Chambre: possède
Chambre "*" --o Categorie: appartient
Client  <--  Reservation: demande
Commis  <--  Reservation: cree
Hotel  *-- "*" Commis : Emploie
Reservation "*" --> "1" Chambre : est reservé par <
ControleurReservation --> MC
ControleurReservation --> Commis
MC --> "*" Categorie
@enduml
```


### CU01-Exercices-Notez une réservation
1. Représentez le fait qu’un Client est responsable d’une Réservation.
1. Représentez le fait qu’une Réservation peut avoir plusieurs Ligne de Réservation.
1. Représentez le fait qu’une Catégorie de chambre regroupe plusieurs Chambres.
1. Est-il nécessaire d’associer un Client à une Chambre? Justifiez la réponse.
1. Représentez le fait qu’un Client désire occuper une Chambre d’une catégorie précise 1. durant la première fin de semaine de mois de juillet. Justifiez les associations à 1. l’aide d’un verbe exprimant la raison d’être de l’association. Indiquez les 1. attributs des classes conceptuelles.
2. Représentez le fait qu’un Commis a confirmé la Réservation à l’aide des 1. informations personnelles (nom et numéro de téléphone) du Client. Justifiez les 1. associations à l’aide d’un verbe d’action. Indiquez les attributs nécessaires.
3. Bâtissez le modèle du domaine partiel du système de l’hôtel. Justifiez les 1. associations à l’aide d’un verbe. Indiquez tous les attributs pertinents


### CU01-CU02-Exercices-RDCU

1. 	Proposez une solution logicielle, sous forme de diagramme dynamique, permettant d’instancier une réservation. Annotez votre solution des principes GRASP.
 
2. 	Proposez une solution logicielle, sous forme de diagramme dynamique, permettant d’instancier une ligne de réservation. Annotez votre solution des principes GRASP.
 
3. 	Proposez une solution logicielle, sous forme de diagramme dynamique, permettant d’associer une ligne de réservation à une catégorie de chambre. Annotez votre solution des principes GRASP.
 
4. 	Proposez une solution logicielle, sous forme de diagramme dynamique, permettant de stocker une réservation. Annotez votre solution des principes GRASP.
 
5. 	Proposez une solution logicielle, sous forme de diagramme dynamique, permettant de repérer une réservation à partir de son numéro de confirmation. Annotez votre solution des principes GRASP.
 
 1. 	Proposez une solution logicielle, sous forme de diagramme dynamique, permettant de créer un client tout en l’associant à une nouvelle réservation. Annotez votre solution des principes GRASP.
 
6. 	Proposez une solution logicielle, sous forme de diagramme dynamique, permettant de détruire une réservation appartenant à un client. Annotez votre solution des principes GRASP.
 
7. 	Proposez une solution logicielle, sous forme de diagramme dynamique, permettant d’imprimer une facture incluant l’information sur une réservation, ses lignes de réservation et les catégories de chambres associées.  Annotez votre solution des principes GRASP.



# CU02 - Noter une réservation avec plusieurs chambres
**Pré(s) condition(s) :**
- Le commis est authentifié.
  
**Post condition(s) :**
- Une réservation est inscrite

**Acteur principal:**
- Le commis à la réception (commis).

**Scénario principal**
1. Un client appelle à l'hôtel pour placer une réservation.
1. Le commis démarre une nouvelle réservation.
1. Le commis entre:
   1. La date d'arrivée;
   2. La date de départ;
   3. Le nom de la catégorie de chambre;
   4. La quantité de chambres.
1. Le système inscrit les informations à la réservation.

Les étapes 3 et 4 sont répétées tant que le client n'indique pas qu'il a terminé.

5. Le commis termine la réservation.
1. Le système affiche toutes les informations entrées.
1. Le commis valide les informations auprès du client et confirme la réservation à l'aide du nom et du numéro de téléphone du client.
1. Le système enregistre la réservation et affiche le numéro de confirmation.
1. Le commis communique le numéro de confirmation au client.

**Scénarios alternatifs**
*. En tout temps, le commis annule la réservation.
  1. Le système supprime la réservation.
  2. Fin du cas d'utilisation.

3a. Il n'y a pas assez de chambres de la catégorie choisie pour la période entrée.
  1. Le système affiche un message d'avertissement.
  1. Le système affiche les disponibilités de toutes les catégories pour la période entrée.
  1. Le commis communique les disponibilités au client.
  1. Retour à 3.




#### CU02-Réservation d'affaires
Jean Hernandez appelle à l'hôtel pour réserver deux chambres identiques, mais pour des périodes légèrement différentes. Son collègue, Patrice Retardataire, arrivera une journée après Jean.
Le commis entrera d'abord:
2011-08-02 comme date d'arrivée
2011-08-07 comme date de départ
"Standard" comme catégorie de chambre
1 pour la quantité
Puis, il entrera:
2011-08-03 comme date d'arrivée
2011-08-07 comme date de départ
"Standard" comme catégorie de chambre
1 pour la quantité
Le commis terminera ensuite la réservation et communiquera le numéro de confirmation à monsieur Hernandez.

#### CU03-Réservation budgétaire
Jean Hernandez planifie des vacances avec son fils. Il appelle à l'hôtel pour réserver deux chambres de catégorie différente pour la même période.
Le commis entrera d'abord:
2011-07-10 comme date d'arrivée
2011-07-18 comme date de départ
"Luxe" comme catégorie de chambre
1 pour la quantité
Puis, il entrera:
2011-07-10 comme date d'arrivée
2011-07-18 comme date de départ
"Standard" comme catégorie de cham¬
Le commis terminera ensuite la réservation et communiquera le numéro de confirmation à monsieur Hernandez.


## Interfaces usagé
```plantuml
@startuml
(*) --> "
{{
salt
{
**Menu Principal**
~~
[Noter une réservation]
}
}}
" as main

main -right-> "
{{
salt
{+
  **Noter une réservation**
  --
  Quantité de chambre | "99"
  Date d'arrivé | "2020-03-03"
  Date de départ | "2020-03-06"
  Catégorie | ^Categorie^ 
[Réserver] |
[Terminer la réservation]
== | ==
**Réservations**
~~ | ~~
  Quantité chambres: | 2
  Date d'arrivé : |  2020-03-03
  Date de départ : | 2020-03-06
  Catégorie: | Luxe
~~ | ~~
  Quantité chambres: | 2
  Date d'arrivé : |  2020-03-03
  Date de départ : | 2020-03-06
  Catégorie: | Luxe
  }
}}
" as reservations

reservations -right-> "
{{
salt
{+
**Menu Principal**
~~ | ~~
  No confirmation: | A1234
== | ==
[Noter une réservation]
  }
}}
" as mainReservationConfirme

@enduml
```

## Modèle du domaine

```plantuml
@startuml
class Hotel <<Conteneur, Object physique>>
class Categorie <<Description d'entité, Catalogue>> {
  nom: String
}
class Reservation <<Transaction>>{
noConfirmation: String
}
class LigneReservation <<Ligne de transaction>> {
  <s>quantity: String</s>
  dateArrive: Date                                                                                                
  dateDepart: Date
}
class Commis <<Role>>
class Chambre <<Objet physique, Contenu dans l'hotel, Produit d'une transaction>>
class Client <<Role>>

Hotel "1" -- "*" Chambre: possède
Chambre "*" -- "1" Categorie: appartient
Client "1" -- "1" Reservation: demande
Commis "1" -- "*" Reservation: cree
Reservation "1" -- "*" LigneReservation: contient
Hotel "1" -- "*" Commis : Emploie
LigneReservation "1" -- "*" Chambre : est reservé par <
@enduml
```

Cette version sous entends que les Catégories sont les même pour tous les Hotels.  Si ce   n’étais pas le cas, on associe Hotel à Catégorie et on enlève l’association entre Hotel et Chambre.

Catégorie peut être traité comme une classe de description ou un catalogue selon le sens par lequel nous utilisons les classes.  
Plusieurs chambres sont décrite par une catégorie (Classe de description)
Une Catégorie (Catalogue) contient plusieurs Chambre.




## Diagramme de séquence système

```plantuml
@startuml
title: Noter plusieurs réservations
skinparam style strictuml
Actor ":Commis" as Commis
Participant ":Systeme" as Systeme
Commis -> Systeme: demarrerReservation()
note right of Commis: Formulaire réservation\ndemander: {quantity,dateArrive,dateDepart, categorie}, retourne: [nomCategorie]] 
Commis <<-- Systeme: FormulaireReservation, historique réservations = []
loop [client n'a pas terminé]
  Commis -> Systeme: reserverChambres(quantity: integer, dateArrive:string, dateDepart:string, nomCategorie: String)
    Commis <<-- Systeme: formulaire réservation, historique réservations

end
 Commis -> Systeme: terminerReservation()
 Commis <<-- Systeme: Menu principale, noConfirmation
@enduml
```

## Contrat
**demarrerReservation()**
 - Précondition:
    - c:Commis est authentifier
  - 
  - Postcondition
    - Une instance r:Reservation a été créée
    - Une association a été cree entre c:Commis et r:Reservation
    - Une instance cl:Client a été créée (? Nom, Prénom, Courriel)
    - Une association entre r:Reservation et cl:Client a été créée

  **reserverChambres(quantity: integer, dateArrive:string, dateDepart:string, nomCategorie: String)**
  - Précondition:
    - c:Commis est authentifier
    - r:Reservation existe

  - Postcondition
    - Une instance lr:LigneReservation a été créé
    - Une association a été créée entre r:Reservation et lr:LigneReservation
    - <u>quantity</u> associations ont été créées entre lr:LigneReservation et Chambre sur la base de correspondance avec categorie.nom == <u>nomCategorie</u>
    - lr.dateArrivé est devenu <u>dateArrivé</u>
    - lr.dateDepart est devenu <u>dateDepart</u>

**terminerReservation()**
  - Précondition:
    - r:Reservation existe

  - Postcondition
    - r.noConfirmation est devenu un numéro unique

## RDCU

### demarrerReservation()
```plantuml
@startuml
skinparam style strictuml
Participant ":ControleurReservation" as S
Participant "r:Reservation" as R
Participant "c:Commis" as C
Participant "cl:Client" as CL
Participant "mc:Map<nom:String,categorie:Categorie>" as MC
Participant "llr:List<:LigneReservation>" as LLR

note right  of MC: Visible par le \ncontroleur puisque \ntout les hotels utilisent \nles mêmes catégories
note left of S:Controleur de session
note right of S: use mc.keys() to get array of category
 -> S: [nomCategorie] = demarrerReservation()
note right of S: Createur
 S -> CL**: cl = Create(?)
 note right of S: Créateur, cohésion et couplage
S -> R**: r=Create( \nc:Commis, \ncl:Client)
note right of R: Createur, r:Reservation possède les lr:LigneReservation
R --> LLR**: llr=Create()
@enduml
```

### reserverChambres
```plantuml
@startuml
skinparam style strictuml
Participant ":ControleurReservation" as S
Participant "mc:Map<nom:String,categorie:Categorie>" as MC
Participant "cat:Categorie" as CAT
Participant "r:Reservation" as R
Participant "lr:LigneReservation" as LR
Participant "llr:List<:LigneReservation>" as LLR

note right  of MC: Visible par le \ncontroleur puisque \ntout les hotels utilisent \nles mêmes catégories
note left of S:Controleur de session
 -> S: reserverChambres(\nquantity:integer\ndateArrive:String, \ndateDepart:String, \ncategorie: String)
 note right of S: Expert en information
S -> MC: cat = get(categorie: String)
 note right of S: Expert en information
S -> CAT: [ch] = getChambresLibre(\nquantity:integer\ndateArrivé: String, \ndateDepart:String)
note right of S: expert en information
S -> R: r=ajouterChambres(\n[ch]:Chambre, \ndateArrive:String, dateDepart: String)
note left of LR: Createur, forte cohesion, faible couplage
R -> LR**: r=create(\n[ch]:Chambre, \ndateArrive:String, dateDepart: String)
note left of LLR: expert en information\nr a une visibilité sur llr\nllr est une liste de ligne de réservation
R -> LLR: ajouterLigneReservarion(llr)

@enduml
```

### terminerReservation
```plantuml
@startuml
skinparam style strictuml
Participant ":ControleurReservation" as S
Participant "r:Reservation" as R

note left of S:Controleur de session
 -> S: noConfirmation = terminerReservation()
 note right of S: Expert en information\nMutateur d'attribut
S -> R: noConfirmation = terminerReservation()

@enduml
```


## Diagramme de classe

```plantuml
@startuml
class ControleurReservation{
  demarrerReservation()
  reserverChambres(quantity:Integer,dateArrive:String, dateDepart:String, categorie: String)
  terminerReservation()
}
class "Map<nom:String,categorie:Categorie>" as MC {
  get(nom:String): Categorie
}

class Hotel <<Conteneur, Object physique>>
class Categorie <<Description d'entité,**Catalogue**>> {
  nom: String
  getChambresLibre(quantity:Integer, dateArrivé: String,dateDepart:String): [Chambre]
}
class Reservation <<Transaction>>{
  noConfirmation: String
  Reservation(commis:Commis, client: Client)
  ajouterChambres([ch]:Chambre,dateArrive:String, dateDepart: String): String (json)
  terminerReservation(): String
}

class LigneReservation <<Ligne de transaction>>{
   dateArrive: Date
  dateDepart: Date
  create([ch]:Chambre,dateArrive:String, dateDepart: String)
}

class Commis <<Role>>
class Chambre <<Objet physique, Contenu dans l'hotel, Produit d'une transaction>>
class Client <<Role>> {
  Client(?)
}

Hotel  *--> "*" Chambre: possède
Chambre "*" --o Categorie: appartient
Client  <--  Reservation: demande
Commis  <--  Reservation: cree
Hotel  *-- "*" Commis : Emploie
Reservation "1" --> "*" LigneReservation: contient
LigneReservation "1" --> "*" Chambre : est reservé par <
ControleurReservation --> MC
ControleurReservation --> Commis
MC --> "*" Categorie
@enduml
```

# CU03-Transférer un séjour de chambre
**Précondition(s) :**
- Le commis est authentifié.

**Postcondition(s) :**
- Le séjour se poursuit dans une autre chambre.

**Acteur principal :** Le commis à la réception

**Scénario principal**
1. Un client désire occuper une chambre différente pour le reste de son séjour.
1. Le commis entre le numéro de chambre dans laquelle le client séjourne.
1. Le système affiche le séjour ainsi que toutes les chambres disponibles de la même catégorie.
1. Le commis informe le client des chambres disponibles puis sélectionne celle que le client préfère.
1. Le système modifie la ligne courante du séjour et inscrit une nouvelle ligne au séjour pour la nouvelle chambre.
1. Le commis remet la clé de la chambre au client et lui indique l'emplacement de la chambre dans l'hôtel.

**Scénarios alternatifs**
3a. Il ne reste plus de chambres disponibles dans la catégorie réservée.
Le système affiche les chambres disponibles pour toutes les catégories.
Retour à 4


## MDD CU01 + CU02 + CU03
LigneRéservation correspond à Sejour. Il faut aussi modifier les multiplicité entre Séjour et Chambre pour 1 – 1. 


```plantuml
@startuml
skinparam style strictuml
hide methods
class Commis 
class Client {
  nom
telephone
}
class Hotel{
telephone
}
class Reservation {
confirmation:Strgin
}
class Sejour {
dateArrive: DateHeure
dateDepart: DateHeure
}
class Chambre {
no
}
class Clé
class Categorie {
nom: String
}
Commis "1" -- "*" Reservation : Effectue
Client "1" -- "*" Reservation : Demande
Reservation "1" -- "*" Sejour : Contient
Sejour "1" -- "*" Chambre : Reserve
Chambre "*" -- "1" Categorie : sont-décrites-par
Hotel "1" -- "*" Commis : Emploie
Hotel "1" -- "*" Chambre : Contient
Chambre "1" -- "*" Clé : est-ouverte*par
Chambre "*" -- "1" Emplacement : sont-situé-à
@enduml
```

## DSS CU03-Transférer un séjour de chambre
```plantuml
@startuml
skinparam style strictuml
Actor ":Commis" as C
participant ":System" as S

C->S: demarrerTransferChanbre(string noChambre)
C<<--S: information du séjour, chambres disponibles

C->S: transfererChambre(string noChambreActuel, string noNouvelleChambre)
C<<--S: confirmation

@enduml
```


### Contrat CU03-demarrerTransfert
**Opération:** demarrerTransfer(numeroChambre)
**Présconditions:**
**Postconditions:**
- Aucune


### Contrat CU03-transférerChambre
**Opération:** transférerChambre(noChambreActuel, noNouvelleChambre)
**Présconditions:**
**Postconditions:**
- Une instance sn:Sejour a été créée
- sn.dateArrive est devenu maintenant
- sn.dateDepart est devenu Chambre.sejour.dateDepart sur la base de correspondance avec noChambreActuel
- Une association a été créée entre sn et Chambre.sejour.réservation sur la base de correspondance avec noChambreActuel
- Une association a été créée entre sn et Chambre sur la base de correspondance avec noNouvelleChambre
- Chambre.Sejour.dateDepart et devenu maintenant sur la base de correspondance avec noChambreActuel


### RDCU CU03-demarrerTransfert

```plantuml
@startuml
skinparam style strictuml
title DémarrerRéservation

participant ":SystemeReservation" as sr
participant ":Reservation" as r

note left of sr : Controleur de facade\n (système) 
->sr : demarrerReservation()
activate sr

note right of sr : Par créateur
create r
sr-->r : create
activate r

participant "llr:List<LigneReservation>" as llr

note right of r: Par créateur
create llr
r --> llr: create

participant "lr:List<Reservation>" as lr

note right of sr : Expert en\n information
sr --> lr: add(r:reservation)
deactivate sr
deactivate r
@enduml
```

### RDCU CU03-transférerChambre

```plantuml
@startuml
skinparam style strictuml
title - réserverCatégorie(arrivée : date, départ : date, nom : string, quantité:int)

participant ":SystemeReservation" as sr
participant ":Reservation" as r
participant "lr:LigneReservation" as lr
participant ":CatalogueCategorie" as cc
participant "lc:List<Categorie>" as lc
participant "llr:List<ligneResevation>" as llr

note left of sr : Controleur de facade\n (système) 
->sr : réserverCatégorie(arrivée : date, départ : date, nom : string, quantité:int)

activate sr
note right of sr: expert en information
sr -> cc : c = getCategorie(nom:String)
note right of cc : expert en information\nCohésion
cc -> lc : c=get(nom:String)

note right of sr : Par expert en information
sr -> r : lr = addReservation(arrivée : date,\n départ : date,\n quantité:int,c:Categorie)

create lr
note right of r: par createur
r--> lr : lr = create(arrivée : date,\n départ : date,\n quantité:int,c:Categorie)

note right of r : Par expert en information
r->llr : add(lr:LigneReservation)
deactivate sr
@enduml
```