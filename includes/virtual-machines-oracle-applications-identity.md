---
author: dlepow
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 07/10/2019
ms.author: danlep
ms.openlocfilehash: 2b28cc742bb981e0db535d39cb0c7fbf027470ea
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "83673771"
---
### <a name="identity-tier"></a>Nivel de identidad 

La asociación entre Microsoft y Oracle le permite configurar una identidad unificada entre Azure, OCI y la aplicación de Oracle. Para el conjunto de aplicaciones de PeopleSoft o JD Edwards EnterpriseOne, se necesita una instancia del servidor HTTP de Oracle (OHS) para configurar el inicio de sesión único entre Azure AD y Oracle IDCS.

OHS actúa como un proxy inverso a la capa de aplicación, lo que significa que todas las solicitudes a las aplicaciones finales pasan por él. Oracle Access Manager WebGate es un complemento del servidor web de OHS que intercepta cada solicitud que va a una aplicación final. Si un recurso al que se está accediendo está protegido (requiere una sesión autenticada), WebGate inicia el flujo de autenticación de OIDC con Identity Cloud Service a través del explorador del usuario. Para más información sobre los flujos compatibles con WebGate de OpenID Connect, consulte la [documentación de Oracle Access Manager](https://docs.oracle.com/cd/E52734_01/oam/AIAAG/GUID-1E927D1B-FB83-425B-8768-85DB441821A4.htm#AIAAG7327).

Con esta configuración, un usuario que ya haya iniciado sesión en Azure AD puede navegar a la aplicación de PeopleSoft o JD Edwards EnterpriseOne sin tener que volver a iniciar sesión, a través de Oracle Identity Cloud Service. Los clientes que implementen esta solución obtienen las ventajas del inicio de sesión único, incluido un conjunto de credenciales único, una mejor experiencia de inicio de sesión, mayor seguridad y un menor costo del departamento de soporte técnico.

Para más información sobre cómo configurar el inicio de sesión único para JD Edwards EnterpriseOne o PeopleSoft con Azure AD, consulte las [notas del producto de Oracle](https://cloud.oracle.com/iaas/whitepapers/deploy_peoplesoft_jdedwards_across_oci_azure.pdf) asociadas.