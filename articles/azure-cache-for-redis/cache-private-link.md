---
title: Azure Cache for Redis con Azure Private Link
description: Un punto de conexión privado de Azure es una interfaz de red que le conecta de forma privada y segura a Azure Cache for Redis de Azure Private Link. En este artículo, obtendrá información sobre cómo crear una memoria caché de Azure, una red virtual de Azure y un punto de conexión privado mediante Azure Portal.
author: curib
ms.author: cauribeg
ms.service: cache
ms.topic: conceptual
ms.date: 3/31/2021
ms.openlocfilehash: e479a743a54e0cf3d7350fa87c3cc5ad8bf332be
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110696521"
---
# <a name="azure-cache-for-redis-with-azure-private-link"></a>Azure Cache for Redis con Azure Private Link

En este artículo, obtendrá información sobre cómo crear una red virtual y una instancia de Azure Cache for Redis con un punto de conexión privado mediante Azure Portal. También aprenderá a agregar un punto de conexión privado a una instancia de Azure Cache for Redis existente.

Un punto de conexión privado de Azure es una interfaz de red que le conecta de forma privada y segura a Azure Cache for Redis de Azure Private Link. 

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)

> [!IMPORTANT]
> Actualmente, no se admiten la redundancia de zona, la compatibilidad con la consola del portal ni la persistencia en cuentas de almacenamiento de firewall. 
>
>

## <a name="create-a-private-endpoint-with-a-new-azure-cache-for-redis-instance"></a>Creación de un punto de conexión privado con una nueva instancia de Azure Cache for Redis 

En esta sección, creará una nueva instancia de Azure Cache for Redis con un punto de conexión privado.

### <a name="create-a-virtual-network"></a>Creación de una red virtual 

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y después seleccione **Crear un recurso**.

    :::image type="content" source="media/cache-private-link/1-create-resource.png" alt-text="Selección de Crear un recurso.":::

2. En la página **Nuevo**, seleccione **Redes** y seleccione **Red virtual**.

3. Seleccione **Agregar** para crear una red virtual.

4. En **Crear red virtual**, escriba o seleccione esta información en la pestaña **Conceptos básicos**:

   | Parámetro      | Valor sugerido  | Descripción |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Suscripción** | Desplácese hacia abajo y seleccione su suscripción. | Suscripción en la que se va a crear esta red virtual. | 
   | **Grupos de recursos** | Desplácese hacia abajo y seleccione un grupo de recursos o **Crear nuevo** y escriba un nombre nuevo para el grupo de recursos. | Nombre del grupo de recursos en el que se van a crear la memoria caché y otros recursos. Al colocar todos los recursos de la aplicación en un grupo de recursos, puede administrarlos o eliminarlos fácilmente. | 
   | **Nombre** | Escriba un nombre de red virtual. | El nombre debe comenzar con una letra o un número, acabar con una letra, un número o un carácter de subrayado y contener solo letras, números, caracteres de subrayado, puntos o guiones. | 
   | **Región** | Desplácese hacia abajo y seleccione una región. | Seleccione una [región](https://azure.microsoft.com/regions/) cerca de otros servicios que vayan a usar la red virtual. |

5. Seleccione la pestaña **Direcciones IP** o el botón **Siguiente: Direcciones IP** situado en la parte inferior de la página.

6. En la pestaña **Direcciones IP**, especifique el **espacio de direcciones IPv4** como uno o varios prefijos de dirección en la notación CIDR (por ejemplo, 192.168.1.0/24).

7. En **Nombre de subred**, seleccione en **predeterminada** para editar las propiedades de la subred.

8. En el panel **Editar subred**, especifique un nombre en **Nombre de subred** y un valor en **Intervalo de direcciones de subred**. Se debe usar la notación CIDR para el intervalo de direcciones de la subred (por ejemplo, 192.168.1.0/24). Debe incluirse en el espacio de direcciones de la red virtual.

9. Seleccione **Guardar**.

10. Seleccione la pestaña **Revisar y crear** o el botón **Revisar y crear**.

11. Compruebe que toda la información es correcta y seleccione **Crear** para aprovisionar la red virtual.

### <a name="create-an-azure-cache-for-redis-instance-with-a-private-endpoint"></a>Creación de una instancia de Azure Cache for Redis con un punto de conexión privado

Para crear una instancia de caché, siga estos pasos.

1. Vuelva a la página principal de Azure Portal o abra el menú de barra lateral y seleccione **Crear un recurso**. 
   
1. En la página **Nuevo**, seleccione **Base de datos** y, a continuación, seleccione **Azure Cache for Redis**.

    :::image type="content" source="media/cache-private-link/2-select-cache.png" alt-text="Selección de Azure Cache for Redis.":::
   
1. En la página **Nueva instancia de Redis Cache**, configure las opciones de la nueva caché.
   
   | Configuración      | Valor sugerido  | Descripción |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Nombre DNS** | Escriba un nombre único global. | El nombre de caché debe ser una cadena comprendida entre 1 y 63 caracteres. La cadena solo debe contener números, letras o guiones. El nombre debe comenzar y terminar por un número o una letra y no puede contener guiones consecutivos. El *nombre de host* de la instancia de caché será *\<DNS name>.redis.cache.windows.net*. | 
   | **Suscripción** | Desplácese hacia abajo y seleccione su suscripción. | La suscripción en la que se creará esta nueva instancia de Azure Cache for Redis. | 
   | **Grupos de recursos** | Desplácese hacia abajo y seleccione un grupo de recursos o **Crear nuevo** y escriba un nombre nuevo para el grupo de recursos. | Nombre del grupo de recursos en el que se van a crear la caché y otros recursos. Al colocar todos los recursos de la aplicación en un grupo de recursos, puede administrarlos o eliminarlos fácilmente. | 
   | **Ubicación** | Desplácese hacia abajo y seleccione una ubicación. | Seleccione una [región](https://azure.microsoft.com/regions/) cerca de otros servicios que vayan a usar la memoria caché. |
   | **Plan de tarifa** | Desplácese hacia abajo y seleccione un [plan de tarifa](https://azure.microsoft.com/pricing/details/cache/). |  El plan de tarifa determina el tamaño, el rendimiento y las características disponibles para la memoria caché. Para más información, consulte la [introducción a Azure Redis Cache](cache-overview.md). |

1. Seleccione la pestaña **Redes** o elija el botón **Redes** situado en la parte inferior de la página.

1. En la pestaña **Redes**, seleccione **Punto de conexión privado** para el método de conectividad.

1. Seleccione el botón **+ Agregar** para crear el punto de conexión privado.

    :::image type="content" source="media/cache-private-link/3-add-private-endpoint.png" alt-text="En redes, agregue un punto de conexión privado.":::

1. En la página **Crear un punto de conexión privado**, defina la configuración del punto de conexión privado con la red virtual y la subred que creó en la última sección y seleccione **Aceptar**. 

1. Seleccione la pestaña **Siguiente: Opciones avanzadas** o seleccione el botón **Siguiente: Opciones avanzadas** en la parte inferior de la página.

1. En la pestaña **Opciones avanzadas** de una instancia de caché básica o estándar, seleccione el botón de alternancia de habilitación si desea habilitar un puerto que no sea TLS.

1. En la pestaña **Opciones avanzadas** de la instancia de caché Premium, configure el puerto no TLS, la agrupación en clústeres y la persistencia de datos.

1. Seleccione el botón **Siguiente: Opciones avanzadas** o elija el botón **Siguiente: Etiquetas** situado en la parte inferior de la página.

1. Opcionalmente, en la pestaña **Etiquetas**, escriba el nombre y el valor si desea clasificar el recurso. 

1. Seleccione **Revisar + crear**. Pasará a la pestaña Revisar y crear, donde Azure validará la configuración.

1. Tras aparecer el mensaje verde Validación superada, seleccione **Crear**.

La caché tarda un tiempo en crearse. Puede supervisar el progreso en la página **Información general** de Azure Cache for Redis. Cuando **Estado** se muestra como **En ejecución**, la memoria caché está lista para su uso. 
    
> [!IMPORTANT]
> 
> Hay una marca `publicNetworkAccess` que está establecida en `Disabled` de manera predeterminada. 
> Esta marca está pensada para que pueda, de manera opcional, permitir el acceso de puntos de conexión públicos y privados a la memoria caché si está establecida en `Enabled`. Si se establece en `Disabled`, solo permitirá el acceso a puntos de conexión privados. Puede establecer el valor en `Disabled` o `Enabled`. Para más información sobre cómo cambiar el valor, consulte las [preguntas más frecuentes](#how-can-i-change-my-private-endpoint-to-be-disabled-or-enabled-from-public-network-access).
>
>

## <a name="create-a-private-endpoint-with-an-existing-azure-cache-for-redis-instance"></a>Creación de un punto de conexión privado con una instancia de Azure Cache for Redis existente 

En esta sección aprenderá a agregar un punto de conexión privado a una instancia de Azure Cache for Redis existente. 

### <a name="create-a-virtual-network"></a>Creación de una red virtual

Para crear una red virtual, siga estos pasos.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y después seleccione **Crear un recurso**.

2. En la página **Nuevo**, seleccione **Redes** y seleccione **Red virtual**.

3. Seleccione **Agregar** para crear una red virtual.

4. En **Crear red virtual**, escriba o seleccione esta información en la pestaña **Conceptos básicos**:

   | Parámetro      | Valor sugerido  | Descripción |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Suscripción** | Desplácese hacia abajo y seleccione su suscripción. | Suscripción en la que se va a crear esta red virtual. | 
   | **Grupos de recursos** | Desplácese hacia abajo y seleccione un grupo de recursos o **Crear nuevo** y escriba un nombre nuevo para el grupo de recursos. | Nombre del grupo de recursos en el que se van a crear la memoria caché y otros recursos. Al colocar todos los recursos de la aplicación en un grupo de recursos, puede administrarlos o eliminarlos fácilmente. | 
   | **Nombre** | Escriba un nombre de red virtual. | El nombre debe comenzar con una letra o un número, acabar con una letra, un número o un carácter de subrayado y contener solo letras, números, caracteres de subrayado, puntos o guiones. | 
   | **Región** | Desplácese hacia abajo y seleccione una región. | Seleccione una [región](https://azure.microsoft.com/regions/) cerca de otros servicios que vayan a usar la red virtual. |

5. Seleccione la pestaña **Direcciones IP** o el botón **Siguiente: Direcciones IP** situado en la parte inferior de la página.

6. En la pestaña **Direcciones IP**, especifique el **espacio de direcciones IPv4** como uno o varios prefijos de dirección en la notación CIDR (por ejemplo, 192.168.1.0/24).

7. En **Nombre de subred**, seleccione en **predeterminada** para editar las propiedades de la subred.

8. En el panel **Editar subred**, especifique un nombre en **Nombre de subred** y un valor en **Intervalo de direcciones de subred**. Se debe usar la notación CIDR para el intervalo de direcciones de la subred (por ejemplo, 192.168.1.0/24). Debe incluirse en el espacio de direcciones de la red virtual.

9. Seleccione **Guardar**.

10. Seleccione la pestaña **Revisar y crear** o el botón **Revisar y crear**.

11. Compruebe que toda la información es correcta y seleccione **Crear** para aprovisionar la red virtual.

### <a name="create-a-private-endpoint"></a>Creación de un punto de conexión privado 

Para crear un punto de conexión privado, siga estos pasos.

1. En Azure Portal, busque **Azure Cache for Redis**. Después, presione Entrar o selecciónelo en las sugerencias de búsqueda.

    :::image type="content" source="media/cache-private-link/4-search-for-cache.png" alt-text="Busque Azure Cache for Redis.":::

2. Seleccione la instancia de caché a la que desea agregar un punto de conexión privado.

3. En el lado izquierdo de la pantalla, seleccione **Puntos de conexión privados**.

4. Seleccione el botón **Punto de conexión privado** para crear el punto de conexión privado.

    :::image type="content" source="media/cache-private-link/5-add-private-endpoint.png" alt-text="Agregue un punto de conexión privado.":::

5. En la **página Crear un punto de conexión privado**, configure las opciones del punto de conexión privado.

   | Configuración      | Valor sugerido  | Descripción |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Suscripción** | Desplácese hacia abajo y seleccione su suscripción. | La suscripción en la que se va a crear este punto de conexión privado. | 
   | **Grupos de recursos** | Desplácese hacia abajo y seleccione un grupo de recursos o **Crear nuevo** y escriba un nombre nuevo para el grupo de recursos. | Nombre del grupo de recursos en el que se van a crear el punto de conexión privado y otros recursos. Al colocar todos los recursos de la aplicación en un grupo de recursos, puede administrarlos o eliminarlos fácilmente. | 
   | **Nombre** | Escriba un nombre de punto de conexión privado. | El nombre debe comenzar con una letra o un número, acabar con una letra, un número o un carácter de subrayado, y puede contener solo letras, números, caracteres de subrayado, puntos o guiones. | 
   | **Región** | Desplácese hacia abajo y seleccione una región. | Seleccione una [región](https://azure.microsoft.com/regions/) cerca de otros servicios que vayan a usar el punto de conexión privado. |

6. Seleccione el botón **Siguiente: Recurso** en la parte inferior de la página.

7. En la pestaña **Recurso**, seleccione su suscripción, elija el tipo de recurso `Microsoft.Cache/Redis` y seleccione la memoria caché a la que desea conectar el punto de conexión privado.

8. Seleccione el botón **Siguiente: Configuración** situado en la parte inferior de la página.

9. En la pestaña **Configuración**, seleccione la red virtual y la subred que creó en la sección anterior.

10. Seleccione el botón **Siguiente: Etiquetas** situado en la parte inferior de la página.

11. Opcionalmente, en la pestaña **Etiquetas**, escriba el nombre y el valor si desea clasificar el recurso.

12. Seleccione **Revisar + crear**. Pasará a la pestaña **Revisar y crear**, donde Azure valida la configuración.

13. Tras aparecer el mensaje verde **Validación superada**, seleccione **Crear**.

> [!IMPORTANT]
> 
> Hay una marca `publicNetworkAccess` que está establecida en `Disabled` de manera predeterminada. 
> Esta marca está pensada para que pueda, de manera opcional, permitir el acceso de puntos de conexión públicos y privados a la memoria caché si está establecida en `Enabled`. Si se establece en `Disabled`, solo permitirá el acceso a puntos de conexión privados. Puede establecer el valor en `Disabled` o `Enabled`. Para más información sobre cómo cambiar el valor, consulte las [preguntas más frecuentes](#how-can-i-change-my-private-endpoint-to-be-disabled-or-enabled-from-public-network-access).
>
>


## <a name="faq"></a>Preguntas más frecuentes

### <a name="why-cant-i-connect-to-a-private-endpoint"></a>¿Por qué no puedo conectarme a un punto de conexión privado?

Si la memoria caché ya está insertada en la red virtual, los puntos de conexión privados no se pueden usar con la instancia de la memoria caché. Si la instancia de caché usa una característica no admitida, que se enumera a continuación, no podrá conectarse a su instancia de punto de conexión privado.

### <a name="what-features-arent-supported-with-private-endpoints"></a>¿Qué características no son compatibles con los puntos de conexión privados?

Actualmente, no se admiten la redundancia de zona, la compatibilidad con la consola del portal ni la persistencia en cuentas de almacenamiento de firewall. 

### <a name="how-can-i-change-my-private-endpoint-to-be-disabled-or-enabled-from-public-network-access"></a>¿Cómo puedo cambiar el punto de conexión privado para que esté deshabilitado o habilitado el acceso desde la red pública?

Hay una marca `publicNetworkAccess` que está `Disabled` de manera predeterminada. Esta marca está pensada para que pueda, de manera opcional, permitir el acceso de puntos de conexión públicos y privados a la memoria caché si está establecida en `Enabled`. Si se establece en `Disabled`, solo permitirá el acceso a puntos de conexión privados. Puede establecer el valor en `Disabled` o `Enabled` en Azure Portal o con una solicitud PATCH de API RESTful. 

Para cambiar el valor en Azure Portal, haga lo siguiente:

1. En Azure Portal, busque **Azure Cache for Redis**.  Después, presione Entrar o selecciónelo en las sugerencias de búsqueda.

2. Seleccione la instancia de caché en la que quiere cambiar el valor de acceso a la red pública.

3. En el lado izquierdo de la pantalla, seleccione **Puntos de conexión privados**.

4. Seleccione el botón **Enable public network access** (Habilitar acceso a la red pública).

Para cambiar el valor mediante una solicitud PATCH de API RESTful, consulte a continuación y modifique el valor para que refleje la marca que quiere para la memoria caché.

```http
PATCH  https://management.azure.com/subscriptions/{subscription}/resourceGroups/{resourcegroup}/providers/Microsoft.Cache/Redis/{cache}?api-version=2020-06-01
{    "properties": {
       "publicNetworkAccess":"Disabled"
   }
}
```

### <a name="how-can-i-have-multiple-endpoints-in-different-virtual-networks"></a>¿Cómo puedo tener varios puntos de conexión en diferentes redes virtuales?

Para tener varios puntos de conexión privados en distintas redes virtuales, la zona DNS privada debe configurarse manualmente en las varias redes virtuales _antes_ de crear el punto de conexión privado. Para obtener más información, vea [Configuración de DNS para puntos de conexión privados de Azure](../private-link/private-endpoint-dns.md). 

### <a name="what-happens-if-i-delete-all-the-private-endpoints-on-my-cache"></a>¿Qué ocurre si se eliminan todos los puntos de conexión privados de la caché?

Una vez que se eliminan los puntos de conexión privados de la caché, la instancia de caché puede quedar inaccesible hasta que se habilite explícitamente el acceso a la red pública o se agregue otro punto de conexión privado. Puede cambiar la marca `publicNetworkAccess` en Azure Portal o mediante una solicitud PATCH de API RESTful. Para más información sobre cómo cambiar el valor, consulte las [preguntas más frecuentes](#how-can-i-change-my-private-endpoint-to-be-disabled-or-enabled-from-public-network-access).

### <a name="are-network-security-groups-nsg-enabled-for-private-endpoints"></a>¿Están habilitados los grupos de seguridad de red (NSG) para los puntos de conexión privados?

No, están deshabilitados para los puntos de conexión privados. Si bien las subredes que contienen el punto de conexión privado pueden tener un grupo de seguridad de red asociado, las reglas no son efectivas en el tráfico procesado por el punto de conexión privado. Debe tener la [aplicación de directivas de red deshabilitada](../private-link/disable-private-endpoint-network-policy.md) para implementar puntos de conexión privados en una subred. El grupo de seguridad de red se sigue aplicando en otras cargas de trabajo hospedadas en la misma subred. Las rutas de cualquier subred de cliente utilizarán un prefijo /32, lo que cambia el comportamiento de enrutamiento predeterminado requiere un UDR similar. 

Controle el tráfico mediante el uso de reglas del grupo de seguridad de red para el tráfico saliente en los clientes de origen. Implementación de rutas individuales con un prefijo /32 para invalidar rutas de punto de conexión privado. Todavía se admiten los registros de flujo de NSG y la información de supervisión de las conexiones salientes y se pueden usar.

### <a name="my-private-endpoint-instance-isnt-in-my-vnet-so-how-is-it-associated-with-my-vnet"></a>Mi instancia de punto de conexión privado no está en mi red virtual, por tanto, ¿cómo se asocia a la red virtual?

Solo está vinculada a la red virtual. Dado que no está en la red virtual, no es necesario modificar las reglas del grupo de seguridad de red para los puntos de conexión dependientes.

### <a name="how-can-i-migrate-my-vnet-injected-cache-to-a-private-endpoint-cache"></a>¿Cómo puedo migrar mi memoria caché insertada en la red virtual a una memoria caché de punto de conexión privado?

Elimine la memoria caché insertada en la red virtual y cree una nueva instancia de memoria caché con un punto de conexión privado. Para más información, consulte [Migración a Azure Cache for Redis](cache-migration-guide.md).

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Azure Private Link, consulte la [documentación de Azure Private Link](../private-link/private-link-overview.md).
* Para comparar las distintas opciones de aislamiento de red para la instancia de memoria caché, consulte [Opciones de aislamiento de red de Azure Cache for Redis](cache-network-isolation.md).
