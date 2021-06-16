---
title: Implementación y configuración de la versión preliminar de Azure Firewall Prémium
description: Aprenda a implementar y configurar Azure Firewall Prémium.
author: vhorne
ms.service: firewall
services: firewall
ms.topic: how-to
ms.date: 05/27/2021
ms.author: victorh
ms.openlocfilehash: 7984cff6b6cf0f70fc50493fd680d7203e09a81b
ms.sourcegitcommit: e832f58baf0b3a69c2e2781bd8e32d4f1ae932c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110585860"
---
# <a name="deploy-and-configure-azure-firewall-premium-preview"></a>Implementación y configuración de la versión preliminar de Azure Firewall Prémium

> [!IMPORTANT]
> Azure Firewall Prémium se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

 La versión preliminar de Azure Firewall Prémium es un firewall de próxima generación con funcionalidades que son necesarias en entornos de alta confidencialidad y regulados. Contiene las características siguientes:

- **Inspección de TLS**: descifra el tráfico saliente, procesa los datos y, luego, los cifra y los envía al destino.
- **IDPS**: sistema de detección y prevención de intrusiones en la red (IDPS) que permite supervisar la actividad malintencionada de la red, registrar información sobre esta actividad, notificarla y, opcionalmente, intentar bloquearla.
- **Filtrado de direcciones URL**: extiende la funcionalidad de filtrado de nombres de dominio completos de Azure Firewall para tener en cuenta una dirección URL completa. Por ejemplo, `www.contoso.com/a/c` en lugar de `www.contoso.com`.
- **Categorías web**: los administradores pueden permitir o denegar el acceso de los usuarios a categorías de sitios web como, por ejemplo, sitios web de apuestas, de redes sociales u otros.

Para más información, consulte [Características de Azure Firewall Prémium](premium-features.md).

Usará una plantilla para implementar un entorno de prueba que tenga una red virtual central (10.0.0.0/16) con tres subredes:
- una subred de trabajo (10.0.10.0/24)
- una subred de Azure Bastion (10.0.20.0/24)
- una subred de firewall (10.0.100.0/24)

Para simplificar, en este entorno de prueba se usa una sola red virtual central. En producción, suele ser más habitual el uso de una [topología de tipo "hub-and-spoke"](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) con redes virtuales emparejadas.

:::image type="content" source="media/premium-deploy/premium-topology.png" alt-text="Topología de red virtual central":::

La máquina virtual de trabajo es un cliente que envía solicitudes HTTP/S a través del firewall.

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="deploy-the-infrastructure"></a>Implementar la infraestructura

La plantilla implementa un entorno de pruebas completo para Azure Firewall Prémium habilitado con IDPS, inspección de TLS, filtrado de direcciones URL y categorías web:

- una instancia de Azure Firewall Prémium y una directiva de firewall nuevas con valores de configuración predefinidos para permitir la validación fácil de sus funcionalidades principales (IDPS, inspección de TLS, filtrado de direcciones URL y categorías web).
- implementa todas las dependencias, incluidas Key Vault y una identidad administrada. En un entorno de producción, es posible que estos recursos ya se hayan creado y no sean necesarios en la misma plantilla.
- genera una entidad de certificación raíz autofirmada y la implementa en la instancia de Key Vault generada.
-  genera una entidad de certificación intermedia derivada y la implementa en una máquina virtual de prueba de Windows (WorkerVM).
- también se implementa un host bastión (BastionHost), que se puede usar para conectarse a la máquina de pruebas de Windows (WorkerVM).



[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.network%2Fazurefirewall-premium%2Fazuredeploy.json)

## <a name="test-the-firewall"></a>Probar el firewall

Ahora puede probar IDPS, la inspección de TLS, el filtrado web y las categorías web.

### <a name="add-firewall-diagnostics-settings"></a>Adición de la configuración de diagnóstico de firewall

Para recopilar registros de firewall, debe agregar la configuración de diagnóstico específica para ello.

1. Seleccione **DemoFirewall** y, en **Supervisión**, elija **Configuración de diagnóstico**.
2. Seleccione **Agregar configuración de diagnóstico**.
3. En **Diagnostic setting name** (Nombre de la configuración de diagnóstico), escriba *fw-diag*.
4. En **registro**, seleccione **AzureFirewallApplicationRule** y **AzureFirewallNetworkRule**.
5. En **Detalles del destino**, seleccione **Enviar a Log Analytics**.
6. Seleccione **Guardar**.

### <a name="idps-tests"></a>Pruebas de IDPS

Para probar IDPS, deberá implementar su propio servidor web interno con un certificado de servidor adecuado. Para más información sobre los requisitos de certificado de la versión preliminar de Azure Firewall Prémium, consulte [Certificados de la versión preliminar de Azure Firewall Prémium](premium-certificates.md).

Puede usar `curl` para controlar varios encabezados HTTP y simular el tráfico malintencionado.

#### <a name="to-test-idps-for-http-traffic"></a>Para probar el tráfico HTTP en IDPS:

1. En la máquina virtual WorkerVM, abra una ventana del símbolo del sistema de administrador.
2. Escriba el siguiente comando en el símbolo del sistema:

   `curl -A "BlackSun" <your web server address>`
3. Verá la respuesta del servidor web.
4. Vaya a los registros de reglas de red de firewall en Azure Portal para buscar una alerta similar al siguiente mensaje:

   :::image type="content" source="media/premium-deploy/alert-message.png" alt-text="Mensaje de alerta":::

   > [!NOTE]
   > Los datos pueden tardar algún tiempo en comenzar a mostrarse en los registros. Espere al menos 20 minutos a que los registros empiecen a mostrar los datos.
5. Agregue una regla de firma para la firma 2008983:

   1. Seleccione **DemoFirewallPolicy** y, en **Configuración**, seleccione **IDPS(preview)** (IDPS [versión preliminar]).
   1. Seleccione la pestaña **Signature rules** (Reglas de firma).
   1. En **Signature ID** (Id. de firma), en el cuadro de texto abierto, escriba *2008983*.
   1. En **Modo**, seleccione **Denegar**.
   1. Seleccione **Guardar**.
   1. Espere a que la implementación finalice antes de continuar.



6. En WorkerVM, vuelva a ejecutar el comando `curl`:

   `curl -A "BlackSun" <your web server address>`

   Puesto que la solicitud HTTP ahora está bloqueada por el firewall, después de que expire el tiempo de espera de la conexión verá la siguiente salida:

   `read tcp 10.0.100.5:55734->10.0.20.10:80: read: connection reset by peer`

7. Vaya a los registros de Supervisión en Azure Portal y busque el mensaje de la solicitud bloqueada.
<!---8. Now you can bypass the IDPS function using the **Bypass list**.

   1. On the **IDPS (preview)** page, select the **Bypass list** tab.
   2. Edit **MyRule** and set **Destination** to *10.0.20.10, which is the ServerVM private IP address.
   3. Select **Save**.
1. Run the test again: `curl -A "BlackSun" http://server.2020-private-preview.com` and now you should get the `Hello World` response and no log alert. --->

#### <a name="to-test-idps-for-https-traffic"></a>Para probar el tráfico HTTP en IDPS

Repita estas pruebas de curl mediante HTTPS en lugar de HTTP. Por ejemplo:

`curl --ssl-no-revoke -A "BlackSun" <your web server address>`

Debería ver los mismos resultados que tenía con las pruebas HTTP.

### <a name="tls-inspection-with-url-filtering"></a>Inspección de TLS con filtrado de direcciones URL

Siga estos pasos para probar la inspección de TLS con el filtrado de direcciones URL.

1. Edite las reglas de aplicación de la directiva de firewall y agregue una nueva regla denominada `AllowURL` a la colección de reglas `AllowWeb`. Configure la dirección URL de destino `www.nytimes.com/section/world`, la dirección IP de origen **\* *, el tipo de destino* URL (versión preliminar)** , seleccione **TLS inspection (preview)** (Inspección de TLS [versión preliminar]) y los protocolos **http, https**.

3. Una vez finalizada la implementación, abra un explorador en WorkerVM, vaya a `https://www.nytimes.com/section/world` y valide que la respuesta HTML se muestra como se espera en el explorador.
4. En Azure Portal, puede ver la dirección URL completa en los registros de supervisión de reglas de aplicación:

      :::image type="content" source="media/premium-deploy/alert-message-url.png" alt-text="Mensaje de alerta que muestra la dirección URL":::

Es posible que algunas páginas HTML parezcan incompletas porque hacen referencia a otras direcciones URL denegadas. Para solucionar este problema, se puede adoptar el siguiente enfoque:

- Si la página HTML contiene vínculos a otros dominios, puede agregar estos dominios a una nueva regla de aplicación donde solo se permita el acceso a estos nombres de dominio completos.
- Si la página HTML contiene vínculos a direcciones URL secundarias, puede modificar la regla y agregar un asterisco a la dirección URL. Por ejemplo: `targetURLs=www.nytimes.com/section/world*`

   Como alternativa, puede agregar una nueva dirección URL a la regla. Por ejemplo: 

   `www.nytimes.com/section/world, www.nytimes.com/section/world/*`

### <a name="web-categories-testing"></a>Pruebas de categorías web

Vamos a crear una regla de aplicación que permita el acceso a sitios web de deportes.
1. En el portal, abra el grupo de recursos y seleccione **DemoFirewallPolicy**.
2. Seleccione **Reglas de aplicación** y, luego, **Agregar una colección de reglas**.
3. En **Nombre**, escriba *GeneralWeb*, en **Prioridad** seleccione *103* y en **Rule collection group** (Grupo de colección de reglas), seleccione **DefaultApplicationRuleCollectionGroup**.
4. En **Reglas**, en **Nombre**, escriba *AllowSports*, en **Origen** *\** en **Protocolo**, *http, https*, seleccione **TLS inspection** (Inspección de TLS), en **Tipo de destino**, seleccione *Web categories (preview)* (Categorías web [versión preliminar]) y, en **Destino**, seleccione *Deportes*.
5. Seleccione **Agregar**.

      :::image type="content" source="media/premium-deploy/web-categories.png" alt-text="Categoría web de deportes":::
6. Cuando finalice la implementación, vaya a **WorkerVM** y abra un explorador web a `https://www.nfl.com`.

   Verá la página web de la NFL; el registro de reglas de aplicación muestra que se encontró coincidencia con una regla **Web Category: Sports** (Categoría web: Deportes) y se permitió la solicitud.

## <a name="next-steps"></a>Pasos siguientes

- [Versión preliminar de Azure Firewall Prémium en Azure Portal](premium-portal.md)