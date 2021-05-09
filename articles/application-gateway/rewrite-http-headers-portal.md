---
title: 'Reescritura de los encabezados HTTP de solicitud y respuesta en el portal: Azure Application Gateway'
description: Obtenga información sobre cómo usar Azure Portal para configurar una instancia de Azure Application Gateway para reescribir los encabezados HTTP en las solicitudes y respuestas que se pasan a través de la puerta de enlace.
services: application-gateway
author: abshamsft
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/13/2019
ms.author: absha
ms.custom: mvc
ms.openlocfilehash: 3427dfafe1918590f215793acbae3d5fd290e604
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108205240"
---
# <a name="rewrite-http-request-and-response-headers-with-azure-application-gateway---azure-portal"></a>Reescribir los encabezados HTTP de solicitud y respuesta con Azure Application Gateway mediante Azure Portal

En este artículo se describe cómo usar el Azure Portal para configurar una instancia de [SKU de Application Gateway v2](./application-gateway-autoscaling-zone-redundant.md) para reescribir los encabezados HTTP en las solicitudes y respuestas.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="before-you-begin"></a>Antes de empezar

Debe tener una instancia de la SKU de Application Gateway v2 para completar los pasos descritos en este artículo. No se admite la reescritura de encabezados en la SKU de v1. Si no tiene la SKU de v2, cree una instancia de [SKU de Application Gateway v2](./tutorial-autoscale-ps.md) antes de comenzar.

## <a name="create-required-objects"></a>Crear los objetos necesarios

Para configurar la reescritura de encabezados HTTP, es preciso completar estos pasos.

1. Cree los objetos necesarios para la reescritura de encabezados HTTP:

   - **Acción de reescritura**: se usa para especificar los campos de solicitud y encabezado de solicitud que se quieren reescribir y el nuevo valor para los encabezados. Puede asociar una o varias condiciones de reescritura con una acción de reescritura.

   - **Condición de reescritura**: Una configuración opcional. Las condiciones de reescritura evalúan el contenido de las solicitudes y respuestas HTTP(S). La acción de reescritura tendrá lugar si la solicitud o respuesta HTTP(S) coinciden con la condición de reescritura.

     Si asocia más de una condición con una acción, la acción se produce solo cuando se cumplen todas las condiciones. En otras palabras, se trata de una operación AND lógica.

   - **Regla de reescritura**: Contiene varias combinaciones de acción de reescritura y condición de reescritura.

   - **Secuencia de reglas**: Ayuda a determinar el orden en el que se ejecutan las reglas de reescritura. Esta configuración es útil cuando hay varias reglas de reescritura en un conjunto de reescritura. Una regla de reescritura que tiene un valor de secuencia de reglas más bajo se ejecuta primero. Si asigna el mismo valor de secuencia de reglas a dos reglas de reescritura, el orden de ejecución es no determinista.

   - **Conjunto de reescritura**: Contiene varias reglas de reescritura que se asociarán con una regla de enrutamiento de solicitudes.

2. Conecte el conjunto de reescritura a una regla de enrutamiento. La configuración de reescritura se conecta al agente de escucha de origen mediante la regla de enrutamiento. Cuando usa una regla de enrutamiento básica, la configuración de reescritura de encabezados se asocia a un agente de escucha de origen y es una reescritura de encabezados global. Cuando usa una regla de enrutamiento basada en rutas, la configuración de reescritura de encabezados se define en la asignación de la ruta de URL. En este caso, solo se aplica al área específica de la ruta de acceso de un sitio.

Puede crear varios conjuntos de reescritura de encabezados HTTP y aplicar cada conjunto de reescritura a varios agentes de escucha. Pero solo puede aplicar un conjunto de reescritura a un agente de escucha específico.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com/) con su cuenta de Azure.

## <a name="configure-header-rewrite"></a>Configuración de la reescritura de encabezados

En este ejemplo, se modificará una dirección URL de redireccionamiento al reescribir el encabezado de ubicación en la respuesta HTTP enviada por una aplicación de back-end.

1. Seleccione **Todos los recursos** y seleccione su puerta de enlace de aplicación.

2. Seleccione **Reescrituras** en el panel izquierdo.

3. Seleccione **Conjunto de reescritura**:

   ![Agregar un conjunto de reescritura](media/rewrite-http-headers-portal/add-rewrite-set.png)

4. Proporcione un nombre para el conjunto de reescritura y asócielo a una regla de enrutamiento:

   - Escriba el nombre del conjunto de reescritura en el cuadro **Nombre**.
   - Seleccione una o varias de las reglas enumeradas en la lista **Reglas de enrutamiento asociadas**. Puede seleccionar solo las reglas que no han asociado con otros conjuntos de reescritura. Las reglas que se han asociado con otros conjuntos de reescritura aparecen atenuadas.
   - Seleccione **Next** (Siguiente).
   
     ![Agregar el nombre y la asociación](media/rewrite-http-headers-portal/name-and-association.png)

5. Crear una regla de reescritura:

   - Seleccione **Agregar regla de reescritura**.

     ![Agregar regla de reescritura](media/rewrite-http-headers-portal/add-rewrite-rule.png)

   - Escriba un nombre para la regla de reescritura en el cuadro **Nombre de la regla de reescritura**. Escriba un número en el cuadro **Secuencia de reglas**.

     ![Agregar un nombre de regla de reescritura](media/rewrite-http-headers-portal/rule-name.png)

6. En este ejemplo, vamos a reescribir el encabezado de ubicación solo cuando contenga una referencia a azurewebsites.net. Para ello, agregue una condición para evaluar si el encabezado de ubicación en la respuesta contiene azurewebsites.net:

   - Seleccione **Agregar condición** y, luego, seleccione el cuadro que contiene las instrucciones **If** para expandirlo.

     ![Adición de una condición](media/rewrite-http-headers-portal/add-condition.png)

   - En la lista **Tipo de variable para comprobar**, seleccione **Encabezado HTTP**.

   - En la lista **Tipo de encabezado**, seleccione **Respuesta**.

   - Dado que en este ejemplo se evalúa el encabezado de ubicación, que es un encabezado común, seleccione **Encabezado común** en **Nombre de encabezado**.

   - En la lista **Encabezado común**, seleccione **Ubicación**.

   - En **Distingue mayúsculas de minúsculas**, seleccione **No**.

   - En la lista **Operador**, seleccione **igual (=)** .

   - Escriba un patrón de expresión regular. En este ejemplo, se usará el patrón `(https?)://.*azurewebsites.net(.*)$`.

   - Seleccione **Aceptar**.

     ![Configurar una condición If](media/rewrite-http-headers-portal/condition.png)

7. Agregar una acción para reescribir el encabezado de ubicación:

   - En la lista **Tipo de acción**, seleccione **Conjunto**.

   - En la lista **Tipo de encabezado**, seleccione **Respuesta**.

   - En **Nombre de encabezado**, seleccione **Encabezado común**.

   - En la lista **Encabezado común**, seleccione **Ubicación**.

   - Escriba el valor de encabezado. En este ejemplo, vamos a usar `{http_resp_Location_1}://contoso.com{http_resp_Location_2}` como valor del encabezado. Este valor reemplazará *azurewebsites.net* con *contoso.com* en el encabezado de ubicación.

   - Seleccione **Aceptar**.

     ![Agregar una acción](media/rewrite-http-headers-portal/action.png)

8. Seleccione **Crear** para crear el conjunto de reescritura:

   ![Seleccionar Crear](media/rewrite-http-headers-portal/create.png)

9. Se abrirá la vista de Conjunto de reescritura. Verifique que el conjunto de reescritura que ha creado está en la lista de conjuntos de reescritura:

   ![Vista de conjunto de reescritura](media/rewrite-http-headers-portal/rewrite-set-list.png)

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo configurar algunos casos de uso comunes, consulte los [escenarios de reescritura de encabezado común](./rewrite-http-headers-url.md).