# Universal Resolver


DIF is introducing a community preview of the Universal Resolver — a core building block of a decentralized identity system. This is a first step in fulfilling DIF’s mission to help individuals and organizations to control their digital identity, without being dependent on any intermediary party.
This tool fulfills a similar purpose as Bind does in the DNS system: resolution of identifiers. However, instead of working with domain names, we work with self-sovereign identifiers that can be created and registered directly by the entities they refer to.

This is important, because identifiers are the basis for any identity and communications system — without identifiers, we cannot have relationships, transactions, data sharing or messaging between entities. Historically identifiers have always been assigned to us by some kind of corporate or state authority. The Universal Resolver lets us build architectures and protocols on top of identifiers that are completely self-sovereign. There is no longer a need for a central authority to issue, maintain and revoke identifiers.

However, just having an identifier is not enough. We need some further information in order to know how to communicate with the entity represented by the identifier. The job of a “Resolver” is to discover and retrieve this further information. At a minimum, this information includes elements such as service endpoints for communicating with the entity, as well as the cryptographic keys associated with it. The Universal Resolver performs this task to enable the basic building blocks of a self-sovereign identity world.

DIF’s Universal Resolver provides a unified interface which can be used to resolve any kind of decentralized identifier. This enables higher level data formats (such as Verifiable Claims) and protocols (such as DIF’s Hub protocol) to be built on top of the identifier layer, no matter which blockchain or other system has been used to register the identifier. Internally, the Universal Resolver achieves this through an architecture consisting of “drivers” for each supported identifier type. A driver can be easily added via a Docker container, a Java API, or a remote HTTP GET call, as illustrated in the following diagram:




# EU 

Resolver
At the moment two implementations of Universal Resolver exist (Java and Python) https://github.com/decentralized-identity/universal-resolver.
The Universal Resolver resolves the DID and forwards the request to the specific DID Driver. Each Universal Resolver service consists of:

Universal Resolver Component
Drivers
Schematic presentation of the Universal Resolver:

A request of the form did:method:did_identifier must be passed to the Universal Resolver. The method specifies in which DID Registry the DID is registered. The resolver must have access to the DID Driver for the method specified. The DID Driver communicates with the DID Registry and obtains the raw DID Document data, transforms the data into a standard DID Document format and returns the formatted DID Document.

A wide range of DID methods/drivers exists at the moment (https://github.com/decentralized-identity/universal-resolver/#drivers) hence the integration of the Universal Resolver and Registrar in EBSI/ESSIF will lead to faster adoption of the SSI, support for a wide range of SSI solutions appearing in the market and the highest level of interoperability.






## Flows 
### Resolver 

1. An agent or a Client sends a resolution request to the Universal Resolver: resolve(did, options):
        a. did is REQUIRED. For example did:ebsi:did_identifier.
        b. options is OPTIONAL:

            i. version-id: the specific version of the DID Document,

            ii. version-time: the version of the DID Document at the given time.

2. Universal Resolver validates that did has the form: did:method:did_identifier. If not, the DID Resolver MUST raise an error.
3. Universal Resolver determines if the input DID method is supported by the DID Resolver that implements this algorithm. If not, the DID Resolver MUST raise an error. Supported methods for EBSI V1 are ebsi-eth and ebsi-fabric.
4. Universal Resolver obtains the DID Document responding to the request executing the Read operation against the requested DID's Decentralized Identifier Registry, as defined by the input DID method.
5. Universal Resolver validates the DID Document data and transforms it into the ESSIF V1 DID Document data model format.


For more details on the DID Document see ESSIF DID Modelling and the example at the bottom.


## Alternatives au Universal Resolver

Il y a autres initiatives qui font l'implementation de la résolution de DID en DID documents, et qui donnent des solutions semblables à Universal Resolver. 

- did-client, a did resolver in command line by Digital Bazaar. 

- Javascript DID-Resolver, by uPort.


# Références 

[Decentralized Identity Foundation](http://identity.foundation)

[Blog post](https://medium.com/decentralized-identity/a-universal-resolver-for-self-sovereign-identifiers-48e6b4a5cc3c)

[Conference web](https://ssimeetup.org/did-resolution-given-did-how-do-retrieve-document-markus-sabadello-webinar-13/)

[Universal Resource Git Repo](https://github.com/decentralized-identity/universal-resolver/)

[Driver development](https://github.com/decentralized-identity/universal-resolver/blob/main/docs/driver-development.md)

[Use case Birth Certificate](https://arxiv.org/pdf/2009.04327.pdf)

[Ultimete SSI beginner's guide](https://tykn.tech/self-sovereign-identity/)

[UniResolver.io de DIF](https://dev.uniresolver.io/)

[European Union Universal Resolver](https://ec.europa.eu/cefdigital/wiki/pages/viewpage.action?pageId=262505873)

[Decentralized Identifier Resolution (DID Resolution) v0.2](https://w3c-ccg.github.io/did-resolution/)


# Expérimentation LaboIDN 






# Document 

## Objectif  
La Decentralized Identity Foundation, DIF, est une organisation dédiée à cultiver les idées et les spécifications émergents, à établir discussions en toute l'industrie, faire l'expérimentation (teste d'hypothèses) et la démonstration des possibilités de l'intéropérabilité. Elle a crée une implementation de référence du DID-Resolver, apellé Universal Resolver, or UR, qui établit les bonnes pratiques préconisées par la DIF dans un module de software. 

Ce document a pour but d'analiser la viabilité et l'applicabilité de l'utilisation de l'UR pour faire la résolution de identificateurs-DID et la génération de documents-DID équivalents. De cette façon, on vise à être capable de traiter et inter-opérer avec des identités décentralisées, independament de leur provenance et appartenance à un réseua blockchain spécifique.  

## Démarche  

Un `DID-Resolver` est une solution qui reçoit un DID en entrée, obtient les données et construit un `DID-Document` standardisé. Cela est une unité fondamentale pour un système d'identité numérique. 

L'`Universal Resolver` est une implementation dévelopée et maintenue par la `Decentralized Identity Foundation`. Comme le `bind` dans un contexte de `DNS` de l'internet, l'Universal Resolver prend l'identificateur, exprimé comme un `DID` et le traduit dans un `DID-Document`, qui contient toutes les informations necessaires pour exchanger de l'information avec le détenteur de l'identité numérique. 

L'identificateur doit être formatée selon le format standard `did:method:did_identifier` et le document-DID doit être disponible au resolveur à la fin. 

L'universal resolver permet la construction d'architectures et protocoles complètement basés sur des identificateurs self-sovereign. Il permet aussi la création, le registre et le maintien de identificateus directement par l'entité à laquelle ils font référence. Alors, il n'y a pas besoin d'une authorité centrale pour l'entretien ou la révocation d'identificateurs. 

 

 

## Analyse des Resultats 

Les résultats des travaux d'expérimentation du LaboIDN par rapport à l'utilisation de l'UR sont les suivantes:  

        Nous avons démontré que l’utilisation de résolveurs « DID » par des services émetteur et consommateur distinct et indépendant, permet l’utiliser d’un ensemble commun d'API sur différents registres, eux-mêmes distinct et indépendant. Plus précisément, l'expérimentation a démontré qu’un service consommateur peut vérifier des justificatifs émis sur différent registre et utilisant un résolveur “DID”.

        L'expérimentation est principalement ancrée par les efforts de standardisation du W3C. Les visées d’Hyperleger et du DIF sont semblables, mais nous n’avons pas été en mesure d'identifier une preuve de concept pleinement implanté et disponible.

        Nous avons démontré que l’utilisation de résolveurs « DID » par des services émetteur et consommateur distinct et indépendant, permet d’utiliser un ensemble commun d'API sur différents registres, eux-mêmes distincts et indépendants. Plus précisément, l'expérimentation adémontré qu’un service consommateur peut vérifier des justificatifs émis sur différent registre et utilisant un résolveur “DID”.

        L'expérimentation est principalement ancrée par les efforts de standardisation du W3C. Les visées d’Hyperleger et du DIF sont semblables, mais nous n’avons pas été en mesure d'identifier une preuve de concept pleinement implanté et disponible.
        L’expérimentation a été rendue possible grâce aux efforts d’interopérabilité du département américain du “Homeland Security” (DHS). Veuillez noter que le fédéral, le laboIDN et 6 entreprises canadiens ont tous juste commencé un projet semblable.

 

 

## Conclusion 



# Référeces

[Decentralized Identity Foundation](http://identity.foundation)

[Blog post](https://medium.com/decentralized-identity/a-universal-resolver-for-self-sovereign-identifiers-48e6b4a5cc3c)

[Conference web](https://ssimeetup.org/did-resolution-given-did-how-do-retrieve-document-markus-sabadello-webinar-13/)

[Universal Resource Git Repo](https://github.com/decentralized-identity/universal-resolver/)

[Driver development](https://github.com/decentralized-identity/universal-resolver/blob/main/docs/driver-development.md)

[Use case Birth Certificate](https://arxiv.org/pdf/2009.04327.pdf)

[Ultimete SSI beginner's guide](https://tykn.tech/self-sovereign-identity/)

[UniResolver.io de la DIF](https://dev.uniresolver.io/)

[European Union Universal Resolver](https://ec.europa.eu/cefdigital/wiki/pages/viewpage.action?pageId=262505873)

[Decentralized Identifier Resolution (DID Resolution) v0.2](https://w3c-ccg.github.io/did-resolution/)

● User-Centric Verifiable Digital Credentials Challenge, ​https://github.com/canada-ca/ucvdcc

● Verifiable Credential Issuer API, ​https://w3c-ccg.github.io/vc-http-api/

● Verifiable Credential Verifier HTTP API, ​https://w3c-ccg.github.io/vc-http-api/

● W3C Verifiable Credentials Data Model, ​https://www.w3.org/TR/vc-data-model/

● W3C Decentralized Identifiers, ​https://w3c.github.io/did-core/

● W3C JSON-LD, ​https://www.w3.org/TR/json-ld11/

● Universal resolver, ​https://github.com/decentralized-identity/universal-resolver/blob/master/README.md

● Example des tests du DHS, https://github.com/canada-ca/vc-examples-ca/tree/master/plugfest-2020



# Definitions 


A DID, or Decentralized Identifier, is a URI composed of three parts: the scheme "did:", a method identifier, and a unique, method-specific identifier generated by the DID method. DIDs are resolvable to DID documents.

Verifiable Data Registry
In order to be resolvable to DID documents, DIDs are typically recorded on an underlying system or network of some kind. Regardless of the specific technology used, any such system that supports recording DIDs and returning data necessary to produce DID documents is called a verifiable data registry. Examples include distributed ledgers, decentralized file systems, databases of any kind, peer-to-peer networks, and other forms of trusted data storage.

DID Document 
DID documents contain information associated with a DID. They typically express verification methods (such as public keys) and services relevant to interactions with the DID subject. A DID document can be serialized according to a particular syntax (see Section § 6. Representations). The generic properties supported in a DID document are specified in § 5. Core Properties. The DID itself is the value of the id property. The properties present in a DID document may be updated according to the applicable operations outlined in § 7. Methods.

DID Method 
DID methods are the mechanism by which a particular type of DID and its associated DID document are created, resolved, updated, and deactivated using a particular verifiable data registry. DID methods are defined using separate DID method specifications (see § 7. Methods).

DID Resolver
A DID resolver is a software and/or hardware component that takes a DID (and associated input metadata) as input and produces a conforming DID document (and associated metadata) as output. This process is called DID resolution. The inputs and outputs of the DID resolution process are defined in § 8. Resolution. The specific steps for resolving a specific type of DID are defined by the relevant DID method specification. Additional considerations for implementing a DID resolver are discussed in [DID-RESOLUTION].






https://www.w3.org/TR/did-core/

https://w3c-ccg.github.io/did-resolution/