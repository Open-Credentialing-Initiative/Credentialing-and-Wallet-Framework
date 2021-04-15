# Credentialing-and-Wallet-Framework



## Identifier
A DID is a globally unique identifier developed specifically for decentralized systems as defined by the W3C DID specification. DIDs enable interoperable decentralized Self-Sovereign Identity management. A DID is associated with exactly one DID Document.

## Credential Structure
A Credential that includes a proof from its issuer. Typically this proof is in the form of a digital signature. Based on the definition provided by the W3C Verifiable Claims Working Group.

## Credential Schemas
Please find the documentation of the credential schemas on https://open-credentialing-initiative.github.io/oci/

## Schemas 
JSON Schemas for aforementioned credentials to be defined and anchored using GS1 Web Vocab

## Signatures
The proofs (including signatures) of Verifiable Credentials and Verifiable Presentation to be generated and verified in conformance with JSON Web Signature 2020 that is one of relevant LD Signature Suites to the date that are registered at Linked Data Cryptographic Suite Registry.

## Verification Method

1.	Signing and verification of Verifiable Credentials to be achieved using key pairs based on Secp256k1 curve. 
2.	Public keys to be represented as JWK that is a supported format by DID Doc spec.
3.	Public keys to be referenced in a respective DID Document and associated with specific proof purposes via the "Verification Relationship" concept that is defined in DID Core spec for further verification.

## Wallet to Wallet communication

Wallet to wallet communication between Credential Issuer and Trading Partner Identity Wallet is based on a set of interoperable and DID method agnostic Aries RFCs that find its roots in working items of Decentralized Identity Foundation. This High Level chart can be used as guidance for the subject workflow.

All messages to be packed (signed/encrypted) in JWM envelopes using DIDComm v2 spec.

Aries RFCs (key ones)
- Issue Credential Protocol v2
- Credential Manifest
- Present Proof Protocol v2
-	Presentation Exchange

Decentralized Identity Foundation specs
-	DIDComm Messaging v2
-	Credential Manifest
-	Presentation Exchange


## Messaging Standard for PI Verification 
To exchange verifiable presentations of VCs in form of a JSON Web Token, the VRS providers integrating the Spherity Credentialing Service are able attach the JWT to the header of the GS1 Lightweight Messaging Standard for PI Verifications.

## Credential Issuance for ATPs 

The full documentation of the APIs implemented based on this framework can be found here: https://docu.cs.spherity.com/

## Interactions between Credential Issuer’s wallet and Trading Partner’s wallet

In order to issue credentials, Credential Issuer will be utilizing the implementation of  “issue-credential” protocol v2 using DID:WEB and/or DID:ETHR identifiers.
Credential Issuer’s wallet offers/issues “CompanyIdentityVerificationCredential” to Trading Partner’s wallet

Note: The latest Spheirty Wallet API docs for Credential Offer flow can be found here.

Please find the flow overview below.
1. Initiate communication from Credential Issuer side 
  * Credential Issuer sends "offer-credential" message to Trading Partner AND provide custom VC ID as "credInput.id" in the request body
2. Trading Partner responds with "request-credential" message in an automatic manner (message filter should be set up during configuration of a Trading Partner)
3. Credential Issuer sends "issue-credential" message to Trading Partner (a manual step on Credential Issuer side)
4. Trading Partner stores a credential from an incoming "issue-credential" message in an automatic manner (message filter should be set up during configuration of a Trading Partner)

## Credential Issuer’s wallet requests/obtains presentation of “CompanyIdentityVerificationCredential” from Trading Partner’s wallet

Note: The latest Spherity Wallet API docs for Request Presentation flow can be found here: https://docu.cs.spherity.com/
Please find the flow overview below.
1. Initiate communication from Credential Issuer side
  * Credential Issuer requests “CompanyIdentityVerificationCredential” from Trading Partner
    * Two API endpoints MUST be triggered one-be-one
      * Create a message filter to save the presentation in case "presentation" message is received from a particular DID of Trading Partner whom you did send "request-presentation" message beforehand
      * Credential Issuer sends "request-presentation" message to Trading Partner
2. Trading Partner responds with "presentation" message in an automatic manner (message filter should be set up during configuration of a Trading Partner)
3. Credential Issuer stores a presentation from an incoming "presentation" message in an automatic manner (message filter should be set up simultaneously at the time of sending "request-presentation" message)

## Interaction between Trading Partner’s wallet and PI Verification messaging service (VRS)

Note: The latest Spherity Wallet API docs for ATP VP Issuance and ATP VP Verification can be found here: https://docu.cs.spherity.com/

### API - Create ATP Credential Verifiable Presentation

This API describes the generation of a signed Verifiable Presentation in form of a JWT. To audit trail all PI messages the piMessageHash is combined with the presentation of the Verifiable Credential.
Verifiable Credential types that can be fetched from the Trading Partners wallet are:

* DSCSA Wholesaler ATP Credential
* DSCSA Manufacturer ATP Credential
* DSCSA Dispenser ATP Credential

Please consider that the Trading Partner needs to have a respective credential in its wallet to successfully create a JWT. Request and response examples can be found here: https://docu.cs.spherity.com/
Within the API the following steps are done:

1.	Create basic log record for an incoming request
  *	“holder” property from request body
  * “vcType” property from request body
  * “corrUUID” property from request body
  * “piMessageHash” property from request body
  * “creator” property from decoded JWT(“email” property from Bearer token)
2.	Fetch ATP VC from database via Holder DID (“holder”) & VC Type (“vcType”)
3.	Throw 404 when no ATP VCs found
4.	Filter the latest ATP VC via Issuance Date (“issuedOn” property that holds UNIX Epoch milliseconds)
5.	Update log record with “verifiableCredential” property based on found ATP VC
6.	Verify ATP VC based on a verification scope that was provided in the request body OR skip in case verification scope for VC is not provided in request’s query params.
  * Update log record with received ATP VC verification result
  * Throw 500 when ATP VC verification result include errors OR return 200 in case query params include gracefulErrors=true
7.	Find wallet record that holds provided Holder DID (“holder”)
8.	Pick the first keypair of Holder DID (“holder”)
9.	Issue ATP VP (JWT) with the latest ATP VC & “piMessageHash” as “nonce” inside
10.	Update log record with “verifiablePresentation” property based on issued ATP VP
11.	Verify ATP VP based on a verification scope that was provided in the request body OR skip in case verification scope for VP is not provided in request’s query params.
  *	Update log record with received ATP VP verification result
  * Throw 500 when ATP VP verification result include errors OR return 200 in case query params include gracefulErrors=true
12.	Return 200 and the latest log record inside response body

This API describes the generation of a signed Verifiable Presentation in form of a JWT. To audit all PI messages the PI MessageHash is combined with the presentation of the Verifiable Credential.

The PI Message Hash is created by the VRS by calculating: SHA256(GTIN+SERIAL+LOT+EXPIRY.substring(0,4))

Next to that it also takes a few request parameters.

### API - Verify ATP Credential Verifiable Presentation

This API describes the verification of a received JWT. The VRS provider needs to send the JWT together with the respective corrUUID to the verifier wallet.
In the verification API the following will be checked::
1.	the signature on the JWT
2.	the signature on the DSCSA ATP Credential issuer
3.	the credential expiration date
4.	the revocation registry

The API will respond with error messages if one of these steps fails. Please find error messages and codes in the 
of the API documentation: https://docu.cs.spherity.com/

Within the API the following steps are done:

1.	Create basic log record for an incoming request
* “verifiablePresentation” property from request body
* “corrUUID” property from request body
* “creator” property from decoded JWT(“email” property from Bearer token)
2.	Decode JWT i.e. the value of “verifiablePresentation”
3.	Update log record with “piMessageHash” property from decoded ATP VP’s “nonce”
4.	Find ATP VC within decoded ATP VP data
5.	Update log record with “verifiableCredential” property based on found ATP VC
6.	Import counterparty into your wallet based on:
* Subject DID (“credentialSubject.id”) value of ATP VC as “did”
* Company name (“credentialSubject.[“Subject Company Name”]) value from ATP VC as “didName”
7.	Verify ATP VP based on a verification scope that was provided in the request body OR skip in case verification scope for VP is not provided in request’s query params.
* Update log record with received ATP VP verification result
* Throw 500 when ATP VP verification result include errors OR return 200 in case query params include gracefulErrors=true
8.	Verify ATP VC based on a verification scope that was provided in the request body OR skip in case verification scope for VC is not provided in request’s query params.
* Update log record with received ATP VC verification result
* Throw 500 when ATP VC verification result include errors OR return 200 in case query params include gracefulErrors=true
9.	Return 200 and the latest log record inside response body

