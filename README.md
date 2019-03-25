# Forge Cloud KMS
A node-forge fork repository to add Google Cloud KMS (asymmetric key) integration.  
Note that you must grant appropriate Cloud KMS IAM permissions to the service account you use (Editor is insufficient to use!).

Currently you should use this with "3072bit RSA key PKCS#1 v1.5 padding - SHA256 digest" asymmetric key, since my only need is that key and other keys have not yet been tested. 

## Modifications from the original repository

 * CloudKmsKey Object (referred as "kmsKeyObj" below): {projectId, keyRing, location, keyName, version}, where location defaults to "global" and version "1"
 * forge.pki.publicKeyFromCloudKms(keyObj) returns Promise
 * forge.cert.signByCloudKms(keyObj, md), where md defaults to forge.md.sha256, return Promise

## Examples
You need to add require statements at the top of the scripts

### Generate self-signed CA certificate from a Cloud KMS asymmetric key
```js
const kmsKey = {projectId: "my-project-id", keyRing: "myring", keyName: "myKey"}

const caInfo = [{
  name: "commonName",
  value: "My Precious CA"
}, {
  name: "countryName",
  value: "JP"
}];
let cert = forge.pki.createCertificate()

cert.serialNumber = "01"
cert.validity.notBefore = new Date()
cert.validity.notAfter = new Date()
cert.validity.notAfter.setFullYear(cert.validity.notBefore.getFullYear() + 100)
cert.setIssuer(caInfo)
cert.setSubject(caInfo)
cert.setExtensions([{
  name: 'basicConstraints',
  cA: true
}]);

// first get public key of the CA to complete cert infomation
forge.pki.publicKeyFromCloudKms(kmsKey)
.then((caPublicKey) => {
        cert.publicKey = caPublicKey

        // cert information completed, sign it by the private key.
        // note that this returns promise
        return cert.signByCloudKms(kmsKey)
}).then(() => {
        const certPem = forge.pki.certificateToPem(cert)
        console.log(certPem)
}).catch(e => console.error(e))
```

and execute this script

```bash
node kms.js | openssl x509 -noout -text
```

## Generate a X.509 client certificate with a Cloud KMS private asymmetric key
```js
const clientPublicKeyPem = "<PEM public key generated with openssl command here>"
const caCertPem = "<generated PEM above here>" 

const kmsKey = {projectId: PROJECT_ID, keyRing: KEY_RING, keyName: KEY_NAME}
const clientInfo = [{
  name: "commonName",
  value: "testingsubject"
}, {
  name: "countryName",
  value: "JP"
}];
const cert = forge.pki.createCertificate()
const cacert = forge.pki.certificateFromPem(caCertPem)

cert.serialNumber = "02"
cert.validity.notBefore = new Date()
cert.validity.notAfter = new Date()
cert.validity.notAfter.setFullYear(cert.validity.notBefore.getFullYear() + 100)
cert.setIssuer(cacert.issuer.attributes)
cert.setSubject(clientInfo)
cert.publicKey = clientPublicKeyPem

cert.signByCloudKms(kmsKey)
.then(() => {
	const certPem = forge.pki.certificateToPem(cert)
	console.log(cert)
	cacert.verify(cert) // this would raise no errors
})
.catch(e => console.error(e))
```
