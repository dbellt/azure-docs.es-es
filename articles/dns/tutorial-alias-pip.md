---
title: 'Tutorial: Creación de un registro de alias de Azure DNS para hacer referencia a una dirección IP pública de Azure'
description: En este tutorial se muestra cómo configurar un registro de alias de Azure DNS para hacer referencia a una dirección IP pública de Azure.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: tutorial
ms.date: 04/19/2021
ms.author: rohink
ms.openlocfilehash: 28e37ad0b404b5275a224c8debab5c11c07948b4
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107738817"
---
# <a name="tutorial-configure-an-alias-record-to-refer-to-an-azure-public-ip-address"></a>Tutorial: Configuración de un registro de alias para hacer referencia a una dirección IP pública de Azure 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una infraestructura de red.
> * Crear una máquina virtual de servidor web con una IP pública.
> * Crear un registro de alias que apunte a la IP pública.
> * Probar el registro de alias.


Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos
Debe tener un nombre de dominio disponible que pueda hospedar en Azure DNS para realizar las pruebas. Debe tener control total de este dominio. El control total incluye la capacidad de establecer los registros de nombre de servidor (NS) para el dominio.

Para obtener instrucciones sobre cómo hospedar el dominio en Azure DNS, vea [Tutorial: Hospedaje del dominio en Azure DNS](dns-delegate-domain-azure-dns.md).

El dominio en el ejemplo que se usa en este tutorial es contoso.com, pero debe usar su propio nombre de dominio.

## <a name="create-the-network-infrastructure"></a>Creación de la infraestructura de red
En primer lugar, cree una red virtual y una subred en la que colocar los servidores web.
1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el panel izquierdo de Azure Portal, seleccione **Crear un recurso**. Escriba *grupo de recursos* en el cuadro de búsqueda y cree un grupo de recursos llamado **RG-DNS-Alias-pip**.
3. Seleccione **Crear un recurso** > **Redes** > **Red virtual**.
4. Cree una red virtual llamada **VNet-Server**. Colóquela en el grupo de recursos **RG-DNS-Alias-pip** y asigne a la subred el nombre **SN-Web**.

## <a name="create-a-web-server-virtual-machine"></a>Crear una máquina virtual de servidor web
1. Seleccione **Crear un recurso** > **Máquina virtual Windows Server 2016**.
2. Escriba **Web-01** en el nombre y coloque la máquina virtual en el grupo de recursos **RG-DNS-Alias-TM**. Escriba un nombre de usuario y contraseña y seleccione **Aceptar**.
3. En **Tamaño**, seleccione una SKU con 8 GB de RAM.
4. En **Configuración**, seleccione la red virtual **VNet-Servers** y la subred **SN-Web**. Para los puertos de entrada públicos, seleccione **HTTP (80)**  > **HTTPS (443)**  > **RDP (3389)** y, a continuación, **Aceptar**.
5. En la página **Resumen**, seleccione **Crear**.

La implementación tarda unos minutos en completarse. La máquina virtual tendrá una NIC asociada, con una dirección IP pública dinámica básica llamada Web-01-ip. La dirección IP pública cambiará cada vez que se reinicie la máquina virtual.

### <a name="install-iis"></a>Instalación de IIS

Instale IIS en **Web-01**.

1. Conéctese a **Web-01** e inicie sesión.
2. En el panel **Administrador del servidor**, seleccione **Agregar roles y características**.
3. Seleccione **Siguiente** tres veces. En la página **Roles de servidor**, seleccione **Servidor web (IIS)** .
4. Seleccione **Agregar características** y, a continuación, seleccione **Siguiente**.
5. Seleccione **Siguiente** cuatro veces y, a continuación, seleccione **Instalar**. Este procedimiento tarda unos minutos en finalizar.
6. Cuando finalice la instalación, seleccione **Cerrar**.
7. Abra un explorador web. Vaya a **localhost** para comprobar que aparece la página web predeterminada de IIS.

## <a name="create-an-alias-record"></a>Crear un registro de alias

Cree un registro de alias que apunte a la dirección IP pública.

1. Seleccione la zona de Azure DNS para abrirla.
2. Seleccione **Conjunto de registros**.
3. En el cuadro de texto **Nombre**, seleccione **web01**.
4. Deje el **Tipo** como un registro **D**.
5. Seleccione la casilla **Conjunto de registros de alias**.
6. Seleccione **Elegir servicio de Azure** y, a continuación, seleccione la dirección IP pública **Web-01-ip**.

## <a name="test-the-alias-record"></a>Probar el registro de alias

1. En el grupo de recursos **RG-DNS-Alias-pip**, seleccione la máquina virtual **Web-01**. Anote la dirección IP pública.
1. Desde un explorador web, vaya al nombre de dominio completo de la máquina virtual Web01-01. Un ejemplo es **web01.contoso.com**. Ahora verá la página web predeterminada de IIS.
2. Cierre el explorador web.
3. Detenga la máquina virtual **Web-01** y, a continuación, reiníciela.
4. Cuando se reinicie la máquina virtual, anote su nueva dirección IP pública.
5. Abra un nuevo explorador. Vaya de nuevo al nombre de dominio completo de la máquina virtual Web01-01. Un ejemplo es **web01.contoso.com**.

Este procedimiento se realiza correctamente porque ha utilizado un registro de alias para apuntar al recurso de dirección IP pública y no un registro A estándar.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos creados para este tutorial, elimine el grupo de recursos **RG-DNS-Alias-pip**.


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un registro de alias para hacer referencia a una dirección IP pública de Azure. Para más información acerca de Azure DNS y sobre las aplicaciones web, continúe con el tutorial sobre estas.

> [!div class="nextstepaction"]
> [Creación de registros DNS para una aplicación web en un dominio personalizado](./dns-web-sites-custom-domain.md)
