---
title: Uso de puntos de conexión privados para integrar Azure Functions con una red virtual
description: En este tutorial se explica cómo conectar una función a una red virtual de Azure y cómo bloquearla con puntos de conexión privados.
ms.topic: article
ms.date: 2/22/2021
ms.openlocfilehash: 0f18712e9881c60754d5729751609f6458104daf
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109715491"
---
# <a name="tutorial-integrate-azure-functions-with-an-azure-virtual-network-by-using-private-endpoints"></a>Tutorial: Integración de Azure Functions con una red virtual de Azure mediante puntos de conexión privados

En este tutorial se explica cómo se utiliza Azure Functions para conectarse a recursos de una red virtual de Azure mediante puntos de conexión privados. Creará una función mediante una cuenta de almacenamiento que está bloqueada tras una red virtual. La red virtual usa un desencadenador de cola de Service Bus.

En este tutorial, hará lo siguiente:

> [!div class="checklist"]
> * Crear una aplicación de funciones en el plan Premium.
> * Crear recursos de Azure, como la instancia de Service Bus, la cuenta de almacenamiento y la red virtual.
> * Bloquear la cuenta de almacenamiento tras un punto de conexión privado.
> * Bloquear la instancia de Service Bus tras un punto de conexión privado.
> * Implementar una aplicación de funciones que usa desencadenadores HTTP y de Service Bus.
> * Bloquear la aplicación de funciones tras un punto de conexión privado.
> * Probar para comprobar si la aplicación de funciones está protegida dentro de la red virtual.
> * Limpieza de recursos.

## <a name="create-a-function-app-in-a-premium-plan"></a>Creación de una aplicación de funciones en un plan Premium

Creará una aplicación de funciones para .NET en el plan Premium, porque en este tutorial se usa C#. En Windows también se admiten otros lenguajes. El plan Premium ofrece escala sin servidor, al mismo tiempo que admite la integración de red virtual.

1. En el menú de Azure Portal o la página **Inicio**, seleccione **Crear un recurso**.

1. En la página **Nuevo**, seleccione **Proceso** > **Aplicación de funciones**.

1. En la página **Aspectos básicos**, use la tabla siguiente para configurar la aplicación de funciones.

    | Configuración      | Valor sugerido  | Descripción |
    | ------------ | ---------------- | ----------- |
    | **Suscripción** | Su suscripción | Suscripción en la que se creará esta aplicación de funciones nueva. |
    | **[Grupo de recursos](../azure-resource-manager/management/overview.md)** |  myResourceGroup | Nombre del grupo de recursos nuevo en el que se creará la aplicación de funciones. |
    | **Nombre de la aplicación de función** | Nombre único globalmente | Nombre que identifica la nueva Function App. Los caracteres válidos son `a-z` (no distingue mayúsculas de minúsculas), `0-9` y `-`.  |
    |**Publicar**| Código | Elija publicar archivos de código o un contenedor de Docker. |
    | **Pila en tiempo de ejecución** | .NET | En este tutorial se usa .NET. |
    |**Región**| Región preferida | Elija una [región](https://azure.microsoft.com/regions/) cerca de usted o cerca de otros servicios a los que tendrán acceso las funciones. |

1. Seleccione **Siguiente: Hospedaje**. En la página **Hospedaje**, escriba la siguiente configuración.

    | Configuración      | Valor sugerido  | Descripción |
    | ------------ | ---------------- | ----------- |
    | **[Cuenta de almacenamiento](../storage/common/storage-account-create.md)** |  Nombre único globalmente |  Cree una cuenta de almacenamiento que use la aplicación de función. Los nombres de cuenta de almacenamiento deben tener entre 3 y 24 caracteres. Solo pueden contener números y letras minúsculas. También puede usar una cuenta existente que debe cumplir los [requisitos de la cuenta de almacenamiento](./storage-considerations.md#storage-account-requirements). |
    |**Sistema operativo**| Windows | En este tutorial se usa Windows. |
    | **[Plan](./functions-scale.md)** | Premium | Plan de hospedaje que define cómo se asignan los recursos a la Function App. De manera predeterminada, cuando seleccione **Premium**, se crea un plan de App Service. El valor predeterminado de **SKU y tamaño** es **EP1**, donde *EP* son las siglas de _Elástico Premium_. Para más información, consulte la lista de las [SKU Premium](./functions-premium-plan.md#available-instance-skus).<br/><br/>Al ejecutar las funciones de JavaScript en un plan Premium, debe elegir una instancia que tenga menos vCPU. Para más información, consulte el apartado sobre la [elección de planes Premium de un solo núcleo](./functions-reference-node.md#considerations-for-javascript-functions).  |

1. Seleccione **Siguiente: Supervisión**. En la página **Supervisión**, escriba la siguiente configuración.

    | Configuración      | Valor sugerido  | Descripción |
    | ------------ | ---------------- | ----------- |
    | **[Application Insights](./functions-monitoring.md)** | Valor predeterminado | Cree un recurso de Application Insights con el mismo nombre de aplicación en la región más cercana compatible. Expanda esta configuración si tiene que cambiar el **Nuevo nombre de recurso** o almacenar los datos en una **Ubicación** distinta en una [geografía de Azure](https://azure.microsoft.com/global-infrastructure/geographies/). |

1. Seleccione **Revisar y crear** para revisar las selecciones de configuración de la aplicación.

1. En la página **Revisar y crear**, revise la configuración. Luego, seleccione **Crear** para aprovisionar e implementar la aplicación de funciones.

1. En la esquina superior derecha del portal, seleccione el icono **Notificaciones** y observe el mensaje **Implementación correcta**.

1. Seleccione **Ir al recurso** para ver la nueva aplicación de función. También puede seleccionar **Anclar al panel**. Dicho anclaje facilita la vuelta a este recurso de aplicación de función desde el panel.

¡Enhorabuena! Creó correctamente la aplicación de funciones prémium.

## <a name="create-azure-resources"></a>Creación de recursos de Azure

A continuación, creará una cuenta de almacenamiento, una instancia de Service Bus y una red virtual. 
### <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Las redes virtuales necesitarán una cuenta de almacenamiento independiente de la que creó con la aplicación de funciones.

1. En el menú de Azure Portal o la página **Inicio**, seleccione **Crear un recurso**.

1. En la página **Nuevo**, busque *cuenta de almacenamiento*. Seleccione **Crear**.

1. En la pestaña **Aspectos básicos**, use la tabla siguiente para configurar las opciones de la cuenta de almacenamiento. Todas las demás opciones de configuración pueden usar los valores predeterminados.

    | Configuración      | Valor sugerido  | Descripción      |
    | ------------ | ---------------- | ---------------- |
    | **Suscripción** | Su suscripción | Suscripción en la que se han creado los recursos. | 
    | **[Grupo de recursos](../azure-resource-manager/management/overview.md)**  | myResourceGroup | El grupo de recursos que creó con la aplicación de funciones. |
    | **Nombre** | mysecurestorage| El nombre de la cuenta de almacenamiento a la que se aplicará el punto de conexión privado. |
    | **[Región](https://azure.microsoft.com/regions/)** | myFunctionRegion | La región en la que creó la aplicación de funciones. |

1. Seleccione **Revisar + crear**. Una vez que se complete la validación, seleccione **Crear**.

### <a name="create-a-service-bus"></a>Creación de un bus de servicio

1. En el menú de Azure Portal o la página **Inicio**, seleccione **Crear un recurso**.

1. En la página **Nuevo**, busque *Service Bus*. Seleccione **Crear**.

1. En la pestaña **Aspectos básicos**, use la tabla siguiente para configurar las opciones de Service Bus. Todas las demás opciones de configuración pueden usar los valores predeterminados.

    | Configuración      | Valor sugerido  | Descripción      |
    | ------------ | ---------------- | ---------------- |
    | **Suscripción** | Su suscripción | Suscripción en la que se han creado los recursos. |
    | **[Grupo de recursos](../azure-resource-manager/management/overview.md)**  | myResourceGroup | El grupo de recursos que creó con la aplicación de funciones. |
    | **Nombre del espacio de nombres** | myServiceBus| Nombre de la instancia de Service Bus al que se aplicará el punto de conexión privado. |
    | **[Ubicación](https://azure.microsoft.com/regions/)** | myFunctionRegion | La región en la que creó la aplicación de funciones. |
    | **Plan de tarifa** | Premium | Elija este nivel para usar puntos de conexión privados con Azure Service Bus. |

1. Seleccione **Revisar + crear**. Una vez que se complete la validación, seleccione **Crear**.

### <a name="create-a-virtual-network"></a>Creación de una red virtual

Los recursos de Azure de este tutorial se integran o se incluyen en una red virtual. Usará puntos de conexión privados para contener el tráfico de red dentro de la red virtual.

En el tutorial se crean dos subredes:
- **default**: subred para los puntos de conexión privados. Desde esta subred se proporcionan direcciones IP privadas.
- **functions**: subred para la integración de la red virtual de Azure Functions. Esta subred se delega a la aplicación de funciones.

Cree la red virtual con la que se integra la aplicación de funciones:

1. En el menú de Azure Portal o la página **Inicio**, seleccione **Crear un recurso**.

1. En la página **Nuevo**, busque *red virtual*. Seleccione **Crear**.

1. En la pestaña **Aspectos básicos**, use la tabla siguiente para configurar las opciones de la red virtual.

    | Configuración      | Valor sugerido  | Descripción      |
    | ------------ | ---------------- | ---------------- |
    | **Suscripción** | Su suscripción | Suscripción en la que se han creado los recursos. | 
    | **[Grupo de recursos](../azure-resource-manager/management/overview.md)**  | myResourceGroup | El grupo de recursos que creó con la aplicación de funciones. |
    | **Nombre** | myVirtualNet| Nombre de la red virtual a la que se conectará la aplicación de funciones. |
    | **[Región](https://azure.microsoft.com/regions/)** | myFunctionRegion | La región en la que creó la aplicación de funciones. |

1. En la pestaña **Direcciones IP**, seleccione **Agregar subred**. Use la tabla siguiente para configurar las opciones de subred.

    :::image type="content" source="./media/functions-create-vnet/1-create-vnet-ip-address.png" alt-text="Captura de pantalla de la vista de configuración de Crear red virtual.":::

    | Configuración      | Valor sugerido  | Descripción      |
    | ------------ | ---------------- | ---------------- |
    | **Nombre de subred** | functions | Nombre de la subred a la que se conectará la aplicación de funciones. | 
    | **Intervalo de direcciones de subred** | 10.0.1.0/24 | El intervalo de direcciones de subred. En la imagen anterior, observe que el espacio de direcciones IPv4 es 10.0.0.0/16. Si el valor fuese 10.1.0.0/16, el intervalo de direcciones de subred recomendado sería 10.1.1.0/24. |

1. Seleccione **Revisar + crear**. Una vez que se complete la validación, seleccione **Crear**.

## <a name="lock-down-your-storage-account"></a>Bloqueo de la cuenta de almacenamiento

Los puntos de conexión privados de Azure se usan para conectarse a recursos específicos de Azure mediante una dirección IP privada. Esta conexión garantiza que el tráfico de red permanece dentro de la red virtual elegida y que el acceso solo está disponible para recursos específicos. 

Cree los puntos de conexión privados para Azure File Storage y Azure Blob Storage mediante la cuenta de almacenamiento:

1. En el menú de la izquierda de la cuenta de almacenamiento nueva, seleccione **Redes**.

1. En la pestaña **Conexiones de puntos de conexión privados**, seleccione **Punto de conexión privado**.

    :::image type="content" source="./media/functions-create-vnet/2-navigate-private-endpoint-store.png" alt-text="Captura de pantalla de la creación de puntos de conexión privados para la cuenta de almacenamiento.":::

1. En la pestaña **Aspectos básicos**, use la configuración de punto de conexión privado que se muestra en la tabla siguiente.

    | Configuración      | Valor sugerido  | Descripción      |
    | ------------ | ---------------- | ---------------- |
    | **Suscripción** | Su suscripción | Suscripción en la que se han creado los recursos. | 
    | **[Grupo de recursos](../azure-resource-manager/management/overview.md)**  | myResourceGroup | Elija el grupo de recursos que ha creado con la aplicación de funciones. |
    | **Nombre** | file-endpoint | El nombre del punto de conexión privado para los archivos de la cuenta de almacenamiento. |
    | **[Región](https://azure.microsoft.com/regions/)** | myFunctionRegion | Elija la región en la que creó la cuenta de almacenamiento. |

1. En la pestaña **Recurso**, use la configuración de punto de conexión privado que se muestra en la tabla siguiente.

    | Configuración      | Valor sugerido  | Descripción      |
    | ------------ | ---------------- | ---------------- |
    | **Suscripción** | Su suscripción | Suscripción en la que se han creado los recursos. | 
    | **Tipo de recurso**  | Microsoft.Storage/storageAccounts | El tipo de recurso de las cuentas de almacenamiento. |
    | **Recurso** | mysecurestorage | La cuenta de almacenamiento que creó. |
    | **Recurso secundario de destino** | archivo | El punto de conexión privado se usará para los archivos de la cuenta de almacenamiento. |

1. En la pestaña **Configuración**, en **Subred**, elija **Valor predeterminado**.

1. Seleccione **Revisar + crear**. Una vez que se complete la validación, seleccione **Crear**. Los recursos de la red virtual ahora pueden comunicarse con los archivos de almacenamiento.

1. Cree otro punto de conexión privado para los blobs. En la pestaña **Recursos**, use la configuración que se muestra en la tabla siguiente. Para todas las demás opciones, utilice los mismos valores que usó para crear el punto de conexión privado para los archivos.

    | Configuración      | Valor sugerido  | Descripción      |
    | ------------ | ---------------- | ---------------- |
    | **Suscripción** | Su suscripción | Suscripción en la que se han creado los recursos. | 
    | **Tipo de recurso**  | Microsoft.Storage/storageAccounts | El tipo de recurso de las cuentas de almacenamiento. |
    | **Nombre** | blob-endpoint | Nombre del punto de conexión privado para los blobs de la cuenta de almacenamiento. |
    | **Recurso** | mysecurestorage | La cuenta de almacenamiento que creó. |
    | **Recurso secundario de destino** | blob | El punto de conexión privado se usará para los blobs de la cuenta de almacenamiento. |
1. Una vez creados los puntos de conexión privados, vuelva a la sección **Firewalls y redes virtuales** de la cuenta de almacenamiento.  
1. Asegúrese de que esté seleccionada la opción **Redes seleccionadas**.  No es necesario agregar una red virtual existente.

Los recursos de la red virtual ahora pueden comunicarse con la cuenta de almacenamiento mediante el punto de conexión privado.
## <a name="lock-down-your-service-bus"></a>Bloqueo de Service Bus

Cree el punto de conexión privado para bloquear la instancia de Service Bus:

1. En el menú de la izquierda de la instancia de Service Bus nueva, seleccione **Redes**.

1. En la pestaña **Conexiones de puntos de conexión privados**, seleccione **Punto de conexión privado**.

    :::image type="content" source="./media/functions-create-vnet/3-navigate-private-endpoint-service-bus.png" alt-text="Captura de pantalla de cómo ir a los puntos de conexión privados para la instancia de Service Bus.":::

1. En la pestaña **Aspectos básicos**, use la configuración de punto de conexión privado que se muestra en la tabla siguiente.

    | Configuración      | Valor sugerido  | Descripción      |
    | ------------ | ---------------- | ---------------- |
    | **Suscripción** | Su suscripción | Suscripción en la que se han creado los recursos. | 
    | **[Grupo de recursos](../azure-resource-manager/management/overview.md)**  | myResourceGroup | El grupo de recursos que creó con la aplicación de funciones. |
    | **Nombre** | sb-endpoint | El nombre del punto de conexión privado para los archivos de la cuenta de almacenamiento. |
    | **[Región](https://azure.microsoft.com/regions/)** | myFunctionRegion | La región en la que creó la cuenta de almacenamiento. |

1. En la pestaña **Recurso**, use la configuración de punto de conexión privado que se muestra en la tabla siguiente.

    | Configuración      | Valor sugerido  | Descripción      |
    | ------------ | ---------------- | ---------------- |
    | **Suscripción** | Su suscripción | Suscripción en la que se han creado los recursos. | 
    | **Tipo de recurso**  | Microsoft.ServiceBus/namespaces | Tipo de recurso de la instancia de Service Bus. |
    | **Recurso** | myServiceBus | El bus de servicio que ha creado anteriormente en el tutorial. |
    | **Subrecurso de destino** | namespace | Punto de conexión privado que se usará para el espacio de nombres de la instancia de Service Bus. |

1. En la pestaña **Configuración**, en **Subred**, elija **Valor predeterminado**.

1. Seleccione **Revisar + crear**. Una vez que se complete la validación, seleccione **Crear**. 
1. Una vez creado el punto de conexión privado, vuelva a la sección **Firewalls y redes virtuales** del espacio de nombres de Service Bus.
1. Asegúrese de que esté seleccionada la opción **Redes seleccionadas**.
1. Seleccione **+ Agregar red virtual existente** para agregar la red virtual creada recientemente.
1. En la pestaña **Agregar redes**, use la configuración de red de la tabla siguiente:

    | Configuración | Valor sugerido | Descripción|
    |---------|-----------------|------------|
    | **Suscripción** | Su suscripción | Suscripción en la que se han creado los recursos. |
    | **Redes virtuales** | myVirtualNet | Nombre de la red virtual a la que se conectará la aplicación de funciones. |
    | **Subredes** | functions | Nombre de la subred a la que se conectará la aplicación de funciones. |

1. Seleccione **Agregar la dirección IP del cliente** para dar acceso a esa IP de cliente actual al espacio de nombres.
    > [!NOTE]
    > Es necesario habilitar la dirección IP del cliente para permitir que Azure Portal [publique mensajes en la cola más adelante en este tutorial](#test-your-locked-down-function-app).
1. Seleccione **Habilitar** para habilitar el punto de conexión de servicio.
1. Seleccione **Agregar** para agregar la red virtual y la subred seleccionadas a las reglas de firewall para la instancia de Service Bus.
1. Seleccione **Guardar** para almacenar las reglas de firewall actualizadas.

Los recursos de la red virtual ahora pueden comunicarse con la instancia de Service Bus mediante el punto de conexión privado.

## <a name="create-a-file-share"></a>Creación de un recurso compartido de archivos

1. En el menú de la izquierda de la cuenta de almacenamiento que creó, seleccione **Recursos compartidos de archivos**.

1. Seleccione **+ Recursos compartidos de archivos**. Para los fines de este tutorial, asigne el nombre *files* al recurso compartido de archivos.

    :::image type="content" source="./media/functions-create-vnet/4-create-file-share.png" alt-text="Captura de pantalla sobre cómo crear un recurso compartido de archivos en la cuenta de almacenamiento.":::

1. Seleccione **Crear**.

## <a name="get-the-storage-account-connection-string"></a>Obtención de la cadena de conexión de la cuenta de almacenamiento

1. En el menú de la izquierda de la cuenta de almacenamiento que creó, seleccione **Claves de acceso**.

1. Seleccione **Mostrar claves**. Copie la cadena de conexión de **key1** y guárdela. Necesitará esta cadena de conexión cuando configure los valores de la aplicación.

    :::image type="content" source="./media/functions-create-vnet/5-get-store-connection-string.png" alt-text="Captura de pantalla sobre cómo obtener la cadena de conexión de una cuenta de almacenamiento.":::

## <a name="create-a-queue"></a>Creación de una cola

Cree la cola en la que el desencadenador de Azure Functions Service Bus obtendrá los eventos:

1. En el menú de la izquierda de la instancia de Service Bus, seleccione **Colas**.

1. Seleccione **Cola**. Para los fines de este tutorial, especifique *queue* como nombre de la nueva cola.

    :::image type="content" source="./media/functions-create-vnet/6-create-queue.png" alt-text="Captura de pantalla sobre cómo crear una cola de Service Bus.":::

1. Seleccione **Crear**.

## <a name="get-a-service-bus-connection-string"></a>Obtención de la cadena de conexión de Service Bus

1. En el menú de la izquierda de Service Bus, seleccione **Directivas de acceso compartido**.

1. Seleccione **RootManageSharedAccessKey**. Copie la **Cadena de conexión principal** y guárdela. Necesitará esta cadena de conexión cuando configure los valores de la aplicación.

    :::image type="content" source="./media/functions-create-vnet/7-get-service-bus-connection-string.png" alt-text="Captura de pantalla sobre cómo obtener la cadena de conexión de Service Bus.":::

## <a name="integrate-the-function-app"></a>Integración de la aplicación de funciones

Para usar la aplicación de funciones con redes virtuales, debe unirla a una subred. Usará una subred específica para la integración de la red virtual de Azure Functions. Usará la subred predeterminada para otros puntos de conexión privados que cree en este tutorial.

1. En el menú de la izquierda de la aplicación de funciones, seleccione **Redes**.

1. En **Integración de red virtual**, seleccione **Haga clic aquí para configurar**.

    :::image type="content" source="./media/functions-create-vnet/8-connect-app-vnet.png" alt-text="Captura de pantalla de cómo ir a la integración de la red virtual.":::

1. Seleccione **Agregar VNET**.

1. En **Red virtual**, seleccione la red virtual que creó anteriormente.

1. Seleccione la subred de **funciones** que creó anteriormente. Seleccione **Aceptar**.  Ahora la aplicación de funciones está integrada con la red virtual.

    :::image type="content" source="./media/functions-create-vnet/9-connect-app-subnet.png" alt-text="Captura de pantalla sobre cómo conectar una aplicación de funciones a una subred.":::

## <a name="configure-your-function-app-settings"></a>Configuración de los valores de la aplicación de funciones

1. En el menú de la izquierda de la aplicación de funciones, seleccione **Configuración**.

1. Para usar la aplicación de funciones con redes virtuales, actualice la configuración de la aplicación que se muestra en la tabla siguiente. Para agregar o editar una configuración, seleccione **+ Nueva configuración de la aplicación** o el icono **Editar** en la columna situada más a la derecha de la tabla de configuración de la aplicación. Cuando termine, seleccione **Save** (Guardar).

    :::image type="content" source="./media/functions-create-vnet/10-configure-app-settings.png" alt-text="Captura de pantalla sobre cómo configurar las opciones de la aplicación de funciones para puntos de conexión privados.":::

    | Configuración      | Valor sugerido  | Descripción      |
    | ------------ | ---------------- | ---------------- |
    | **AzureWebJobsStorage** | mysecurestorageConnectionString | La cadena de conexión de la cuenta de almacenamiento que ha creado. Esta cadena de conexión de almacenamiento proviene de la sección [Obtención de la cadena de conexión de la cuenta de almacenamiento](#get-the-storage-account-connection-string). Esta configuración permite que la aplicación de funciones utilice la cuenta de almacenamiento segura para las operaciones habituales en tiempo de ejecución. | 
    | **WEBSITE_CONTENTAZUREFILECONNECTIONSTRING**  | mysecurestorageConnectionString | La cadena de conexión de la cuenta de almacenamiento que ha creado. Esta configuración permite que la aplicación de funciones utilice la cuenta de almacenamiento segura para Azure Files, que se usa durante la implementación. |
    | **WEBSITE_CONTENTSHARE** | archivos | El nombre del recurso compartido de archivos creado en la cuenta de almacenamiento. Use esta configuración con WEBSITE_CONTENTAZUREFILECONNECTIONSTRING. |
    | **SERVICEBUS_CONNECTION** | myServiceBusConnectionString | Cree una configuración de aplicación para la cadena de conexión de la instancia de Service Bus. Esta cadena de conexión de almacenamiento proviene de la sección [Obtención de la cadena de conexión de Service Bus](#get-a-service-bus-connection-string).|
    | **WEBSITE_CONTENTOVERVNET** | 1 | Cree esta configuración de aplicación. Un valor de 1 permite escalar la aplicación de funciones cuando la cuenta de almacenamiento está restringida a una red virtual. |
    | **WEBSITE_DNS_SERVER** | 168.63.129.16 | Cree esta configuración de aplicación. Cuando la aplicación se integre con una red virtual, usará el mismo servidor DNS que esta. La aplicación de funciones necesita esta configuración para poder trabajar con zonas privadas de Azure DNS. Se requiere cuando se utilizan puntos de conexión privados. Esta configuración y WEBSITE_VNET_ROUTE_ALL enviarán todas las llamadas salientes desde la aplicación a la red virtual. |
    | **WEBSITE_VNET_ROUTE_ALL** | 1 | Cree esta configuración de aplicación. Cuando la aplicación se integra con una red virtual, usa el mismo servidor DNS que esta. La aplicación de funciones necesita esta configuración para poder trabajar con zonas privadas de Azure DNS. Se requiere cuando se utilizan puntos de conexión privados. Esta configuración y WEBSITE_DNS_SERVER enviarán todas las llamadas salientes desde la aplicación a la red virtual. |

1. En la vista **Configuración**, seleccione la pestaña **Configuración del tiempo de ejecución de la función**.

1. Establezca **Supervisión de la escala en tiempo de ejecución** en **Activada**. Después, seleccione **Guardar**. El escalado controlado por el entorno de ejecución permite conectar funciones de desencadenador no HTTP a los servicios que se ejecutan dentro de la red virtual.

    :::image type="content" source="./media/functions-create-vnet/11-enable-runtime-scaling.png" alt-text="Captura de pantalla sobre cómo habilitar el escalado controlado por el entorno de ejecución para Azure Functions.":::

## <a name="deploy-a-service-bus-trigger-and-http-trigger"></a>Implementación de un desencadenador de Service Bus y un desencadenador HTTP

1. En GitHub, vaya al repositorio de ejemplo siguiente. Contiene una aplicación de funciones y dos funciones, un desencadenador HTTP y un desencadenador de cola de Service Bus.

    <https://github.com/Azure-Samples/functions-vnet-tutorial>

1. En la parte superior de la página, seleccione **Bifurcar** para crear una bifurcación de este repositorio en su propia organización o cuenta de GitHub.

1. En el menú de la izquierda de la aplicación de funciones, seleccione **Centro de implementación**. Luego, seleccione **Configuración**.

1. En la pestaña **Configuración**, use la configuración de implementación que se muestra en la tabla siguiente.

    | Configuración      | Valor sugerido  | Descripción      |
    | ------------ | ---------------- | ---------------- |
    | **Origen** | GitHub | Debe haber creado un repositorio de GitHub para el código de ejemplo del paso 2. | 
    | **Organización**  | myOrganization | La organización en la que está insertado el repositorio. Por lo general, se trata de su cuenta. |
    | **Repositorio** | functions-vnet-tutorial | Repositorio bifurcado desde https://github.com/Azure-Samples/functions-vnet-tutorial. |
    | **Rama** | main (principal) | La rama principal del repositorio que creó. |
    | **Pila en tiempo de ejecución** | .NET | El código de ejemplo está en C#. |
    | **Versión** | .NET Core 3.1 | Versión del entorno de ejecución. |

1. Seleccione **Guardar**. 

    :::image type="content" source="./media/functions-create-vnet/12-deploy-portal.png" alt-text="Captura de pantalla sobre cómo implementar código de Azure Functions mediante el portal.":::

1. La implementación inicial debería tardar unos minutos. Cuando la aplicación se implementa correctamente, en la pestaña **Registros**, verá un mensaje de estado **correcto (activo)** . Si es necesario, actualice la página.

¡Enhorabuena! Implementó correctamente la aplicación de funciones de ejemplo.

## <a name="lock-down-your-function-app"></a>Bloqueo de la aplicación de funciones

Ahora cree el punto de conexión privado para bloquear la aplicación de funciones. Este punto de conexión privado conectará la aplicación de funciones de manera privada y segura a la red virtual mediante una dirección IP privada. 

Para más información, consulte la [documentación sobre los puntos de conexión privados](../private-link/private-endpoint-overview.md).

1. En el menú de la izquierda de la aplicación de funciones, seleccione **Redes**.

1. En **Conexiones de punto de conexión privado**, seleccione **Configurar las conexiones de punto de conexión privado**.

    :::image type="content" source="./media/functions-create-vnet/14-navigate-app-private-endpoint.png" alt-text="Captura de pantalla sobre cómo navegar a un punto de conexión privado de la aplicación de funciones.":::

1. Seleccione **Agregar**.

1. En el panel que se abre, use la configuración de punto de conexión privado siguiente:

    :::image type="content" source="./media/functions-create-vnet/15-create-app-private-endpoint.png" alt-text="Captura de pantalla sobre cómo crear un punto de conexión privado de la aplicación de funciones. El nombre es functionapp-endpoint. La suscripción es &quot;Private Test Sub CACHHAI&quot;. La red virtual es MyVirtualNet-tutorial. La subred es la predeterminada.":::

1. Seleccione **Aceptar** para agregar el punto de conexión privado. 
 
¡Enhorabuena! Protegió correctamente la aplicación de funciones, la instancia de Service Bus y la cuenta de almacenamiento al agregar puntos de conexión privados.

### <a name="test-your-locked-down-function-app"></a>Prueba de la aplicación de funciones bloqueada

1. En el menú de la izquierda de la aplicación de funciones, seleccione **Funciones**.

1. Seleccione **ServiceBusQueueTrigger**.

1. En el menú de la izquierda, seleccione **Monitor**. 
 
Verá que no puede supervisar la aplicación. El explorador no tiene acceso a la red virtual, por lo que no puede acceder directamente a los recursos de esa red. 
 
A continuación, se muestra una forma alternativa de supervisar la función mediante el uso de Application Insights:

1. En el menú de la izquierda de la aplicación de funciones, seleccione **Application Insights**. Luego, seleccione **Ver datos de Application Insights**.

    :::image type="content" source="./media/functions-create-vnet/16-app-insights.png" alt-text="Captura de pantalla sobre cómo ver Application Insights para una aplicación de funciones.":::

1. En el menú de la izquierda, seleccione **Métricas activas**.

1. Abra una nueva pestaña. En el menú de la izquierda de Service Bus, seleccione **Colas**.

1. Seleccione la cola.

1. En el menú de la izquierda, seleccione **Service Bus Explorer**. En **Enviar**, elija **Texto/sin formato** para **Tipo de contenido**. Luego, escriba un mensaje. 

1. Seleccione **Enviar** para enviar el mensaje.

    :::image type="content" source="./media/functions-create-vnet/17-send-service-bus-message.png" alt-text="Captura de pantalla sobre cómo enviar mensajes de Service Bus mediante el portal.":::

1. En la pestaña **Live Metrics**, debería ver que se activó el desencadenador de la cola de Service Bus. Si no es así, vuelva a enviar el mensaje desde **Service Bus Explorer**.

    :::image type="content" source="./media/functions-create-vnet/18-hello-world.png" alt-text="Captura de pantalla sobre cómo ver mensajes mediante Live Metrics para las aplicaciones de funciones.":::

¡Enhorabuena! Probó correctamente la configuración de la aplicación de funciones con puntos de conexión privados.

## <a name="understand-private-dns-zones"></a>Descripción de las zonas DNS privadas
Usó un punto de conexión privado para conectarse a los recursos de Azure. Está estableciendo una conexión a una dirección IP privada en lugar del punto de conexión público. Los servicios de Azure existentes están configurados para usar un DNS existente para conectarse al punto de conexión público. Debe reemplazar la configuración de DNS para conectarse al punto de conexión privado.

Se crea una zona DNS privada para cada recurso de Azure configurado con un punto de conexión privado. También se crea un registro de DNS para cada dirección IP privada asociada con el punto de conexión privado.

En este tutorial se han creado las siguientes zonas DNS:

- privatelink.file.core.windows.net
- privatelink.blob.core.windows.net
- privatelink.servicebus.windows.net
- privatelink.azurewebsites.net

[!INCLUDE [clean-up-section-portal](../../includes/clean-up-section-portal.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, creó una aplicación de funciones, una cuenta de almacenamiento y una instancia de Service Bus de nivel Premium. Protegió todos estos recurso tras puntos de conexión privados. 

Use los vínculos siguientes para obtener más información sobre las opciones de red y puntos de conexión privados de Azure Functions:

- [Opciones de redes en Azure Functions](./functions-networking-options.md)
- [Plan prémium de Azure Functions](./functions-premium-plan.md)
- [Puntos de conexión privados de Service Bus](../service-bus-messaging/private-link-service.md)
- [Puntos de conexión privados de Azure Storage](../storage/common/storage-private-endpoints.md)
