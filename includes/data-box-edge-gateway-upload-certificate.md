---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 06/26/2019
ms.author: alkohli
ms.openlocfilehash: 09d9b5bbf3f9ca7a4eef37891d03c9c865e7f74b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67448626"
---
Un certificado SSL correcto garantiza que envía la información cifrada al servidor correcto. Además del cifrado, el certificado también permite la autenticación. Puede cargar su propio certificado SSL de confianza mediante la interfaz de PowerShell del dispositivo.

1. [Conéctese a la interfaz de PowerShell](#connect-to-the-powershell-interface).
2. Use el cmdlet `Set-HcsCertificate` para cargar el certificado. Cuando se le solicite, proporcione los siguientes parámetros:

   - `CertificateFilePath`: ruta de acceso al recurso compartido que contiene el archivo de certificado en formato *.pfx*.
   - `CertificatePassword`: contraseña usada para proteger el certificado.
   - `Credentials`: nombre de usuario para obtener acceso al recurso compartido que contiene el certificado. Proporcione la contraseña para el recurso compartido de red cuando se le solicite.

     En el ejemplo siguiente se muestra el uso de este cmdlet:

     ```
     Set-HcsCertificate -Scope LocalWebUI -CertificateFilePath "\\myfileshare\certificates\mycert.pfx" -CertificatePassword "mypassword" -Credential "Username"
     ```

