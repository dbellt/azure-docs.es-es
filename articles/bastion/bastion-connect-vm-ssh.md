---
title: Conexión a una máquina virtual Linux con Azure Bastion
description: Aprenda a conectarse a una máquina virtual Linux mediante Azure Bastion.
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: how-to
ms.date: 02/12/2021
ms.author: cherylmc
ms.openlocfilehash: a6f9add11b6632d1ee26041ade0ade45ec58716d
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110538087"
---
# <a name="connect-using-ssh-to-a-linux-virtual-machine-using-azure-bastion"></a>Conexión mediante SSH a una máquina virtual Linux con Azure Bastion

En este artículo se muestra cómo establecer una conexión SSH segura y sin problemas con sus máquinas virtuales Linux en una red virtual de Azure. Puede conectarse a una máquina virtual directamente desde Azure Portal. Cuando se utiliza Azure Bastion, las máquinas virtuales no requieren un cliente, un agente ni software adicional. Para más información sobre Azure Bastion, consulte la página de [información general](bastion-overview.md).

Puede usar Azure Bastion para conectar a una máquina virtual Linux mediante SSH. Puede utilizar tanto el nombre de usuario y la contraseña como las claves SSH para realizar la autenticación. Puede conectarse a la máquina virtual con claves SSH mediante una de estas opciones:

* Una clave privada que se escribe manualmente.
* Un archivo que contiene la información de la clave privada.

La clave privada SSH debe tener un formato que empieza con `"-----BEGIN RSA PRIVATE KEY-----"` y finaliza con `"-----END RSA PRIVATE KEY-----"`.

## <a name="prerequisites"></a>Prerrequisitos

Asegúrese de haber configurado un host de Azure Bastion para la red virtual donde reside la máquina virtual. Para más información, consulte [Create an Azure Bastion host](./tutorial-create-host-portal.md) (Creación de un host de Azure Bastion). Cuando el servicio Bastion se aprovisiona e implementa en la red virtual, puede usarlo para conectarse a cualquier máquina virtual de esta red virtual. 

Cuando se conecta mediante Bastion, se presupone que usa RDP para conectarse a una máquina virtual Windows y SSH para conectarse a las máquinas virtuales Linux. Para información sobre cómo conectarse a una máquina virtual Windows, consulte [Conexión a una máquina virtual: Windows](bastion-connect-vm-rdp.md).

### <a name="required-roles"></a>Roles necesarios

Para crear una conexión, se requieren los siguientes roles:

* Rol de lector en la máquina virtual
* Rol de lector en la tarjeta de interfaz de red con la dirección IP privada de la máquina virtual
* Rol de lector en el recurso de Azure Bastion

### <a name="ports"></a>Puertos

Para conectarse a la máquina virtual Linux mediante SSH, debe tener abiertos los siguientes puertos en la máquina virtual:

* Puerto de entrada: SSH (22)

## <a name="connect-using-username-and-password"></a><a name="username"></a>Conexión: mediante un nombre de usuario y una contraseña

1. Abra [Azure Portal](https://portal.azure.com). Vaya a la máquina virtual a la que quiere conectarse y, a continuación, haga clic en **Conectar** y seleccione **Bastion** en la lista desplegable.

   :::image type="content" source="./media/bastion-connect-vm-ssh/connect.png" alt-text="Captura de pantalla que muestra la información general de una máquina virtual en Azure Portal con la opción Conectar seleccionada.":::
1. Después de seleccionar Bastion, haga clic en **Usar Bastion**. Si no aprovisionó Bastion para la red virtual, consulte [Configure Bastion](./quickstart-host-portal.md) (Configuración de Bastion).
1. En la página **Conectar mediante Azure Bastion**, especifique los valores pertinentes en **Nombre de usuario** y **Contraseña**.

   :::image type="content" source="./media/bastion-connect-vm-ssh/password.png" alt-text="Autenticación de contraseña":::
1. Seleccione **Conectar** para conectarse a la máquina virtual.

## <a name="connect-manually-enter-a-private-key"></a><a name="privatekey"></a>Conexión: con una clave privada escrita

1. Abra [Azure Portal](https://portal.azure.com). Vaya a la máquina virtual a la que quiere conectarse y, a continuación, haga clic en **Conectar** y seleccione **Bastion** en la lista desplegable.

   :::image type="content" source="./media/bastion-connect-vm-ssh/connect.png" alt-text="Captura de pantalla que muestra la información general de una máquina virtual en Azure Portal con la opción Conectar seleccionada.":::
1. Después de seleccionar Bastion, haga clic en **Usar Bastion**. Si no aprovisionó Bastion para la red virtual, consulte [Configure Bastion](./quickstart-host-portal.md) (Configuración de Bastion).
1. En la página **Conectar mediante Azure Bastion**, especifique los valores pertinentes en **Nombre de usuario** y **Clave privada SSH**.

   :::image type="content" source="./media/bastion-connect-vm-ssh/ssh-private-key.png" alt-text="Autenticación de clave privada SSH":::
1. Escriba la clave privada en el área de texto **Clave privada SSH** (o péguela directamente).
1. Seleccione **Conectar** para conectarse a la máquina virtual.

## <a name="connect-using-a-private-key-file"></a><a name="ssh"></a>Conexión: con un archivo de clave privada

1. Abra [Azure Portal](https://portal.azure.com). Vaya a la máquina virtual a la que quiere conectarse y, a continuación, haga clic en **Conectar** y seleccione **Bastion** en la lista desplegable.

   :::image type="content" source="./media/bastion-connect-vm-ssh/connect.png" alt-text="Captura de pantalla que muestra la información general de una máquina virtual en Azure Portal con la opción Conectar seleccionada.":::
1. Después de seleccionar Bastion, haga clic en **Usar Bastion**. Si no aprovisionó Bastion para la red virtual, consulte [Configure Bastion](./quickstart-host-portal.md) (Configuración de Bastion).
1. En la página **Conectar mediante Azure Bastion**, especifique los valores pertinentes en **Nombre de usuario** y **Clave privada SSH del archivo local**.

   :::image type="content" source="./media/bastion-connect-vm-ssh/private-key-file.png" alt-text="Archivo de clave privada SSH":::

1. Busque el archivo y, luego, seleccione **Abrir**.
1. Seleccione **Conectar** para conectarse a la máquina virtual. Al hacer clic en Connect (Conectar), la conexión SSH con esta máquina virtual se abrirá directamente en Azure Portal. Esta conexión se realiza a través de HTML5 mediante el puerto 443 en el servicio Bastion a través de la dirección IP privada de la máquina virtual.

## <a name="connect-using-a-private-key-stored-in-azure-key-vault"></a><a name="akv"></a>Conexión: Uso de una clave privada almacenada en Azure Key Vault

>[!NOTE]
>La actualización del portal para esta característica se está implementando actualmente en algunas regiones.
>

1. Abra [Azure Portal](https://portal.azure.com). Vaya a la máquina virtual a la que quiere conectarse y, a continuación, haga clic en **Conectar** y seleccione **Bastion** en la lista desplegable.

   :::image type="content" source="./media/bastion-connect-vm-ssh/connect.png" alt-text="Captura de pantalla que muestra la información general de una máquina virtual en Azure Portal con la opción Conectar seleccionada.":::
1. Después de seleccionar Bastion, haga clic en **Usar Bastion**. Si no aprovisionó Bastion para la red virtual, consulte [Configure Bastion](./quickstart-host-portal.md) (Configuración de Bastion).
1. En la página **Conectar mediante Azure Bastion**, especifique el valor pertinente en **Nombre de usuario** y seleccione **Clave privada SSH de Azure Key Vault**.

   :::image type="content" source="./media/bastion-connect-vm-ssh/ssh-key-vault.png" alt-text="Clave privada SSH de Azure Key Vault":::
1. Seleccione la lista desplegable **Azure Key Vault** y elija el recurso en el que ha almacenado la clave privada SSH. Si no ha configurado un recurso de Azure Key Vault, consulte [Creación de un almacén de claves](../key-vault/general/quick-create-portal.md) y almacene la clave privada SSH como el valor de un nuevo secreto de Key Vault.

   :::image type="content" source="./media/bastion-connect-vm-ssh/key-vault.png" alt-text="Azure Key Vault":::

   Asegúrese de que tiene los permisos de acceso **Enumerar** y **Obtener** para los secretos almacenados en el recurso de Key Vault. Para asignar y modificar directivas de acceso para el recurso de Key Vault, consulte [Asignación de una directiva de acceso de Key Vault](../key-vault/general/assign-access-policy-portal.md).
1. Seleccione la lista desplegable **Azure Key Vault Secret** (Secreto de Azure Key Vault) y elija el secreto de Key Vault que contiene el valor de la clave privada SSH.
1. Seleccione **Conectar** para conectarse a la máquina virtual. Al hacer clic en **Conectar**, la conexión SSH con esta máquina virtual se abrirá directamente en Azure Portal. Esta conexión se realiza a través de HTML5 mediante el puerto 443 en el servicio Bastion a través de la dirección IP privada de la máquina virtual.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Bastion, consulte las [preguntas frecuentes sobre Bastion](bastion-faq.md). 
