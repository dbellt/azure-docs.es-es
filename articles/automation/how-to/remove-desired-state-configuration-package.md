---
title: Eliminación de DSC y nodos de Automation State Configuration
description: En este artículo se explica cómo quitar un documento de configuración de Azure Automation State Configuration (DSC) asignado y anular el registro de un nodo administrado.
titleSuffix: Azure Automation
services: automation
ms.subservice: dsc
ms.date: 04/16/2021
ms.topic: how-to
ms.openlocfilehash: 341a29d3aa8376a6308582bba026fab8fcadca35
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108021177"
---
# <a name="how-to-remove-a-configuration-and-node-from-automation-state-configuration"></a>Cómo quitar una configuración y un nodo de Automation State Configuration

En este artículo se explica cómo anular el registro de un nodo administrado por Automation State Configuration y quitar de forma segura una configuración de Desired State Configuration (DSC) de PowerShell de los nodos administrados. Para los nodos de Windows y Linux, debe [anular el registro del nodo](#unregister-a-node) y [eliminar la configuración](#delete-a-configuration-from-the-azure-portal). Solo para los nodos de Linux, también puede eliminar los paquetes DSC desde los nodos. Consulte [Eliminación del paquete DSC de un nodo de Linux](#remove-the-dsc-package-from-a-linux-node).

## <a name="unregister-a-node"></a>Anular el registro de un nodo

Si ya no quiere que un nodo se pueda administrar mediante State Configuration (DSC), puede anular el registro desde Azure Portal o con Azure PowerShell si sigue estos pasos.

Cuando se anula el registro de un nodo en el servicio, solo establece la configuración del Administrador de configuración local de forma que el nodo deja de conectarse al servicio. Esto no afecta a la configuración que se aplica actualmente al nodo y deja los archivos relacionados en su lugar en el nodo. Opcionalmente, puede limpiar esos archivos. Consulte [Eliminación de una configuración](#delete-a-configuration).

### <a name="unregister-in-the-azure-portal"></a>Anulación del registro en Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque y seleccione **Cuentas de Automation**.
1. En la página **Cuentas de Automation**, seleccione su cuenta de Automation en la lista.
1. En la cuenta de Automation, seleccione **State Configuration (DSC)** en **Administración de configuración**.
1. En la página **State configuration (DSC)** (Configuración de estado [DSC]), haga clic en la pestaña **Nodos**.
1. En la pestaña **Nodos**, seleccione el nombre del nodo cuyo registro quiere anular.
1. En el panel de dicho nodo, haga clic en **Anular registro**.

   :::image type="content" source="./media/remove-desired-state-configuration-package/unregister-node.png" alt-text="Captura de pantalla de la página de detalles del nodo donde está resaltado el botón Anular registro." lightbox="./media/remove-desired-state-configuration-package/unregister-node.png":::

### <a name="unregister-using-powershell"></a>Anulación del registro mediante PowerShell

También puede anular el registro de un nodo mediante el cmdlet de PowerShell [Unregister-AzAutomationDscNode](/powershell/module/az.automation/unregister-azautomationdscnode).

>[!NOTE]
>Si su organización sigue utilizando los módulos AzureRM en desuso, puede usar [Unregister-AzureRmAutomationDscNode](/powershell/module/azurerm.automation/unregister-azurermautomationdscnode).

## <a name="delete-a-configuration"></a>Eliminación de una configuración

Cuando esté listo para quitar un documento de configuración de DSC importado (que es un archivo Managed Object Format (MOF) o .mof) asignado a uno o varios nodos, siga estos pasos.

### <a name="delete-a-configuration-from-the-azure-portal"></a>Eliminación de una configuración en Azure Portal

Puede eliminar las configuraciones de los nodos de Windows y Linux desde Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque y seleccione **Cuentas de Automation**.
1. En la página **Cuentas de Automation**, seleccione su cuenta de Automation en la lista.
1. En la cuenta de Automation, seleccione **State Configuration (DSC)** en **Administración de configuración**.
1. En la página **State Configuration (DSC)** , haga clic en la pestaña **Configuraciones** y seleccione el nombre de la configuración que quiere eliminar.

   :::image type="content" source="./media/remove-desired-state-configuration-package/configurations-tab.png" alt-text="Captura de pantalla de la pestaña Configuraciones." lightbox="./media/remove-desired-state-configuration-package/configurations-tab.png":::

1. En la página de detalles de la configuración, haga clic en **Eliminar** para quitar la configuración.

   :::image type="content" source="./media/remove-desired-state-configuration-package/delete-extension.png" alt-text="Captura de pantalla de la eliminación de una extensión." lightbox="./media/remove-desired-state-configuration-package/delete-extension.png":::

## <a name="manually-delete-a-configuration-file-from-a-node"></a>Eliminación manual de un archivo de configuración de un nodo

Si no quiere usar Azure Portal, puede eliminar manualmente los archivos de configuración .mof como se muestra a continuación.

### <a name="delete-a-windows-configuration-using-powershell"></a>Eliminación de una configuración de Windows mediante PowerShell

Para quitar un documento de configuración de DSC importado (.mof), use el cmdlet [Remove-DscConfigurationDocument](/powershell/module/psdesiredstateconfiguration/remove-dscconfigurationdocument).

### <a name="delete-a-linux-configuration"></a>Eliminación de una configuración de Linux

Los archivos de configuración se almacenan en /etc/opt/omi/conf/dsc/configuration/. Quite los archivos .mof de este directorio para eliminar la configuración del nodo.

## <a name="remove-the-dsc-package-from-a-linux-node"></a>Eliminación del paquete DSC de un nodo de Linux

Este paso es opcional. Al anular el registro de un nodo de Linux de State Configuration (DSC) no se quitan los paquetes DSC y OMI de la máquina. Use los siguientes comandos para quitar los paquetes, así como todos los registros y datos relacionados.

Para buscar los nombres de paquetes y otros detalles pertinentes, consulte el repositorio de GitHub [Desired State Configuration de PowerShell para Linux](https://github.com/Microsoft/PowerShell-DSC-for-Linux).

### <a name="rpm-based-systems"></a>Sistemas basados en RPM

```bash
RPM -e <package name>
``` 

### <a name="dpkg-based-systems"></a>Sistemas basados en DPKG

```bash
dpkg -P <package name>
```

 ## <a name="next-steps"></a>Pasos siguientes

- Si quiere volver a registrar el nodo, o registrar uno nuevo, consulte [Registro de una máquina virtual para su administración mediante State Configuration](/azure/automation/tutorial-configure-servers-desired-state#register-a-vm-to-be-managed-by-state-configuration).

- Si quiere volver a agregar la configuración y volver a compilarla, consulte [Compilación de configuraciones de DSC en Azure Automation State Configuration](/azure/automation/automation-dsc-compile).