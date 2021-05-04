---
title: 'Tutorial: Acceso a la nube privada'
description: Aprenda a acceder a una nube privada de Azure VMware Solution.
ms.topic: tutorial
ms.date: 03/13/2021
ms.openlocfilehash: d2046acaf483022f977559fe74254ca58b8c42ea
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107945654"
---
# <a name="tutorial-access-an-azure-vmware-solution-private-cloud"></a>Tutorial: Acceso a una nube privada de Azure VMware Solution

Azure VMware Solution no permite administrar la nube privada con la instancia local de vCenter. Tendrá que conectarse a la instancia de vCenter de Azure VMware Solution a través de un jumpbox. 

En este tutorial, creará un jumpbox en el grupo de recursos que creó en el [tutorial anterior](tutorial-configure-networking.md) e iniciará sesión en vCenter de Azure VMware Solution. Este jumpbox es una máquina virtual (VM) de Windows que se encuentra en la misma red virtual que creó.  Proporciona acceso tanto a vCenter como a NSX Manager. 

En este tutorial aprenderá a:

> [!div class="checklist"]
> * Crear una máquina virtual Windows para acceder a vCenter de Azure VMware Solution.
> * Iniciar sesión en vCenter desde esta máquina virtual.

## <a name="create-a-new-windows-virtual-machine"></a>Creación de una nueva máquina virtual Windows.

1. En el grupo de recursos, seleccione **+ Agregar**, busque y seleccione **Microsoft Windows 10** y, a continuación, seleccione **Crear**.

   :::image type="content" source="media/tutorial-access-private-cloud/ss8-azure-w10vm-create.png" alt-text="Captura de pantalla que muestra cómo agregar una nueva máquina virtual Windows 10 para un jumpbox." border="true":::

1. Escriba la información requerida en los campos y seleccione **Revisar y crear**. 

   Para más información sobre los campos, consulte la tabla siguiente.

   | Campo | Valor |
   | --- | --- |
   | **Suscripción** | El valor se rellena previamente con la suscripción a la que pertenece el grupo de recursos. |
   | **Grupos de recursos** | El valor se rellena previamente para el grupo de recursos actual que creó en el tutorial anterior.  |
   | **Nombre de la máquina virtual** | Introduzca un nombre único para la máquina virtual. |
   | **Región** | Seleccione la ubicación geográfica de la máquina virtual. |
   | **Opciones de disponibilidad** | Deje el valor predeterminado seleccionado. |
   | **Imagen** | Seleccione la imagen de la máquina virtual. |
   | **Tamaño** | Deje el valor de tamaño predeterminado. |
   | **Tipo de autenticación**  | Seleccione **Contraseña**. |
   | **Nombre de usuario** | Escriba el nombre de usuario para iniciar sesión en la máquina virtual. |
   | **Contraseña** | Escriba la contraseña para iniciar sesión en la máquina virtual. |
   | **Confirmar contraseña** | Escriba la contraseña para iniciar sesión en la máquina virtual. |
   | **Puertos de entrada públicos** | Seleccione **Ninguno**. Si selecciona Ninguno, puede usar el [acceso JIT](../security-center/security-center-just-in-time.md#jit-configure) para controlar el acceso a la máquina virtual solo cuando desee acceder a ella. Como alternativa, puede usar [Azure Bastion](../bastion/tutorial-create-host-portal.md) si quiere tener acceso al servidor de jumpbox de forma segura desde Internet sin exponer ningún puerto de red.  |


1. Una vez superada la validación, seleccione **Crear** para iniciar el proceso de creación de la máquina virtual.

## <a name="connect-to-the-local-vcenter-of-your-private-cloud"></a>Conexión a la instancia local de vCenter de la nube privada.

1. En el jumpbox, inicie sesión en el cliente de vSphere con el inicio de sesión único de VMware vCenter con un nombre de usuario de administrador de la nube y compruebe que la interfaz de usuario se muestra correctamente.

1. En Azure Portal, seleccione su nube privada y haga clic en **Administrar** > **Identidad**. 

   Se muestran las direcciones URL y las credenciales de usuario para vCenter y NSX-T Manager de la nube privada.

   :::image type="content" source="media/tutorial-access-private-cloud/ss4-display-identity.png" alt-text="Mostrar las direcciones URL y las credenciales del administrador de NSX y vCenter de la nube privada" border="true" lightbox="media/tutorial-access-private-cloud/ss4-display-identity.png":::.

1. Vaya a la máquina virtual que creó en el paso anterior y conéctese a ella. 

   Si necesita ayuda para conectarse a la máquina virtual, consulte [Conexión a una máquina virtual](../virtual-machines/windows/connect-logon.md#connect-to-the-virtual-machine) para los detalles.

1. En la máquina virtual Windows, abra un explorador y vaya a las direcciones URL del administrador de NSX-T y vCenter en dos pestañas. 

1. En la pestaña de vCenter, escriba las credenciales del usuario `cloudadmin@vmcp.local` del paso anterior.

   :::image type="content" source="media/tutorial-access-private-cloud/ss5-vcenter-login.png" alt-text="Inicio de sesión en vCenter de la nube privada" border="true":::.

   :::image type="content" source="media/tutorial-access-private-cloud/ss6-vsphere-client-home.png" alt-text="Portal de vCenter" border="true":::.

1. En la segunda pestaña del explorador, inicie sesión en el administrador de NSX-T.

   :::image type="content" source="media/tutorial-access-private-cloud/ss10-nsx-manager-home.png" alt-text="En la segunda pestaña del explorador, inicie sesión en el administrador de NSX-T." border="true":::



## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido cómo:

> [!div class="checklist"]
> * Crear una máquina virtual Windows para conectarse a vCenter
> * Iniciar sesión en vCenter desde la máquina virtual

Continúe con el siguiente tutorial para aprender a crear una red virtual a fin de configurar la administración local de los clústeres de nube privada.

> [!div class="nextstepaction"]
> [Creación de una red virtual](tutorial-configure-networking.md)


