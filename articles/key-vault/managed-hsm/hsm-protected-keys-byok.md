---
title: 'Generación y transferencia de claves protegidas con HSM para HSM administrado de Azure Key Vault: Azure Key Vault | Microsoft Docs'
description: Use este artículo para ayudarle a planear, generar y transferir sus propias claves protegidas con HSM para usar con HSM administrado. También se denomina BYOK o "Bring Your Own Key".
services: key-vault
author: amitbapat
tags: azure-resource-manager
ms.service: key-vault
ms.topic: conceptual
ms.date: 02/04/2021
ms.author: ambapat
ms.openlocfilehash: cc9037db3289d7fb3287a8994a8ff6a68fc0583a
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107790588"
---
# <a name="import-hsm-protected-keys-to-managed-hsm-byok"></a>Importación de claves protegidas con HSM en HSM administrado (BYOK)

 HSM administrado de Azure Key Vault admite la importación de claves generadas en su módulo de seguridad de hardware (HSM) local; las claves nunca dejarán el límite de protección de HSM. Con frecuencia este escenario se conoce como *Bring Your Own Key* (BYOK). En HSM administrado se usan los adaptadores Marvell LiquidSecurity HSM (validados como FIPS 140-2 de nivel 3) para proteger las claves.

Use la información de este artículo como ayuda para planear, generar y transferir sus propias claves protegidas con HSM para utilizarlas con HSM administrado.

> [!NOTE]
> Esta funcionalidad no está disponible para Azure China 21Vianet. Este método de importación solo está disponible para [HSM compatibles](#supported-hsms). 

Para obtener más información y ver un tutorial con el que empezar a trabajar con HSM administrado, consulte [¿Qué es HSM administrado?](overview.md).

## <a name="overview"></a>Información general

A continuación se muestra información general del proceso: Los pasos específicos que se deben realizar se describen más adelante en el artículo.

* En HSM administrado, genere una clave (conocida como *clave de intercambio de claves* o [KEK]). La KEK debe ser una clave RSA-HSM que tenga solo la operación de clave `import`. 
* Descargue la clave pública KEK como un archivo .pem.
* Transfiera la clave pública KEK a un equipo sin conexión que esté conectado a un HSM local.
* En la estación de trabajo sin conexión, use la herramienta BYOK que proporciona su proveedor de HSM para crear un archivo BYOK. 
* La clave de destino se cifra con una KEK, que permanece cifrada hasta que se transfiere al HSM administrado. Solo la versión cifrada de la clave sale del HSM local.
* Una KEK generada dentro de un HSM administrado no es exportable. Los HSM aplican la regla de que no existe ninguna versión clara de una KEK fuera de un HSM administrado.
* La KEK debe estar en el mismo HSM administrado al que se va a importar la clave de destino.
* Cuando el archivo BYOK se carga en HSM administrado, un HSM administrado usa la clave privada KEK para descifrar el material de la clave de destino e importarlo como una clave de HSM. Esta operación tiene lugar enteramente dentro del HSM. La clave de destino siempre permanece en el límite de la protección de HSM.


## <a name="prerequisites"></a>Requisitos previos

Para usar los comandos de la CLI de Azure, debe tener los siguientes elementos:

* Una suscripción a Microsoft Azure. Si no tiene una, puede registrarse para una [evaluación gratuita](https://azure.microsoft.com/pricing/free-trial).
* La CLI de Azure, versión 2.12.0 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).
* Un HSM administrado de la [lista de HSM admitidos](#supported-hsms) en su suscripción. Consulte [Quickstart: Aprovisionamiento y activación de un HSM administrado mediante la CLI de Azure](quick-create-cli.md) para aprovisionar y activar un HSM administrado.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Para iniciar sesión en Azure mediante la CLI puede escribir:

```azurecli
az login
```

Para más información sobre las opciones de inicio de sesión mediante la CLI, consulte [Inicio de sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli)

## <a name="supported-hsms"></a>HSM compatibles

|Nombre del proveedor|Tipo de proveedor|Modelos de HSM compatibles|Más información|
|---|---|---|---|
|nCipher|Fabricante,<br/>HSM como servicio|<ul><li>Familia nShield de HSM</li><li>nShield como servicio</ul>|[Nueva herramienta BYOK y documentación de nCipher](https://www.ncipher.com/products/key-management/cloud-microsoft-azure)|
|Thales|Fabricante|<ul><li>Familia Luna HSM 7 con la versión del firmware 7.3 o posterior</li></ul>| [Herramienta y documentación de Luna BYOK](https://supportportal.thalesgroup.com/csm?id=kb_article_view&sys_kb_id=3892db6ddb8fc45005c9143b0b961987&sysparm_article=KB0021016)|
|Fortanix|Fabricante,<br/>HSM como servicio|<ul><li>Self-Defending Key Management Service (SDKMS)</li><li>Equinix SmartKey</li></ul>|[Exportación de claves de SDKMS a proveedores de nube para BYOK: Azure Key Vault](https://support.fortanix.com/hc/en-us/articles/360040071192-Exporting-SDKMS-keys-to-Cloud-Providers-for-BYOK-Azure-Key-Vault)|
|Marvell|Fabricante|Todos los HSM de LiquidSecurity con<ul><li>versión de firmware 2.0.4 o posterior</li><li>versión de firmware 3.2 o posterior</li></ul>|[Herramienta BYOK y documentación de Marvell](https://www.marvell.com/products/security-solutions/nitrox-hs-adapters/exporting-marvell-hsm-keys-to-cloud-azure-key-vault.html)|
|Cryptomathic|ISV (sistema de administración de claves empresariales)|Varias marcas y modelos de HSM, como<ul><li>nCipher</li><li>Thales</li><li>Utimaco</li></ul>Consulte el [sitio de Cryptomathic para obtener más información](https://www.cryptomathic.com/azurebyok).|[Herramienta BYOK y documentación de Cryptomathic](https://www.cryptomathic.com/azurebyok)|
|Securosys SA|Fabricante, HSM como servicio|Familia Primus HSM, Securosys Clouds HSM|[Herramienta y documentación de BYOK de Primus](https://www.securosys.com/primus-azure-byok)|
|StorMagic|ISV (sistema de administración de claves empresariales)|Varias marcas y modelos de HSM, como<ul><li>Utimaco</li><li>Thales</li><li>nCipher</li></ul>Consulte el [sitio web de StorMagic para más información](https://stormagic.com/doc/svkms/Content/Integrations/Azure_KeyVault_BYOK.htm).|[BYOK de SvKMS y Azure Key Vault](https://stormagic.com/doc/svkms/Content/Integrations/Azure_KeyVault_BYOK.htm)|
|IBM|Fabricante|IBM 476x, CryptoExpress|[IBM Enterprise Key Management Foundation](https://www.ibm.com/security/key-management/ekmf-bring-your-own-key-azure)|
|Utimaco|Fabricante,<br/>HSM como servicio|u.trust Anchor, CryptoServer|[Guía de integración y herramienta BYOK de Utimaco](https://support.hsm.utimaco.com/support/downloads/byok)|
||||


## <a name="supported-key-types"></a>Tipos de clave admitidos

|Nombre de clave|Tipo de clave|Tamaño o curva de la clave|Origen|Descripción|
|---|---|---|---|---|
|Clave de intercambio de claves (KEK)|RSA| 2048 bits<br />3072 bits<br />4096 bits|HSM administrado|Un par de claves RSA respaldadas por HSM generado en HSM administrado|
|Clave de destino|
||RSA|2048 bits<br />3072 bits<br />4096 bits|HSM del proveedor|La clave que se va a transferir al HSM administrado|
||EC|P-256<br />P-384<br />P-521|HSM del proveedor|La clave que se va a transferir al HSM administrado|
||Clave simétrica (oct-HSM)|128 bits<br />192 bits<br />256 bits|HSM del proveedor|La clave que se va a transferir al HSM administrado|
||||
## <a name="generate-and-transfer-your-key-to-the-managed-hsm"></a>Generación y transferencia de la clave al HSM administrado

Para generar y transferir la clave a un HSM administrado:

* [Paso 1: Generación de una KEK](#step-1-generate-a-kek)
* [Paso 2: Descarga de la clave pública KEK](#step-2-download-the-kek-public-key)
* [Paso 3: Generación y preparación de la clave para la transferencia](#step-3-generate-and-prepare-your-key-for-transfer)
* [Paso 4: Transferencia de la clave a HSM administrado](#step-4-transfer-your-key-to-managed-hsm)

### <a name="step-1-generate-a-kek"></a>Paso 1: Generación de una KEK

Una KEK es una clave RSA que se genera en un HSM administrado. La KEK se usa para cifrar la clave que quiere importar (la clave de *destino*).

La KEK debe:
- Ser una clave RSA-HSM (2048 bits; 3072 bits; o 4096 bits)
- Generarse en el mismo HSM administrado al que quiere importar la clave de destino
- Crearse con las operaciones de clave permitidas establecidas en `import`

> [!NOTE]
> La KEK debe tener "import" como la única operación de clave permitida. "import" es mutuamente excluyente con todas las demás operaciones de clave.

Use el comando [az keyvault key create](/cli/azure/keyvault/key#az_keyvault_key_create) para crear una KEK que tenga operaciones de clave establecidas en `import`. Registre el identificador de clave (`kid`) que se devuelve con el siguiente comando (usará el valor `kid` en el [paso 3](#step-3-generate-and-prepare-your-key-for-transfer)).

```azurecli-interactive
az keyvault key create --kty RSA-HSM --size 4096 --name KEKforBYOK --ops import --hsm-name ContosoKeyVaultHSM
```
---


### <a name="step-2-download-the-kek-public-key"></a>Paso 2: Descarga de la clave pública KEK

Use [az keyvault key download](/cli/azure/keyvault/key#az_keyvault_key_download) para descargar la clave pública KEK en un archivo .pem. La clave de destino que importa se cifra mediante la clave pública KEK.

```azurecli-interactive
az keyvault key download --name KEKforBYOK --hsm-name ContosoKeyVaultHSM --file KEKforBYOK.publickey.pem
```
---

Transfiera el archivo KEKforBYOK.publickey.pem al equipo sin conexión, ya que lo necesitará en el paso siguiente.

### <a name="step-3-generate-and-prepare-your-key-for-transfer"></a>Paso 3: Generación y preparación de la clave para la transferencia

Consulte la documentación del proveedor del HSM para descargar e instalar la herramienta BYOK. Siga las instrucciones del proveedor de HSM para generar una clave de destino y, luego, cree un paquete de transferencia de claves (un archivo BYOK). La herramienta BYOK usará el valor de `kid` del [paso 1](#step-1-generate-a-kek) y el archivo KEKforBYOK.publickey.pem que descargó en el [paso 2](#step-2-download-the-kek-public-key) para generar una clave de destino cifrada en un archivo BYOK.

Transfiera el archivo BYOK al equipo conectado.

> [!NOTE] 
> No se admite la importación de claves RSA de 1024 bits. Actualmente no se admite la importación de una clave de curva elíptica (EC).
>
> **Problema conocido**: La importación de una clave de destino RSA 4K desde los HSM Luna solo se admite con el firmware 7.4.0 o una versión más reciente.

### <a name="step-4-transfer-your-key-to-managed-hsm"></a>Paso 4: Transferencia de la clave a HSM administrado

Para completar la importación de claves, transfiera el paquete de transferencia de claves (un archivo BYOK) del equipo desconectado al equipo conectado a Internet. Use el comando [az keyvault key import](/cli/azure/keyvault/key#az_keyvault_key_import) para cargar el archivo BYOK en el HMS administrado.

```azurecli-interactive
az keyvault key import --hsm-name ContosoKeyVaultHSM --name ContosoFirstHSMkey --byok-file KeyTransferPackage-ContosoFirstHSMkey.byok
```

Si la carga se realiza correctamente, la CLI de Azure muestra las propiedades de la clave importada.

## <a name="next-steps"></a>Pasos siguientes

Ahora puede usar esta clave protegida con HSM en el HSM administrado. Para más información, vea esta [comparativa de precios y características](https://azure.microsoft.com/pricing/details/key-vault/).
