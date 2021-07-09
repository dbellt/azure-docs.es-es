---
title: Integración de Key Vault con la entidad de certificación DigiCert
description: En este artículo se describe cómo integrar Key Vault con la entidad de certificación DigiCert para que pueda aprovisionar, administrar e implementar certificados para la red.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: how-to
ms.date: 06/02/2020
ms.author: sebansal
ms.openlocfilehash: 99be94db2dd904db09b8b265b74442d5ba31b629
ms.sourcegitcommit: 42ac9d148cc3e9a1c0d771bc5eea632d8c70b92a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2021
ms.locfileid: "109845607"
---
# <a name="integrating-key-vault-with-digicert-certificate-authority"></a>Integración de Key Vault con la entidad de certificación DigiCert

Azure Key Vault permite aprovisionar, administrar e implementar fácilmente certificados digitales para una red y habilitar las comunicaciones seguras para las aplicaciones. Un certificado digital es una credencial electrónica que establece la prueba de identidad en una transacción electrónica. 

Los usuarios de Azure Key Vault pueden generar certificados DigiCert directamente desde sus almacenes de claves. Key Vault una asociación de confianza con la entidad de certificación DigiCert. Esta asociación garantiza la administración del ciclo de vida de los certificados de un extremo a otro para los certificados emitidos por DigiCert.

Para más información general acerca de los certificados, consulte [Certificados de Azure Key Vault](./about-certificates.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar los procedimientos de este artículo, necesitará tener:
* Un almacén de claves. Puede usar un almacén de claves existente o crear uno completando los pasos de uno de estos artículos de inicio rápido:
   - [Creación de un almacén de claves mediante la CLI de Azure](../general/quick-create-cli.md)
   - [Creación de un almacén de claves mediante Azure PowerShell](../general/quick-create-powershell.md)
   - [Creación de un almacén de claves mediante Azure Portal](../general/quick-create-portal.md)
*   Una cuenta CertCentral de DigiCert activada. [Regístrese](https://www.digicert.com/account/signup/) para la cuenta CertCentral.
*   Permisos de nivel de administrador en sus cuentas.


### <a name="before-you-begin"></a>Antes de empezar

Asegúrese de disponer de la siguiente información de la cuenta CertCentral de DigiCert:
-   Id. de la cuenta CertCentral
-   Identificador de la organización
-   Clave de API

## <a name="add-the-certificate-authority-in-key-vault"></a>Incorporación de una entidad de certificación en Key Vault 
Después de recopilar la información anterior de la cuenta CertCentral de DigiCert, puede agregar DigiCert a la lista de entidades de certificación del almacén de claves.

### <a name="azure-portal"></a>Portal de Azure

1.  Para agregar la entidad de certificación DigiCert, vaya al almacén de claves en el que quiere agregar DigiCert. 
2.  En la página de propiedades de Key Vault, seleccione **Certificados**.
3.  Seleccione la pestaña **Entidades de certificación**: :::image type="content" source="../media/certificates/how-to-integrate-certificate-authority/select-certificate-authorities.png" alt-text="Captura de pantalla que muestra la selección de la pestaña Entidades de certificación.":::
4.  Seleccione **Agregar**: :::image type="content" source="../media/certificates/how-to-integrate-certificate-authority/add-certificate-authority.png" alt-text="Captura de pantalla que muestra el botón Agregar en la pestaña Entidades de certificación.":::
5.  En **Creación de una entidad de certificación**, escriba estos valores:
    -   **Nombre**: un nombre de emisor identificable. Por ejemplo, **DigiCertCA**.
    -   **Proveedor**: **DigiCert**.
    -   **Id. de cuenta**: id. de la cuenta CertCentral de DigiCert.
    -   **Contraseña de cuenta**: escriba la clave de API que generó en la cuenta CertCentral de DigiCert.
    -   **Id. de organización**: el id. de organización de la cuenta CertCentral de DigiCert. 

1. Seleccione **Crear**.
   
DigicertCA ahora está en la lista de entidades de certificación.


### <a name="azure-powershell"></a>Azure PowerShell

Puede usar Azure PowerShell para crear y administrar recursos de Azure mediante comandos o scripts. En Azure se hospeda Azure Cloud Shell, un entorno de shell interactivo que puede usar mediante Azure Portal en un explorador.

Si decide instalar y usar PowerShell localmente, necesitará la versión 1.0.0 o posterior del módulo de Azure AZ PowerShell para completar los procedimientos. Escriba `$PSVersionTable.PSVersion` para determinar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure AZ PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Login-AzAccount` para crear una conexión con Azure:

```azurepowershell-interactive
Login-AzAccount
```

1.  Cree un grupo de recursos de Azure mediante [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. 

    ```azurepowershell-interactive
    New-AzResourceGroup -Name ContosoResourceGroup -Location EastUS
    ```

2. Cree un almacén de claves que tenga un nombre único. Aquí, `Contoso-Vaultname` es el nombre del almacén de claves.

   - **Nombre del almacén**: `Contoso-Vaultname`
   - **Nombre del grupo de recursos**: `ContosoResourceGroup`
   - **Ubicación**: `EastUS`.

    ```azurepowershell-interactive
    New-AzKeyVault -Name 'Contoso-Vaultname' -ResourceGroupName 'ContosoResourceGroup' -Location 'EastUS'
   ```

3. Defina variables para los siguientes valores de la cuenta CertCentral de DigiCert:

   - **Id. de cuenta** 
   - **Identificador de la organización** 
   - **Clave de API** 

   ```azurepowershell-interactive
   $accountId = "myDigiCertCertCentralAccountID"
   $org = New-AzKeyVaultCertificateOrganizationDetail -Id OrganizationIDfromDigiCertAccount
   $secureApiKey = ConvertTo-SecureString DigiCertCertCentralAPIKey -AsPlainText –Force
   ```

4. Establezca el emisor. Al hacerlo agregará DigiCert como entidad de certificación en el almacén de claves. [Más información sobre los parámetros.](/powershell/module/az.keyvault/Set-AzKeyVaultCertificateIssuer)
   ```azurepowershell-interactive
   Set-AzKeyVaultCertificateIssuer -VaultName "Contoso-Vaultname" -Name "TestIssuer01" -IssuerProvider DigiCert -AccountId $accountId -ApiKey $secureApiKey -OrganizationDetails $org -PassThru
   ```

5. Establezca la directiva para el certificado y emisión del certificado desde DigiCert directamente a Key Vault:

   ```azurepowershell-interactive
   $Policy = New-AzKeyVaultCertificatePolicy -SecretContentType "application/x-pkcs12" -SubjectName "CN=contoso.com" -IssuerName "TestIssuer01" -ValidityInMonths 12 -RenewAtNumberOfDaysBeforeExpiry 60
   Add-AzKeyVaultCertificate -VaultName "Contoso-Vaultname" -Name "ExampleCertificate" -CertificatePolicy $Policy
   ```

La entidad de certificación DigiCert emite ahora el certificado al almacén de claves especificado.

## <a name="troubleshoot"></a>Solución de problemas

Si el certificado emitido se encuentra en el estado deshabilitado en Azure Portal, vea la operación de certificados para revisar el mensaje de error de DigiCert para el certificado:

:::image type="content" source="../media/certificates/how-to-integrate-certificate-authority/certificate-operation-select.png" alt-text="Captura de pantalla que muestra la pestaña Operación de certificados.":::

Mensaje de error "Realice una operación de combinación para completar esta solicitud de certificado".
   
Combine la CSR firmada por la entidad de certificación para completar la solicitud. Para información sobre la combinación de una CSR, consulte [Creación y combinación de una CSR](./create-certificate-signing-request.md).

Para más información, consulte [Operaciones de certificados en la referencia de la API de REST de Key Vault](/rest/api/keyvault). Para obtener información sobre cómo establecer permisos, vea [Almacenes: creación o actualización](/rest/api/keyvault/vaults/createorupdate) y [Almacenes: actualización de la directiva de acceso](/rest/api/keyvault/vaults/updateaccesspolicy).

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

- **¿Puedo generar un certificado comodín de DigiCert mediante KeyVault?** 
   
  Sí, aunque depende de cómo haya configurado la cuenta de DigiCert.
- **¿Cómo puedo crear un certificado OV SSL o EV SSL con DigiCert?**
 
   Key Vault admite la creación de certificados OV SSL y EV SSL. Al crear un certificado, seleccione **Configuración de directiva avanzada** y, a continuación, especifique el tipo de certificado. Valores admitidos: OV SSL, EV SSL
   
   Puede crear este tipo de certificado en Key Vault si la cuenta de DigiCert lo permite. Para este tipo de certificado, DigiCert realiza la validación. Si se produce un error en la validación, el equipo de soporte técnico de DigiCert puede ayudarle. Puede agregar información al crear un certificado definiendo la información en `subjectName`.

  Por ejemplo,     `SubjectName="CN = docs.microsoft.com, OU = Microsoft Corporation, O = Microsoft Corporation, L = Redmond, S = WA, C = US"`.
   
- **¿Se tarda más tiempo en crear un certificado de DigiCert a través de la integración que en adquirirlo directamente de DigiCert?**
   
   No. Al crear un certificado, el proceso de comprobación puede tardar tiempo. DigiCert controla ese proceso.


## <a name="next-steps"></a>Pasos siguientes

- [Autenticación, solicitudes y respuestas](../general/authentication-requests-and-responses.md)
- [Guía del desarrollador de Key Vault](../general/developers-guide.md)