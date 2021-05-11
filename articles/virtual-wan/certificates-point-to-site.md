---
title: Generación y exportación de certificados para conexiones VPN de usuario | Azure Virtual WAN
description: Obtenga información sobre cómo crear un certificado raíz autofirmado, exportar una clave pública y generar certificados de cliente para conexiones de VPN de usuario (punto a sitio) de Virtual WAN mediante PowerShell.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 04/27/2021
ms.author: cherylmc
ms.openlocfilehash: d126f68ed1eebae9dda7ad957748cb3258b5ad89
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108165306"
---
# <a name="generate-and-export-certificates-for-user-vpn-connections"></a>Generación y exportación de certificados para conexiones VPN de usuario

Las conexiones de VPN de usuario (punto a sitio) utilizan certificados para realizar la autenticación. En este artículo, se muestra cómo crear un certificado raíz autofirmado y generar certificados de cliente usando PowerShell en Windows 10 o Windows Server 2016.

Debe realizar los pasos de este artículo en un equipo que ejecute Windows 10 o Windows Server 2016. Los cmdlets de PowerShell que se usan para generar certificados forman parte del sistema operativo y no funcionan en otras versiones de Windows. El equipo con Windows 10 o Windows Server 2016 solo es necesario para generar los certificados. Una vez que se generan los certificados, puede cargarlos o instalarlos en cualquier sistema operativo cliente compatible.

[!INCLUDE [Export public key](../../includes/vpn-gateway-generate-export-certificates-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Continúe con los [Pasos de Virtual WAN para la conexión VPN de usuario](virtual-wan-about.md)
