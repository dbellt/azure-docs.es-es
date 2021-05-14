---
title: Migración de instancias heredadas de Azure DNS Private Zones al nuevo modelo de recursos
titleSuffix: Azure DNS
description: En esta guía se proporcionan instrucciones paso a paso sobre cómo migrar zonas DNS privadas heredadas al modelo de recursos más reciente
services: dns
author: rohinkoul
ms.service: dns
ms.topic: how-to
ms.date: 06/18/2019
ms.author: rohink
ms.openlocfilehash: cf39a2b5853368f529c524798c6ac6486f148d46
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108745080"
---
# <a name="migrating-legacy-azure-dns-private-zones-to-new-resource-model"></a>Migración de zonas privadas de Azure DNS heredadas al nuevo modelo de recursos

Durante la versión preliminar pública, las zonas DNS privadas se crearon mediante el recurso "dnszones" con la propiedad "zoneType" establecida en "Private". Estas zonas no se admiten después del 31 de diciembre de 2019 y deben migrarse al modelo de recursos de disponibilidad general, que hace uso del tipo de recurso "privateDnsZones" en lugar de "dnszones". El proceso de migración es sencillo y le proporcionaremos un script de PowerShell para automatizar el proceso. En esta guía se proporcionan instrucciones paso a paso para migrar las zonas privadas de Azure DNS al nuevo modelo de recursos.

Para averiguar los recursos de dnszones que requieren migración, ejecute el comando siguiente en la CLI de Azure.
```azurecli
az account set --subscription <SubscriptionId>
az network dns zone list --query "[?zoneType=='Private']"
```

## <a name="prerequisites"></a>Requisitos previos

Asegúrese de que ha instalado la versión más reciente de Azure PowerShell. Para obtener más información sobre Azure PowerShell (Az) y cómo instalarlo, visite https://docs.microsoft.com/powershell/azure/new-azureps-module-az.

Asegúrese de que ha instalado el módulo Az.PrivateDns para Azure PowerShell. Para instalar este módulo, abra una ventana de PowerShell con privilegios elevados (modo administrativo) y escriba el siguiente comando

```powershell
Install-Module -Name Az.PrivateDns
```

>[!IMPORTANT]
>El proceso de migración está completamente automatizado y no se espera que ocasione tiempo de inactividad. Sin embargo, si usa las zonas privadas de Azure DNS (versión preliminar) en un entorno de producción crítico, debe ejecutar el siguiente proceso de migración durante un período de mantenimiento planeado. Asegúrese de no modificar la configuración ni los conjuntos de registros de las zonas DNS privadas mientras se ejecuta el script de migración.

## <a name="installing-the-script"></a>Instalación del script

Abra una ventana de PowerShell con privilegios elevados (modo administrativo) y ejecute el siguiente comando

```powershell
install-script PrivateDnsMigrationScript
```

Escriba "A" cuando se le pida que instale el script.

![Instalación del script](./media/private-dns-migration-guide/install-migration-script.png)

También puede obtener la versión más reciente del script de PowerShell manualmente desde https://www.powershellgallery.com/packages/PrivateDnsMigrationScript.

>[!IMPORTANT]
>El script de migración no debe ejecutarse en Azure Cloud Shell sino en una máquina virtual o en una máquina local conectada a Internet.

## <a name="running-the-script"></a>Ejecutar el script

Ejecute el siguiente comando para ejecutar el script.

```powershell
PrivateDnsMigrationScript.ps1
```

![Ejecutar el script](./media/private-dns-migration-guide/running-migration-script.png)

### <a name="enter-the-subscription-id-and-sign-in-to-azure"></a>Escriba el identificador de la suscripción e inicie sesión en Azure.

Se le pedirá que escriba el identificador de suscripción que contiene las zonas DNS privadas que quiere migrar. Se le pedirá que inicie sesión en su cuenta de Azure. Complete el inicio de sesión para que el script pueda acceder a los recursos de zona DNS privada en la suscripción.

![Inicio de sesión en Azure](./media/private-dns-migration-guide/login-migration-script.png)

### <a name="select-the-dns-zones-you-want-to-migrate"></a>Seleccione las zonas DNS que quiere migrar.

El script obtendrá la lista de todas las zonas DNS privadas en la suscripción y le pedirá que confirme cuáles quiere migrar. Escriba "A" para migrar todas las zonas DNS privadas. Una vez que haya realizado este paso, el script creará nuevas zonas DNS privadas con el nuevo modelo de recursos y copiará los datos en la nueva zona DNS. Este paso no afectará de ninguna forma a las zonas DNS privadas existentes.

![Seleccionar zonas DNS](./media/private-dns-migration-guide/migratezone-migration-script.png)

### <a name="switching-dns-resolution-to-the-new-dns-zones"></a>Cambio de la resolución DNS a las nuevas zonas DNS

Una vez que se han copiado las zonas y los registros al nuevo modelo de recursos, el script le pedirá que cambie la resolución DNS a las nuevas zonas DNS. Este paso quita la asociación entre las zonas DNS privadas heredadas y las redes virtuales. Cuando la zona heredada se desvincula de las redes virtuales, las nuevas zonas DNS creadas en el paso anterior automáticamente toman el control de la resolución DNS para esas redes virtuales.

Seleccione "A" para cambiar la resolución de DNS de todas las redes virtuales.

![Cambiar la resolución de nombres](./media/private-dns-migration-guide/switchresolution-migration-script.png)

### <a name="verify-the-dns-resolution"></a>Comprobación de la resolución DNS

Antes de continuar, compruebe que la resolución DNS en las zonas DNS funciona según lo previsto. Puede iniciar sesión en las máquinas virtuales de Azure y enviar una consulta nslookup a las zonas migradas para comprobar que la resolución DNS está funcionando.

![Verificar la resolución de nombres](./media/private-dns-migration-guide/verifyresolution-migration-script.png)

Si descubre que las consultas de DNS no se resuelven, espere unos minutos y vuelva a intentar las consultas. Si las consultas de DNS funcionan según lo previsto, escriba "Y" cuando el script se lo pida para quitar la red virtual de la zona DNS privada.

![Confirmar resolución de nombres](./media/private-dns-migration-guide/confirmresolution-migration-script.png)

>[!IMPORTANT]
>Si, por cualquier motivo, la resolución DNS de las zonas migradas no funciona según lo previsto, escriba "N" en el paso anterior y el script volverá a cambiar la resolución DNS a las zonas heredadas. Cree una incidencia de soporte técnico y podremos ayudarle con la migración de las zonas DNS.

## <a name="cleanup"></a>Limpieza

Este paso eliminarán las zonas DNS heredadas y solo se debe ejecutar después comprobar que la resolución DNS funciona según lo previsto. Se le pedirá que elimine todas las zonas DNS privadas. Escriba "Y" en cada aviso después de comprobar que la resolución DNS de esas zonas funciona correctamente.

![Limpieza](./media/private-dns-migration-guide/cleanup-migration-script.png)

## <a name="update-your-automation"></a>Actualización de la automatización

Si está usando automatización que incluye plantillas, scripts de PowerShell o código personalizado desarrollado con el SDK, debe actualizar la automatización para que use el nuevo modelo de recursos para las zonas DNS privadas. A continuación se incluyen los vínculos a la documentación de los nuevos CLI, PowerShell y SDK de DNS privados.
* [API REST de zonas privadas de Azure DNS](/rest/api/dns/privatedns/privatezones)
* [CLI de zonas privadas de Azure DNS](/cli/azure/network/private-dns/link/vnet)
* [PowerShell de zonas privadas de Azure DNS](/powershell/module/az.privatedns/)
* [SDK de zonas privadas de Azure DNS](/dotnet/api/overview/azure/privatedns/management)

## <a name="need-further-help"></a>Ayuda adicional

Cree una incidencia de soporte técnico si necesita ayuda adicional con el proceso de migración o si por cualquier motivo los pasos enumerados anteriormente no funcionan. Incluya el archivo de transcripción generado por el script de PowerShell con su incidencia de soporte técnico.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo crear una zona privada en Azure DNS mediante [Azure PowerShell](./private-dns-getstarted-powershell.md) o la [CLI de Azure](./private-dns-getstarted-cli.md).

* Obtenga información sobre algunos [escenarios de zona privada](./private-dns-scenarios.md) habituales que pueden realizarse con zonas privadas en Azure DNS.

* Para ver algunas preguntas y respuestas comunes sobre las zonas privadas en Azure DNS, como el comportamiento específico que se puede esperar con determinadas clases de operaciones, consulte [Preguntas frecuentes de DNS privado](./dns-faq-private.yml).

* Visite [Información general sobre zonas y registros de DNS](dns-zones-records.md) para obtener información sobre zonas y registros DNS.

* Obtenga información sobre las demás [funcionalidades de red](../networking/fundamentals/networking-overview.md) clave de Azure.