---
title: 'Especificación Bring Your Own Key: Azure Key Vault | Microsoft Docs'
description: En este documento se describe la especificación Bring Your Own Key.
services: key-vault
author: amitbapat
manager: devtiw
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: conceptual
ms.date: 02/04/2021
ms.author: ambapat
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 1005176576e2c0db8450dda8ea15d7b3d6934c88
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110665112"
---
# <a name="bring-your-own-key-specification"></a>Especificación Bring Your Own Key

En este documento se describen las especificaciones para importar claves protegidas con HSM de HSM locales de los clientes en Key Vault.

## <a name="scenario"></a>Escenario

Un cliente de Key Vault puede transferir de forma segura una clave desde su HSM local fuera de Azure, en la instancia de Azure Key Vault de respaldo de HSM. En general, el proceso de importación de una clave generada fuera de Key Vault se conoce como Bring Your Own Key (BYOK).

A continuación se indican los requisitos:
* La clave que se va a transferir nunca existe fuera de un HSM en formato de texto sin formato.
* Fuera de un HSM, la clave que se va a transferir siempre está protegida por una clave contenida en el HSM de Azure Key Vault.

## <a name="terminology"></a>Terminología

|Nombre de clave|Tipo de clave|Origen|Descripción|
|---|---|---|---|
|Clave de intercambio de claves (KEK)|RSA|HSM de Azure Key Vault|Un par de claves RSA respaldadas por HSM generado en Azure Key Vault.
Clave de encapsulado|AES|HSM del proveedor|Una clave AES [efímera] generada por HSM local.
Clave de destino|RSA, EC, AES (solo HSM administrado)|HSM del proveedor|La clave que se va a transferir al HSM de Azure Key Vault.

**Clave de intercambio de claves**: Una clave respaldada por HSM que el cliente genera en el almacén de claves donde se importará la clave BYOK. Esta KEK debe tener las siguientes propiedades:

* Es una clave RSA-HSM (4096, 3072 o 2048 bits).
* Tendrá el parámetro key_ops fijo (solo ‘import’), que permitirá su uso solo durante BYOK.
* Debe estar en el mismo almacén donde se va a importar la clave de destino.

## <a name="user-steps"></a>Pasos de usuario

Para realizar una transferencia de claves, un usuario debe realizar los pasos siguientes:

1. Generar la KEK.
2. Recuperar la clave pública de la KEK.
3. Usar la herramienta BYOK suministrada por el proveedor de HSM para importar la KEK en el HSM de destino y exportar la clave de destino protegida por la KEK.
4. Importar la clave de destino protegida en Azure Key Vault.

Los clientes utilizan la herramienta BYOK y la documentación suministrada por el proveedor de HSM para completar el paso 3. Genera un Blob de transferencia de claves (un archivo ".byok").


## <a name="hsm-constraints"></a>Restricciones de HSM

Los módulos HSM existentes pueden aplicar restricciones en la clave que administran, entre las que se incluyen:
* Es posible que sea necesario configurar el módulo HSM para permitir la exportación basada en el encapsulado de claves.
* Es posible que sea necesario marcar la clave de destino CKA_EXTRACTABLE para que el módulo HSM permita la exportación controlada.
* En algunos casos, es posible que la KEK y la clave de encapsulado deban marcarse como CKA_TRUSTED. Esto permite su uso para encapsular claves en el módulo HSM.

Por lo general, la configuración del HSM de origen está fuera del ámbito de esta especificación. Microsoft espera que el proveedor de HSM genere la documentación que acompaña a la herramienta BYOK de modo que incluya estos pasos de configuración.

> [!NOTE]
> Los pasos 1, 2 y 4 descritos a continuación se pueden realizar con otras interfaces, como Azure PowerShell y Azure Portal. También se pueden realizar mediante programación con funciones equivalentes en Key Vault SDK.

### <a name="step-1-generate-kek"></a>Paso 1: Generar la KEK

Use el comando **az keyvault key create** para crear KEK con las operaciones de clave establecidas para importar. Anote el identificador de clave, "kid", que devuelve el siguiente comando.

```azurecli
az keyvault key create --kty RSA-HSM --size 4096 --name KEKforBYOK --ops import --vault-name ContosoKeyVaultHSM
```

### <a name="step-2-retrieve-the-public-key-of-the-kek"></a>Paso 2: Recuperar la clave pública de la KEK.

Descargar la parte de la clave pública de la KEK y guardarla en un archivo PEM.

```azurecli
az keyvault key download --name KEKforBYOK --vault-name ContosoKeyVaultHSM --file KEKforBYOK.publickey.pem
```

### <a name="steps-3-generate-key-transfer-blob-using-hsm-vendor-provided-byok-tool"></a>Paso 3: Generar el blob de transferencia de claves con la herramienta de BYOK suministrada por el proveedor de HSM.

El cliente usará la herramienta BYOK suministrada por el proveedor de HSM para crear un blob de transferencia de claves (almacenado como un archivo ".byok"). La clave pública de la KEK (como un archivo .pem) será una de las entradas de esta herramienta.

#### <a name="key-transfer-blob"></a>Blob de transferencia de claves
A largo plazo, Microsoft optaría por usar el mecanismo de CKM_RSA_AES_KEY_WRAP de PKCS#11 para transferir la clave de destino a Azure Key Vault, ya que este mecanismo genera un único blob y, lo que es más importante, la clave AES intermedia se controla mediante los dos HSM y está garantizado que será efímera. Este mecanismo no está actualmente disponible en algunos HSM, pero la combinación de protección de la clave de destino con CKM_AES_KEY_WRAP_PAD mediante una clave AES y la protección de la clave AES con CKM_RSA_PKCS_OAEP genera un blob equivalente.

El texto no cifrado de la clave de destino depende del tipo de clave: 
* Para una clave RSA, la codificación ASN.1 DER de la clave privada [RFC3447] encapsulada en PKCS#8 [RFC5208] 
* Para una clave EC, la codificación ASN.1 DER de la clave privada [RFC5915] encapsulada en PKCS#8 [RFC5208].
* Para una clave de octeto, los bytes sin formato de la clave.

Los bytes de la clave de texto no cifrado se transforman mediante el mecanismo CKM_RSA_AES_KEY_WRAP:
* Se genera una clave AES efímera y se cifra con la clave RSA de encapsulado mediante RSA-OAEP con SHA1.
* La clave de texto no cifrado codificada se cifra con la clave AES mediante el encapsulado de claves AES con relleno.
* La clave AES cifrada y la clave de texto no cifrado cifrada se concatenan para generar el blob de texto cifrado final.

El formato del blob de transferencia usa la serialización compacta de cifrado web JSON (RFC7516) principalmente como un vehículo para entregar los metadatos necesarios al servicio para su correcto descifrado.

Si se utiliza CKM_RSA_AES_KEY_WRAP_PAD, la serialización de JSON del blob de transferencia será:

```json
{
  "schema_version": "1.0.0",
  "header":
  {
    "kid": "<key identifier of the KEK>",
    "alg": "dir",
    "enc": "CKM_RSA_AES_KEY_WRAP"
  },
  "ciphertext":"BASE64URL(<ciphertext contents>)",
  "generator": "BYOK tool name and version; source HSM name and firmware version"
}

```

* kid = identificador de clave de la KEK. Para Key Vault, las claves tienen el siguiente aspecto: https://ContosoKeyVaultHSM.vault.azure.net/keys/mykek/eba63d27e4e34e028839b53fac905621
* alg = algoritmo. 
* dir = modo directo, es decir, el kid de referencia se usa para proteger directamente el texto cifrado, que es una representación precisa de CKM_RSA_AES_KEY_WRAP.
* generator = campo informativo que denota el nombre y la versión de la herramienta BYOK, y el fabricante y el modelo del módulo HSM de origen. Esta información está destinada a la solución de problemas y el soporte técnico.

El blob JSON se almacena en un archivo con una extensión ".byok" para que los clientes de Azure PowerShell o la CLI de Azure lo traten correctamente cuando se usan los comandos "Add-AzKeyVaultKey" (PSH) o "az keyvault key import" (CLI).

### <a name="step-4-upload-key-transfer-blob-to-import-hsm-key"></a>Paso 4: Cargar el blob de transferencia de claves para importar la clave de HSM

El cliente transferirá el blob de transferencia de claves (archivo ".byok") a una estación de trabajo en línea y, a continuación, ejecutará un comando **az keyvault key import** para importar este blob como una nueva clave respaldada por HSM en Key Vault. 

Para importar una clave RSA, use este comando:
```azurecli
az keyvault key import --vault-name ContosoKeyVaultHSM --name ContosoFirstHSMkey --byok-file KeyTransferPackage-ContosoFirstHSMkey.byok --ops encrypt decrypt
```
Para importar una clave EC, debe especificar el tipo de clave y el nombre de la curva.

```azurecli
az keyvault key import --vault-name ContosoKeyVaultHSM --name ContosoFirstHSMkey --byok-file --kty EC-HSM --curve-name "P-256" KeyTransferPackage-ContosoFirstHSMkey.byok --ops sign verify
```


Cuando se ejecuta el comando anterior, se produce el envío de una solicitud de la API de REST de la siguiente manera:

```
PUT https://contosokeyvaulthsm.vault.azure.net/keys/ContosoFirstHSMKey?api-version=7.0
```

Cuerpo de la solicitud al importar una clave RSA:
```json
{
  "key": {
    "kty": "RSA-HSM",
    "key_ops": [
      "decrypt",
      "encrypt"
    ],
    "key_hsm": "<Base64 encoded BYOK_BLOB>"
  },
  "attributes": {
    "enabled": true
  }
}
```

Cuerpo de la solicitud al importar una clave EC:
```json
{
  "key": {
    "kty": "EC-HSM",
    "crv": "P-256",
    "key_ops": [
      "sign",
      "verify"
    ],
    "key_hsm": "<Base64 encoded BYOK_BLOB>"
  },
  "attributes": {
    "enabled": true
  }
}
```

el valor "key_hsm" es el contenido completo del archivo KeyTransferPackage-ContosoFirstHSMkey.byok codificado en formato Base64.

## <a name="references"></a>Referencias
- [Guía del desarrollador de Key Vault](../general/developers-guide.md)

## <a name="next-steps"></a>Pasos siguientes
* Instrucciones de BYOK detalladas: [Importación de claves protegidas con HSM en Key Vault (BYOK)](hsm-protected-keys-byok.md)

