# Upper Secondary School Diploma


# How to Issue JSON-LD-"Upper Secondary School Diploma" using Aca-py 0.7.1

The following Readme is is adapted from the official Aca-Py Demo-Readme:
https://github.com/hyperledger/aries-cloudagent-python/blob/main/demo/AliceWantsAJsonCredential.md 

Aca-py has the capability to issue and verify both Indy and JSON-LD (W3C compliant) credentials.

The JSON-LD support is documented [here](../JsonLdCredentials.md) - this document will provide some additional detail in how to use the demo and admin api to issue and prove JSON-LD credentials.

Schema used: https://raw.githubusercontent.com/pherbke/schoolDiploma/master/schemas/schoolDeploma_schema.json


## Setup Agents to Issue JDON-LD Credentials

Clone this repository to a directory on your local:

```bash
git clone https://github.com/hyperledger/aries-cloudagent-python.git
cd aries-cloudagent-python/demo
```

Open up a second shell (so you have 2 shells open in the `demo` directory) and in one shell:

```bash
LEDGER_URL=http://dev.greenlight.bcovrin.vonx.io ./run_demo faber --did-exchange --aip 20 --cred-type json-ld
```

... and in the other:

```bash
LEDGER_URL=http://dev.greenlight.bcovrin.vonx.io ./run_demo alice
```

Copy the "invitation" json text from the Faber shell and paste into the Alice shell to establish a connection between the two agents.

Now open up two browser windows to the [Faber](http://localhost:8021/api/doc) and [Alice](http://localhost:8031/api/doc) admin api swagger pages.

Using the Faber admin api, you have to create a DID with the appropriate:

- DID method ("key" or "sov")
- key type "ed25519" or "bls12381g2" (corresponding to signature types "Ed25519Signature2018" or "BbsBlsSignature2020")
- if you use DID method "sov" you must use key type "ed25519"

Note that "did:sov" must be a public DID (i.e. registered on the ledger) but "did:key" is not.

For example, in Faber's swagger page call the `/wallet/did/create` endpoint with the following payload:

```
{
  "method": "key",
  "options": {
    "key_type": "bls12381g2" // or ed25519
  }
}
```

This will return something like:

```
{
  "result": {
    "did": "did:key:zUC71KdwBhq1FioWh53VXmyFiGpewNcg8Ld42WrSChpMzzskRWwHZfG9TJ7hPj8wzmKNrek3rW4ZkXNiHAjVchSmTr9aNUQaArK3KSkTySzjEM73FuDV62bjdAHF7EMnZ27poCE",
    "verkey": "mV6482Amu6wJH8NeMqH3QyTjh6JU6N58A8GcirMZG7Wx1uyerzrzerA2EjnhUTmjiSLAp6CkNdpkLJ1NTS73dtcra8WUDDBZ3o455EMrkPyAtzst16RdTMsGe3ctyTxxJav",
    "posture": "wallet_only",
    "key_type": "bls12381g2",
    "method": "key"
  }
}
```

You do *not* create a schema or cred def for a JSON-LD credential (these are only required for "indy" credentials).

You will need to create a DID as above for Alice as well (`/wallet/did/create` etc ...).

Congradulations, you are now ready to start issuing JSON-LD credentials!

- You have two agents with a connection established between the agents - you will need to copy Faber's `connection_id` into the examples below.
- You have created a (non-public) DID for Faber to use to sign/issue the credentials - you will need to copy the DID that you created above into the examples below (as `issuer`).
- You have created a (non-public) DID for Alice to use as her `credentialSubject.id` - this is required for Alice to sign the proof (the `credentialSubject.id` is not required, but then the provided presentation can't be verified).

To issue a credential, use the `/issue-credential-2.0/send` (or `/issue-credential-2.0/create-offer`) endpoint, you can test with the example payload of the file `diploma_fictionalStudent.json` from the folder `exampleCredentials/` (just replace the "connection_id", "issuer" key, "credentialSubject.id" and "proofType" with appropriate values:

```
{
    "connection_id": "41637230-c158-4b98-99fe-a0c706ad4541",
    "filter": {
      "ld_proof": {
        "credential": {
          "@context": [
            "https://www.w3.org/2018/credentials/v1",
            "https://raw.githubusercontent.com/pherbke/educationalCredentials/main/schemas/upperSecCert.json"
          ],
          "type": ["VerifiableCredential", "UniversityDegreeCredential"],
          "issuer": {
            "id": "did:key:zUC7L3Dvdvnt8EHzekfj8iLEp1cRrBvQYHy8MCFEDY74qBb4a6nrpTd5k4NhpEXJ7e7kGaqiohzNpzB2dEebG6zXdwSYQXDbhdn16qzVrTZkvSArVpijn3qo2HgcA2PefDvkGpB",
            "schoolType" : "Upper Secondary School",
            "name" : "Engelbert-Kaempfer-Gymnasium",
            "info" : "Städt. Gymnasium für Jungen und Mädchen - Sekundarstufen I und II -",
            "number" : "168890",
            "url" : "https://www.ekg-lemgo.de/",
            "eqfLevel" : "4",
            "eqfCategory" : "344",
            "country" : "DE",
            "countryISO3166" : "276",
            "federalStateISO31662" : "DE-NW"
          }
... 
```

Note that if you have the "auto" settings on, this is all you need to do.  Otherwise you need to call the `/send-request`, `/store`, etc endpoints to complete the protocol.

To see the issued credential, call the `/credentials/w3c` endpoint on Alice's admin api - this will return the content from the `storedCredential.json` as in the folder `result/`:

```
{
    "contexts": [
      "https://w3id.org/security/bbs/v1",
      "https://www.w3.org/2018/credentials/v1",
      "https://raw.githubusercontent.com/pherbke/educationalCredentials/main/schemas/upperSecCert.json"
    ],
    "expanded_types": [
      "https://www.w3.org/2018/credentials#VerifiableCredential",
      "UniversityDegreeCredential"
    ],
    "schema_ids": [],
    "issuer_id": "did:key:zUC7L3Dvdvnt8EHzekfj8iLEp1cRrBvQYHy8MCFEDY74qBb4a6nrpTd5k4NhpEXJ7e7kGaqiohzNpzB2dEebG6zXdwSYQXDbhdn16qzVrTZkvSArVpijn3qo2HgcA2PefDvkGpB",
    "subject_ids": [
      "did:key:zUC795mecXi5mttgFmmAeo5wvZSJvBerdu2yaUPDWyPdsb2p9xexWmuJLeg9D59QvWK491MY4gqvT14WdBKAYiZNeLZ83GfxAz2qEkZXMB5zUhHDdU7e7YdBEMFHZYev5mzn9rd"
    ],
    "proof_types": [
      "BbsBlsSignature2020"
    ],
    "cred_value": {
      "@context": [
        "https://www.w3.org/2018/credentials/v1",
        "https://raw.githubusercontent.com/pherbke/educationalCredentials/main/schemas/upperSecCert.json",
        "https://w3id.org/security/bbs/v1"
      ],
      "type": [
        "VerifiableCredential",
        "UniversityDegreeCredential"
      ],
      "issuer": {
        "id": "did:key:zUC7L3Dvdvnt8EHzekfj8iLEp1cRrBvQYHy8MCFEDY74qBb4a6nrpTd5k4NhpEXJ7e7kGaqiohzNpzB2dEebG6zXdwSYQXDbhdn16qzVrTZkvSArVpijn3qo2HgcA2PefDvkGpB",
        "schoolType": "Upper Secondary School",
        "name": "Engelbert-Kaempfer-Gymnasium",
        "info": "Städt. Gymnasium für Jungen und Mädchen - Sekundarstufen I und II -",
        "number": "168890",
        "url": "https://www.ekg-lemgo.de/",
        "eqfLevel": "4",
        "eqfCategory": "344",
        "country": "DE",
        "countryISO3166": "276",
        "federalStateISO31662": "DE-NW"
      }
...
```

Used ISCED codes: 
https://ec.europa.eu/assets/eac/education/tools/iscedf/codes_en.htm
