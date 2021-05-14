---
title: Solución de problemas de implementación de Cloud Services (clásico) | Microsoft Docs
description: Hay varios problemas comunes que pueden surgir al implementar servicios en la nube en Azure. Este artículo proporciona soluciones a algunos de ellos.
ms.topic: troubleshooting
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 39c69d69012345cd636964ca813ac93186946cb9
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108771602"
---
# <a name="troubleshoot-azure-cloud-services-classic-deployment-problems"></a>Solución de problemas de implementación de Azure Cloud Services (clásico)

> [!IMPORTANT]
> [Azure Cloud Services (soporte extendido)](../cloud-services-extended-support/overview.md) es un nuevo modelo de implementación basado en Azure Resource Manager para el producto Azure Cloud Services. Con este cambio, se ha modificado el nombre del modelo de implementación basado en Azure Cloud Services para Azure Service Manager a Cloud Services (clásico), y todas las implementaciones nuevas deben usar [Cloud Services (soporte extendido)](../cloud-services-extended-support/overview.md).

Al implementar un paquete de aplicación del servicio en la nube en Azure, puede obtener información sobre la implementación en el panel **Propiedades** del Portal de Azure. Puede usar los detalles de este panel para ayudarle a solucionar problemas con el servicio en la nube y proporcionar esta información al soporte técnico de Azure al abrir una nueva solicitud de soporte técnico.

Puede encontrar el panel **Propiedades** panel de la siguiente manera:

* En Azure Portal, haga clic en la implementación del servicio en la nube, haga clic en **Toda la configuración** y después en **Propiedades**.

> [!NOTE]
> El contenido del panel **Propiedades** se puede copiar en el Portapapeles. Para ello, es preciso hacer clic en el icono de la esquina superior derecha del panel.
>
>

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="problem-i-cannot-access-my-website-but-my-deployment-is-started-and-all-role-instances-are-ready"></a>Problema: no puedo acceder a mi sitio web, pero la implementación se ha iniciado y todas las instancias de rol están listas
El vínculo de la dirección URL del sitio web que se muestra en el portal no incluye el puerto. El puerto predeterminado para los sitios web es 80. Si la aplicación está configurada para ejecutarse en otro puerto, debe agregar el puerto correcto a la dirección URL al acceder al sitio web.

1. En el Portal de Azure, haga clic en la implementación del servicio en la nube.
2. En el panel **Propiedades** de Azure Portal, compruebe los puertos de las instancias de rol (en **Puntos de conexión de entrada**).
3. Si el puerto no es el 80, agregue el valor de puerto correcto para la dirección URL al acceder a la aplicación. Para especificar un puerto no predeterminado, escriba la dirección URL, seguida de dos puntos (:) y del número de puerto, sin espacios.

## <a name="problem-my-role-instances-recycled-without-me-doing-anything"></a>Problema: mis instancias de rol se reciclan automáticamente
Se produce automáticamente una recuperación del servicio cuando Azure detecta nodos con problemas y, por consiguiente, mueve instancias de rol a nodos nuevos. Cuando esto ocurre, puede que vea que las instancias de rol se reciclan automáticamente. Para averiguar si se produjo la recuperación del servicio:

1. En el Portal de Azure, haga clic en la implementación del servicio en la nube.
2. En el panel **Propiedades** del Portal de Azure, revise la información y determine si la recuperación del servicio se produjo mientras observaba el reciclaje de roles.

Los roles también se reciclarán aproximadamente una vez al mes durante las actualizaciones del sistema operativo del host y del sistema operativo invitado.  
Para obtener más información, consulte la entrada del blog [Role Instance Restarts Due to OS Upgrades](/archive/blogs/kwill/role-instance-restarts-due-to-os-upgrades)

## <a name="problem-i-cannot-do-a-vip-swap-and-receive-an-error"></a>Problema: No puedo realizar un intercambio de VIP y recibo un error
No se permite un intercambio de VIP si una actualización de implementación está en curso. Las actualizaciones de implementación pueden ocurrir automáticamente cuando:

* Hay un nuevo sistema operativo invitado y ha configurado actualizaciones automáticas
* Se produce una recuperación de servicio.

Para averiguar si alguna actualización automática impide la realización de un intercambio de VIP:

1. En el Portal de Azure, haga clic en la implementación del servicio en la nube.
2. En el panel **Propiedades** de Azure Portal, examine el valor de **Estado**. Si el valor es **Listo**, active **Última operación** para comprobar si se produjo recientemente una operación que pudiera evitar el intercambio de VIP.
3. Repita los pasos 1 y 2 para la implementación de producción.
4. Si una actualización automática está en curso, espere a que finalice antes de intentar realizar el intercambio de VIP.

## <a name="problem-a-role-instance-is-looping-between-started-initializing-busy-and-stopped"></a>Problema: Una instancia de rol entra en un bucle entre Iniciado, Inicializando, Ocupado y Detenido
Esta condición puede indicar un problema con el código de la aplicación, el paquete o el archivo de configuración. En ese caso, podría ver que el estado cambia cada pocos minutos y en Azure Portal puede aparecer algo como **Reciclando**, **Ocupado** o **Inicializando**. Esto indica que hay algún problema con la aplicación que impide que la instancia de rol se ejecute.

Para más información acerca de cómo solucionar este problema, consulte la entrada de blog [Azure PaaS Compute Diagnostics Data](/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data) (Datos de diagnóstico de proceso de PaaS de Azure) y [Problemas comunes que causan el reciclaje de los roles](cloud-services-troubleshoot-common-issues-which-cause-roles-recycle.md).

## <a name="problem-my-application-stopped-working"></a>Problema: Mi aplicación dejó de funcionar
1. En el Portal de Azure, haga clic en la instancia de rol.
2. En el panel **Propiedades** del Portal de Azure, tenga en cuenta las condiciones siguientes para resolver el problema:
   * Si la instancia de rol se ha detenido recientemente (puede comprobar el valor de **Recuento de anulados**), es posible que la implementación se esté actualizando. Espere para ver si la instancia de rol reanuda el funcionamiento por sí misma.
   * Si la instancia de rol está en estado **Ocupado**, compruebe el código de aplicación para ver si se controla el evento [StatusCheck](/previous-versions/azure/reference/ee758135(v=azure.100)) . Debe agregar o corregir el código que controla este evento.
   * Revise los datos de diagnóstico y los escenarios de solución de problemas en la entrada del blog [Azure PaaS Compute Diagnostics Data](/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data)(Datos de diagnóstico de proceso de PaaS de Azure).

> [!WARNING]
> Si recicla el servicio en la nube, se restablecen las propiedades de la implementación, con lo que se elimina eficazmente la información del problema original.
>
>

## <a name="next-steps"></a>Pasos siguientes
Vea más [artículos de solución de problemas](./cloud-services-allocation-failures.md) para servicios en la nube.

Para más información acerca de cómo solucionar los problemas de los roles de los servicios en la nube mediante el uso de datos de diagnóstico de equipos de PaaS de Azure, consulte la [serie de blogs de Kevin Williamson](/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data).