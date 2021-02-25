# Notes sur Identity Hub - DIF


## C'est quoi?
--- 

Identity Hubs are decentralized, off-chain, personal datastores that put control over personal data in the hands of users. They allow users to store their sensitive data — identity information, official documents, app data, etc. — in a way that prevents anyone from using their data without their explicit permission. 

From the app developers perspective, ID Hubs are also a great tool to reduce the complexity of data management and compliance by storing all sensitive data in the users’ Hub. This implementation reduces the developers’ risk of privacy violations and data breaches because the data would no longer be stored in the app.

Les utilisateurs peuvent utiliser leurs hubs d'identification pour partager en toute sécurité leurs données avec d'autres personnes, applications et entreprises, tout en conservant le contrôle et la propriété. Les utilisateurs peuvent fournir un accès limité à la quantité minimale de données nécessaires dans un scénario donné.

Du point de vue des développeurs d'applications, ID Hubs est également un excellent outil pour réduire la complexité de la gestion et de la conformité des données en stockant toutes les données sensibles dans le Hub des utilisateurs. Cette mise en œuvre réduit le risque de violation de la confidentialité et de violation de données par les développeurs, car les données ne seraient plus stockées dans l'application.

Une propriété importante de l'ID Hub est sa capacité à partager entre les appareils, ce qui signifie que les utilisateurs peuvent utiliser leurs ID Hub avec n'importe quel fournisseur. 

La spécification de l'ID Hub est en cours d'élaboration par les membres de la Decentralized Identity Foundation (DIF). Les membres de la fondation partagent l'engagement de rechercher et d'élaborer les futures normes qui s'appliqueront à la protection de notre identité.


## Installation

Installez la librairie du sdk dans votre projet avec le npm suivant: 

    npm install @decentralized-identity/hub-sdk-js

## Adressabilité 

Hubs let you securely store and share data. A Hub is a datastore containing semantic data objects at well-known locations. Each object in a Hub is signed by an identity and accessible via a globally recognized API format that explicitly maps to semantic data objects. Hubs are addressable via unique identifiers maintained in a global namespace.

Une seule entité (personne, entreprise, agent, etc) peut avoir plusieurs instances d'un hub, adressés par un URI routing lié à identificateur de l'entité. 

Il y a deux types de descripteurs de endpoint: user, qui est associé au DID de l'entité, et host, qui redirectionne les demandes au site ou l'infrastructure est presente. 

User endpoint descriptor example: 

    "service": [{
    "type": "IdentityHub",
    "publicKey": "did:foo:123#key-1",
    "serviceEndpoint": {
        "@context": "schema.identity.foundation/hub",
        "@type": "UserServiceEndpoint",
        "instances": ["did:bar:456", "did:zaz:789"]
    }
    }]

Host endpoint descriptor example: 

    "service": [{
    "type": "IdentityHub",
    "publicKey": "did:bar:456#key-1",
    "serviceEndpoint": {
        "@context": "schema.identity.foundation/hub",
        "@type": "HostServiceEndpoint",
        "locations": ["https://hub1.bar.com/.identity", "https://hub2.bar.com/blah/.identity"]
    }
    }]


## Synchronization de données: 


Hub instances must sync data without requiring master-slave relationships or forcing a single implementation for storage or application logic. This requires a shared replication protocol for broadcasting and resolving changes. The protocol for reproducing Hub state across multiple instances is in the formative phases of definition/selection, but should be relatively straightforward to integrate on top of any NoSQL datastore.

## Serialization et exportation de données 


Le hub doit être capable d'exporter ses données serialisées. 

Compatibilité, portabilité et contrôle de l'usager sur ses données. 

Travail en évolution, encore à finir par le DIF.


## Hub protocol URI Scheme 


Pour ne pas avoir une dépendence entre les données d'identité du proprietaire et des instances de hubs individuels-spécifiques, le URI scheme proposé est le suivant 

hub://did:foo:123abc

Les user-agents qui connaissent ce schema font la résolution, via le Universal Resolver pour trouver les instances des hubs endpoints via les services endpoint qui'il trouve dans le UR. 

## Authentication


Elle est faite selon les schemas DIF/W3C DID Auth. Voir le lien: https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-spring2018/blob/master/draft-documents/did_auth_draft.md

Authorization

Elle est faite via une couche de permissions implementée bare bones. Le proprietaire du hub peut donner l'accès, et peut le contraindre à certains types de données. Il aura plus d'implementations disponibles au futur, qui sont des work in progress chez DIF. 

Voir: https://github.com/decentralized-identity/identity-hub/blob/master/docs/permissions.md

## API

Because of the sensitive nature of the data being transmitted to Identity Hubs, the Identity Hub request and response API may look a bit different to developers who are used to a traditional REST service API. Most of the differences are based on the high level of security and privacy decentralized identity inherently demands.

## Commits

Toute donnée presente dans un hub est representée par une série de commits, tout comme on fait chez github. 

Pour écrire une donnée dans une hub, on fait un commit et l'envoie au hub. 

Pour lire une donnée, on prend tous les commits puis on réconstruit l'état actuel de l'objet par l'application séquentielle des commits, dans l'ordre chronologique. 

Avantages de la stratégie des commits: 

- it facilitates the hub's replication protocol, enabling multiple hub instances to sync data.
- it creates an auditable history of changes to an object, especially when each commit is signed by a DID.
- it eases implementation for use cases that need offline data modification and require conflict resolution.

Example de commit: 

    // JWT headers
    {
    "alg": "RS256",
    "kid": "did:foo:123abc#key-abc",
    "interface": "Collections",
    "context": "https://schema.org",
    "type": "MusicPlaylist",
    "operation": "create",
    "committed_at": "2018-10-24T18:39:10.10:00Z",
    "commit_strategy": "basic",
    "sub": "did:bar:456def",

    // Example metadata about the object that is intended to be "public"
    "meta": {
        "tags": ["classic rock", "rock", "rock n roll"],
        "cache-intent": "full"
    }
    }

    // JWT body
    {
    "@context": "http://schema.org/",
    "@type": "MusicPlaylist",
    "description": "The best rock of the 60s, 70s, and 80s",
    "tracks": ["..."],
    }

    // JWT signature
    uQRqsaky-SeP3m9QPZmTGtRtMoKzyg6wwWF...

## Requete d'écriture et format de réponse 

Sont de messages auto-contenus, qui encapsulent tout ce qu'elles en ont besoin pour être protegées durant le transport. 

Toute requête est un objet JSON qui est signé et crypté, comme detaillé dans la doc de authentification. 

L'envelope est signé avec la clé du "iss" DID (usager), et crypté avec la clé DID du hub. 

Example:  

    {
    iss: 'did:foo:123abc',
    sub: 'did:bar:456def',
    aud: 'did:baz:789ghi',
    "@context": "https://schema.identity.foundation/0.1",
    '@type': 'WriteRequest',

    
    // The commit in JSON Serialization Format
    // See: https://tools.ietf.org/html/rfc7515#section-3.1
    commit: {
        protected: "ewogICJpbnRlcmZhY2...",
        
        // Optional metadata information not protected by the JWT signature
        header: {
        "iss": "did:foo:123abc"
        },

        payload: "ewogICJAY29udGV4dCI6...",
        signature: "b7V2UpDPytr-kMnM_YjiQ3E0J2..."
    }
    }

Réponse: 

    {
    "@context": "https://schema.identity.foundation/0.1",
    "@type": "WriteResponse",
    "developer_message": "completely optional message from the hub",
    "revisions": ["aHashOfTheCommitSubmitted"]
    }

## Requête de lecture et format de réponse 

Les objets sont formés par la cummulation séquencielle d'une série de commits, qui doit être reconstruite dans la bonne séquence. 

Example: 

    {
    "@context": "https://schema.identity.foundation/0.1",
    "@type": "ObjectQueryRequest",
    "iss": "did:foo:123abc",
    "sub": "did:bar:456def",
    "aud": "did:baz:789ghi",
    "query": {
        "interface": "Collections",
        "context": "http://schema.org",
        "type": "MusicPlaylist",
        
        // Optional object_id filters
        "object_id": ["3a9de008f526d239..", "a8f3e7..."]
    }
    }

Réponse 

    {
    "@context": "https://schema.identity.foundation/0.1",
    "@type": "ObjectQueryResponse",
    "developer_message": "completely optional",
    "objects": [
        {
        // object metadata
        "interface": "Collections",
        "context": "http://schema.org",
        "type": "MusicPlaylist",
        "id": "3a9de008f526d239...",
        "created_by": "did:foo:123abc",
        "created_at": "2018-10-24T18:39:10.10:00Z",
        "sub": "did:foo:123abc",
        "commit_strategy": "basic",
        "meta": {
            "tags": ["classic rock", "rock", "rock n roll"],
            "cache-intent": "full"
        }
        },
        // ...more objects
    ]

    // potential pagination token
    "skip_token": "ajfl43241nnn1p;u9390",
    }


## Requete de Commit et format de réponse 

    {
    "@context": "https://schema.identity.foundation/0.1",
    "@type": "CommitQueryRequest",
    "iss": "did:foo:123abc",
    "sub": "did:bar:456def",
    "aud": "did:baz:789ghi",
    "query": {
        "object_id": ["3a9de008f526d239..."],
        "revision": ["abc", "def", ...]
    },
    }

Réponse  

    {
    "@context": "https://schema.identity.foundation/0.1",
    "@type": "CommitQueryResponse",
    "developer_message": "completely optional",
    "commits": [
        {
        protected: "ewogICJpbnRlcmZhY2UiO...",
        header: {
            "iss": "did:foo:123abc",
            // Hubs may add additional information to the unprotected headers for convenience
            "rev": "aHashOfTheCommit",
        },
        payload: "ewogICJAY29udGV4dCI6ICdo...",
        signature: "b7V2UpDPytr-kMnM_YjiQ3E0J2..."
        },
        // ...
    ],

    // potential pagination token
    "skip_token": "ajfl43241nnn1p;u9390",
    }

## Interfaces 
--- 

Pour faciliter les intéractions et l'enregistrement de données, le hub offre un ensemble d'interfaces standardisées que peuvent être apellées: 

- Profile :  Le'objet descripteur princial de l'entité proprietaire  (schema agnostique)
- Permissions : Le document de controle d'acess JSON 
- Actions : Un endpoint connu pour le relais des actions vers le propriétaire de l'identité
- Stores : Espace de stockage 1: 1 de portée qui est directement affecté à un autre DID externe
- Collections : Les collections d'identité de l'entité propriétaire (accès limité)
- Services : toute fonctionnalité personnalisée basée sur les services que l'identité expose


### Profile 
--- 

Chaque Hub a un objet de profil qui décrit l'entité propriétaire. L'objet de profil doit utiliser le schéma et l'objet qui représentent le mieux l'entité. Pour définir le profil d'un DID, envoyez une requête d'objet à l'interface Profil:

    {
    "@context": "https://schema.identity.foundation/0.1",
    "@type": "ObjectQueryRequest",
    "iss": "did:foo:123abc",
    "sub": "did:bar:456def",
    "aud": "did:baz:789ghi",
    "query": {
        "interface": "Profile",
    }
    }


### Permissions 
---

Tous les accès et manipulations des données d'identité sont soumis aux autorisations établies par l'entité propriétaire. 

### Actions 
---

L'interface Actions sert à envoyer à une identité cible des objets sémantiquement significatifs qui transmettent une intention à l'expéditeur, ce qui implique souvent la charge de données de l'objet. L'interface Actions n'est pas limitée à de simples communications centrées sur l'humain. Il est plutôt conçu comme un canal universel par lequel les identités peuvent effectuer toutes sortes d'activités, d'échanges et de notifications.


###  Stores 
---

La meilleure façon de décrire Stores est une variante à portée DID 1: 1 de l'API window.localStorage à portée d'origine du DOM du W3C. La principale différence est que cette forme de stockage d'objets persistants par paires transcende les fournisseurs, les plates-formes et les appareils. Pour chaque relation de stockage entre le propriétaire du DID et les DID externes, le concentrateur doit créer une zone de stockage basée sur les documents clé-valeur. Le propriétaire du DID ou le DID externe peut stocker des données JSON non structurées dans le document, en fonction des clés qu'ils spécifient. Le responsable de la mise en œuvre du concentrateur peut choisir de limiter l'espace disponible du document de stockage, avec la possibilité d'étendre la limite de stockage en fonction des critères définis par celui-ci.

### Collections 
--- 

La découverte de données est un problème depuis la création du Web. La plupart des tentatives précédentes pour résoudre ce problème partent du principe que la découverte concerne les entités individuelles fournissant un mappage de leur propre API et schémas de données spécifiques au service. Bien que vous puissiez certainement créer un format commun pour exprimer différentes API et schémas de données, vous vous retrouvez avec le même problème de base: une mer de services qui ne peuvent pas interagir efficacement sans examen, effort et intégration spécifiques. Les concentrateurs évitent entièrement ce problème en reconnaissant que le problème de la découverte de données est qu'elle repose sur la découverte. Au lieu de cela, les Hubs affirment que la localisation et la récupération de données devraient être un processus implicitement connaissable.

Les collections fournissent une interface pour accéder aux objets de données sur tous les hubs, quelle que soit leur implémentation. Cette interface n'exerce presque aucune opinion sur les schémas de données utilisés par les entités. Pour ce faire, l'interface Hub Collection permet aux objets de n'importe quel schéma d'être stockés, indexés et accessibles de manière unifiée.

Avec Collections, vous stockez, interrogez et récupérez des données en fonction du schéma et du type de données que vous recherchez. Voici quelques exemples d'objets de données provenant d'une variété de schémas courants auxquels les entités peuvent écrire et accéder via le Hub d'un utilisateur:

Localisez toutes les offres qu'un utilisateur pourrait souhaiter partager avec des applications (http://schema.org/Offer)

    {
    "@context": "https://schema.identity.foundation/0.1",
    "@type": "ObjectQueryRequest",
    "iss": "did:foo:123abc",
    "sub": "did:bar:456def",
    "aud": "did:baz:789ghi",
    "query": {
        "interface": "Collections",
        "context": "http://schema.org",
        "type": "Offer",
    }
    }

### Services 
---

Services offer a means to surface custom service calls an identity wishes to expose publicly or in an access-limited fashion. Services should not require the Hub host to directly execute code the service calls describe; service descriptions should link to a URI where execution takes place.

Performing a Request request to the base Services interface will respond with an object that contains an entry for every service description the requesting entity is permitted to access.


    // request
    {
    "iss": "did:foo:123abc",
    "sub": "did:bar:456def",
    "aud": "did:baz:789ghi",
    "@context": "https://schema.identity.foundation/0.1",
    "@type": "ServicesRequest",
    }

    // response
    {
    "@context": "https://schema.identity.foundation/0.1",
    "@type": "ServicesResponse",
    developer_message: "optional message",
    services: [{
        // Open API service descriptors
    }]
    }

----- 

# Références
## DIF 

https://github.com/decentralized-identity/hub-sdk-js

https://github.com/decentralized-identity/identity-hub

https://github.com/decentralized-identity/identity-hub/blob/master/explainer.md

https://github.com/WebOfTrustInfo/rwot6-santabarbara/blob/master/final-documents/did-auth.md

## Microsoft

DID Registration first test - https://didproject.azurewebsites.net/docs/registration-test.html

DID Registration prod - https://didproject.azurewebsites.net/docs/registration.html

Hub tutorial - https://didproject.azurewebsites.net/docs/hub-sdk-tutorial.html

Authenticator - https://didproject.azurewebsites.net/docs/credential-authenticator.html

Complete code - https://github.com/decentralized-identity/hub-sdk-js/blob/master/src/example.ts


## Complement 

JSON Web Key JWK - https://tools.ietf.org/html/rfc7517

Blockchain Identity projects : https://github.com/peacekeeper/blockchain-identity





