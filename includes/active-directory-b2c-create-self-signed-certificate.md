---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 01/27/2021
ms.author: mimart
ms.openlocfilehash: 32a47a34d9a4c8562dd1cec44f5a2ec519cb1a7f
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107984424"
---
Si aún no tiene un certificado, puede usar un certificado autofirmado para este tutorial. Un certificado autofirmado es un certificado de seguridad que no está firmado por una entidad de certificación (CA) y no proporciona las garantías de seguridad de un certificado firmado por una CA. 

# <a name="windows"></a>[Windows](#tab/windows)

En Windows, use el cmdlet [New-SelfSignedCertificate](/powershell/module/pki/new-selfsignedcertificate) de PowerShell para generar un certificado.

1. Ejecute este comando de PowerShell para generar un certificado autofirmado. Modifique el argumento `-Subject` según corresponda para su aplicación y el nombre del inquilino de Azure AD B2C. También puede ajustar la fecha de `-NotAfter` para especificar una expiración diferente para el certificado.

    ```PowerShell
    New-SelfSignedCertificate `
        -KeyExportPolicy Exportable `
        -Subject "CN=yourappname.yourtenant.onmicrosoft.com" `
        -KeyAlgorithm RSA `
        -KeyLength 2048 `
        -KeyUsage DigitalSignature `
        -NotAfter (Get-Date).AddMonths(12) `
        -CertStoreLocation "Cert:\CurrentUser\My"
    ```

1. Abra **Administrar certificados de usuario** > **Usuario actual** > **Personal** > **Certificados** > *yourappname.yourtenant.onmicrosoft.com*.
1. Seleccione el certificado y, a continuación, seleccione **Acción** > **Todas las tareas** > **Exportar**.
1. Seleccione **Sí** > **Siguiente** > **Sí, exportar la clave privada** > **Siguiente**.
1. Acepte los valores predeterminados para **Formato de archivo de exportación**.
1. Proporcionar una contraseña para el certificado.

Para que Azure AD B2C acepte la contraseña del archivo .pfx, esta debe estar cifrada con la opción TripleDES-SHA1 de la utilidad de exportación del almacén de certificados de Windows en lugar de AES256-SHA256.

# <a name="macos"></a>[macOS](#tab/macos)

En macOS, use el [Asistente para Certificados](https://support.apple.com/guide/keychain-access/aside/glosa3ed0609/11.0/mac/11.0) de Acceso a llaveros para generar un certificado.

1. Siga las instrucciones que encontrará en [Creación de certificados autofirmados en Acceso a Llaveros en el equipo Mac](https://support.apple.com/guide/keychain-access/kyca8916/mac).
1. En la aplicación Acceso a llaveros del equipo Mac, seleccione el certificado que ha creado.
1. Elija **Archivo** > **Exportar elementos**.
1. Seleccione un nombre de archivo para guardar el certificado. Por ejemplo, **self-signed-certificate.p12**.
1. En **Formato de archivo**, seleccione **Personal Information Exchange (.p12)** .
1. Seleccione **Guardar**.
1. Escriba una **Contraseña** y luego seleccione **Comprobar**.
1. Cambie la extensión del archivo a `.pfx`. Por ejemplo, **self-signed-certificate.pfx**.

---
