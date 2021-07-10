---
title: 'Inicio rápido: Configuración de Azure Bastion y conexión a una máquina virtual mediante una dirección IP privada y un explorador'
titleSuffix: Azure Bastion
description: Aprenda a crear un host de Azure Bastion desde una máquina virtual y a conectarse a ella de forma segura mediante el explorador y una dirección IP privada.
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: quickstart
ms.date: 02/18/2021
ms.author: cherylmc
ms.openlocfilehash: e51d8633418a0a00afb8a6055c05f9c77d93f3cb
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110540460"
---
# <a name="quickstart-connect-to-a-vm-securely-through-a-browser-via-private-ip-address"></a>Inicio rápido: conexión a una máquina virtual de forma segura mediante un explorador y una dirección IP privada

Puede conectarse a una máquina virtual mediante el explorador con Azure Bastion y Azure Portal. En este artículo de inicio rápido se muestra cómo configurar Azure Bastion en función de la configuración de la máquina virtual y conectarse a la máquina virtual mediante el portal. La máquina virtual no necesita una dirección IP pública, software cliente, agente ni una configuración especial. Cuando se aprovisiona el servicio, la experiencia de RDP/SSH está disponible para todas las máquinas virtuales de la misma red virtual. Para más información sobre Azure Bastion, consulte [¿Qué es Azure Bastion?](bastion-overview.md)

## <a name="prerequisites"></a><a name="prereq"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. Si no tiene ninguna, [cree una gratis](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Para poder conectarse a una máquina virtual mediante el explorador con Bastion, debe poder iniciar sesión en Azure Portal.

* Una máquina virtual Windows en una red virtual. Si no tiene una máquina virtual, cree una mediante [Inicio rápido: creación de una máquina virtual](../virtual-machines/windows/quick-create-portal.md).

  * Si necesita valores de ejemplo, consulte los [Valores de ejemplo](#values) proporcionados.
  * Si ya tiene una red virtual, asegúrese de seleccionarla en la pestaña Redes al crear la máquina virtual.
  * Si aún no tiene una, puede crearla al mismo tiempo que la máquina virtual.
  * No es necesario tener una dirección IP pública para esta máquina virtual para poder conectarse mediante Azure Bastion.

* Roles de máquina virtual obligatorios:
  * Rol de lector en la máquina virtual.
  * Rol de lector en la tarjeta de interfaz de red con la dirección IP privada de la máquina virtual.
  
* Puertos de máquina virtual obligatorios:
  * Puertos de entrada: RDP (3389)

 >[!NOTE]
 >En este momento no se admite el uso de Azure Bastion con las zonas de DNS privado de Azure. Antes de empezar, asegúrese de que la red virtual en la que planea implementar el recurso de Bastion no esté vinculada a una zona de DNS privado.
 >

### <a name="example-values"></a><a name="values"></a>Valores del ejemplo

Puede usar los siguientes valores de ejemplo al crear esta configuración, o puede sustituirlos por los propios.

**Valores básicos de la red virtual y la máquina virtual:**

|**Nombre** | **Valor** |
| --- | --- |
| Máquina virtual| TestVM |
| Resource group | TestRG1 |
| Region | Este de EE. UU. |
| Virtual network | VNet1 |
| Espacio de direcciones | 10.1.0.0/16 |
| Subredes | FrontEnd: 10.1.0.0/24 |

**Valores de Azure Bastion:**

|**Nombre** | **Valor** |
| --- | --- |
| Nombre | VNet1-bastion |
| + Nombre de subred | AzureBastionSubnet |
| Direcciones de AzureBastionSubnet | Una subred dentro del espacio de direcciones de la red virtual con una máscara de subred/27. Por ejemplo, 10.1.1.0/27.  |
| Dirección IP pública |  Crear nuevo |
| Nombre de la dirección IP pública | VNet1-ip  |
| SKU de la dirección IP pública |  Estándar  |
| Asignación  | estática |

## <a name="create-a-bastion-host"></a><a name="createvmset"></a>Creación de un host de Bastion

Hay varias maneras de configurar un host bastión. En los pasos siguientes creará un host bastión en Azure Portal directamente desde la máquina virtual. A crear un host desde una máquina virtual, varias opciones de configuración se rellenarán automáticamente con los valores de la máquina virtual o la red virtual.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya a la máquina virtual a la que quiere conectarse y seleccione **Conectar**.

   :::image type="content" source="./media/quickstart-host-portal/vm-connect.png" alt-text="Captura de pantalla de configuración de máquina virtual." lightbox="./media/quickstart-host-portal/vm-connect.png":::
1. En el menú desplegable, seleccione **Bastion**.

   :::image type="content" source="./media/quickstart-host-portal/bastion.png" alt-text="Captura de pantalla de la lista desplegable de Bastion." lightbox="./media/quickstart-host-portal/bastion.png":::
1. En la **página TestVM | Conectar**, seleccione **Usar Bastion**.

   :::image type="content" source="./media/quickstart-host-portal/select-bastion.png" alt-text="Captura de pantalla de Usar Bastion.":::

1. En la página **Conectar mediante Azure Bastion**, configure los valores.

   * **Paso 1:** los valores se rellenan previamente porque crea el host bastión directamente desde una máquina virtual.

   * **Paso 2:** el espacio de direcciones se rellena previamente con un espacio de direcciones sugerido. AzureBastionSubnet debe tener un espacio de direcciones de /27, o mayor (/26, /25, etc.).

   :::image type="content" source="./media/quickstart-host-portal/create-subnet.png" alt-text="Captura de pantalla de creación de la subred de Bastion.":::

1. Haga clic en **Crear subred** para crear AzureBastionSubnet.
1. Después de que se crea la subred, la página avanza automáticamente al **paso 3**. En el paso 3, use los valores siguientes:

   * **Nombre**: nombre del host bastión.
   * **Dirección IP pública**: seleccione **Crear nueva**.
   * **Nombre de dirección IP pública:** nombre del recurso de la dirección IP pública.
   * **SKU de dirección IP pública:** preconfigurado como **Estándar**
   * **Asignación:** Configurado previamente como **Estático**. No se puede usar una asignación dinámica para Azure Bastion.
   * **Grupo de recursos**: el mismo que la máquina virtual.

   :::image type="content" source="./media/quickstart-host-portal/create-bastion.png" alt-text="Captura de pantalla del paso 3.":::
1. Después de completar los valores, seleccione **Create Azure Bastion using defaults** (Usar valores predeterminados para crear Azure Bastion). Azure valida la configuración y luego crea el host. El host y sus recursos tardan unos cinco minutos en crearse e implementarse.

## <a name="connect"></a><a name="connect"></a>Conexión

Una vez implementado Bastion en la red virtual, la pantalla cambia a la página Conectar.

1. Escriba el nombre de usuario y la contraseña de la máquina virtual. A continuación, seleccione **Conectar**.

   :::image type="content" source="./media/quickstart-host-portal/connect.png" alt-text="Captura de pantalla que muestra el cuadro de diálogo Conectar usando Azure Bastion.":::
1. La conexión RDP con esta máquina virtual a se abrirá directamente en Azure Portal (mediante HTML5) con el puerto 443 y el servicio Bastion.

   :::image type="content" source="./media/quickstart-host-portal/connected.png" alt-text="Conexión RDP":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado de usar la red virtual y las máquinas virtuales, elimine el grupo de recursos y todos los recursos que contiene:

1. Escriba el nombre del grupo de recursos en el cuadro **Buscar** de la parte superior del portal y selecciónelo en los resultados de la búsqueda.

1. Seleccione **Eliminar grupo de recursos**.

1. En **TYPE THE RESOURCE GROUP NAME** (ESCRIBIR EL NOMBRE DEL GRUPO DE RECURSOS), escriba el grupo de recursos y seleccione **Delete** (Eliminar).

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha creado un host bastión para la red virtual y, a continuación, se ha conectado a una máquina virtual de forma segura mediante Bastion. Después, puede continuar con el paso siguiente si quiere conectarse a un conjunto de escalado de máquinas virtuales.

> [!div class="nextstepaction"]
> [Conexión a un conjunto de escalado de máquinas virtuales mediante Azure Bastion](bastion-connect-vm-scale-set.md)
