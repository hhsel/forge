# Forge Cloud KMS
A node-forge fork repository to add Google Cloud KMS (asymmetric key) integration.  
Note that you must grant appropriate Cloud KMS IAM permissions to the service account you use (Editor is insufficient to use!).

## Modifications from the original repository

 * CloudKmsKey Object (referred as "kmsKeyObj" below): {projectId, keyRing, location, keyName, version}, where location defaults to "global" and version "1"
 * forge.pki.publicKeyFromCloudKms(keyObj) returns Promise
 * forge.cert.signByCloudKms(keyObj, md), where md defaults to forge.md.sha256

## Example
```js
const kmsKey = {projectId: "my-project-id", keyRing: "myring", keyName: "myKey"}

forge.pki.publicKeyFromCloudKms(kmsKey)
    .then(res => {console.log(res)})
    .catch(err => console.error(err))
```
