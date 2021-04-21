---
title: Configuración de Azure Functions con una red virtual
description: En este artículo se muestra cómo realizar ciertas tareas de redes virtuales para Azure Functions.
ms.topic: conceptual
ms.date: 3/13/2021
ms.custom: template-how-to
ms.openlocfilehash: c123b20e163731f9a872a969f2f1564479b6e308
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107718438"
---
# <a name="how-to-configure-azure-functions-with-a-virtual-network"></a>Configuración de Azure Functions con una red virtual

En este artículo se muestra cómo realizar tareas relacionadas con la configuración de la aplicación de funciones para conectarse a una red virtual y ejecutarse en ella. Para obtener más información sobre Azure Functions y las redes, vea [Opciones de redes de Azure Functions](functions-networking-options.md).

## <a name="restrict-your-storage-account-to-a-virtual-network"></a>Restricción de la cuenta de almacenamiento a una red virtual 

Al crear una aplicación de funciones, debe crear una cuenta de Azure Storage de uso general compatible con Blob, Queue y Table Storage, o vincular a una. Puede reemplazar esta cuenta de almacenamiento por una que esté protegida con puntos de conexión de servicio o punto de conexión privado. 

> [!NOTE]  
> Esta característica funciona actualmente para todas las SKU compatibles con la red virtual de Windows en el plan dedicado (App Service) y para los planes Premium. No se admite el plan de consumo. 

Para configurar una función con una cuenta de almacenamiento restringida a una red privada:

1. Cree una función con una cuenta de almacenamiento que no tenga habilitados los puntos de conexión de servicio.

1. Configure la función para conectarse a la red virtual.

1. Cree o configure una cuenta de almacenamiento diferente.  Será la cuenta de almacenamiento que se proteja con los puntos de conexión de servicio y se conecte a la función.

1. [Cree un recurso compartido de archivos](../storage/files/storage-how-to-create-file-share.md#create-a-file-share) en la cuenta de almacenamiento protegida.

1. Habilite los puntos de conexión de servicio o el punto de conexión privado para la cuenta de almacenamiento.  
    * Si se usan conexiones de punto de conexión privado, la cuenta de almacenamiento necesitará un punto de conexión privado para los subrecursos `file` y `blob`.  Si se usan ciertas funcionalidades como Durable Functions, también necesitará que se pueda acceder a `queue` y `table` a través de una conexión de punto de conexión privado.
    * Si usa puntos de conexión de servicio, habilite la subred dedicada a las aplicaciones de funciones para las cuentas de almacenamiento.

1. Copie el contenido del archivo y el blob de la cuenta de almacenamiento de la aplicación de funciones en la cuenta de almacenamiento protegida y el recurso compartido de archivos.

1. Copie la cadena de conexión para esta cuenta de almacenamiento.

1. Actualice el contenido de **Configuración de la aplicación** que se encuentra en **Configuración** para la aplicación de funciones a lo siguiente:

    | Nombre del valor | Value | Comentario |
    |----|----|----|
    | `AzureWebJobsStorage`| Cadena de conexión de almacenamiento | Esta es la cadena de conexión para una cuenta de almacenamiento protegida. |
    | `WEBSITE_CONTENTAZUREFILECONNECTIONSTRING` |  Cadena de conexión de almacenamiento | Esta es la cadena de conexión para una cuenta de almacenamiento protegida. |
    | `WEBSITE_CONTENTSHARE` | Recurso compartido de archivos | El nombre del recurso compartido de archivos creado en la cuenta de almacenamiento protegida donde residen los archivos de implementación del proyecto. |
    | `WEBSITE_CONTENTOVERVNET` | 1 | Nueva configuración |
    | `WEBSITE_VNET_ROUTE_ALL` | 1 | Fuerza todo el tráfico saliente a través de la red virtual. Obligatorio cuando la cuenta de almacenamiento usa conexiones de punto de conexión privado. |
    | `WEBSITE_DNS_SERVER` | `168.63.129.16` | El servidor DNS que usa la aplicación. Obligatorio cuando la cuenta de almacenamiento usa conexiones de punto de conexión privado. |

1. Seleccione **Guardar** para guardar la configuración de la aplicación. Si cambia la configuración de la aplicación, esta se reiniciará.  

Una vez que se reinicia la aplicación de funciones, estará conectado a una cuenta de almacenamiento protegida.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Opciones de redes de Azure Functions](functions-networking-options.md)

