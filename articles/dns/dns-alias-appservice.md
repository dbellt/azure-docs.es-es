---
title: Hospedaje de aplicaciones web de Azure con equilibrio de carga en el vértice de zona
description: Uso de un registro de alias de Azure DNS para hospedar aplicaciones web con equilibrio de carga en el vértice de zona
services: dns
author: rohinkoul
ms.service: dns
ms.topic: how-to
ms.date: 04/27/2021
ms.author: rohink
ms.openlocfilehash: 726cc63ecbd06e2cc4610be65828bd5e897d9fd0
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108745206"
---
# <a name="host-load-balanced-azure-web-apps-at-the-zone-apex"></a>Hospedaje de aplicaciones web de Azure con equilibrio de carga en el vértice de zona

El protocolo DNS evita la asignación de cualquier elemento que no sea un registro D o AAAA en el vértice de la zona. Un vértice de zona de ejemplo es contoso.com. Esta restricción presenta un problema para los propietarios de aplicaciones que tienen aplicaciones con equilibrio de carga detrás de Traffic Manager. No es posible señalar al perfil de Traffic Manager desde el registro de vértice de zona. Como consecuencia, los propietarios de aplicaciones deben usar una solución alternativa. Un redireccionamiento en la capa de la aplicación debe redirigirse desde el vértice de zona a otro dominio. Un ejemplo es un redireccionamiento de `contoso.com` a `www.contoso.com`. Esa organización presenta un único punto de error en cuanto a la función de redireccionamiento.

Con los registros de alias, ya no tendrá este problema. Puede apuntar su registro de vértice de zona a un perfil de Traffic Manager que tenga puntos de conexión externos. También puede apuntar al mismo perfil de Traffic Manager que se usa para otros dominios dentro de la zona DNS.

Por ejemplo, puede hacer que `contoso.com` y `www.contoso.com` apunten al mismo el perfil de Traffic Manager. Esta configuración funcionará siempre que el perfil de Traffic Manager solo tenga configurados puntos de conexión externos.

En este artículo, aprenderá a crear un registro de alias para el vértice del dominio. A continuación, configurará los puntos de conexión del perfil de Traffic Manager para las aplicaciones web.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Debe tener un nombre de dominio disponible que pueda hospedar en Azure DNS para realizar las pruebas. Debe tener control total de este dominio. El control total incluye la capacidad de establecer los registros de nombre de servidor (NS) para el dominio.

Para obtener instrucciones sobre cómo hospedar el dominio en Azure DNS, vea [Tutorial: Hospedaje del dominio en Azure DNS](dns-delegate-domain-azure-dns.md).

El dominio en el ejemplo que se usa en este tutorial es contoso.com, pero debe usar su propio nombre de dominio.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos que contenga todos los recursos usados en este artículo.

## <a name="create-app-service-plans"></a>Creación de planes de App Service

Cree dos planes web de App Service en el grupo de recursos. Use la tabla siguiente para obtener ayuda para la configuración. Para más información sobre cómo crear un plan de App Service, consulte [Administración de un plan de App Service en Azure](../app-service/app-service-plan-manage.md).


|Nombre  |Sistema operativo  |Location  |Nivel de precios  |
|---------|---------|---------|---------|
|ASP-01     |Windows|Este de EE. UU.|D1-Shared para desarrollo/pruebas|
|ASP-02     |Windows|Centro de EE. UU.|D1-Shared para desarrollo/pruebas|

## <a name="create-app-services"></a>Creación de servicios de aplicaciones

Cree dos aplicaciones web, una en cada plan de App Service.

1. En la esquina superior izquierda de la página de Azure Portal, seleccione **Crear un recurso**.
2. Escriba **Aplicación web** en la barra de búsqueda y presione Entrar.
3. Seleccione **Aplicación web**.
4. Seleccione **Crear**.
5. Acepte los valores predeterminados y use la tabla siguiente para configurar las dos aplicaciones web:

   |Nombre<br>(debe ser único dentro de .azurewebsites.net)|Grupo de recursos |Pila en tiempo de ejecución|Region|Plan de App Service/ubicación
   |---------|---------|-|-|-------|
   |App-01|Usar existente<br>Seleccionar el grupo de recursos|.NET Core 2.2|Este de EE. UU.|ASP-01(D1)|
   |App-02|Usar existente<br>Seleccionar el grupo de recursos|.NET Core 2.2|Centro de EE. UU.|ASP-02(D1)|

### <a name="gather-some-details"></a>Recopilar algunos detalles

Ahora debe anotar la dirección IP y el nombre de host de las aplicaciones web.

1. Abra el grupo de recursos y seleccione la primera aplicación web (**App-01** en este ejemplo).
2. En la columna izquierda, seleccione **Propiedades**.
3. Anote la dirección que se muestra en **URL** y, en **Direcciones IP de salida**, anote la primera dirección IP de la lista. Usará esta información más adelante cuando configure los puntos de conexión de Traffic Manager.
4. Repita el proceso con **App-02**.

## <a name="create-a-traffic-manager-profile"></a>Crear un perfil de Traffic Manager

Cree un perfil de Traffic Manager en el grupo de recursos. Use los valores predeterminados y escriba un nombre único dentro del espacio de nombres trafficmanager.net.

Para obtener más información, consulte [Guía de inicio rápido: Creación de un perfil de Traffic Manager para una aplicación web de alta disponibilidad](../traffic-manager/quickstart-create-traffic-manager-profile.md).

### <a name="create-endpoints"></a>Creación de puntos de conexión

Ahora puede crear los puntos de conexión de las dos aplicaciones web.

1. Abra el grupo de recursos y seleccione el perfil de Traffic Manager.
2. En la columna izquierda, seleccione **Puntos de conexión**.
3. Seleccione **Agregar**.
4. Use la tabla siguiente para configurar los puntos de conexión:

   |Tipo  |Nombre  |Destino  |Location  |Configuración de encabezado personalizado|
   |---------|---------|---------|---------|---------|
   |Punto de conexión externo     |End-01|Dirección IP que anotó para App-01|Este de EE. UU.|Host:\<the URL you recorded for App-01\><br>Ejemplo: **host:app-01.azurewebsites.net**|
   |Punto de conexión externo     |End-02|Dirección IP que anotó para App-02|Centro de EE. UU.|Host:\<the URL you recorded for App-02\><br>Ejemplo: **host:app-02.azurewebsites.net**

## <a name="create-dns-zone"></a>Creación de una zona DNS

Puede usar una zona DNS existente para las pruebas o puede crear una zona. Para crear y delegar una nueva zona DNS en Azure, consulte [Tutorial: Hospedaje del dominio en Azure DNS](dns-delegate-domain-azure-dns.md).

## <a name="add-a-txt-record-for-custom-domain-validation"></a>Adición de un registro TXT para la validación de un dominio personalizado

Al agregar un nombre de host personalizado a las aplicaciones web, buscará un registro TXT específico para validar el dominio.

1. Abra el grupo de recursos y seleccione la zona DNS.
2. Seleccione **Conjunto de registros**.
3. Agregue el conjunto de registros con la siguiente tabla. Para el valor, use la dirección URL de la aplicación web real que registró anteriormente:

   |Nombre  |Tipo  |Value|
   |---------|---------|-|
   |@     |TXT|App-01.azurewebsites.net|


## <a name="add-a-custom-domain"></a>Adición de un dominio personalizado

Agregue un dominio personalizado para las dos aplicaciones web.

1. Abra el grupo de recursos y seleccione su primera aplicación web.
2. En la columna izquierda, seleccione **Dominios personalizados**.
3. En **Dominios personalizados**, seleccione **Agregar un dominio personalizado**.
4. En **Dominio personalizado**, escriba el nombre de su dominio personalizado. Por ejemplo, contoso.com.
5. Seleccione **Validar**.

   El dominio debe superar la validación y mostrar marcas de verificación verdes junto a **Disponibilidad de nombre de host** y **Propiedad del dominio**.
5. Seleccione **Agregar dominio personalizado**.
6. Para ver el nuevo nombre de host en **Nombres de host asignados al sitio**, actualice el explorador. La actualización en la página no siempre muestra los cambios inmediatamente.
7. Repita este procedimiento con la segunda aplicación web.

## <a name="add-the-alias-record-set"></a>Incorporación del conjunto de registros de alias

Ahora cree un registro de alias para el vértice de la zona.

1. Abra el grupo de recursos y seleccione la zona DNS.
2. Seleccione **Conjunto de registros**.
3. Agregue el conjunto de registros con la siguiente tabla:

   |Nombre  |Tipo  |Conjunto de registros de alias  |Tipo de alias  |Recurso de Azure|
   |---------|---------|---------|---------|-----|
   |@     |A|Sí|Recurso de Azure|Traffic Manager: su perfil|


## <a name="test-your-web-apps"></a>Prueba de las aplicaciones web

Ahora puede comprobar que puede llegar a la aplicación web y que se está equilibrando la carga.

1. Abra un explorador web y vaya a su dominio. Por ejemplo, contoso.com. Verá la página de aplicación web predeterminada.
2. Detenga su primera aplicación web.
3. Cierre el explorador web y espere unos minutos.
4. Inicie el explorador web y vaya a su dominio. Debería seguir viendo la página de aplicación web predeterminada.
5. Detenga la segunda aplicación web.
6. Cierre el explorador web y espere unos minutos.
7. Inicie el explorador web y vaya a su dominio. Debería ver el Error 403, que indica que la aplicación web se ha detenido.
8. Inicie la segunda aplicación web.
9. Cierre el explorador web y espere unos minutos.
10. Inicie el explorador web y vaya a su dominio. Debería ver de nuevo la página de aplicación web predeterminada.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los registros de alias, consulte los artículos siguientes:

- [Tutorial: Configuración de un registro de alias para hacer referencia a una dirección IP pública de Azure](tutorial-alias-pip.md)
- [Tutorial: Configuración de un registro de alias para admitir nombres de dominio de Apex con Traffic Manager](tutorial-alias-tm.md)
- [Preguntas más frecuentes sobre DNS](./dns-faq.yml)

Para información sobre cómo migrar un nombre de DNS activo, consulte [Migración de un nombre de DNS activo a Azure App Service](../app-service/manage-custom-dns-migrate-domain.md).