---
title: Ver y modificar los nombres de host | Microsoft Docs
description: Cómo ver y cambiar los nombres de host para máquinas virtuales de Azure, roles de trabajo y web para la resolución de nombres
services: virtual-network
documentationcenter: na
author: genlin
manager: dcscontentpm
ms.assetid: c668cd8e-4e43-4d05-acc3-db64fa78d828
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/14/2021
ms.author: genli
ms.openlocfilehash: d793dddcfd51c9d9bd2527298d958482a9216b88
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110080626"
---
# <a name="viewing-and-modifying-hostnames"></a>Ver y modificar los nombres de host
Para permitir que el nombre de host haga referencia a las instancias de rol, debe establecer el valor del nombre de host en el archivo de configuración de servicio de cada rol. Para hacer esto, agregue el nombre de host que quiera al atributo **vmName** del elemento **Rol**. El valor del atributo **vmName** se usa como base para el nombre de host de cada instancia de rol. Por ejemplo, si el atributo **vmName** es *webrole* y hay tres instancias de ese rol, los nombres de host de las instancias serán *webrole0*, *webrole1* y *webrole2*. No es necesario especificar un nombre de host para máquinas virtuales en el archivo de configuración, porque el nombre de host de una máquina virtual se rellena según el nombre de esa máquina virtual. Para obtener más información sobre cómo configurar un servicio de Microsoft Azure, consulte [Esquema de configuración del servicio de Azure (archivo de .cscfg)](/previous-versions/azure/reference/ee758710(v=azure.100))

## <a name="viewing-hostnames"></a>Ver nombres de host
Puede ver los nombres de host de máquinas virtuales e instancias de rol en un servicio en la nube mediante cualquiera de las herramientas siguientes.

### <a name="service-configuration-file"></a>Archivo de configuración de servicio
Puede descargar en el Portal de Azure el archivo de configuración de servicio de un servicio implementado desde la hoja **Configurar** del servicio. Después, puede buscar el atributo **vmName** del elemento **Nombre de rol** para ver el nombre de host. Tenga en cuenta que ese nombre de host se usa como base para cada nombre de host de cada instancia de rol. Por ejemplo, si el atributo **vmName** es *webrole* y hay tres instancias de ese rol, los nombres de host de las instancias serán *webrole0*, *webrole1* y *webrole2*.

### <a name="remote-desktop"></a>Escritorio remoto
Una vez haya habilitado el Escritorio remoto (Windows), la comunicación remota de Windows PowerShell (Windows) o las conexiones SSH (Linux y Windows) en las máquinas virtuales o instancias de rol, podrá ver el nombre de host de una conexión del Escritorio remoto activa de varias maneras:

* Escriba el nombre de host en el símbolo del sistema o en un terminal SSH.
* Escriba ipconfig/all en el símbolo del sistema (sólo Windows).
* Consulte el nombre del equipo en la configuración del sistema (sólo Windows).

### <a name="azure-service-management-rest-api"></a>API de REST de Administración de servicios de Azure
Desde un cliente REST, siga estas instrucciones:

1. Asegúrese de que tiene un certificado de cliente para conectarse al Portal de Azure. Para obtener un certificado de cliente, siga los pasos que aparecen en el artículo sobre [cómo descargar e importar la configuración de publicación y la información de suscripción](/previous-versions/dynamicsnav-2013/dn385850(v=nav.70)). 
2. Establezca una entrada de encabezado denominada x-ms-version con un valor de 2013-11-01.
3. Envíe una solicitud con el formato siguiente: `https://management.core.windows.net/<subscription-id>/services/hostedservices/<service-name>?embed-detail=true`
4. Busque el elemento **HostName** de cada elemento **RoleInstance**.

> [!WARNING]
> Asimismo, puede ver el sufijo de dominio interno del servicio en la nube desde la respuesta a la llamada REST comprobando el elemento **InternalDnsSuffix** o ejecutando ipconfig /all desde un símbolo del sistema en una sesión de Escritorio remoto (Windows) o cat /etc/resolv.conf desde un terminal SSH (Linux).
> 
> 

## <a name="modifying-a-hostname"></a>Modificar un nombre de host
Puede modificar el nombre de host de cualquier máquina virtual o instancia de rol al cargar un archivo de configuración de servicio modificado o al cambiar el nombre del equipo desde una sesión de Escritorio remoto.

## <a name="next-steps"></a>Pasos siguientes
[resolución de nombres DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md)

[Esquema de configuración del servicio de Azure (.cscfg)](/previous-versions/azure/reference/ee758710(v=azure.100))

[Esquema de configuración de Azure Virtual Network](/previous-versions/azure/reference/jj157100(v=azure.100))

[Especificación de la configuración de DNS usando los archivos de configuración de red](/previous-versions/azure/virtual-network/virtual-networks-specifying-a-dns-settings-in-a-virtual-network-configuration-file)
