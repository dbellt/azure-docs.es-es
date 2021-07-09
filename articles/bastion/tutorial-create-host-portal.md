---
title: 'Tutorial: Creación de un host de Azure Bastion: máquina virtual Windows: portal'
description: Aprenda a crear un host de Azure Bastion y a conectarse a una máquina virtual Windows.
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: tutorial
ms.date: 04/27/2021
ms.author: cherylmc
ms.openlocfilehash: 2479b736809bbbe0dc2810b168dada6536ab8f91
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110540495"
---
# <a name="tutorial-configure-bastion-and-connect-to-a-windows-vm-through-a-browser"></a>Tutorial: Configuración de Bastion y conexión a una máquina virtual Windows mediante un explorador web

En este tutorial se muestra cómo conectarse a una máquina virtual mediante el explorador web con Azure Bastion y Azure Portal. En Azure Portal, implemente Bastion en la red virtual. Después de implementar Bastion, conéctese a una máquina virtual mediante su dirección IP privada con Azure Portal. La máquina virtual no necesita una dirección IP pública ni un software especial. Cuando se aprovisiona el servicio, la experiencia de RDP/SSH está disponible para todas las máquinas virtuales de la misma red virtual. Para más información sobre Azure Bastion, consulte [¿Qué es Azure Bastion?](bastion-overview.md)

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un host bastión para la red virtual.
> * Eliminar la dirección IP pública de una máquina virtual.
> * Conectarse a una máquina virtual Windows.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

* Una red virtual.
* Una máquina virtual Windows en la red virtual.
* Los siguientes roles necesarios:
  * Rol Lector en la máquina virtual.
  * Rol de lector en la tarjeta de interfaz de red con la dirección IP privada de la máquina virtual.
  * Rol Lector en el recurso de Azure Bastion.

* Puertos: Para conectarse a la máquina virtual Windows, debe tener abiertos los siguientes puertos en ella:
  * Puertos de entrada: RDP (3389)

 >[!NOTE]
 >En este momento no se admite el uso de Azure Bastion con las zonas de DNS privado de Azure. Antes de empezar, asegúrese de que la red virtual en la que planea implementar el recurso de Bastion no esté vinculada a una zona de DNS privado.
 >

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="create-a-bastion-host"></a><a name="createhost"></a>Creación de un host de Bastion

Esta sección le ayuda a crear el objeto de bastión en la red virtual. Esto es necesario para crear una conexión segura con una máquina virtual de la red virtual.

1. En la página **Inicio**, seleccione **+ Crear un recurso**.
1. En la página **Nuevo**, en el cuadro de búsqueda, escriba **Bastion** y, a continuación, seleccione **Buscar** para obtener los resultados de la búsqueda. En el resultado de la búsqueda de **Bastion**, compruebe que el publicador es Microsoft.
1. Seleccione **Crear**.
1. En la página **Crear una instancia de Bastion**, configure un nuevo recurso de Bastion.

   :::image type="content" source="./media/tutorial-create-host-portal/create.png" alt-text="Captura de pantalla de la página Crear un portal de Bastion." lightbox="./media/tutorial-create-host-portal/create-expand.png":::

    * **Suscripción**: suscripción de Azure que quiere usar para crear un recurso de Bastion.
    * **Grupo de recursos**: grupo de recursos en el que se creará el recurso de Bastion. Si no tiene un grupo de recursos existente, puede crear uno.
    * **Nombre**: nombre del nuevo recurso de Bastion.
    * **Región**: región pública de Azure en la que se creará el recurso.
    * **Red virtual**: red virtual en la que se creará el recurso de Bastion. Puede crear una red virtual en el portal durante este proceso, o bien usar una ya existente. Si usa una red virtual existente, asegúrese de que tenga suficiente espacio de direcciones libre para adaptarse a los requisitos de subred de Bastion. Si no ve la red virtual en la lista desplegable, asegúrese de que ha seleccionado el grupo de recursos correcto.
    * **Subred**: una vez que cree o seleccione una red virtual, aparecerá el campo subred. subred de la red virtual en la que se implementará el nuevo host de Bastion. La subred se dedicará al host de Bastion. Seleccione **Administrar configuración de subred** y cree la subred de Azure Bastion. Seleccione **+Subred** y cree una subred con las siguientes directrices:

         * La subred debe tener el nombre **AzureBastionSubnet**.
         * La subred debe ser al menos /27 o mayor.

      No es necesario rellenar los campos adicionales. Seleccione **Aceptar** y, en la parte superior de la página, seleccione **Crear una instancia de Bastion** para volver a la página de configuración de Bastion.
    * **Dirección IP pública**: dirección IP pública del recurso de Bastion en la que se accederá a RDP/SSH (a través del puerto 443). Cree una dirección IP pública. La dirección debe estar en la misma región que el recurso de Bastion que está creando. Esta dirección IP no se corresponde a ninguna de las máquinas virtuales a las que desea conectarse. Es la dirección IP pública del recurso de host de Bastion.
    * **Nombre de dirección IP pública**: nombre del recurso de la dirección IP pública. En este tutorial, puede dejar el valor predeterminado.
    * **SKU de la dirección IP pública**: Esta configuración se rellena de forma predeterminada como **Estándar**. Azure Bastion solo usa o admite la SKU de IP pública estándar.
    * **Asignación**: Esta configuración se rellena de forma predeterminada como **Estática**.

1. Cuando termine de especificar la configuración, seleccione **Revisar y crear**. Esto valida los valores. Una vez que se supera la validación, puede crear el recurso de Bastion.

   :::image type="content" source="./media/tutorial-create-host-portal/validation.png" alt-text="Captura de pantalla de la página de validación.":::
1. Revise la configuración. A continuación, en la parte inferior de la página, seleccione **Create** (Crear).
1. Verá un mensaje en el que se le indica que la implementación está en curso. El estado se mostrará en esta página a medida que se creen los recursos. El recurso de Bastion tarda aproximadamente cinco minutos en crearse e implementarse.

## <a name="remove-a-vm-public-ip-address"></a>Eliminación de una dirección IP pública de máquina virtual

[!INCLUDE [Remove a public IP address from a VM](../../includes/bastion-remove-ip.md)]

## <a name="connect-to-a-vm"></a>Conexión a una máquina virtual

[!INCLUDE [Connect to a Windows VM](../../includes/bastion-vm-rdp.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, elimine los recursos mediante los siguientes pasos:

1. Escriba el nombre del grupo de recursos en el cuadro de **búsqueda** de la parte superior del portal. Cuando vea el grupo de recursos en los resultados de la búsqueda, selecciónelo.
1. Seleccione **Eliminar grupo de recursos**.
1. En **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS:** , escriba el nombre del grupo de recursos y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un host bastión y lo ha asociado a una red virtual. A continuación, eliminó la dirección IP pública de una máquina virtual y se conectó a ella. Puede optar por usar grupos de seguridad de red con la subred de Azure Bastion. Para ello, consulte:

> [!div class="nextstepaction"]
> [Trabajar con Grupos de seguridad de red](bastion-nsg.md)
