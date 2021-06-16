---
title: 'Tutorial: Diagnóstico de problemas de enrutamiento en una red de máquina virtual con Azure Portal'
titleSuffix: Azure Network Watcher
description: En este tutorial, aprenderá a diagnosticar un problema de enrutamiento de red de máquina virtual mediante la funcionalidad de próximo salto de Azure Network Watcher.
services: network-watcher
documentationcenter: network-watcher
author: damendo
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 01/07/2021
ms.author: damendo
ms.custom: mvc
ms.openlocfilehash: 7a056d3a998be5d8edee71557c0333f4c5cb8fd9
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112082368"
---
# <a name="tutorial-diagnose-a-virtual-machine-network-routing-problem-using-the-azure-portal"></a>Tutorial: Diagnosticar un problema de enrutamiento de red de máquina virtual con Azure Portal

Al implementar una máquina virtual (VM), Azure crea varias rutas predeterminadas para ella. Es posible crear rutas personalizadas para reemplazar las predeterminadas de Azure. En ocasiones, una ruta personalizada puede dar lugar a una VM que no pueda comunicarse con otros recursos. En este tutorial aprenderá a:

> [!div class="checklist"]
> * Crear una VM
> * Probar la comunicación con una dirección URL mediante la funcionalidad de próximo salto de Network Watcher
> * Probar la comunicación con una dirección IP
> * Diagnosticar un problema de enrutamiento y obtener información acerca de cómo resolverlo

Si lo prefiere, puede diagnosticar un problema de enrutamiento de red de máquina virtual mediante [Azure CLI](diagnose-vm-network-routing-problem-cli.md) o [Azure PowerShell](diagnose-vm-network-routing-problem-powershell.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-a-vm"></a>Crear una VM

1. Seleccione **+ Crear un recurso** en la esquina superior izquierda de Azure Portal.
2. Seleccione **Compute** y **Windows Server 2016 Datacenter** o **Ubuntu Server 17.10 VM**.
3. Escriba o seleccione la siguiente información, acepte los valores predeterminados para el resto de la configuración y luego seleccione **Aceptar**:

    |Configuración|Valor|
    |---|---|
    |Nombre|myVm|
    |Nombre de usuario| Escriba un nombre de usuario de su elección.|
    |Contraseña| Escriba una contraseña de su elección. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../virtual-machines/windows/faq.yml?toc=%2fazure%2fnetwork-watcher%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm-).|
    |Subscription| Seleccione su suscripción.|
    |Resource group| Haga clic en **Crear nuevo** y escriba **myResourceGroup**.|
    |Location| Seleccione **Este de EE. UU**.|

4. Seleccione un tamaño para la máquina virtual y luego **Seleccionar**.
5. En **Configuración**, acepte todos los valores predeterminados y seleccione **Aceptar**.
6. En **Crear** de la página **Resumen**, seleccione **Crear** para iniciar la implementación de la máquina virtual. La maquina virtual tarda unos minutos en implementarse. Espere a que la VM finalice la implementación antes de continuar con los pasos restantes.

## <a name="test-network-communication"></a>Prueba de la comunicación de red

Para probar la comunicación de red con Network Watcher, primero debe habilitar un monitor de red en al menos una región de Azure y, a continuación, usar la funcionalidad de próximo salto de Network Watcher para probar la comunicación.

### <a name="enable-network-watcher"></a>Habilitación de Network Watcher

Si ya dispone de un monitor de red habilitado en al menos una región, vaya a la sección [Usar próximo salto](#use-next-hop).

1. En el portal, seleccione **Todos los servicios**. En el **cuadro Filtrar**, escriba *Network Watcher*. Cuando aparezca la opción **Network Watcher** en los resultados, selecciónela.
2. Seleccione **Regiones** para expandirla y, a continuación, seleccione **...**  a la derecha de la opción **Este de EE. UU.** , tal y como se muestra en la siguiente imagen:

    ![Habilitación de Network Watcher](./media/diagnose-vm-network-traffic-filtering-problem/enable-network-watcher.png)

3. Seleccione **Habilitar Network Watcher**.

### <a name="use-next-hop"></a>Use el próximo salto

Azure crea rutas automáticamente a los destinos predeterminados. Es posible crear rutas personalizadas que reemplacen las rutas predeterminadas. En ocasiones, las rutas personalizadas pueden provocar errores de comunicación. Use la funcionalidad de próximo salto de Network Watcher para determinar la ruta que Azure utiliza para enrutar el tráfico.

1. En Azure Portal, seleccione **Próximo salto**, en **Network Watcher**.
2. Seleccione su suscripción, escriba o seleccione los siguientes valores y, a continuación, seleccione **Próximo salto**, tal y como se muestra en la siguiente figura:

    |Configuración                  |Valor                                                   |
    |---------                |---------                                               |
    | Resource group          | Seleccione myResourceGroup                                 |
    | Máquina virtual         | Seleccione myVm                                            |
    | interfaz de red       | myvm: el nombre de la interfaz de red puede ser diferente.   |
    | Dirección IP de origen       | 10.0.0.4                                               |
    | Dirección IP de destino  | 13.107.21.200: una de las direcciones para <www.bing.com>. |

    ![Próximo salto](./media/diagnose-vm-network-routing-problem/next-hop.png)

    Después de unos segundos, el resultado le informa que el tipo de próximo salto es **Internet** y que el **Id. de tabla de enrutamiento** es **Ruta del sistema**. Este resultado le permite saber que hay una ruta de sistema válida para el destino.

3. Cambie la **Dirección IP de destino** a *172.31.0.100* y seleccione **Próximo salto** de nuevo. El resultado que se devuelve le informa que **Ninguno** es el **Tipo de próximo salto** y que el **Id. de tabla de enrutamiento** también es **Ruta del sistema**. Este resultado le permite saber que, aunque no hay una ruta de sistema válida para el destino, no hay ningún próximo salto para enrutar el tráfico al destino.

## <a name="view-details-of-a-route"></a>Vista de los detalles de una ruta

1. Para analizar el enrutamiento detenidamente, revise las rutas efectivas para la interfaz de red. En el cuadro de búsqueda en la parte superior del portal, escriba *myvm* (o el nombre de la interfaz de red que haya comprobado). Seleccione **myvm** cuando aparezca en los resultados de búsqueda.
2. Seleccione **Rutas efectivas** en **SUPPORT + TROUBLESHOOTING** (Soporte técnico y solución de problemas), como se muestra en la siguiente imagen:

    ![Rutas eficaces](./media/diagnose-vm-network-routing-problem/effective-routes.png)

    Cuando ejecutó la prueba con 13.107.21.200 en [Próximo salto de uso](#use-next-hop), se utilizó la ruta con el prefijo de dirección 0.0.0.0/0 para enrutar el tráfico a la dirección, ya que ninguna otra ruta incluye la dirección. De forma predeterminada, todas las direcciones que no se especifican en el prefijo de dirección de otra ruta se enrutan a internet.

    Sin embargo, cuando ejecutó la prueba con 172.31.0.100, el resultado informó de que no había un tipo de próximo salto. Como puede ver en la imagen anterior, a pesar de que hay una ruta predeterminada para el prefijo 172.16.0.0/12, lo que incluye la dirección 172.31.0.100, el **TIPO DE PRÓXIMO SALTO** es **Ninguno**. Azure crea una ruta predeterminada a 172.16.0.0/12, pero no especifica un tipo de próximo salto hasta que haya un motivo para hacerlo. Por ejemplo, si se agrega el intervalo de dirección 172.16.0.0/12 al espacio de direcciones de la red virtual, Azure cambia el **TIPO DE PRÓXIMO SALTO** a **Red Virtual** para la ruta. A continuación, se mostraría una marca de verificación de **Red virtual** como el **TIPO DE PRÓXIMO SALTO**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no sea necesario, elimine el grupo de recursos y todos los recursos que contiene:

1. Escriba *myResourceGroup* en el cuadro **Buscar** que se encuentra en la parte superior del portal. Seleccione **myResourceGroup** cuando lo vea en los resultados de la búsqueda.
2. Seleccione **Eliminar grupo de recursos**.
3. Escriba *myResourceGroup* para **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS:** y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, se creó una VM y diagnosticó el enrutamiento de red a partir de la VM. Se informó que Azure crear varias rutas predeterminadas y prueba el enrutamiento a dos destinos diferentes. Obtenga más información sobre [enrutamiento en Azure](../virtual-network/virtual-networks-udr-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) y cómo [crear rutas personalizadas](../virtual-network/manage-route-table.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-route).

Para las conexiones de VM salientes, también puede determinar la latencia, tráfico de red permitido o denegado entre la VM y un punto de conexión y la ruta que se usa para un punto de conexión, mediante la funcionalidad de [resolución de problemas](network-watcher-connectivity-portal.md) de Network Watcher. Obtenga información acerca de cómo puede supervisar la comunicación entre una máquina virtual y un punto de conexión, como una dirección IP o una URL, en el transcurso del tiempo con la funcionalidad de supervisión de conexión de Network Watcher.

> [!div class="nextstepaction"]
> [Supervisar una conexión de red](connection-monitor.md)