---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 05/28/2021
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 24bbd88c9a31172f0dcf51bebda1d74eb339557d
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110721623"
---
[!INCLUDE [P2S FAQ All](vpn-gateway-faq-p2s-all-include.md)]

### <a name="what-should-i-do-if-im-getting-a-certificate-mismatch-when-connecting-using-certificate-authentication"></a>¿Qué debo hacer si obtengo un error de coincidencia de certificado al realizar la conexión mediante la autenticación de certificado?

Desactive **"Verificar la identidad del servidor validando el certificado"** o **agregue el FQDN del servidor junto con el certificado** al crear un perfil de forma manual. Para ello, ejecute **rasphone** desde un símbolo del sistema y elija el perfil en la lista desplegable.

En general, no se recomienda omitir la validación de la identidad del servidor, pero con la autenticación de certificado de Azure, se usa el mismo certificado para la validación del servidor en el protocolo de túnel VPN (IKEv2/SSTP) y el protocolo EAP. Dado que el certificado de servidor y el FQDN ya están validados por el protocolo de túnel de VPN, se produce una redundancia si se vuelven a validar en EAP.

![Autenticación de punto a sitio](./media/vpn-gateway-faq-p2s-all-include/servercert.png "Certificado de servidor")

### <a name="can-i-use-my-own-internal-pki-root-ca-to-generate-certificates-for-point-to-site-connectivity"></a>¿Puedo usar mi propio CA raíz de PKI interna para generar certificados de conectividad de punto a sitio?

Sí. Anteriormente, solo podían usarse certificados raíz autofirmados. Todavía puede cargar 20 certificados raíz.

### <a name="can-i-use-certificates-from-azure-key-vault"></a>¿Puedo usar certificados de Azure Key Vault?

No.

### <a name="what-tools-can-i-use-to-create-certificates"></a>¿Qué herramientas puedo usar para crear certificados?

Puede usar la solución Enterprise PKI (la PKI interna), Azure PowerShell, MakeCert y OpenSSL.

### <a name="are-there-instructions-for-certificate-settings-and-parameters"></a><a name="certsettings"></a>¿Hay instrucciones para los parámetros y la configuración de certificados?

* **Solución PKI interna/Enterprise PKI:** consulte los pasos de [Generación de certificados](../articles/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#generatecert).

* **Azure PowerShell:** consulte el artículo [Azure PowerShell](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md) para conocer los pasos.

* **MakeCert:** consulte el artículo [MakeCert](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site-makecert.md) para conocer los pasos.

* **OpenSSL:** 

    * Al exportar certificados, asegúrese de convertir el certificado raíz en Base64.

    * Para el certificado de cliente:

      * Al crear la clave privada, especifique la longitud como 4096.
      * Al crear el certificado para el parámetro *-extensions*, especifique *usr_cert*.
