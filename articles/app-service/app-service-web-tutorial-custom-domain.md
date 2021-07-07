---
title: 'Tutorial: Asignación de un nombre DNS personalizado existente'
description: Aprenda a agregar un nombre de dominio DNS personalizado (dominio individualizado) a una aplicación web, un back-end de una aplicación móvil o una aplicación de API en Azure App Service.
keywords: app service, azure app service, domain mapping, domain name, existing domain, hostname, vanity domain
ms.assetid: dc446e0e-0958-48ea-8d99-441d2b947a7c
ms.topic: tutorial
ms.date: 05/27/2021
ms.custom: mvc, seodec18, devx-track-azurepowershell
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./app-service-web-tutorial-custom-domain-uiex
ms.openlocfilehash: 6977bbe8d9854a771e31d2dcc0d49353ff29c4bf
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111591588"
---
# <a name="tutorial-map-an-existing-custom-dns-name-to-azure-app-service"></a>Tutorial: Asignación de un nombre DNS personalizado existente a Azure App Service

[Azure App Service](overview.md) proporciona un servicio de hospedaje web muy escalable y con aplicación de revisiones de un modo automático. En este tutorial se muestra cómo asignar un nombre del Sistema de nombres de dominio (DNS) personalizado existente a Azure App Service.

En este tutorial aprenderá a:

> [!div class="checklist"]
> * Asignar un subdominio mediante un [registro CNAME](https://en.wikipedia.org/wiki/CNAME_record).
> * Asignar un dominio raíz mediante un [registro A](https://en.wikipedia.org/wiki/List_of_DNS_record_types#A).
> * Asignar un [dominio con carácter comodín](https://en.wikipedia.org/wiki/Wildcard_DNS_record) mediante un registro CNAME.
> * Redirigir la dirección URL predeterminada a un directorio personalizado.


## <a name="1-prepare-your-environment"></a>1. Preparación del entorno

* [Cree una aplicación de App Service](./index.yml) o use alguna aplicación que haya creado para otro tutorial.
* Asegúrese de que puede editar los registros DNS del dominio personalizado. Si aún no tiene un dominio personalizado, puede [adquirir un dominio de App Service](manage-custom-dns-buy-domain.md).

    > [!NOTE]
    > Para editar los registros DNS, se requiere acceso al registro DNS del proveedor de dominios, como GoDaddy. Por ejemplo, para agregar entradas DNS para `contoso.com` y `www.contoso.com`, debe poder configurar las opciones de DNS del dominio raíz de `contoso.com`.


## <a name="2-prepare-the-app"></a>2. Preparación de la aplicación

Para asignar un nombre DNS personalizado a una aplicación web, el [plan de App Service](overview-hosting-plans.md) de dicha aplicación debe ser un nivel de pago, no **Gratis (F1)** .

#### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Abra [Azure Portal](https://portal.azure.com) e inicie sesión con su cuenta de Azure.

#### <a name="select-the-app-in-the-azure-portal"></a>Selección de la aplicación en Azure Portal

1. Busque y seleccione **App Services**.

   ![Captura de pantalla que muestra la selección de App Services.](./media/app-service-web-tutorial-custom-domain/app-services.png)

1. En la página **App Services**, seleccione el nombre de la aplicación de Azure.

   ![Captura de pantalla que muestra cómo ir a una aplicación de Azure en el portal.](./media/app-service-web-tutorial-custom-domain/select-app.png)

    Consulte la página de administración de la aplicación de App Service.

<a name="checkpricing" aria-hidden="true"></a>

#### <a name="check-the-pricing-tier"></a>Comprobar el plan de tarifa

1. En el panel izquierdo de la página de la aplicación, desplácese hasta la sección **Configuración** y seleccione **Escalar verticalmente (plan de App Service)** .

   ![Captura de pantalla que muestra el menú Escalar verticalmente (plan de App Service).](./media/app-service-web-tutorial-custom-domain/scale-up-menu.png)

1. El nivel actual de la aplicación aparece resaltado con un cuadro azul. Asegúrese de que la aplicación no está en el nivel **F1**. No se admite DNS personalizado en el nivel **F1**.

   ![Captura de pantalla que muestra los planes de tarifa recomendados.](./media/app-service-web-tutorial-custom-domain/check-pricing-tier.png)

1. Si el plan de App Service no está en el nivel **F1**, cierre la página **Escalar verticalmente** y vaya directamente a [3. Obtención de un identificador de comprobación de dominio](#3-get-a-domain-verification-id).

<a name="scaleup" aria-hidden="true"></a>

#### <a name="scale-up-the-app-service-plan"></a>Escalado vertical del plan de App Service

1. Seleccione cualquiera de los niveles no gratuitos (**D1**, **B1**, **B2**, **B3**, o cualquier nivel de la categoría **Producción**). Para ver las opciones adicionales, seleccione **Ver opciones adicionales**.

1. Seleccione **Aplicar**.

   ![Captura de pantalla que muestra la comprobación del plan de tarifa.](./media/app-service-web-tutorial-custom-domain/choose-pricing-tier.png)

   Cuando vea la siguiente notificación, significará que la operación de escalado se habrá completado.

   ![Captura de pantalla que muestra la confirmación de la operación de escalado.](./media/app-service-web-tutorial-custom-domain/scale-notification.png)


## <a name="3-get-a-domain-verification-id"></a>3. Obtención de un identificador de comprobación de dominio

Para agregar un dominio personalizado a la aplicación, tiene que confirmar la propiedad del dominio agregando un identificador de comprobación como un registro TXT con el proveedor de dominio. 

1. En el panel izquierdo de la página de la aplicación, seleccione **Dominios personalizados**. 
1. Copie el identificador del cuadro **Id. de verificación del dominio personalizado** en la página **Dominios personalizados** para realizar el siguiente paso.

    ![Captura de pantalla que muestra el identificador en el cuadro Id. de verificación del dominio personalizado.](./media/app-service-web-tutorial-custom-domain/get-custom-domain-verification-id.png)

    > [!WARNING]
    > La adición de identificadores de verificación de dominio al dominio personalizado puede impedir las entradas DNS pendientes y ayudar a evitar las adquisiciones de subdominios. En el caso de los dominios personalizados configurados previamente sin este identificador de verificación, debe agregar el identificador al registro de DNS para protegerlos del mismo riesgo. Para más información sobre esta amenaza común de alta gravedad, consulte [Adquisición de subdominios](../security/fundamentals/subdomain-takeover.md).
    
<a name="info"></a>

3. **(Solo registro A)** Para asignar un [registro A](https://en.wikipedia.org/wiki/List_of_DNS_record_types#A), se necesita la dirección IP externa de la aplicación. En la página **Dominios personalizados**, copie el valor de **Dirección IP**.

   ![Captura de pantalla que muestra cómo ir a una aplicación de Azure en el portal.](./media/app-service-web-tutorial-custom-domain/mapping-information.png)


## <a name="4-create-the-dns-records"></a>4. Creación de los registros DNS

1. Inicie sesión en el sitio web de su proveedor de dominios.

    > [!NOTE]
    > Si lo desea, puede usar Azure DNS para administrar los registros DNS para su dominio y configurar un nombre DNS personalizado para Azure App Service. Para más información, consulte el [Tutorial: Hospedaje del dominio en Azure DNS](../dns/dns-delegate-domain-azure-dns.md).

1. Busque la página de administración de registros DNS. 

    > [!NOTE]
    > Cada proveedor de dominios tiene su propia interfaz de registros DNS, así que consulte la documentación del proveedor. Busque áreas del sitio etiquetadas como **Nombre de dominio**, **DNS** o **Administración del servidor del nombres**.
    >
    > A menudo, se puede encontrar la página de registros DNS al ver la información de la cuenta y al buscar un vínculo, como **Mis dominios**. Vaya a dicha página y, después, busque un vínculo que se llame algo así como **Zone file**, **DNS Records** o **Advanced configuration**.

   La captura de pantalla siguiente es un ejemplo de página de registros DNS:

   ![Captura de pantalla que muestra una página de registros DNS de ejemplo.](../../includes/media/app-service-web-access-dns-records-no-h/example-record-ui.png)

1. Seleccione **Agregar** o el widget adecuado para crear un registro. 

1. Seleccione el tipo de registro que se va a crear y siga las instrucciones. Puede usar un [registro CNAME](https://en.wikipedia.org/wiki/CNAME_record) o un [registro D](https://en.wikipedia.org/wiki/List_of_DNS_record_types#A) para asignar un nombre DNS personalizado a App Service.

    > [!NOTE]
    > **Qué registro elegir**
    > 
    > * Para asignar el dominio raíz (por ejemplo, `contoso.com`), use un [registro A](https://en.wikipedia.org/wiki/List_of_DNS_record_types#A). No use el registro CNAME para el registro raíz (para obtener información, consulte [RFC 1912, sección 2.4](https://datatracker.ietf.org/doc/html/rfc1912#section-2.4)).
    > * Para asignar un subdominio (por ejemplo, `www.contoso.com`), use un [registro CNAME](https://en.wikipedia.org/wiki/CNAME_record).
    > * Puede asignar un subdominio directamente a la dirección IP de la aplicación con un registro A, pero es posible que [la dirección IP cambie](overview-inbound-outbound-ips.md#when-inbound-ip-changes). En su lugar, el registro CNAME se asigna al nombre de host predeterminado de la aplicación, que es menos susceptible de cambiar.
    > * Para asignar un [dominio con comodín](https://en.wikipedia.org/wiki/Wildcard_DNS_record) (por ejemplo, `*.contoso.com`), utilice un registro CNAME.
    
# <a name="cname"></a>[CNAME](#tab/cname)

Para un subdominio como `www` en `www.contoso.com`, cree dos registros según la tabla siguiente:

| Tipo de registro | Host | Value | Comentarios |
| - | - | - |
| CNAME | `<subdomain>` (por ejemplo, `www`) | `<app-name>.azurewebsites.net` | La propia asignación de dominio. |
| TXT | `asuid.<subdomain>` (por ejemplo, `asuid.www`) | [El identificador de comprobación que obtuvo anteriormente](#3-get-a-domain-verification-id). | App Service accede al registro TXT `asuid.<subdomain>` para comprobar la propiedad del dominio personalizado. |

![Captura de pantalla que muestra cómo ir a una aplicación de Azure en el portal.](./media/app-service-web-tutorial-custom-domain/cname-record.png)
    
# <a name="a"></a>[A](#tab/a)

- Para un dominio raíz como `contoso.com`, cree dos registros según la tabla siguiente:

    | Tipo de registro | Host | Value | Comentarios |
    | - | - | - |
    | A | `@` | D www Dirección IP de [Copiar la dirección IP de la aplicación](#3-get-a-domain-verification-id) | La propia asignación de dominio (`@` normalmente representa el dominio raíz). |
    | TXT | `asuid` | [El identificador de comprobación que obtuvo anteriormente](#3-get-a-domain-verification-id). | App Service accede al registro TXT `asuid.<subdomain>` para comprobar la propiedad del dominio personalizado. Para el dominio raíz, use `asuid`. |
    
    ![Captura de pantalla que muestra una página de registros DNS.](./media/app-service-web-tutorial-custom-domain/a-record.png)
    
- Para asignar un subdominio como `www.contoso.com` con un registro D, en lugar de un registro CNAME recomendado, tanto el registro D como el registro TXT deberían parecerse a la tabla siguiente:

    |Tipo de registro|Host|Value|
    |--- |--- |--- |
    |A|<subdomain> (por ejemplo, www)|D www Dirección IP de Copiar la dirección IP de la aplicación|
    |TXT|asuid.<subdomain> (por ejemplo, asuid.www)|El identificador de comprobación que obtuvo anteriormente.|
    
# <a name="wildcard-cname"></a>[Carácter comodín (CNAME)](#tab/wildcard)

Para un nombre con carácter comodín como `*` en `*.contoso.com`, cree dos registros según la tabla siguiente:

| Tipo de registro | Host | Value | Comentarios |
| - | - | - |
| CNAME | `*` | `<app-name>.azurewebsites.net` | La propia asignación de dominio. |
| TXT | `asuid` | [El identificador de comprobación que obtuvo anteriormente](#3-get-a-domain-verification-id). | App Service accede al registro TXT `asuid` para comprobar la propiedad del dominio personalizado. |

![Captura de pantalla que muestra cómo ir a una aplicación de Azure.](./media/app-service-web-tutorial-custom-domain/cname-record-wildcard.png)
    
-----

> [!NOTE]
> En el caso de ciertos proveedores, como GoDaddy, los cambios en los registros DNS no entran en vigor hasta que se selecciona un vínculo **Guardar cambios** independiente.

<a name="a" aria-hidden="true"></a>

<a name="enable-a" aria-hidden="true"></a>

<a name="wildcard" aria-hidden="true"></a>

<a name="cname" aria-hidden="true"></a>

## <a name="5-enable-the-mapping-in-your-app"></a>5. Habilitación de la asignación en la aplicación

1. En el panel izquierdo de la página de la aplicación en Azure Portal, seleccione **Dominios personalizados**.

    ![Captura de pantalla que muestra el menú Dominios personalizados.](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

1. Seleccione **Agregar dominio personalizado**.

    ![Captura de pantalla que muestra el elemento Agregar nombre de host.](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

# <a name="cname"></a>[CNAME](#tab/cname)

3. Escriba el nombre de dominio completo para el que ha agregado un registro CNAME, como `www.contoso.com`.

1. Seleccione **Validar**. Aparece la página **Agregar dominio personalizado**.

1. Asegúrese de que en **Tipo de registro de nombre de host** está seleccionado **CNAME (www\.example.com o cualquier subdominio)** . Seleccione **Agregar dominio personalizado**.

    ![Captura de pantalla que muestra el botón Agregar dominio personalizado.](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname.png)

    El nuevo dominio personalizado puede tardar un tiempo en reflejarse en la página **Dominios personalizados** de la aplicación. Actualice el explorador para actualizar los datos.

    ![Captura de pantalla que muestra la adición del registro CNAME.](./media/app-service-web-tutorial-custom-domain/cname-record-added.png)

    > [!NOTE]
    > Una etiqueta de advertencia para el dominio personalizado significa que aún no está enlazado a un certificado TLS/SSL. Todas las solicitudes HTTPS al dominio personalizado desde un explorador recibirán un error o una advertencia, en función del explorador. Para agregar un enlace TLS, consulte [Protección de un nombre DNS personalizado con un enlace TLS/SSL en Azure App Service](configure-ssl-bindings.md).

    Si olvidó un paso o cometió un error tipográfico en alguna parte anteriormente, aparecerá un error de comprobación en la parte inferior de la página.

    ![Captura de pantalla que muestra un error de comprobación.](./media/app-service-web-tutorial-custom-domain/verification-error-cname.png)

# <a name="a"></a>[A](#tab/a)

3. Escriba el nombre de dominio completo para el que ha configurado el registro A, como `contoso.com`. 

1. Seleccione **Validar**. Se abre la página **Agregar dominio personalizado**.

1. Asegúrese de que el **tipo de registro de nombre de host** esté establecido en el **registro D (ejemplo.com)** . Seleccione **Agregar dominio personalizado**.

    ![Captura de pantalla que muestra la adición de un nombre DNS a la aplicación.](./media/app-service-web-tutorial-custom-domain/validate-domain-name.png)

    El nuevo dominio personalizado puede tardar un tiempo en reflejarse en la página **Dominios personalizados** de la aplicación. Actualice el explorador para actualizar los datos.

    ![Captura de pantalla que muestra la adición de un registro A.](./media/app-service-web-tutorial-custom-domain/a-record-added.png)

    > [!NOTE]
    > Una etiqueta de advertencia para el dominio personalizado significa que aún no está enlazado a un certificado TLS/SSL. Todas las solicitudes HTTPS al dominio personalizado desde un explorador recibirán un error o una advertencia, en función del explorador. Para agregar un enlace TLS, consulte [Protección de un nombre DNS personalizado con un enlace TLS/SSL en Azure App Service](configure-ssl-bindings.md).
    
    Si olvidó un paso o cometió un error tipográfico en alguna parte anteriormente, aparecerá un error de comprobación en la parte inferior de la página.
    
    ![Captura de pantalla que muestra un error de comprobación.](./media/app-service-web-tutorial-custom-domain/verification-error.png)
    
# <a name="wildcard-cname"></a>[Carácter comodín (CNAME)](#tab/wildcard)

3. Escriba un nombre de dominio completo que coincida con el dominio con caracteres comodín. Por ejemplo, para el ejemplo de `*.contoso.com` puede usar `sub1.contoso.com`, `sub2.contoso.com`, `*.contoso.com` o cualquier otra cadena que coincida con el patrón de carácter comodín. A continuación, seleccione **Validar**.

    El botón **Agregar dominio personalizado** está activado.

1. Asegúrese de que en **Tipo de registro de nombre de host** está seleccionado **Registro CNAME (www\.example.com o cualquier subdominio)** . Seleccione **Agregar dominio personalizado**.

    ![Captura de pantalla que muestra la adición de un nombre DNS a la aplicación.](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname-wildcard.png)

    El nuevo dominio personalizado puede tardar un tiempo en reflejarse en la página **Dominios personalizados** de la aplicación. Actualice el explorador para actualizar los datos.

    > [!NOTE]
    > Una etiqueta de advertencia para el dominio personalizado significa que aún no está enlazado a un certificado TLS/SSL. Todas las solicitudes HTTPS al dominio personalizado desde un explorador recibirán un error o una advertencia, en función del explorador. Para agregar un enlace TLS, consulte [Protección de un nombre DNS personalizado con un enlace TLS/SSL en Azure App Service](configure-ssl-bindings.md).

-----


## <a name="6-test-in-a-browser"></a>6. Prueba en un explorador

Vaya a los nombres DNS que configuró anteriormente.

![Captura de pantalla que muestra cómo ir a una aplicación de Azure.](./media/app-service-web-tutorial-custom-domain/app-with-custom-dns.png)

<a name="resolve-404-not-found" aria-hidden="true"></a>

Si recibe un error HTTP 404 (No encontrado) al navegar a la dirección URL del dominio personalizado, las dos causas más comunes son:

* En el dominio personalizado configurado falta un registro A o un registro CNAME. Es posible que haya eliminado el registro DNS después de habilitar la asignación en la aplicación. Compruebe si los registros DNS están configurados correctamente mediante una herramienta de <a href="https://www.nslookup.io/">búsqueda de DNS en línea</a>.
* El cliente del explorador ha almacenado en caché la dirección IP antigua del dominio. Borre la memoria caché y pruebe la resolución DNS de nuevo. En un equipo Windows, borre la memoria caché con `ipconfig /flushdns`.


## <a name="migrate-an-active-domain"></a>Migración de un dominio activo

Para migrar un sitio en vivo y su nombre de dominio DNS a App Service sin tiempo de inactividad, consulte [Migración de un nombre DNS activo a Azure App Service](manage-custom-dns-migrate-domain.md).

<a name="virtualdir" aria-hidden="true"></a>

## <a name="redirect-to-a-custom-directory"></a>Redirección a un directorio personalizado

> [!NOTE]
> De forma predeterminada, App Service dirige las solicitudes web al directorio raíz del código de la aplicación. Sin embargo, algunos marcos web no se inician en el directorio raíz. Por ejemplo, [Laravel](https://laravel.com/) se inicia en el subdirectorio `public`. Para continuar con el ejemplo de DNS de `contoso.com`, se podría acceder a la aplicación en `http://contoso.com/public`, pero normalmente, en su lugar, quiere redirigir `http://contoso.com` al directorio `public`.

Aunque se trata de un escenario común, en realidad no implica la asignación de DNS personalizados, sino que se trata de personalizar el directorio virtual dentro de la aplicación.

1. Seleccione **Configuración de la aplicación** en el panel izquierdo de la página de la aplicación web.

1. En la parte inferior de la página, el directorio virtual raíz `/` apunta a `site\wwwroot` de forma predeterminada, que es el directorio raíz del código de la aplicación. Cambie esta configuración para que, en su lugar, apunte, por ejemplo, a `site\wwwroot\public` y después guarde los cambios.

    ![Captura de pantalla que muestra la personalización de un directorio virtual.](./media/app-service-web-tutorial-custom-domain/customize-virtual-directory.png)

1. Una vez finalizada la operación, vaya a la ruta de acceso raíz de la aplicación en el explorador (por ejemplo, `http://contoso.com` o `http://<app-name>.azurewebsites.net`) para comprobarlo.


## <a name="automate-with-scripts"></a>Automatizar con scripts

Puede automatizar la administración de dominios personalizados con scripts mediante la [CLI de Azure](/cli/azure/install-azure-cli) o [Azure PowerShell](/powershell/azure/).

#### <a name="azure-cli"></a>Azure CLI

El comando siguiente agrega un nombre DNS personalizado configurado a una aplicación de App Service.

```azurecli 
az webapp config hostname add \
    --webapp-name <app-name> \
    --resource-group <resource_group_name> \
    --hostname <fully_qualified_domain_name>
``` 

Para más información, consulte [Asignación de un dominio personalizado a una aplicación web](scripts/cli-configure-custom-domain.md).

#### <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

El comando siguiente agrega un nombre DNS personalizado configurado a una aplicación de App Service.

```powershell  
Set-AzWebApp `
    -Name <app-name> `
    -ResourceGroupName <resource_group_name> ` 
    -HostNames @("<fully_qualified_domain_name>","<app-name>.azurewebsites.net")
```

Para obtener más información, vea [Asignación de un dominio personalizado a una aplicación web](scripts/powershell-configure-custom-domain.md).


## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente tutorial para aprender a enlazar un certificado TLS/SSL personalizado a una aplicación web.

> [!div class="nextstepaction"]
> [Protección de un nombre DNS personalizado con un enlace TLS/SSL en Azure App Service](configure-ssl-bindings.md)
