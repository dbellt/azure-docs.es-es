---
title: Requisitos previos del host del conector ECMA de Azure AD
description: En este artículo se describen los requisitos previos y los requisitos de hardware que necesita para usar el host del conector ECMA de Azure AD.
services: active-directory
author: billmath
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 05/28/2021
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2c668380103088fb0f434d513cf7490c39e4ead2
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111570257"
---
# <a name="prerequisites-for-the-azure-ad-ecma-connector-host"></a>Requisitos previos del host del conector ECMA de Azure AD

>[!IMPORTANT]
> La versión preliminar de aprovisionamiento local se encuentra actualmente en una versión preliminar solo por invitación. Puede solicitar acceso a la funcionalidad [aquí](https://aka.ms/onpremprovisioningpublicpreviewaccess). Abriremos la versión preliminar para más clientes y conectores durante los próximos meses a medida que nos preparamos para la disponibilidad general.

En este artículo encontrará una guía sobre los requisitos previos necesarios para usar el host del conector ECMA de Azure AD.  

La instalación y configuración del host del conector ECMA de Azure AD es un proceso. Use el flujo siguiente como guía para el proceso.

 ![Flujo de instalación](./media/on-premises-ecma-prerequisites/flow-1.png)  

Para más información sobre la instalación y configuración, consulte:
   - [Instalación del host del conector ECMA de Azure AD](on-premises-ecma-install.md)
   - [Configuración del host del conector ECMA de Azure AD y el agente de aprovisionamiento](on-premises-ecma-configure.md)
   - [Configuración del conector SQL genérico del host del conector ECMA de Azure AD](on-premises-sql-connector-configure.md)

## <a name="on-premises-pre-requisites"></a>Requisitos previos locales
 - Un sistema de destino, como una base de datos SQL, en el que se puedan crear, actualizar y eliminar usuarios.
 - Un conector ECMA 2.0 o posterior para ese sistema de destino, que admita operaciones de exportación, recuperación de esquemas y, opcionalmente, importación completa o importación diferencial. Si no tiene un conector ECMA listo durante la configuración, aún puede validar el flujo de un extremo a otro si tiene una instancia de SQL Server en su entorno y usar el conector SQL genérico.
 - Un equipo con Windows Server 2016 o posterior con una dirección TCP/IP accesible desde Internet, conectividad con el sistema de destino, y conectividad saliente a login.microsoftonline.com (por ejemplo, una máquina virtual de Windows Server 2016 hospedada en IaaS de Azure o detrás de un proxy). El servidor debe tener al menos 3 GB de RAM.
 - Un equipo con .NET Framework 4.7.1

## <a name="cloud-requirements"></a>Requisitos de la nube

 - Un inquilino de Azure AD con Azure AD Premium P1 o Premium P2 (o EMS E3 o E5). 
    [!INCLUDE [active-directory-p1-license.md](../../../includes/active-directory-p1-license.md)]

 - Rol Administrador híbrido para configurar el agente de aprovisionamiento y los roles administrador de aplicaciones o administrador de la nube para configurar el aprovisionamiento en Azure Portal.


## <a name="next-steps"></a>Pasos siguientes

- [Instalación del host del conector ECMA de Azure AD](on-premises-ecma-install.md)
- [Configuración del host del conector ECMA de Azure AD](on-premises-ecma-configure.md)
- [Conector SQL genérico](on-premises-sql-connector-configure.md)
- [Tutorial: Conector SQL genérico del host del conector ECMA de Azure AD](tutorial-ecma-sql-connector.md)
