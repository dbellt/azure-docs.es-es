---
author: billmath
ms.service: active-directory
ms.subservice: cloud-provisioning
ms.topic: include
ms.date: 10/16/2019
ms.author: billmath
ms.openlocfilehash: 6d95e40623f17a39145778a2fc067dccc68fd872
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "95556190"
---
## <a name="steps-to-enable-single-sign-on"></a>Pasos para habilitar el inicio de sesión único
El aprovisionamiento en la nube funciona con el inicio de sesión único.  Actualmente no hay ninguna opción para habilitar el inicio de sesión único (SSO) cuando se instala el agente; sin embargo, puede usar los pasos siguientes para habilitar el inicio de sesión único y usarlo. 

### <a name="step-1-download-and-extract-azure-ad-connect-files"></a>Paso 1: Descarga y extracción de archivos de Azure AD Connect
1.  En primer lugar, descargue la versión más reciente de [Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594).
2.  Abra un símbolo del sistema con privilegios de administración y vaya al msi que acaba de descargar.
3.  Ejecute lo siguiente: `msiexec /a C:\filepath\AzureADConnect.msi /qb TARGETDIR=C:\filepath\extractfolder`.
4. Cambie filepath y extractfolder para que coincidan con la ruta de acceso del archivo y el nombre de la carpeta de extracción.  El contenido debe estar ahora en la carpeta de extracción.

### <a name="step-2-import-the-seamless-sso-powershell-module"></a>Paso 2: Importación del módulo de PowerShell de SSO de conexión directa

1. Descargue e instale [Azure AD PowerShell](/powershell/azure/active-directory/overview).
2. Examine la carpeta `%programfiles%\Microsoft Azure Active Directory Connect`.
3. Importe el módulo de PowerShell de SSO de conexión directa mediante este comando: `Import-Module .\AzureADSSO.psd1`.

### <a name="step-3-get-the-list-of-active-directory-forests-on-which-seamless-sso-has-been-enabled"></a>Paso 3: Obtención de la lista de bosques de Active Directory en la que se habilitó el SSO de conexión directa

1. Ejecute PowerShell como administrador. En PowerShell, llame a `New-AzureADSSOAuthenticationContext`. Cuando se le solicite, escriba las credenciales de administrador global de su inquilino.
2. Llame a `Get-AzureADSSOStatus`. Este comando proporciona la lista de bosques de Active Directory (examine la lista "Dominios") en la que se ha habilitado esta característica.

### <a name="step-4-enable-seamless-sso-for-each-active-directory-forest"></a>Paso 4: Habilitación de SSO de conexión directa para cada bosque de Active Directory

1. Llame a `Enable-AzureADSSOForest`. Cuando se le pida, escriba las credenciales del administrador de dominio para el bosque de Active Directory deseado.

   > [!NOTE]
   >El nombre de usuario de las credenciales de administrador de dominio debe especificarse con el formato de nombre de cuenta SAM (contoso\johndoe o contoso.com\johndoe). La parte de dominio del nombre de usuario se usa para localizar el controlador de dominio del administrador de dominio con DNS.

   >[!NOTE]
   >La cuenta de administrador de dominio usada no puede ser miembro del grupo Usuarios protegidos. De lo contrario, la operación presentará un error.

2. Repita los pasos anteriores para cada bosque de Active Directory en el que desea configurar la característica.

### <a name="step-5-enable-the-feature-on-your-tenant"></a>Paso 5. Habilite la característica en su inquilino

Para activar la característica en su inquilino, llame a `Enable-AzureADSSO -Enable $true`.