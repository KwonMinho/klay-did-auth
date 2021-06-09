# klay-did-client

해당 모듈은 자바스크립트 기반의 클레이튼 DID 레지스트리의 클라이언트 모듈이다.

해당 모듈은 서브 모듈인 auth가 있다. <br/>


<br />

### Example code
```js

    const KlayDIDClient = require('klay-did-auth');
    const klayDID = new KlayDIDClient({ network: '테스트넷', regABI: 'DID 레지스트리 ABI File Path', regAddr: 'DID 레지스트리 주소'});
    
    
    async function test(){
        const document = await klayDID.getDocument('did:kt:eFefe...');
        const nonce = await klayDID.getNonce('did:kt:eFefe...');
    
        //or login({path: 'key store file(json)', password: '1234'});
        klayDID.auth.login({account: '0x..', privateKey: '0x..'});
    
        console.log(klayDID.auth.isLogin())
        // >> true    
    
        const result1 = await klayDID.createDocument();
        if(result1.status == -1 || result1.status == -2){
            console.log(result1.msg); // >> Login is required  or Failed create document 
        }
    
        const result2 = await klayDID.addPubKey('did:kt:eFefe...', 'EcdsaSecp256k1RecoveryMethod2020', '0xdmFkem..', 'did:kt:Femfd..');
        if(result2.status ==1){
            console.log(result2.msg); // >> Success add pubKey
        }    
    }
    
    test();
```
<br />

### 목차 <br />
[1. Create](#create) <br/>
[2. Sign Verification](#sign-verification) <br/>
[3. Delegate](#delegate) <br/>
[4. Add attribute](#add-attribute) <br/>
[5. Read Utils](#read-utils) <br/>
[6. Revoke Attribute](#revoke-attribute) <br/>
[7. Signature Data](#signature-data) <br/>
[8. Submodule Auth](#submodule-auth) <br/>


<br />

## Create


### document

```js
await klayDID.createDocument()
```
<br/>

### keypair(private/public key)

```js
const keypair = await klayDID.createPairKey(type)
```

type의 값은 `EcdsaSecp256k1RecoveryMethod2020`와 `EcdsaSecp256k1VerificationKey2019`있다.

`EcdsaSecp256k1RecoveryMethod2020`은 keypair{private key, account address}

`EcdsaSecp256k1VerificationKey2019`은 keypair{private key, public key}

private key: 32byte (hex string), public key: 33byte (hex string), account address: 0x{hex string}
<br/>

## Sign Verification


### create sign

```js
const signature = await klayDID.sign(data, type, privateKey)
```
해당 함수는 일반적으로 did auth 과정에서 사용되는 signature를 만들기 위한 함수이다.

data는 사인할 때 사용되는 값을 의미한다.

type의 값은 `EcdsaSecp256k1RecoveryMethod2020`와 `EcdsaSecp256k1VerificationKey2019`있다.

type의 값을 잘못 넣으면 signature 값은 0x00

만약, did registry에서 사용되어지는 signature를 만들고 싶다면, 아래와 같은 규칙을 따른다.
1. type의 값은 `EcdsaSecp256k1RecoveryMethod2020`
2. data는 [여기](#signature-data)에 설명되어 있는 값으로 구성한다. 

**privateKey는 type에 맞는 키를 사용해야한다.**

signature 값은 { signature: 0x(hex string), VRS obj }인 object가 반환

VRS obj는 type이 `EcdsaSecp256k1RecoveryMethod2020` 일 때만 null 아니고, {v: int, r: string, s: string}으로 구성된다.

<br/>

### verification

```js
const isValid = await klayDID.isValidSign(signature, data, publicKey)
```

isValid의 값은 bool 타입이다.

signature는 0x(hex string)

data는 signature에 사용된 data을 의미한다. (string 타입)

public key는 did document의 public key list에 있는 public key object{id, keyType, pubKeyData}이다.

<br/>

## Delegate

```js
await klayDID.setController(did, delegate)
```
did는 업데이트할 문서의 주체를 의미한다(ex. did:kt:dF2..)

delegate는 해당 did document의 편집할 수 있는 권한을 가질 대리인을 의미한다.

delegate는 did format을 따른다.

<br/>
```js
await klayDID.setControllerBySigner(did, delegate, signature)
```
signature는 `controller`의(public key의 controller가 아니다) private key로 사인한 값이다.

signature는 {uint8 v, bytes32 r, bytes32 s}로 이루어진 object 타입이다.

signature에 사용되는 데이터 값은 [여기](#signature-data)에 설명되어 있다.

did, delegate는 `setController`와 동일하다.

<br/>

## Add attribute(public key, service)


### public key

```js
await klayDID.addPubKey(did, type, publicKey, controller)
```
did는 업데이트할 문서의 주체를 의미한다(ex. did:kt:dF2..)

type의 값은 `EcdsaSecp256k1RecoveryMethod2020`와 `EcdsaSecp256k1VerificationKey2019`있다.

1. type이 `EcdsaSecp256k1RecoveryMethod2020`일 때, publicKey는 account address(0x{20byte})
2. type이 `EcdsaSecp256k1VerificationKey2019`일 때, publicKey는 33byte (hex string)

controller는 해당 public key의 private key를 가지고 있는 주체이다.

<br/>

```js
await klayDID.addPubKeyBySigner(did, type, publicKey, controller, signature)
```

해당 함수는 public key list에  public key를 `did document의 controller`(public key의 controller가 아니다)에 의해서 업데이트하는 함수이다.

signature는 `controller`의(public key의 controller가 아니다) private key로 사인한 값이다.

signature는 {uint8 v, bytes32 r, bytes32 s}로 이루어진 object 타입이다.

signature에 사용되는 데이터 값은 [여기](#signature-data)에 설명되어 있다.

did, type, publicKey, controller는 `addPubKey`와 동일하다.


<br/>

### service
```js
await klayDID.addService(did, scvId, scvType, scvEndPoint)
```
did는 업데이트할 문서의 주체를 의미한다(ex. did:kt:dF2..)

scvId는 해당 서비스 항목의 식별자를 의미한다

만약 scvId가 'storage-service'로 한다면, 문서에서는 scvId가 did:kt:dF2...#storage-service로 저장된다.

scvId,scvType 그리고 scvEndpoint의 값은 [W3C DIDs specification](https://www.w3.org/TR/did-core/#services)를 따른다.

<br/>
```js
await klayDID.addServiceBySinger(did, scvId, scvType, scvEndPoint, signature)
```

해당 함수는  `did document의 controller`(public key의 controller가 아니다)에 의해서 업데이트하는 함수이다.

signature는 `controller`의(public key의 controller가 아니다) private key로 사인한 값이다.

signature는 {uint8 v, bytes32 r, bytes32 s}로 이루어진 object 타입이다.

signature에 사용되는 데이터 값은 [여기](#signature-data)에 설명되어 있다.

did, scvId, scvType, scvEndPoint는 `addService`와 동일하다.

<br/>

## Read Utils

### read document
```js
const document = await klayDID.getDocument(did)
```
did는 가져올 document의 주체를 의미한다.

document의 예시는 아래와 같다.

```json
{
  "contexts": ['https://www.w3.org/ns/did/v1'],
  "id": "did:kt:f269E60fF7280e3E11b7EEd7B76b5C005105D121"
  "controller": "did:kt:5C005105D121f269E60fF7280e3E11b7EEd7B76b",
  "publicKeys":[
    "did:kt:f269E60fF7280e3E11b7EEd7B76b5C005105D121#key-1",
    "EcdsaSecp256k1RecoveryMethod2020",
    "did:kt:f269E60fF7280e3E11b7EEd7B76b5C005105D121",
    "0xf269E60fF7280e3E11b7EEd7B76b5C005105D121",
    "false" // isDisable
  ],
  "service": [
	{
	  "id": "did:kt:f269E60fF7280e3E11b7EEd7B76b5C005105D121#some-service",
	  "type": "SomeServiceType",
	  "serviceEndpoint": "Some URL"
	}
  ]
}
```
<br/>

### read nonce
```js
const document = await klayDID.getNonce(did)
```
did는 가져올 document의 주체를 의미한다.

nonce는 레지스트리에서 사용할 [signature](#signature-data)를 만들때 사용된다.

<br/>

<br/>

### extract public key
```js
const pubKeyObj = await klayDID.extractPubKey(document, pubKeyID)
```
document는 `getDocument('did:kt:0Fkdmf..')을 반환 값

pubKeyID = `did:kt:0Fkdmf..#key-1`

pubKeyObj = {id, keyType, controller, pubKeyData, disable} 

<br/>

## Revoke Attribute

### pubKey
```js
await klayDID.revokePubKey(did, pubKeyId)
```
pubKeyId는 did document에 있는 public key object에서 id를 의미한다.

Example) 문서에 있는 public key의 ID가 did:kt:dsfsF...xx#key-1 있다면, pubKeyId는 `key-1`이다.

<br/>

```js
await klayDID.revokePubKeyBySinger(did, pubKeyId, signature )
```

signature는 `controller`의(public key의 controller가 아니다) private key로 사인한 값이다.

signature는 {uint8 v, bytes32 r, bytes32 s}로 이루어진 object 타입이다.

signature에 사용되는 데이터 값은 [여기](#signature-data)에 설명되어 있다.

did, pubKeyId는 `revokePubKey`와 동일하다.


<br/>

### service
```js
await klayDID.revokeService(did, scvId)
```
scvId는 did document에 있는 serivce object에서 id를 의미한다.

Example) 문서에 있는 serivce의 ID가 did:kt:dsfsF...xx#company 있다면, scvId는 `company`이다.


<br/>
```js
await klayDID.revokeServiceBySinger(did, scvId)
```

signature는 `controller`의(public key의 controller가 아니다) private key로 사인한 값이다.

signature는 {uint8 v, bytes32 r, bytes32 s}로 이루어진 object 타입이다.

signature에 사용되는 데이터 값은 [여기](#signature-data)에 설명되어 있다.

did, scvId는 `revokeService`와 동일하다.

<br/>

### document

해당 모듈에서는 did document의 폐기를 지원하지 않는다.

왜냐하면 **한번 폐기한 document는 다시 되돌리 수 가 없기 때문에**, 신중하게 폐기해야한다.

caver-js 모듈을 사용해서 직접 `didLedger의 deactivatedDom(did)` 실행해라.

`deactivatedDom`는 오직 did의 소유자만 할 수 있다. (대리인은 불가능)


<br/>

## Signature Data

`addPubKeyBySigner`나 `addServiceBySinger`에서 사용되어지는 signature의 data는 아래와 같이 구성된다.

```
\x19Klaytn Signed Message:\n + length of message + message
```

\x19Klaytn Signed Message:\n, length of message는 클레이튼 고유의 prefix이다.

signature를 만들 때 `klayDID.sign` 함수를 사용한다면, 위의 prefix를 별도로 입력할 필요없다.

message는 아래와 같은 규칙을 따른다.

```
function name + DIDLedger address + nonce[did] + did  
```
function name은 아래와 같다.
1. klayDID.addPubKeyBySigner이고 type이 `EcdsaSecp256k1RecoveryMethod2020`일 때, function name은 `addAddrKey`이다.
2. klayDID.addPubKeyBySigner이고 type이 `EcdsaSecp256k1VerificationKey2019`일 때, function name은 `addPubKey`이다.
3. klayDID.addServiceBySinger이라면, function name은 `addService`이다.
4. klayDID.disableKey이라면, function name은 `disableKey`이다.
5. klayDID.disableService이라면, function name은 `disableService`이다.

DIDLedger address는 현재 배포되어있는 did registry의 contract 주소이다. (***주의 lowercase***)

nonce[did]는 현재 did registry에 저장되어 있는 did의 nonce 값이다. `klayDID.getNonce`로 얻을 수 있다.

did는 document의 주체를 의미한다.


<br/>

## Submodule Auth

<br/>

### login

```js
 klayDID.auth.login(keyInfo)
```

해당 메소드는 Contract의 `send` opreation을 하기 위하여, 현재 인스턴스한 klaytDID에 in memory 형태로 account와 private key를 설정하는 메소드이다.  

keyinfo는 아래와 같은 object이다. 
1. {path: 'keystorefile.json', password: '1234'}
2. {address: '0xfFEdf..', privateKey: 0xFdfmfkdivcvcv....}

***해당 메서드를 선행하여 실행하지 않으면, `klayDID.addPubKey`, `klayDID.createDocument` 등의 메소드를 사용할 수 없다.***

<br/>
