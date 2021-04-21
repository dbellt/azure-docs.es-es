---
title: Creación y combinación de solicitudes de firma de certificado en Azure Key Vault
description: Aprenda a crear solicitudes de firma de certificado en Azure Key Vault.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: tutorial
ms.date: 06/17/2020
ms.author: sebansal
ms.openlocfilehash: a4d079855e5aa05adb84b62d686d9f386608f7bb
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107752145"
---
# <a name="create-and-merge-a-csr-in-key-vault"></a>Creación y combinación de solicitudes de firma de certificado en Key Vault

Azure Key Vault admite el almacenamiento de los certificados digitales emitidos por cualquier entidad de certificación (CA). Admite la creación de una solicitud de firma de certificado (CSR) con un par de claves privada y pública. La solicitud de firma de certificado puede firmarla cualquier entidad de certificación (una entidad de una empresa interna o una entidad pública externa). Una solicitud de firma de certificado es un mensaje que se envía a una entidad de certificación para solicitar un certificado digital.

Para más información general acerca de los certificados, consulte [Certificados de Azure Key Vault](./about-certificates.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="add-certificates-in-key-vault-issued-by-partnered-cas"></a>Incorporación a Key Vault de certificados emitidos por entidades de certificación asociadas

Key Vault se asocia con las dos siguientes entidades de certificación para simplificar la creación de certificados.

|Proveedor|Tipo de certificado|Configuración  
|--------------|----------------------|------------------|  
|DigiCert|Key Vault ofrece certificados SSL OV o EV con DigiCert| [Guía de integración](./how-to-integrate-certificate-authority.md)
|GlobalSign|Key Vault ofrece certificados SSL OV o EV con GlobalSign| [Guía de integración](https://support.globalsign.com/digital-certificates/digital-certificate-installation/generating-and-importing-certificate-microsoft-azure-key-vault)

## <a name="add-certificates-in-key-vault-issued-by-non-partnered-cas"></a>Incorporación a Key Vault de certificados emitidos por entidades de certificación no asociadas

Siga estos pasos para agregar un certificado de entidades de certificación no asociadas con Key Vault (por ejemplo, GoDaddy no es una entidad de certificación de Key Vault de confianza).

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Vaya al almacén de claves al que desea agregar el certificado.
1. En la página de propiedades, seleccione **Certificados**.
1. Seleccione la pestaña **Generar o importar**.
1. En la pantalla **Crear un certificado**, elija los siguientes valores:
    - **Método de creación de certificados**: Generar.
    - **Nombre del certificado**: ContosoManualCSRCertificate.
    - **Tipo de entidad de certificación (CA)** : certificado emitido por una entidad de certificación no integrada.
    - **Asunto**: `"CN=www.contosoHRApp.com"`.
     > [!NOTE]
     > Si utiliza un nombre distintivo relativo (RDN) que tiene una coma (,) en el valor, encapsule el valor que contiene el carácter especial entre comillas dobles. 
     >
     > Entrada de ejemplo en **Asunto**: `DC=Contoso,OU="Docs,Contoso",CN=www.contosoHRApp.com`
     >
     > En este ejemplo, el RDN `OU` contiene un valor con una coma en el nombre. La salida resultante para `OU` es **Docs, Contoso**.
1. Seleccione los restantes valores que desee y, después, seleccione **Crear** para agregar el certificado a la lista **Certificados**.

    ![Captura de pantalla de las propiedades del certificado](../media/certificates/create-csr-merge-csr/create-certificate.png)  

1. En la lista **Certificados**, seleccione el nuevo certificado. El estado actual del certificado es **disabled** (deshabilitado), ya que la entidad de certificación aún no lo ha emitido.
1. En la pestaña **Operación de certificados**, seleccione **Descargar CSR**.

   ![Captura de pantalla que resalta el botón Descargar CSR.](../media/certificates/create-csr-merge-csr/download-csr.png)

1. Pida a la entidad de certificación que firme la solicitud de firma de certificado (.csr).
1. Una vez firmada la solicitud, seleccione **Combinar la solicitud firmada** en la pestaña **Operación de certificados** para agregar el certificado firmado a Key Vault.

La solicitud de certificado ahora se ha combinado correctamente.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Cree una directiva de certificados. Como la CA de este escenario no está asociada, en **IssuerName**  se selecciona **Desconocido** y Key Vault no inscribe ni renueva el certificado.

   ```azure-powershell
   $policy = New-AzKeyVaultCertificatePolicy -SubjectName "CN=www.contosoHRApp.com" -ValidityInMonths 1  -IssuerName Unknown
   ```
     > [!NOTE]
     > Si utiliza un nombre distintivo relativo (RDN) que tiene una coma (,) en el valor, use comillas simples para todo el valor o el conjunto de valores y encapsule el valor que contiene el carácter especial entre comillas dobles. 
     >
     >Entrada de ejemplo de **SubjectName**: `$policy = New-AzKeyVaultCertificatePolicy -SubjectName 'OU="Docs,Contoso",DC=Contoso,CN=www.contosoHRApp.com' -ValidityInMonths 1  -IssuerName Unknown`. En este ejemplo, el valor `OU` se lee como **Docs, Contoso**. Este formato funciona con todos los valores que contienen una coma.
     > 
     > En este ejemplo, el RDN `OU` contiene un valor con una coma en el nombre. La salida resultante para `OU` es **Docs, Contoso**.

1. Cree la solicitud de firma de certificado.

   ```azure-powershell
   $csr = Add-AzKeyVaultCertificate -VaultName ContosoKV -Name ContosoManualCSRCertificate -CertificatePolicy $policy
   $csr.CertificateSigningRequest
   ```

1. Pida a la entidad de certificación que firme la solicitud de firma de certificado. `$csr.CertificateSigningRequest` es la solicitud de firma de certificado codificada base para el certificado. Este blob se puede volcar en el sitio web de solicitud de certificados del emisor. Este paso varía de una entidad a otra. Consulte las instrucciones de la entidad de certificación sobre cómo ejecutar este paso. También puede usar herramientas como certreq u openssl para obtener la solicitud de certificado firmada y completar el proceso de generación de un certificado.

1. Combine la solicitud firmada en Key Vault. Una vez firmada la solicitud de certificado, puede combinarla con el par de claves pública y privada inicial creado en Azure Key Vault.

    ```azure-powershell-interactive
    Import-AzKeyVaultCertificate -VaultName ContosoKV -Name ContosoManualCSRCertificate -FilePath C:\test\OutputCertificateFile.cer
    ```

La solicitud de certificado ahora se ha combinado correctamente.

---

## <a name="add-more-information-to-the-csr"></a>Incorporación de más información a la solicitud de certificado firmada

Si desea agregar más información al crear la solicitud de certificado firmada, defínala en **SubjectName**. Puede agregar información como:
- País
- Ciudad o situación
- Estado o provincia
- Organización
- Unidad organizativa

Ejemplo

   ```azure-powershell
   SubjectName="CN = docs.microsoft.com, OU = Microsoft Corporation, O = Microsoft Corporation, L = Redmond, S = WA, C = US"
   ```

> [!NOTE]
> Si solicita un certificado de validación del dominio (DV) con información adicional, es posible que la entidad de certificación rechace la solicitud si no puede validar toda la información de la solicitud. Es posible que sea más adecuado agregar información si se solicita un certificado de validación de la organización (OV).

## <a name="faqs"></a>Preguntas más frecuentes

- ¿Cómo se supervisa o administra una CSR?

     Consulte [Supervisión y administración de la creación de certificados](./create-certificate-scenarios.md).

- ¿Qué ocurre si veo un **error del tipo "La clave pública del certificado de entidad final en el contenido del certificado X.509 especificado no coincide con la parte pública de la clave privada especificada. Compruebe si el certificado es válido"** ?

     Este error se produce si no combina la CSR firmada con la solicitud de CSR que ha iniciado. Todas las CSR nuevas que cree tienen una clave privada, que debe coincidir al combinar la solicitud firmada.

- Cuando se combina una CSR, ¿se combinará toda la cadena?

     Sí, se combinará toda la cadena, siempre que el usuario haya recuperado el archivo .p7b que va a combinar.

- ¿Qué ocurre si el certificado emitido está en estado deshabilitado en Azure Portal?

     En la pestaña **Operación de certificados** puede ver el mensaje de error de ese certificado.

- ¿Qué ocurre si veo un **error del tipo "El nombre de sujeto proporcionado no es un nombre de X500 válido**?

     Este error puede producirse si **SubjectName** incluye algún carácter especial. Consulte las notas de Azure Portal y las instrucciones de PowerShell.

---

## <a name="next-steps"></a>Pasos siguientes

- [Autenticación, solicitudes y respuestas](../general/authentication-requests-and-responses.md)
- [Guía del desarrollador de Key Vault](../general/developers-guide.md)
- [Referencia de la API REST de Azure Key Vault](/rest/api/keyvault)
- [Almacenes: crear o actualizar](/rest/api/keyvault/vaults/createorupdate)
- [Almacenes: actualizar directiva de acceso](/rest/api/keyvault/vaults/updateaccesspolicy)