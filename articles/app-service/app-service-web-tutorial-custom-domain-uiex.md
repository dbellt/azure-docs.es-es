---
title: 'Tutorial: Asignación de un nombre DNS personalizado existente'
description: Aprenda a agregar un nombre de dominio DNS personalizado (dominio individualizado) a una aplicación web, un back-end de una aplicación móvil o una aplicación de API en Azure App Service.
keywords: app service, azure app service, domain mapping, domain name, existing domain, hostname, vanity domain
ms.assetid: dc446e0e-0958-48ea-8d99-441d2b947a7c
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 08/25/2020
ms.custom: mvc, seodec18, devx-track-azurepowershell
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: bfd3429531f1473e099e3296ebfebd50c6d4d864
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110787923"
---
# <a name="tutorial-map-an-existing-custom-dns-name-to-azure-app-service"></a>Tutorial: Asignación de un nombre DNS personalizado existente a Azure App Service

En este tutorial se muestra cómo asignar cualquier <abbr title="Un nombre de dominio adquirido a un registrador de dominios, como GoDaddy, o un subdominio del dominio adquirido.">nombre de dominio DNS personalizado existente</abbr> en <abbr title="Un servicio basado en HTTP para hospedar aplicaciones web, API REST y aplicaciones de back-end para dispositivos móviles.">Azure App Service</abbr>.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Asignar un subdominio mediante el uso de un <abbr title="Un registro de nombre canónico DNS asigna un nombre de dominio a otro.">Registro CNAME</abbr>.
> * Asignar un dominio raíz mediante el uso de un <abbr title="Un registro de dirección en DNS asigna un nombre de host a una dirección IP.">Registro D</abbr>.
> * Asignar un dominio con carácter comodín mediante el uso de un registro CNAME.
> * Redirigir la dirección URL predeterminada a un directorio personalizado.

<hr/> 

## <a name="1-prepare-your-environment"></a>1. Preparación del entorno

* [Cree una aplicación de App Service](./index.yml) o use alguna aplicación que haya creado para otro tutorial.
* Asegúrese de que puede editar los registros DNS del dominio personalizado. Si aún no tiene un dominio personalizado, puede [adquirir un dominio de App Service](manage-custom-dns-buy-domain.md).

    <details>
        <summary>¿Qué necesito para editar registros DNS?</summary>
        Se requiere acceso al registro DNS del proveedor de dominios, como GoDaddy. Por ejemplo, para agregar entradas DNS para <code>contoso.com</code> y <code>www.contoso.com</code>, debe poder configurar las opciones de DNS del dominio raíz de <code>contoso.com</code>.
    </details>

<hr/> 

## <a name="2-prepare-the-app"></a>2. Preparación de la aplicación

Para asignar un nombre DNS personalizado a una aplicación, el <abbr title="Especifica la ubicación, el tamaño y las características de la granja de servidores web que hospeda la aplicación.">Plan de App Service</abbr> debe ser un nivel de pago (no el <abbr title="Un nivel de Azure App Service en el que la aplicación se ejecuta en las mismas máquinas virtuales que otras aplicaciones, incluidas las aplicaciones de otros clientes. Este nivel está pensado para las fases de desarrollo y pruebas.">**nivel gratis (F1)**</abbr>). Para más información, consulte [Introducción a los planes de Azure App Service](overview-hosting-plans.md).

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

<hr/> 

<a name="cname" aria-hidden="true"></a>

## <a name="3-get-a-domain-verification-id"></a>3. Obtención de un identificador de comprobación de dominio

Para agregar un dominio personalizado a la aplicación, tiene que confirmar la propiedad del dominio agregando un identificador de comprobación como un registro TXT con el proveedor de dominio. 

1. En el panel izquierdo de la página de la aplicación, seleccione **Dominios personalizados**. 
1. Copie el identificador del cuadro **Id. de verificación del dominio personalizado** en la página **Dominios personalizados** para realizar el siguiente paso.

    ![Captura de pantalla que muestra el identificador en el cuadro Id. de verificación del dominio personalizado.](./media/app-service-web-tutorial-custom-domain/get-custom-domain-verification-id.png)

    <details>
        <summary>¿Por qué necesito esto?</summary>
        La adición de identificadores de verificación de dominio al dominio personalizado puede impedir las entradas DNS pendientes y ayudar a evitar las adquisiciones de subdominios. En el caso de los dominios personalizados configurados previamente sin este identificador de verificación, debe agregar el identificador al registro de DNS para protegerlos del mismo riesgo. Para más información sobre esta amenaza común de alta gravedad, consulte <a href="/azure/security/fundamentals/subdomain-takeover">Adquisición de subdominios</a>.
    </details>
    
<a name="info"></a>

3. **(Solo registro D)** Para asignar un <abbr title="Un registro de dirección en DNS asigna un nombre de host a una dirección IP.">Registro D</abbr>, necesita la dirección IP externa de la aplicación. En la página **Dominios personalizados**, copie el valor de **Dirección IP**.

   ![Captura de pantalla que muestra cómo ir a una aplicación de Azure en el portal.](./media/app-service-web-tutorial-custom-domain/mapping-information.png)

<hr/> 

## <a name="4-create-the-dns-records"></a>4. Creación de los registros DNS

1. Inicie sesión en el sitio web de su proveedor de dominios.

    <details>
        <summary>¿Puedo administrar DNS desde mi proveedor de dominios mediante Azure?</summary>
        Si lo desea, puede usar Azure DNS para administrar los registros DNS para su dominio y configurar un nombre DNS personalizado para Azure App Service. Para más información, consulte <a href="/azure/dns/dns-delegate-domain-azure-dns">Tutorial: Hospedaje del dominio en Azure DNS</a>.
    </details>

1. Busque la página de administración de registros DNS. 

    <details>
        <summary>¿Cómo encuentro la página?</summary>
        <p>Cada proveedor de dominios tiene su propia interfaz de registros DNS, así que consulte la documentación del proveedor. Busque áreas del sitio etiquetadas como <strong>Nombre de dominio</strong>, <strong>DNS</strong> o <strong>Administración del servidor del nombres</strong>.</p>
        <p>A menudo, se puede encontrar la página de registros DNS al ver la información de la cuenta y al buscar un vínculo, como <strong>Mis dominios</strong>. Vaya a dicha página y, después, busque un vínculo que se llame algo así como <strong>Zone file</strong>, <strong>DNS Records</strong> o <strong>Advanced configuration</strong>.</p>
    </details>

   La captura de pantalla siguiente es un ejemplo de página de registros DNS:

   ![Captura de pantalla que muestra una página de registros DNS de ejemplo.](../../includes/media/app-service-web-access-dns-records-no-h/example-record-ui.png)

1. Seleccione **Agregar** o el widget adecuado para crear un registro. 

1. Seleccione el tipo de registro que se va a crear y siga las instrucciones. Puede usar un <abbr title="Un registro de nombre canónico en DNS asigna un nombre de dominio (un alias) a otro (el nombre canónico).">Registro CNAME</abbr> o un <abbr title="Un registro de dirección en DNS asigna un nombre de host a una dirección IP.">Registro D</abbr> para asignar un nombre DNS personalizado a App Service. 

    <details>
        <summary>¿Qué registro debo elegir?</summary>
        <div>
            <ul>
            <li>Para asignar el dominio raíz (por ejemplo, <code>contoso.com</code>), use un registro D. No use el registro CNAME para el registro raíz (para obtener información, consulte esta <a href="https://en.wikipedia.org/wiki/CNAME_record">entrada de Wikipedia</a>).</li>
            <li>Para asignar un subdominio (por ejemplo, <code>www.contoso.com</code>), use un registro CNAME.</li>
            <li>Puede asignar un subdominio directamente a la dirección IP de la aplicación con un registro A, pero es posible que <a href="/azure/app-service/overview-inbound-outbound-ips#when-inbound-ip-changes">la dirección IP cambie</a>. En su lugar, el registro CNAME se asigna al nombre de host de la aplicación, que es menos susceptible de cambiar.</li>
            <li>Para asignar un <a href="https://en.wikipedia.org/wiki/Wildcard_DNS_record">dominio con comodín</a> (por ejemplo, <code>*.contoso.com</code>), utilice un registro CNAME.</li>
            </ul>
        </div>
    </details>
    
# <a name="cname"></a>[CNAME](#tab/cname)

Para un subdominio como `www` en `www.contoso.com`, cree dos registros según la tabla siguiente:

| Tipo de registro | Host | Value | Comentarios |
| - | - | - |
| CNAME | `<subdomain>` (por ejemplo, `www`) | `<app-name>.azurewebsites.net` | La propia asignación de dominio. |
| TXT | `asuid.<subdomain>` (por ejemplo, `asuid.www`) | [El identificador de comprobación que obtuvo anteriormente](#3-get-a-domain-verification-id). | App Service accede al registro TXT `asuid.<subdomain>` para comprobar la propiedad del dominio personalizado. |

![Captura de pantalla que muestra cómo ir a una aplicación de Azure en el portal.](./media/app-service-web-tutorial-custom-domain/cname-record.png)
    
# <a name="a"></a>[A](#tab/a)

Para un dominio raíz como `contoso.com`, cree dos registros según la tabla siguiente:

| Tipo de registro | Host | Value | Comentarios |
| - | - | - |
| A | `@` | D www Dirección IP de [Copiar la dirección IP de la aplicación](#3-get-a-domain-verification-id) | La propia asignación de dominio (`@` normalmente representa el dominio raíz). |
| TXT | `asuid` | [El identificador de comprobación que obtuvo anteriormente](#3-get-a-domain-verification-id). | App Service accede al registro TXT `asuid.<subdomain>` para comprobar la propiedad del dominio personalizado. Para el dominio raíz, use `asuid`. |

![Captura de pantalla que muestra una página de registros DNS.](./media/app-service-web-tutorial-custom-domain/a-record.png)

<details>
<summary>¿Qué ocurre si quiero asignar un subdominio con un registro D?</summary>
Para asignar un subdominio como `www.contoso.com` con un registro D, en lugar de un registro CNAME recomendado, tanto el registro D como el registro TXT deberían parecerse a la tabla siguiente:

<div class="table-scroll-wrapper"><table class="table"><caption class="visually-hidden">Tabla 3</caption>
<thead>
<tr>
<th>Tipo de registro</th>
<th>Host</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>A</td>
<td><code>&lt;subdomain&gt;</code> (por ejemplo, <code>www</code>)</td>
<td>D www Dirección IP de <a href="#info" data-linktype="self-bookmark">Copiar la dirección IP de la aplicación</a></td>
</tr>
<tr>
<td>TXT</td>
<td><code>asuid.&lt;subdomain&gt;</code> (por ejemplo, <code>asuid.www</code>)</td>
<td><a href="#3-get-a-domain-verification-id" data-linktype="self-bookmark">El identificador de comprobación que obtuvo anteriormente</a>.</td>
</tr>
</tbody>
</table></div>
</details>

# <a name="wildcard-cname"></a>[Carácter comodín (CNAME)](#tab/wildcard)

Para un nombre con carácter comodín como `*` en `*.contoso.com`, cree dos registros según la tabla siguiente:

| Tipo de registro | Host | Value | Comentarios |
| - | - | - |
| CNAME | `*` | `<app-name>.azurewebsites.net` | La propia asignación de dominio. |
| TXT | `asuid` | [El identificador de comprobación que obtuvo anteriormente](#3-get-a-domain-verification-id). | App Service accede al registro TXT `asuid` para comprobar la propiedad del dominio personalizado. |

![Captura de pantalla que muestra cómo ir a una aplicación de Azure.](./media/app-service-web-tutorial-custom-domain/cname-record-wildcard.png)
    
-----

<details>
<summary>Los cambios se borran al salir de la página.</summary>
<p>En el caso de ciertos proveedores, como GoDaddy, los cambios en los registros DNS no entran en vigor hasta que se selecciona un vínculo <strong>Guardar cambios</strong> independiente.</p>
</details>

<hr/>

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

    <details>
        <summary>¿Qué significa la etiqueta de advertencia <strong>No seguro</strong>?</summary>
        Una etiqueta de advertencia para el dominio personalizado significa que aún no está enlazado a un certificado TLS/SSL. Todas las solicitudes HTTPS al dominio personalizado desde un explorador recibirán un error o una advertencia, en función del explorador. Para agregar un enlace TLS, consulte <a href="/azure/app-service/configure-ssl-bindings">Protección de un nombre DNS personalizado con un enlace TLS/SSL en Azure App Service</a>.
    </details>

    Si olvidó un paso o cometió un error tipográfico en alguna parte anteriormente, aparecerá un error de comprobación en la parte inferior de la página.

    ![Captura de pantalla que muestra un error de comprobación.](./media/app-service-web-tutorial-custom-domain/verification-error-cname.png)

<a name="a" aria-hidden="true"></a>

<a name="enable-a" aria-hidden="true"></a>

# <a name="a"></a>[A](#tab/a)

3. Escriba el nombre de dominio completo para el que ha configurado el registro A, como `contoso.com`. 

1. Seleccione **Validar**. Se abre la página **Agregar dominio personalizado**.

1. Asegúrese de que el **tipo de registro de nombre de host** esté establecido en el **registro D (ejemplo.com)** . Seleccione **Agregar dominio personalizado**.

    ![Captura de pantalla que muestra la adición de un nombre DNS a la aplicación.](./media/app-service-web-tutorial-custom-domain/validate-domain-name.png)

    El nuevo dominio personalizado puede tardar un tiempo en reflejarse en la página **Dominios personalizados** de la aplicación. Actualice el explorador para actualizar los datos.

    ![Captura de pantalla que muestra la adición de un registro A.](./media/app-service-web-tutorial-custom-domain/a-record-added.png)

    <details>
        <summary>¿Qué significa la etiqueta de advertencia <strong>No seguro</strong>?</summary>
        Una etiqueta de advertencia para el dominio personalizado significa que aún no está enlazado a un certificado TLS/SSL. Todas las solicitudes HTTPS al dominio personalizado desde un explorador recibirán un error o una advertencia, en función del explorador. Para agregar un enlace TLS, consulte <a href="/azure/app-service/configure-ssl-bindings">Protección de un nombre DNS personalizado con un enlace TLS/SSL en Azure App Service</a>.
    </details>
    
    Si olvidó un paso o cometió un error tipográfico en alguna parte anteriormente, aparecerá un error de comprobación en la parte inferior de la página.
    
    ![Captura de pantalla que muestra un error de comprobación.](./media/app-service-web-tutorial-custom-domain/verification-error.png)
    
<a name="wildcard" aria-hidden="true"></a>

# <a name="wildcard-cname"></a>[Carácter comodín (CNAME)](#tab/wildcard)

3. Escriba un nombre de dominio completo que coincida con el dominio con caracteres comodín. Por ejemplo, para el ejemplo de `*.contoso.com` puede usar `sub1.contoso.com`, `sub2.contoso.com`, `*.contoso.com` o cualquier otra cadena que coincida con el patrón de carácter comodín. A continuación, seleccione **Validar**.

    El botón **Agregar dominio personalizado** está activado.

1. Asegúrese de que en **Tipo de registro de nombre de host** está seleccionado **Registro CNAME (www\.example.com o cualquier subdominio)** . Seleccione **Agregar dominio personalizado**.

    ![Captura de pantalla que muestra la adición de un nombre DNS a la aplicación.](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname-wildcard.png)

    El nuevo dominio personalizado puede tardar un tiempo en reflejarse en la página **Dominios personalizados** de la aplicación. Actualice el explorador para actualizar los datos.

    <details>
        <summary>¿Qué significa la etiqueta de advertencia <strong>No seguro</strong>?</summary>
        Una etiqueta de advertencia para el dominio personalizado significa que aún no está enlazado a un certificado TLS/SSL. Todas las solicitudes HTTPS al dominio personalizado desde un explorador recibirán un error o una advertencia, en función del explorador. Para agregar un enlace TLS, consulte <a href="/azure/app-service/configure-ssl-bindings">Protección de un nombre DNS personalizado con un enlace TLS/SSL en Azure App Service</a>.
    </details>

-----

<hr/> 

## <a name="6-test-in-a-browser"></a>6. Prueba en un explorador

Vaya a los nombres DNS que configuró anteriormente.

![Captura de pantalla que muestra cómo ir a una aplicación de Azure.](./media/app-service-web-tutorial-custom-domain/app-with-custom-dns.png)

<a name="resolve-404-not-found" aria-hidden="true"></a>
<details>
<summary>Recibo un error HTTP 404 (No encontrado).</summary>
<ul>
<li>En el dominio personalizado configurado falta un registro A o un registro CNAME.  Compruebe si los registros DNS están expuestos mediante una herramienta de <a href="https://www.nslookup.io/">búsqueda de DNS en línea</a>.</li>
<li>El cliente del explorador ha almacenado en caché la dirección IP antigua del dominio. Borre la memoria caché y pruebe la resolución DNS de nuevo. En un equipo Windows, borre la memoria caché con <code>ipconfig /flushdns</code>.</li>
</ul>
</details>

<hr/> 

## <a name="migrate-an-active-domain"></a>Migración de un dominio activo

Para migrar un sitio en vivo y su nombre de dominio DNS a App Service sin tiempo de inactividad, consulte [Migración de un nombre DNS activo a Azure App Service](manage-custom-dns-migrate-domain.md).

<hr/> 

<a name="virtualdir" aria-hidden="true"></a>

## <a name="redirect-to-a-custom-directory"></a>Redirección a un directorio personalizado

<details>
<summary>¿Necesito esto?</summary>
<p>Depende de la aplicación. De forma predeterminada, App Service dirige las solicitudes web al directorio raíz del código de la aplicación. Sin embargo, algunos marcos web no se inician en el directorio raíz. Por ejemplo, <a href="https://laravel.com/">Laravel</a> se inicia en el subdirectorio <code>public</code>. Para continuar con el ejemplo de DNS de <code>contoso.com</code>, se podría acceder a la aplicación en <code>http://contoso.com/public</code>, pero, en su lugar, quiere redirigir <code>http://contoso.com</code> al directorio <code>public</code>. </p>
</details>

Aunque se trata de un escenario común, en realidad no implica la asignación de dominios personalizados, sino que se trata de personalizar el directorio virtual dentro de la aplicación.

1. Seleccione **Configuración de la aplicación** en el panel izquierdo de la página de la aplicación web.

1. En la parte inferior de la página, el directorio virtual raíz `/` apunta a `site\wwwroot` de forma predeterminada, que es el directorio raíz del código de la aplicación. Cambie esta configuración para que, en su lugar, apunte, por ejemplo, a `site\wwwroot\public` y después guarde los cambios.

    ![Captura de pantalla que muestra la personalización de un directorio virtual.](./media/app-service-web-tutorial-custom-domain/customize-virtual-directory.png)

1. Una vez finalizada la operación, vaya a la ruta de acceso raíz de la aplicación en el explorador (por ejemplo, `http://contoso.com` o `http://<app-name>.azurewebsites.net`) para comprobarlo.

<hr/> 

## <a name="automate-with-scripts"></a>Automatizar con scripts

Puede automatizar la administración de dominios personalizados con scripts mediante la [CLI de Azure](/cli/azure/install-azure-cli) o [Azure PowerShell](/powershell/azure/).

#### <a name="azure-cli"></a>Azure CLI

El comando siguiente agrega un nombre DNS personalizado configurado a una aplicación de App Service.

```bash 
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

<hr/> 

## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente tutorial para aprender a enlazar un certificado TLS/SSL personalizado a una aplicación web.

> [!div class="nextstepaction"]
> [Protección de un nombre DNS personalizado con un enlace TLS/SSL en Azure App Service](configure-ssl-bindings.md)
