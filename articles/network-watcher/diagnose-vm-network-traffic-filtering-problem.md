---
title: 'Inicio rápido: Diagnóstico de problemas de filtrado del tráfico de red de una máquina virtual: Azure Portal'
titleSuffix: Azure Network Watcher
description: En esta guía de inicio rápido, aprenderá a diagnosticar los problemas que surgen al filtrar el tráfico de las máquinas virtuales utilizando la funcionalidad Comprobación del flujo de IP de Azure Network Watcher.
services: network-watcher
documentationcenter: network-watcher
author: damendo
ms.author: damendo
editor: ''
ms.date: 04/20/2018
ms.assetid: ''
ms.topic: quickstart
ms.service: network-watcher
ms.workload: infrastructure
ms.tgt_pltfrm: network-watcher
ms.devlang: na
tags:
- azure-resource-manager
ms.custom:
- mvc
- mode-portal
ms.openlocfilehash: e8a9ce38e29f743fd40a6998d8bdc300080b2cee
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107537763"
---
# <a name="quickstart-diagnose-a-virtual-machine-network-traffic-filter-problem-using-the-azure-portal"></a>Inicio rápido: Diagnóstico de problemas al filtrar el tráfico de red de las máquinas virtuales con Azure Portal

En esta guía de inicio rápido, va a implementar una máquina virtual y a comprobar las comunicaciones entre una dirección IP y una dirección URL u otra dirección IP. Además, va a determinar la causa de un error de comunicación y cómo puede resolverlo.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-a-vm"></a>Crear una VM

1. Seleccione **+ Crear un recurso** en la esquina superior izquierda de Azure Portal.
2. Seleccione **Proceso** y, luego, **Windows Server 2016 Datacenter** o una versión de **Ubuntu Server**.
3. Escriba o seleccione la siguiente información, acepte los valores predeterminados para el resto de la configuración y luego seleccione **Aceptar**:

    |Configuración|Value|
    |---|---|
    |Nombre|myVm|
    |Nombre de usuario| Escriba un nombre de usuario de su elección.|
    |Contraseña| Escriba una contraseña de su elección. La contraseña debe tener un mínimo de 12 caracteres y cumplir los requisitos de complejidad definidos.|
    |Suscripción| Seleccione su suscripción.|
    |Resource group| Haga clic en **Crear nuevo** y escriba **myResourceGroup**.|
    |Location| Seleccione **Este de EE. UU**.|

4. Seleccione un tamaño para la máquina virtual y luego **Seleccionar**.
5. En **Configuración**, acepte todos los valores predeterminados y seleccione **Aceptar**.
6. En **Crear** de la página **Resumen**, seleccione **Crear** para iniciar la implementación de la máquina virtual. La maquina virtual tarda unos minutos en implementarse. Espere a que la VM finalice la implementación antes de continuar con los pasos restantes.

## <a name="test-network-communication"></a>Prueba de la comunicación de red

Para probar la comunicación de red con Network Watcher, primero se habilita un monitor de red en al menos una región de Azure y, a continuación, se usa la funcionalidad Comprobación del flujo de IP de Network Watcher.

### <a name="enable-network-watcher"></a>Habilitación de Network Watcher

Si ya dispone de un monitor de red habilitado en al menos una región, vaya a la sección [Uso de la funcionalidad Comprobación del flujo de IP](#use-ip-flow-verify).

1. En el portal, seleccione **Todos los servicios**. En el **cuadro Filtrar**, escriba *Network Watcher*. Cuando aparezca la opción **Network Watcher** en los resultados, selecciónela.
2. Habilite un monitor de red en la región Este de EE. UU., porque esa es la región de la máquina virtual que se implementó en un paso anterior. Seleccione **Regiones** para expandirla y, a continuación, seleccione **...**  a la derecha de la opción **Este de EE. UU.** , tal y como se muestra en la siguiente imagen:

    ![Habilitación de Network Watcher](./media/diagnose-vm-network-traffic-filtering-problem/enable-network-watcher.png)

3. Seleccione **Habilitar Network Watcher**.

### <a name="use-ip-flow-verify"></a>Uso de la funcionalidad Comprobación del flujo de IP

Cuando se crea una máquina virtual, Azure permite y deniega el tráfico de entrada y salida de la máquina virtual conforme a unos valores predeterminados. Si lo desea, puede invalidar después los valores predeterminados de Azure para permitir o denegar otros tipos de tráfico.

1. En el portal, seleccione **Todos los servicios**. En el cuadro **Todos los servicios** *Filtro*, escriba *Network Watcher*. Cuando aparezca la opción **Network Watcher** en los resultados, selecciónela.
2. Seleccione **Comprobación del flujo de IP** en **HERRAMIENTAS DE DIAGNÓSTICO DE RED**.
3. Seleccione su suscripción, escriba o seleccione los siguientes valores y seleccione **Comprobar**, tal y como se muestra en la siguiente imagen:

    |Configuración            |Value                                                                                              |
    |---------          |---------                                                                                          |
    | Resource group    | Seleccione myResourceGroup                                                                            |
    | Máquina virtual   | Seleccione myVm                                                                                       |
    | interfaz de red | myVm: el nombre de la interfaz de red del portal que se creó al crear la máquina virtual es diferente. |
    | Protocolo          | TCP                                                                                               |
    | Dirección         | Salida                                                                                          |
    | Dirección IP local  | 10.0.0.4                                                                                          |
    | Puerto local      | 60000                                                                                                |
    | Dirección IP remota | 13.107.21.200: una de las direcciones para <www.bing.com>.                                             |
    | Puerto remoto       | 80                                                                                                |

    ![Comprobación de flujo de IP](./media/diagnose-vm-network-traffic-filtering-problem/ip-flow-verify-outbound.png)

    Después de unos segundos, el resultado devuelto le informa de que una regla de seguridad llamada **AllowInternetOutbound** ha permitido el acceso. Al ejecutar la comprobación, Network Watcher crea automáticamente un monitor de red en la región Este de EE.UU., si tuviera un monitor de red en una región distinta de la región Este de EE. UU. antes de ejecutar la comprobación.
4. Realice el paso 3 de nuevo con la **Dirección IP remota** cambiada a **172.31.0.100**. El resultado devuelto le informa de que una regla de seguridad llamada **DefaultOutboundDenyAll** ha denegado el acceso.
5. Realice el paso 3 de nuevo con la **Dirección** cambiada a **Entrada**, el **Puerto Local** cambiado a **80** y el **Puerto remoto**, a **60000**. El resultado devuelto le informa de que una regla de seguridad llamada **DefaultInboundDenyAll** ha denegado el acceso.

Ahora que sabe qué reglas de seguridad están permitiendo o denegando el tráfico de entrada y salida en una máquina virtual, puede determinar cómo deben resolverse los problemas.

## <a name="view-details-of-a-security-rule"></a>Ver detalles de una regla de seguridad

1. Para determinar por qué las reglas de los pasos 3 a 5 de **Uso de la funcionalidad Comprobación del flujo de IP** permiten o deniegan la comunicación, revise las reglas de seguridad eficaces de la interfaz de red en la máquina virtual. En el cuadro de búsqueda que aparece en la parte superior del portal, escriba *myvm*. Cuando la interfaz de red **myvm** (o cualquiera que sea el nombre de la interfaz de red) aparezca en los resultados de la búsqueda, selecciónela.
2. Seleccione **Reglas de seguridad eficaces** en **SUPPORT + TROUBLESHOOTING** (Soporte técnico y solución de problemas), como se muestra en la siguiente imagen:

    ![Reglas de seguridad eficaces](./media/diagnose-vm-network-traffic-filtering-problem/effective-security-rules.png)

    En el paso 3 de **Uso de la funcionalidad Comprobación del flujo de IP** ha aprendido que el motivo por el se permitió la comunicación era la regla **AllowInternetOutbound**. En la imagen anterior puede ver que el **DESTINO** de la regla es **Internet**. Sin embargo, no queda claro el modo en que 13.107.21.200, la dirección que probó en el paso 3 de **Uso de la funcionalidad Comprobación del flujo de IP** está relacionada con **Internet**.
3. Seleccione la regla **AllowInternetOutBound** y **Destino**, tal y como se muestra en la siguiente imagen:

    ![Prefijos de regla de seguridad](./media/diagnose-vm-network-traffic-filtering-problem/security-rule-prefixes.png)

    Uno de los prefijos de la lista es **12.0.0.0/6**, que comprende el intervalo de direcciones IP de 12.0.0.1 a 15.255.255.254. Como 13.107.21.200 está dentro de ese intervalo, la regla **AllowInternetOutBound** permite el tráfico de salida. Asimismo, no aparece ninguna regla con prioridad mayor (número menor) en la imagen del paso 2 que anule esta regla. Cierre el cuadro **Prefijos de direcciones**. Para denegar la comunicación de salida con 13.107.21.200, podría agregar una regla de seguridad con una prioridad mayor que denegara la salida del puerto 80 hacia la dirección IP.
4. Al ejecutar la comprobación de salida para 172.131.0.100 en el paso 4 de **Uso de la funcionalidad Comprobación del flujo de IP**, ha aprendido que la regla **DefaultOutboundDenyAll** denegaba la comunicación. Esta regla equivale a la regla **DenyAllOutBound** que se muestra en la imagen del paso 2 que especifica **0.0.0.0/0** como **DESTINO**. La regla deniega la comunicación de salida hacia 172.131.0.100 porque la dirección no está incluida como **DESTINO** en las **Reglas de salida** que aparecen en la imagen. Para permitir la comunicación de salida, podría agregar una regla de seguridad con prioridad mayor que permitiera el tráfico de salida hacia el puerto 80 para la dirección 172.131.0.100.
5. Al ejecutar la comprobación de entrada para 172.131.0.100 en el paso 5 de **Uso de la funcionalidad Comprobación del flujo de IP**, ha aprendido que la regla **DefaultInboundDenyAll** denegaba la comunicación. Esta regla equivale a la regla **DenyAllInBound** que se muestra en la imagen del paso 2. La regla **DenyAllInBound** se aplicaba porque no existía ninguna otra regla de prioridad mayor que permitiera el puerto 80 de entrada a la máquina virtual desde 172.31.0.100. Para permitir la comunicación de entrada, podría agregar una regla de seguridad con una prioridad mayor que permitiera el tráfico de entrada en el puerto 80 del tráfico procedente de 172.31.0.100.

Las pruebas de esta guía de inicio rápido permiten comprobar la configuración de Azure. Si estas pruebas devuelven resultados esperados pero sigue teniendo problemas de red, asegúrese de que no hay un firewall entre la máquina virtual y el punto de conexión con el que se está comunicando y que el sistema operativo de la máquina virtual no tiene un firewall que permita o deniegue la comunicación.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no sea necesario, elimine el grupo de recursos y todos los recursos que contiene:

1. Escriba *myResourceGroup* en el cuadro **Buscar** que se encuentra en la parte superior del portal. Seleccione **myResourceGroup** cuando lo vea en los resultados de la búsqueda.
2. Seleccione **Eliminar grupo de recursos**.
3. Escriba *myResourceGroup* para **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS:** y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado una máquina virtual y diagnosticado los filtros del tráfico de entrada y de salida. Ha aprendido que las reglas de grupo de seguridad de red permiten o deniegan el tráfico de entrada y salida de una máquina virtual. Más información acerca de las [reglas de seguridad](../virtual-network/network-security-groups-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) y [cómo crearlas](../virtual-network/manage-network-security-group.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-security-rule).

Incluso con la aplicación de los filtros de tráfico adecuados, la comunicación con una máquina virtual puede seguir experimentando errores debido a la configuración del enrutamiento. Para más información acerca de cómo diagnosticar los problemas de enrutamiento de red de la máquina virtual, consulte [Diagnose VM routing problems](diagnose-vm-network-routing-problem.md) (Diagnóstico de problemas de enrutamiento de máquina virtual). Para diagnosticar problemas con los filtros de tráfico, la latencia o el enrutamiento de salida mediante una herramienta, consulte [Solución de problemas de conexiones](network-watcher-connectivity-portal.md).
