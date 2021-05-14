---
title: 'Azure VMware Solution by CloudSimple: acceso al cliente de vSphere'
description: Describe cómo acceder a vCenter en la nube privada.
author: shortpatti
ms.author: v-patsho
ms.date: 08/30/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: e3f827bbd1e500c9cea809939194757bb9fe7881
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108181065"
---
# <a name="access-your-private-cloud-vcenter-portal"></a>Acceso al portal de vCenter de la nube privada

Puede iniciar el portal de vCenter de la nube privada desde Azure Portal o el portal de CloudSimple.  El portal de vCenter le permite administrar la infraestructura de VMware en su nube privada.

## <a name="before-you-begin"></a>Antes de empezar

Debe establecerse la conexión de red y la resolución de nombres DNS debe estar habilitada para acceder al portal de vCenter.  Puede establecer la conexión de red a la nube privada con cualquiera de las opciones siguientes.

* [Conexión desde una red local a CloudSimple mediante ExpressRoute](on-premises-connection.md)
* [Configuración de una conexión VPN en la nube privada de CloudSimple](set-up-vpn.md)

Para configurar la resolución de nombres DNS de los componentes de la infraestructura de VMware de la nube privada, consulte [Configuración de DNS para la resolución de nombres en el acceso a vCenter de la nube privada desde estaciones de trabajo en el entorno local](on-premises-dns-setup.md).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="access-vcenter-from-azure-portal"></a>Acceso a vCenter desde Azure Portal

Puede iniciar el portal de vCenter de la nube privada desde Azure Portal.

1. Seleccione **Todos los servicios**.

2. Busque **Servicios de CloudSimple**.

3. Seleccione el servicio de CloudSimple de la nube privada al que desea conectarse.

4. En la página **Información general**, haga clic en **View VMware Private Clouds** (Ver nubes privadas de VMware).

    ![Información general del servicio CloudSimple](media/cloudsimple-service-overview.png)

5. Seleccione la nube privada en la lista y, después, haga clic en **Launch vSphere Client** (Iniciar cliente vSphere).

    ![Inicio del cliente vSphere](media/cloudsimple-service-launch-vsphere-client.png)

## <a name="access-vcenter-from-cloudsimple-portal"></a>Acceso a vCenter desde el portal de CloudSimple

Puede iniciar el portal de vCenter de la nube privada desde el portal de CloudSimple.

1. Acceda al [portal de CloudSimple](access-cloudsimple-portal.md).

2. En **Recursos** seleccione la nube privada a la que desea acceder y haga clic en **Launch vSphere Client** (Iniciar cliente vSphere).

    ![Iniciar cliente de vSphere: Recursos](media/cloudsimple-portal-resources-launch-vcenter.png)

3. También puede iniciar el portal de vCenter desde la pantalla de resumen de la nube privada.

    ![Iniciar cliente de vSphere: Resumen](media/cloudsimple-resources-summary-launch-vcenter.png)

## <a name="next-steps"></a>Pasos siguientes

* [Creación y administración de redes VLAN y subredes para las nubes privadas](create-vlan-subnet.md)
* [Modelo de permisos de la nube privada de VMware vCenter de CloudSimple](learn-private-cloud-permissions.md)