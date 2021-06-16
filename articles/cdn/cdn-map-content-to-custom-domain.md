---
title: 'Tutorial: Adición de un dominio personalizado al punto de conexión'
titleSuffix: Azure Content Delivery Network
description: Use este tutorial para agregar un dominio personalizado a un punto de conexión de Azure Content Delivery Network, de modo que el nombre de dominio sea visible en la dirección URL.
services: cdn
author: jessie-jyy
manager: KumudD
ms.service: azure-cdn
ms.topic: tutorial
ms.date: 04/12/2021
ms.author: yuajia
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 3b64efab10a9d52209f04dfd43a43654eb55467a
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110706948"
---
# <a name="tutorial-add-a-custom-domain-to-your-endpoint"></a>Tutorial: Adición de un dominio personalizado al punto de conexión

En este tutorial se muestra cómo agregar un dominio personalizado a un punto de conexión de Azure Content Delivery Network (CDN). 

El nombre del punto de conexión en el perfil de CDN es un subdominio de azureedge.net. De forma predeterminada, al entregar contenido, el dominio del perfil de CDN se incluye en la dirección URL.

Por ejemplo, **https://contoso.azureedge.net/photo.png**.

Azure CDN permite asociar un dominio personalizado a un punto de conexión de CDN. Esta opción le permite entregar el contenido con un dominio personalizado en la dirección URL en lugar de en el dominio predeterminado.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> - Crear un registro DNS de CNAME
> - Asociar el dominio personalizado a un punto de conexión de CDN
> - Comprobar el dominio personalizado

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

* Para poder completar los pasos de este tutorial, cree un perfil de CDN y al menos un punto de conexión de CDN. 
    * Para más información, consulte [Inicio rápido: Creación de un perfil y un punto de conexión de Azure CDN](cdn-create-new-endpoint.md).

* Si no dispone de un dominio personalizado, adquiera uno con un proveedor de dominios. 
    * Para más información, consulte [Compra de un nombre de dominio personalizado](../app-service/manage-custom-dns-buy-domain.md).

* Si usa Azure para hospedar sus [dominios DNS](../dns/dns-overview.md), delegue su dominio personalizado en Azure DNS. 
    * Para más información, vea [Delegación de un dominio en DNS de Azure](../dns/dns-delegate-domain-azure-dns.md).

* Si utiliza un proveedor de dominios para controlar su dominio DNS, diríjase a [Creación de un registro DNS de CNAME](#create-a-cname-dns-record).


## <a name="create-a-cname-dns-record"></a>Creación de un registro DNS de CNAME

Para poder usar un dominio personalizado con un punto de conexión de Azure CDN, primero debe crear un registro de nombre canónico (CNAME) con Azure DNS o con su proveedor de DNS que apunte a su punto de conexión de CDN. 

Un registro CNAME es un registro de DNS que asigna un nombre de dominio de origen a un nombre de dominio de destino. 

En el caso de Azure CDN, el nombre de dominio de origen es su nombre de dominio personalizado y el nombre de dominio de destino es el nombre de host del punto de conexión de CDN. 

Azure CDN enruta el tráfico dirigido al dominio personalizado de origen al nombre de host del punto de conexión de CDN de destino después de comprobar el registro CNAME.

Un dominio personalizado y su subdominio no se pueden asociar con más de un punto de conexión a la vez. 

Utilice varios registros CNAME para diferentes subdominios del mismo dominio personalizado de distintos servicios de Azure.

Puede asignar un dominio personalizado con diferentes subdominios al mismo punto de conexión de CDN.

> [!NOTE]
> Este tutorial utiliza el tipo de registro CNAME. Si usa los tipos de registro A o AAAA, siga los mismos pasos que se indican a continuación, pero reemplace CNAME por el tipo de registro que prefiera.

---
# <a name="azure-dns"></a>[**Azure DNS**](#tab/azure-dns)

Azure DNS usa registros de alias para los recursos de Azure de la misma suscripción.

Para agregar un registro de alias para el punto de conexión de Azure CDN:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. En el menú de la izquierda, seleccione **Todos los recursos** y, después, la zona de Azure DNS para el dominio personalizado.

3. En la zona DNS de su dominio personalizado, seleccione **+ Conjunto de registros**.

4. En **Agregar conjunto de registros**, escriba o seleccione la siguiente información:

    | Configuración | Valor |
    | ------- | ----- |
    | Nombre  | Escriba el alias que desea usar para el punto de conexión de CDN. Por ejemplo, **www**. |
    | Tipo  | Seleccione **CNAME.** . |
    | Conjunto de registros de alias | Seleccione **Sí**. |
    | Tipo de alias | Seleccione **Recurso de Azure**. |
    | Elija una suscripción | Seleccione su suscripción. |
    | Recurso de Azure | Seleccione el punto de conexión de Azure CDN. |

5. Cambie el **TTL** del registro por su valor.

6. Seleccione **Aceptar**.

# <a name="dns-provider"></a>[**Proveedor de DNS**](#tab/dns-provider)

## <a name="map-the-temporary-cdnverify-subdomain"></a>Asignación del subdominio temporal cdnverify

Al asignar un dominio existente que esté en producción, existen consideraciones especiales. 

Mientras se registra el dominio personalizado en Azure Portal, se puede producir un breve período de inactividad en el dominio.

Para evitar la interrupción del tráfico web, asigne su dominio personalizado al nombre de host de su punto de conexión de CDN con el subdominio **cdnverify** de Azure. Este proceso crea una asignación de CNAME temporal. 

Con este método, los usuarios pueden acceder a un dominio sin interrupción mientras se realiza la asignación de DNS. 

Si las consideraciones sobre tiempos de inactividad en producción no son un problema, puede asignar directamente su dominio personalizado a su punto de conexión de CDN. Vaya a [Asignación del dominio personalizado permanente](#map-the-permanent-custom-domain).

Para crear un registro CNAME con el subdominio cdnverify:

1. Inicie sesión en el sitio web del proveedor de DNS de su dominio personalizado.

2. Cree una entrada de registro CNAME para el dominio personalizado y rellene los campos como se muestra en la tabla siguiente (los nombres de campo pueden variar):

    | Source                    | Tipo  | Destination                     |
    |---------------------------|-------|---------------------------------|
    | cdnverify. www.contoso.com | CNAME | cdnverify.contoso.azureedge.net |

    - Origen: escriba un nombre de dominio personalizado, incluido el subdominio cdnverify, con el siguiente formato: **cdnverify.\<custom-domain-name>**
        - Por ejemplo, **cdnverify. www.contoso.com**

    - Escriba:  Escriba o seleccione el **CNAME**.

    - Destino: escriba el nombre de host del punto de conexión de CDN, incluido el subdominio cdnverify, con el siguiente formato: **cdnverify.\<endpoint-name>.azureedge.net**. 
        - Por ejemplo, **cdnverify.contoso.azureedge.net**

3. Guarde los cambios.

## <a name="map-the-permanent-custom-domain"></a>Asignación de un dominio personalizado permanente

En esta sección, asignará el dominio personalizado permanente al punto de conexión de CDN. 

Para crear un registro CNAME para un dominio personalizado:

1. Inicie sesión en el sitio web del proveedor de dominios de su dominio personalizado.

2. Cree una entrada de registro CNAME para el dominio personalizado y rellene los campos como se muestra en la tabla siguiente (los nombres de campo pueden variar):

    | Source          | Tipo  | Destination           |
    |-----------------|-------|-----------------------|
    | www.contoso.com | CNAME | contoso.azureedge.net |

    - Origen: Escriba el nombre de dominio personalizado.
        - Por ejemplo, **www.contoso.com**

    - Escriba:  Escriba o seleccione el **CNAME**.

    - Destino: Escriba el nombre de host del punto de conexión de CDN con el siguiente formato: **\<endpoint-name>.azureedge.net**. 
        - Por ejemplo, **contoso.azureedge.net**

3. Guarde los cambios.

4. Si previamente ha creado un registro CNAME de un subdominio cdnverify temporal, elimínelo. 

5. La primera vez que use este dominio personalizado en producción, siga los pasos de [Asociación del dominio personalizado a un punto de conexión de CDN](#associate-the-custom-domain-with-your-cdn-endpoint) y [Comprobación del dominio personalizado](#verify-the-custom-domain).

---
## <a name="associate-the-custom-domain-with-your-cdn-endpoint"></a>Asociación del dominio personalizado a un punto de conexión de CDN

Una vez que haya registrado un dominio personalizado, puede agregarlo a su punto de conexión de CDN. 


---
# <a name="azure-portal"></a>[**Azure Portal**](#tab/azure-portal)

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y vaya al perfil de CDN que contenga el punto de conexión que desea asignar a un dominio personalizado.
    
2. En la página **Perfil de CDN**, seleccione el punto de conexión de CDN que se va a asociar con el dominio personalizado.

    :::image type="content" source="media/cdn-map-content-to-custom-domain/cdn-endpoint-selection.png" alt-text="Selección de punto de conexión de CDN" border="true":::
    
3. Seleccione **+ Dominio personalizado**. 

   :::image type="content" source="media/cdn-map-content-to-custom-domain/cdn-custom-domain-button.png" alt-text="Botón Agregar dominio personalizado" border="true":::

4. En **Agregar un dominio personalizado**, el **Nombre de host del punto de conexión** se rellena previamente y se deriva de la dirección URL del punto de conexión de CDN: **\<endpoint-hostname>** .azureedge.net. No se puede modificar.

5. Para **Nombre de host personalizado**, escriba el dominio personalizado, incluido el subdominio, que se usará como el dominio de origen del registro CNAME. 
    1. Por ejemplo, **www.contoso.com** o **cdn.contoso.com**. **No utilice el nombre de subdominio cdnverify**.

    :::image type="content" source="media/cdn-map-content-to-custom-domain/cdn-add-custom-domain.png" alt-text="Adición de un dominio personalizado" border="true":::

6. Seleccione **Agregar**.

   Azure comprueba que el registro CNAME existe para el nombre de dominio personalizado que ha especificado. Si el registro CNAME es correcto, el dominio personalizado se valida. 

   Puede tardar algún tiempo la propagación de la nueva configuración de dominio personalizado a todos los nodos perimetrales de la red CDN: 
    - En los perfiles de **Azure CDN Estándar de Microsoft**, la propagación se completa normalmente en 10 minutos. 
    - En los perfiles de **Azure CDN estándar**, la propagación normalmente se completa en un minuto. 
    - En los perfiles **Azure CDN Estándar de Verizon** y **Azure CDN Premium de Verizon**, la propagación se completa normalmente en 10 minutos.   

# <a name="powershell"></a>[**PowerShell**](#tab/azure-powershell)

1. Inicie sesión en Azure PowerShell:

```azurepowershell-interactive
    Connect-AzAccount

```
2. Use [New-AzCdnCustomDomain](/powershell/module/az.cdn/new-azcdncustomdomain) para asignar el dominio personalizado al punto de conexión de CDN. 

    * Reemplace **myendpoint8675.azureedge.net** por la dirección URL del punto de conexión.
    * Reemplace **myendpoint8675** por el nombre del punto de conexión de la red CDN.
    * Reemplace **www.contoso.com** por el nombre de dominio personalizado.
    * Reemplace **myCDN** por el nombre del perfil de CDN.
    * Reemplace **myResourceGroupCDN** por el nombre del grupo de recursos.

```azurepowershell-interactive
    $parameters = @{
        Hostname = 'myendpoint8675.azureedge.net'
        EndPointName = 'myendpoint8675'
        CustomDomainName = 'www.contoso.com'
        ProfileName = 'myCDN'
        ResourceGroupName = 'myResourceGroupCDN'
    }
    New-AzCdnCustomDomain @parameters
```

Azure comprueba que el registro CNAME existe para el nombre de dominio personalizado que ha especificado. Si el registro CNAME es correcto, el dominio personalizado se valida. 

   Puede tardar algún tiempo la propagación de la nueva configuración de dominio personalizado a todos los nodos perimetrales de la red CDN: 

- En los perfiles de **Azure CDN Estándar de Microsoft**, la propagación se completa normalmente en 10 minutos. 
- En los perfiles de **Azure CDN estándar**, la propagación normalmente se completa en un minuto. 
- En los perfiles **Azure CDN Estándar de Verizon** y **Azure CDN Premium de Verizon**, la propagación se completa normalmente en 10 minutos.   


---
## <a name="verify-the-custom-domain"></a>Comprobación del dominio personalizado

Después de haber completado el registro del dominio personalizado, compruebe que hace referencia a su punto de conexión de CDN.
 
1. Asegúrese de que tiene contenido público almacenado en caché en el punto de conexión. Por ejemplo, si su punto de conexión de CDN está asociado a una cuenta de almacenamiento, Azure CDN almacenará el contenido en la caché, en un contenedor público. Configure el contenedor para permitir el acceso público y compruebe que contiene al menos un archivo para probar el dominio personalizado.

2. En el explorador, vaya a la dirección del archivo usando el dominio personalizado. Por ejemplo, si el dominio personalizado es `www.contoso.com`, la dirección URL al archivo almacenado en caché debe ser similar a la siguiente: `http://www.contoso.com/my-public-container/my-file.jpg`. Compruebe que el resultado es igual que al acceder directamente al punto de conexión de CDN en **\<endpoint-hostname>** .azureedge.net.

## <a name="clean-up-resources"></a>Limpieza de recursos

---
# <a name="azure-portal"></a>[**Azure Portal**](#tab/azure-portal-cleanup)

Si desea dejar de asociar el punto de conexión a un dominio personalizado, siga estos pasos para quitar el dominio personalizado:
 
1. Vaya al proveedor DNS, elimine el registro CNAME del dominio personalizado o actualice el registro CNAME del dominio personalizado a un punto de conexión que no pertenezca a Azure CDN.

    > [!Important]
    > Para evitar las entradas DNS pendientes y los riesgos de seguridad que crean, a partir del 9 de abril de 2021 Azure CDN requiere la eliminación de los registros CNAME en los puntos de conexión de Azure CDN antes de que se puedan eliminar los recursos. Los recursos incluyen los dominios personalizados de Azure CDN, los perfiles o puntos de conexión de Azure CDN, o los grupos de recursos de Azure que tienen dominios personalizado de Azure CDN habilitados.

2. En el perfil de CDN, seleccione el punto de conexión con el dominio personalizado que desee quitar.

3. En la página **Punto de conexión**, en Dominios personalizados, haga clic con el botón derecho en el dominio personalizado que desea quitar y, después, seleccione **Eliminar** en el menú contextual. Seleccione **Sí**.

   El dominio personalizado se desasocia del punto de conexión.

# <a name="powershell"></a>[**PowerShell**](#tab/azure-powershell-cleanup)

Si desea dejar de asociar el punto de conexión a un dominio personalizado, siga estos pasos para quitar el dominio personalizado:

1. Vaya al proveedor DNS, elimine el registro CNAME del dominio personalizado o actualice el registro CNAME del dominio personalizado a un punto de conexión que no pertenezca a Azure CDN.

    > [!Important]
    > Para evitar las entradas DNS pendientes y los riesgos de seguridad que crean, a partir del 9 de abril de 2021 Azure CDN requiere la eliminación de los registros CNAME en los puntos de conexión de Azure CDN antes de que se puedan eliminar los recursos. Los recursos incluyen los dominios personalizados de Azure CDN, los perfiles o puntos de conexión de Azure CDN, o los grupos de recursos de Azure que tienen dominios personalizado de Azure CDN habilitados.

2. Use [Remove-AzCdnCustomDomain](/powershell/module/az.cdn/remove-azcdncustomdomain) para quitar el dominio personalizado del punto de conexión:

    * Reemplace **myendpoint8675** por el nombre del punto de conexión de la red CDN.
    * Reemplace **www.contoso.com** por el nombre de dominio personalizado.
    * Reemplace **myCDN** por el nombre del perfil de CDN.
    * Reemplace **myResourceGroupCDN** por el nombre del grupo de recursos.

    ```azurepowershell-interactive
        $parameters = @{
            CustomDomainName = 'www.contoso.com'
            EndPointName = 'myendpoint8675'
            ProfileName = 'myCDN'
            ResourceGroupName = 'myResourceGroupCDN'
        }
        Remove-AzCdnCustomDomain @parameters
    ```
---

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> - Crear un registro DNS de CNAME
> - Asociar el dominio personalizado a un punto de conexión de CDN
> - Comprobar el dominio personalizado

En el siguiente tutorial aprenderá a configurar el protocolo HTTPS en un dominio personalizado de Azure CDN.

> [!div class="nextstepaction"]
> [Tutorial: Configuración de HTTPS en un dominio personalizado de Azure CDN](cdn-custom-ssl.md)
