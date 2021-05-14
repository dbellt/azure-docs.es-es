---
title: 'Tutorial: Creación de un registro de alias para admitir nombres de dominio apex con Traffic Manager'
titleSuffix: Azure DNS
description: Este tutorial muestra cómo configurar un registro de alias de Azure DNS para admitir el uso de su nombre de dominio apex con Traffic Manager.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: tutorial
ms.date: 04/19/2021
ms.author: rohink
ms.openlocfilehash: e0101133c68142845a8ada50d9921d341cf10ad0
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107738799"
---
# <a name="tutorial-configure-an-alias-record-to-support-apex-domain-names-with-traffic-manager"></a>Tutorial: configurar un registro de alias de para admitir nombres de dominio de vértice con Traffic Manager 

Puede crear un registro de alias para el vértice de nombre de dominio para hacer referencia a un perfil de Azure Traffic Manager. Un ejemplo es contoso.com. En lugar de un servicio de redirección, puede configurar Azure DNS para hacer referencia a un perfil de Traffic Manager directamente desde la zona. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una infraestructura de red y una máquina virtual de host.
> * Crear un perfil de Traffic Manager.
> * Crear un registro de alias.
> * Probar el registro de alias.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos
Debe tener un nombre de dominio disponible que pueda hospedar en Azure DNS para realizar las pruebas. Debe tener control total de este dominio. El control total incluye la capacidad de establecer los registros de nombre de servidor (NS) para el dominio.

Para instrucciones sobre cómo hospedar el dominio en Azure DNS, consulte [Tutorial: Hospedaje del dominio en Azure DNS](dns-delegate-domain-azure-dns.md).

El dominio en el ejemplo que se usa en este tutorial es contoso.com, pero debe usar su propio nombre de dominio.

## <a name="create-the-network-infrastructure"></a>Creación de la infraestructura de red

En primer lugar, cree una red virtual y una subred en la que colocar los servidores web.

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
2. En el portal, en la parte superior izquierda, seleccione **Crear un recurso**. Escriba *grupo de recursos* en el cuadro de búsqueda y cree un grupo de recursos llamado **RG-DNS-Alias-TM**.
3. Seleccione **Crear un recurso** > **Redes** > **Red virtual**.
4. Cree una red virtual llamada **VNet-Servers**. Colóquela en el grupo de recursos **RG-DNS-Alias-TM** y asigne a la subred el nombre **SN-Web**.

## <a name="create-two-web-server-virtual-machines"></a>Crear dos máquinas virtuales de servidor web

1. Seleccione **Crear un recurso** > **Máquina virtual Windows Server 2016**.
2. Escriba **Web-01** en el nombre y coloque la máquina virtual en el grupo de recursos **RG-DNS-Alias-TM**. Escriba un nombre de usuario y una contraseña y seleccione **Aceptar**.
3. En **Tamaño**, seleccione una SKU con 8 GB de RAM.
4. En **Configuración**, seleccione la red virtual **VNet-Servers** y la subred **SN-Web**.
5. Seleccione **Dirección IP pública**. En **Asignación**, seleccione **Estática** y, a continuación, seleccione **Aceptar**.
6. Para los puertos de entrada públicos, seleccione **HTTP (80)**  > **HTTPS (443)**  > **RDP (3389)** y, a continuación, **Aceptar**.
7. En la página **Resumen**, seleccione **Crear**. Este procedimiento tarda unos minutos en finalizar.

Repita este procedimiento para crear otra máquina virtual denominada **Web-02**.

### <a name="add-a-dns-label"></a>Agregar una etiqueta DNS

Las direcciones IP públicas necesitan una etiqueta DNS para funcionar con Traffic Manager.
1. En el grupo de recursos **RG-DNS-Alias-TM**, seleccione la dirección IP pública **Web-01-ip**.
2. En **Configuración**, seleccione **Configuración**.
3. En el cuadro de texto de la etiqueta de nombre DNS, escriba **web01pip**.
4. Seleccione **Guardar**.

Repita este procedimiento para la dirección IP pública **Web-02-ip** con **web02pip** para la etiqueta de nombre DNS.

### <a name="install-iis"></a>Instalación de IIS

Instale IIS en **Web-01** y **Web 02**.

1. Conéctese a **Web-01** e inicie sesión.
2. En el panel **Administrador del servidor**, seleccione **Agregar roles y características**.
3. Seleccione **Siguiente** tres veces. En la página **Roles de servidor**, seleccione **Servidor web (IIS)** .
4. Seleccione **Agregar características** y, a continuación, seleccione **Siguiente**.
5. Seleccione **Siguiente** cuatro veces. A continuación, seleccione **Instalar**. Este procedimiento tarda unos minutos en finalizar.
6. Cuando finalice la instalación, seleccione **Cerrar**.
7. Abra un explorador web. Vaya a **localhost** para comprobar que aparece la página web predeterminada de IIS.

Repita este procedimiento para instalar IIS en **Web-02**.


## <a name="create-a-traffic-manager-profile"></a>Crear un perfil de Traffic Manager

1. Abra el grupo de recursos **RG-DNS-Alias-TM** y seleccione la dirección IP pública **Web-01-ip**. Tome nota de la dirección IP para su uso posterior. Repita este paso para la dirección IP pública **Web-02-ip**.
1. Seleccione **Crear un recurso** > **Redes** > **Perfil de Traffic Manager**.
2. En nombre, escriba **TM-alias-test**. Colóquelo en el grupo de recursos **RG-DNS-Alias-TM**.
3. Seleccione **Crear**.
4. Cuando finalice la implementación, seleccione **Ir al recurso**.
5. En la página del perfil de Traffic Manager, en **Configuración**, seleccione **Puntos de conexión**.
6. Seleccione **Agregar**.
7. Para **Tipo**, seleccione **Punto de conexión externo** y en **Nombre** escriba **EP-Web01**.
8. En el cuadro de texto **Nombre de dominio completo (FQDN) o dirección IP**, escriba la dirección IP de **Web-01-ip** que anotó anteriormente.
9. Seleccione la misma **Ubicación** que la de los otros recursos y después seleccione **Aceptar**.

Repita este procedimiento para agregar el punto de conexión **Web-02** mediante la dirección IP que anotó anteriormente para **Web-02-ip**.

## <a name="create-an-alias-record"></a>Crear un registro de alias

Cree un registro de alias que señale al perfil de Traffic Manager.

1. Seleccione la zona de Azure DNS para abrirla.
2. Seleccione **Conjunto de registros**.
3. Deje el cuadro de texto **Nombre** vacío para representar el vértice de nombre de dominio. Un ejemplo es contoso.com.
4. Deje el **Tipo** como un registro **D**.
5. Seleccione la casilla **Conjunto de registros de alias**.
6. Seleccione **Elegir servicio de Azure** y seleccione el perfil de Traffic Manager **TM-alias-test**.

## <a name="test-the-alias-record"></a>Probar el registro de alias

1. Desde un explorador web, vaya al vértice de nombre de dominio. Un ejemplo es contoso.com. Verá la página web predeterminada de IIS. Cierre el explorador web.
2. Apague la máquina virtual **Web-01**. Espere unos minutos para que se cierre por completo.
3. Abra un nuevo explorador web y busque el vértice de nombre de dominio de nuevo.
4. Verá de nuevo la página web predeterminada de IIS, puesto que Traffic Manager ha controlado la situación y dirige el tráfico a **Web-02**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos creados para este tutorial, elimine el grupo de recursos **RG-DNS-Alias-TM**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha creado un registro de alias para usar el nombre de dominio de vértice para hacer referencia a un perfil de Traffic Manager. Para más información acerca de Azure DNS y sobre las aplicaciones web, continúe con el tutorial sobre estas.

> [!div class="nextstepaction"]
> [Hospedaje de aplicaciones web con equilibrio de carga en el vértice de la zona](./dns-alias-appservice.md)
