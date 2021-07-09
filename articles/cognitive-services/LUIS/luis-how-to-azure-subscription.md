---
title: Creación y administración de recursos de LUIS
titleSuffix: Azure Cognitive Services
description: Aprenda a usar y administrar recursos de Azure para la aplicación LUIS.
services: cognitive-services
author: aahill
ms.author: aahi
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/17/2021
ms.openlocfilehash: 60e5d37f57d6a87d9190650f044a1ffc991e60fb
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110096097"
---
# <a name="how-to-create-and-manage-luis-resources"></a>Creación y administración de recursos de LUIS

Utilice este artículo para obtener información sobre los tipos de recursos de Azure que puede usar con LUIS y cómo administrarlos.

## <a name="authoring-resource"></a>Recurso de creación

Un recurso de creación le permite crear, administrar, entrenar, probar y publicar las aplicaciones. Hay un [plan de tarifa](https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/) disponible para el recurso de creación de LUIS: el nivel gratis (F0) que le proporciona:

* 1 millón de transacciones de creación 
* 1 000 solicitudes de punto de conexión de predicción de prueba al mes.

Puede utilizar [v3.0-preview LUIS Programmatic APIs](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c2f) (API de programación de LUIS v3.0-versión preliminar) para administrar los recursos de creación. 

## <a name="prediction-resource"></a>Recurso de predicción

Un recurso de predicción permite consultar el punto de conexión de predicción más allá de las 1 000 solicitudes proporcionadas por el recurso de creación. El recurso de predicción dispone de dos [planes de tarifa](https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/):

* El recurso de predicción (F0) gratis, que le otorga 10 000 solicitudes de punto de conexión de predicción al mes.
* Recurso de predicción (S0) estándar, que es el plan de pago. 

Puede utilizar [v3.0-preview LUIS Endpoint API](https://westus.dev.cognitive.microsoft.com/docs/services/luis-endpoint-api-v3-0-preview/operations/5f68f4d40a511ce5a7440859) (API de punto de conexión de LUIS v3.0-versión preliminar) para administrar los recursos de predicción.

> [!Note]
> * También puede usar un [recurso de varios servicios](../cognitive-services-apis-create-account-cli.md?tabs=multiservice) para obtener un único punto de conexión que puede utilizar para varios servicios de Cognitive Services.
> * LUIS proporciona dos tipos de recursos F0 (nivel gratuito): uno para las transacciones de creación y otro para las transacciones de predicción. Si está agotando la cuota de transacciones de predicción gratuitas, asegúrese de que está usando el recurso de predicción F0, que le otorga 10 000 transacciones gratuitas al mes, y no el recurso de creación, que le da 1000 transacciones de predicción al mes.
> * Debe crear las aplicaciones de LUIS en las [regiones](luis-reference-regions.md#publishing-regions) donde desee publicar y consultar contenido.

## <a name="create-luis-resources"></a>Creación de recursos de LUIS

Para crear recursos de LUIS, puede usar el portal de LUIS, [Azure Portal](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesLUISAllInOne) o la CLI de Azure. Después de crear los recursos, deberá asignarlos a las aplicaciones para que las utilicen.

# <a name="luis-portal"></a>[Portal de LUIS](#tab/portal)

### <a name="create-a-luis-authoring-resource-using-the-luis-portal"></a>Creación de un recurso de creación de LUIS mediante el portal de LUIS

1. Inicie sesión en el [portal de LUIS](https://www.luis.ai), seleccione su país o región, y acepte las condiciones de uso. Si ve la sección **Aplicaciones** en el portal, ya existe un recurso de LUIS y puede omitir el paso siguiente.

2. En la ventana **Choose an authoring** (Elección de una creación) que aparece, busque una suscripción de Azure y un recurso de creación de LUIS. Si no tiene un recursos, puede crearlo.

    :::image type="content" source="./media/luis-how-to-azure-subscription/choose-authoring-resource.png" alt-text="Elija un tipo de recurso de creación de Language Understanding.":::
    
    Al crear un recurso de creación, especifique la siguiente información:
    * **Nombre de inquilino**: el inquilino al que está asociada la suscripción de Azure.
    * **Nombre de la suscripción de Azure**: la suscripción que se facturará para el recurso.
    * **Nombre del grupo de recursos de Azure**: nombre personalizado del grupo de recursos que elija o cree. Los grupos de recursos le permiten agrupar los recursos de Azure para el acceso y la administración.
    * **Nombre del recurso de Azure**: un nombre personalizado que elija y que utilice como parte de la dirección URL para las consultas de punto de conexión de creación y predicción.
    * **Plan de tarifa**: el plan de tarifa determina el número máximo de transacciones por segundo y mes.

### <a name="create-a-luis-prediction-resource-using-the-luis-portal"></a>Creación de un recurso de predicción de LUIS mediante el portal de LUIS

[!INCLUDE [Create LUIS Prediction resource in LUIS portal](./includes/add-prediction-resource-portal.md)]

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

### <a name="create-luis-resources-in-the-azure-cli"></a>Creación de recursos de LUIS en la CLI de Azure

Use la [CLI de Azure](/cli/azure/install-azure-cli) para crear cada recursos de manera individual.

> [!TIP]
> * El recurso de creación `kind` es `LUIS.Authoring`  
> * El recurso de predicción `kind` es `LUIS` 

1. Inicie sesión en la CLI de Azure.

    ```azurecli
    az login
    ```

    Este comando abre un explorador para que pueda seleccionar la cuenta correcta y proporcionar autenticación.

2. Defina un recurso de creación de LUIS de tipo `LUIS.Authoring`, denominado `my-luis-authoring-resource`. Créelo en el grupo de recursos _existente_ denominado `my-resource-group` para la región `westus`.

    ```azurecli
    az cognitiveservices account create -n my-luis-authoring-resource -g my-resource-group --kind LUIS.Authoring --sku F0 -l westus --yes
    ```

3. Cree un recurso de punto de conexión de predicción de LUIS de tipo `LUIS`, denominado `my-luis-prediction-resource`. Créelo en el grupo de recursos _existente_ denominado `my-resource-group` para la región `westus`. Si quiere contar con una capacidad de proceso superior a la que proporciona el nivel de servicio gratis, cambie `F0` a `S0`. Más información sobre [planes de tarifa y la capacidad de proceso](luis-limits.md#key-limits).

    ```azurecli
    az cognitiveservices account create -n my-luis-prediction-resource -g my-resource-group --kind LUIS --sku F0 -l westus --yes
    ```

---


## <a name="assign-luis-resources"></a>Asignación de recursos de LUIS

La creación de un recurso no significa necesariamente que se pueda utilizar, lo tiene que asignar a las aplicaciones. Puede asignar un recurso de creación de una sola aplicación o de todas las aplicaciones en LUIS.

# <a name="luis-portal"></a>[Portal de LUIS](#tab/portal)

### <a name="assign-resources-using-the-luis-portal"></a>Asignación de recursos mediante el portal de LUIS

**Asignación de un recurso de creación a todas las aplicaciones** 

 En el siguiente procedimiento se asigna el recurso de creación a todas las aplicaciones.

1. Inicie sesión en el [portal de LUIS](https://www.luis.ai).
1. En la esquina superior derecha, seleccione su cuenta de usuario y, a continuación, seleccione **Configuración**.
1. En la página **User Settings** (Configuración de usuario), seleccione **Add authoring resource** (Agregar recurso de creación) y, a continuación, seleccione un recurso de creación existente. Seleccione **Guardar**.

**Asignación de un recurso a una aplicación específica**

En el siguiente procedimiento se asigna un recurso a una aplicación específica.

1. Inicie sesión en el [portal de LUIS](https://www.luis.ai). Seleccione una aplicación de la lista **Mis aplicaciones**.
1. Vaya a **Administrar** > **Recursos de Azure**:

    :::image type="content" source="./media/luis-how-to-azure-subscription/manage-azure-resources-prediction.png" alt-text="Elección de un tipo de recurso de predicción de Language Understanding." lightbox="./media/luis-how-to-azure-subscription/manage-azure-resources-prediction.png":::

1. En la pestaña **Prediction resource** (Recurso de predicción) o **Authoring resource** (Recurso de creación), seleccione el botón **Add prediction resource** (Agregar recurso de predicción) o **Add authoring resource** (Agregar recurso de creación), respectivamente.
1. Use los campos del formulario para buscar el recurso correcto y, a continuación, seleccione **Save** (Guardar).

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

## <a name="assign-prediction-resource-programmatically"></a>Asignación de recursos de predicción mediante programación 

En el caso de procesos automatizados, como las canalizaciones de CI/CD, puede automatizar la asignación de un recurso de LUIS a una aplicación de LUIS mediante los pasos siguientes:

1. Obtenga un [token de Azure Resource Manager](https://resources.azure.com/api/token?plaintext=true) que es una cadena alfanumérica de caracteres. Este token expira, por lo que debe usarlo de inmediato. También puede usar el siguiente comando de la CLI de Azure.

    ```azurecli
    az account get-access-token --resource=https://management.core.windows.net/ --query accessToken --output tsv
    ```
    
1. Use el token para solicitar los recursos de entorno de ejecución de LUIS en distintas suscripciones. Utilice la API para [obtener la cuenta de Azure de LUIS](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5be313cec181ae720aa2b26c) a la que tiene acceso su cuenta de usuario.

    Esta API POST requiere los siguientes valores:

    |Encabezado|Value|
    |--|--|
    |`Authorization`|El valor de `Authorization` es `Bearer {token}`. El valor del token debe ir precedido de la palabra `Bearer` y un espacio.|
    |`Ocp-Apim-Subscription-Key`|Su clave de creación.|

    La API devuelve una matriz de objetos JSON que representan las suscripciones a LUIS. Los valores devueltos incluyen el identificador de suscripción, el grupo de recursos y el nombre de recurso, devuelto como `AccountName`. Busque en la matriz el elemento que corresponde al recurso de LUIS que desea asignar a la aplicación de LUIS.

1. Asigne el token al recurso de LUIS mediante la API [Assign a LUIS Azure accounts to an application](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5be32228e8473de116325515).

    Esta API POST requiere los siguientes valores:

    |Tipo|Configuración|Value|
    |--|--|--|
    |Encabezado|`Authorization`|El valor de `Authorization` es `Bearer {token}`. El valor del token debe ir precedido de la palabra `Bearer` y un espacio.|
    |Encabezado|`Ocp-Apim-Subscription-Key`|Su clave de creación.|
    |Encabezado|`Content-type`|`application/json`|
    |QueryString|`appid`|El id. de la aplicación LUIS.
    |Body||{`AzureSubscriptionId`: Id. de suscripción,<br>`ResourceGroup`: nombre del grupo de recursos que tiene el recurso de predicción,<br>`AccountName`: nombre del recurso de predicción}|

    Cuando esta API finaliza correctamente, devuelve `201 - created status`.

---

## <a name="unassign-a-resource"></a>Anulación de la asignación de un recurso

Al anular la asignación de un recurso, no se elimina de Azure; solo se desvincula de LUIS.

# <a name="luis-portal"></a>[Portal de LUIS](#tab/portal)

## <a name="unassign-resources-using-luis-portal"></a>Cancelación de asignación de recursos mediante el portal de LUIS

1. Inicie sesión en el [portal de LUIS](https://www.luis.ai) y seleccione una aplicación de la lista **Mis aplicaciones**.
1. Vaya a **Administrar** > **Recursos de Azure**.
1. Seleccione el botón **Cancelar la asignación del recurso** para el recurso.

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

## <a name="unassign-prediction-resource-programmatically"></a>Cancelación de asignación de recursos de predicción mediante programación

1. Obtenga un [token de Azure Resource Manager](https://resources.azure.com/api/token?plaintext=true) que es una cadena alfanumérica de caracteres. Este token expira, por lo que debe usarlo de inmediato. También puede usar el siguiente comando de la CLI de Azure.

    ```azurecli
    az account get-access-token --resource=https://management.core.windows.net/ --query accessToken --output tsv
    ```
 
1. Use el token para solicitar los recursos de entorno de ejecución de LUIS en distintas suscripciones. Use la [API Get LUIS Azure accounts](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5be313cec181ae720aa2b26c), a la que su cuenta de usuario tiene acceso.

    Esta API POST requiere los siguientes valores:

    |Encabezado|Value|
    |--|--|
    |`Authorization`|El valor de `Authorization` es `Bearer {token}`. El valor del token debe ir precedido de la palabra `Bearer` y un espacio.|
    |`Ocp-Apim-Subscription-Key`|Su clave de creación.|

    La API devuelve una matriz de objetos JSON que representan las suscripciones a LUIS. Los valores devueltos incluyen el identificador de suscripción, el grupo de recursos y el nombre de recurso, devuelto como `AccountName`. Busque en la matriz el elemento que corresponde al recurso de LUIS que desea asignar a la aplicación de LUIS.

1. Asigne el token al recurso de LUIS mediante [Unassign a LUIS Azure account from an application](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5be32554f8591db3a86232e1/console) API.

    DELETE API requiere los siguientes valores:

    |Tipo|Configuración|Value|
    |--|--|--|
    |Encabezado|`Authorization`|El valor de `Authorization` es `Bearer {token}`. El valor del token debe ir precedido de la palabra `Bearer` y un espacio.|
    |Encabezado|`Ocp-Apim-Subscription-Key`|Su clave de creación.|
    |Encabezado|`Content-type`|`application/json`|
    |QueryString|`appid`|El id. de la aplicación LUIS.
    |Body||{`AzureSubscriptionId`: Id. de suscripción,<br>`ResourceGroup`: nombre del grupo de recursos que tiene el recurso de predicción,<br>`AccountName`: nombre del recurso de predicción}|

    Cuando esta API finaliza correctamente, devuelve `200 - OK status`.

---

## <a name="resource-ownership"></a>Propiedad de recurso

Los recursos de LUIS, como todos los recursos de Azure, pertenecen a la suscripción que los contiene.

Para cambiar la propiedad de un recurso, puede realizar una de estas acciones:
* Transferir la [propiedad](../../cost-management-billing/manage/billing-subscription-transfer.md) de la suscripción.
* Exportar la aplicación de LUIS como un archivo y, a continuación, importar la aplicación en otra suscripción. La exportación está disponible en la página **Mis aplicaciones** del portal de LUIS.

## <a name="resource-limits"></a>Límites de recursos

### <a name="authoring-key-creation-limits"></a>Limites al establecimiento de claves de creación

Puede tener hasta 10 claves de creación por región y suscripción. Las regiones de publicación son diferentes de las regiones de creación. Asegúrese de que crea la aplicación en la región de creación que corresponde a la región de publicación en la que quiere ubicar su aplicación cliente. Para obtener información sobre cómo se asignan las regiones de creación a las regiones de publicación, consulte [Regiones de creación y publicación](luis-reference-regions.md). 

Para obtener más información sobre los límites de la clave, consulte [Límites de la clave](luis-limits.md#key-limits).

### <a name="errors-for-key-usage-limits"></a>Errores de los límites de uso de claves

El límite de uso depende del plan de tarifa.

Si supera su cuota de transacciones por segundo (TPS), recibirá un error HTTP 429. Si supera su cuota de transacciones por mes (TPM), recibirá un error HTTP 403.

## <a name="change-the-pricing-tier"></a>Cambio del plan de tarifa

1.  En [Azure Portal](https://portal.azure.com), vaya a **Todos los recursos** y seleccione el recurso.

    :::image type="content" source="./media/luis-usage-tiers/find.png" alt-text="Captura de pantalla que muestra una suscripción a LUIS en Azure Portal." lightbox="./media/luis-usage-tiers/find.png":::

1.  En el menú de la izquierda, seleccione **Plan de tarifa** para ver los planes de tarifa disponibles.
1.  Elija el plan de tarifa que desee y haga clic en **Seleccionar** para guardar el cambio. Una vez completado el cambio de precios, aparecerá una notificación en la parte superior derecha con la actualización del plan de tarifa.

## <a name="view-azure-resource-metrics"></a>Visualización de las métricas de recursos de Azure

## <a name="view-a-summary-of-azure-resource-usage"></a>Visualización de un resumen del uso de recursos de Azure
En Azure Portal puede ver la información de uso de LUIS. En la página **Información general** se muestra un resumen, incluidas las llamadas y los errores recientes. Si realiza una solicitud de punto de conexión de LUIS, espere hasta cinco minutos para que aparezca el cambio.

:::image type="content" source="./media/luis-usage-tiers/overview.png" alt-text="Captura de pantalla que muestra la página Información general." lightbox="./media/luis-usage-tiers/overview.png":::

## <a name="customizing-azure-resource-usage-charts"></a>Personalización de los gráficos de uso de los recursos de Azure
La página **Métricas** proporciona una vista más detallada de los datos. Puede configurar los gráficos de métricas para un **período de tiempo** y una **métrica** específicos.

:::image type="content" source="./media/luis-usage-tiers/metrics.png" alt-text="Captura de pantalla que muestra la página Métricas." lightbox="./media/luis-usage-tiers/metrics.png":::

## <a name="total-transactions-threshold-alert"></a>Alerta de umbral de transacciones totales
Si desea saber cuándo se alcanza un umbral de transacciones determinado (por ejemplo 10 000 transacciones), puede crear una alerta:

1. En el menú de la izquierda, seleccione **Alertas**.
2. En el menú superior, seleccione **Nueva regla de alertas**.

    :::image type="content" source="./media/luis-usage-tiers/alerts.png" alt-text="Captura de pantalla que muestra la página de reglas de alerta." lightbox="./media/luis-usage-tiers/alerts.png":::

3. Haga clic en **Agregar condición**.

    :::image type="content" source="./media/luis-usage-tiers/alerts-2.png" alt-text="Captura de pantalla que muestra la página Agregar condición para las reglas de alerta." lightbox="./media/luis-usage-tiers/alerts-2.png":::

4. Seleccione **Llamadas totales**.

    :::image type="content" source="./media/luis-usage-tiers/alerts-3.png" alt-text="Captura de pantalla que muestra la página Llamadas totales para las alertas." lightbox="./media/luis-usage-tiers/alerts-3.png":::

5. Desplácese hacia abajo hasta la sección **Lógica de alerta**, establezca los atributos como desee y haga clic en **Listo**.

    :::image type="content" source="./media/luis-usage-tiers/alerts-4.png" alt-text="Captura de pantalla que muestra la página Lógica de alerta." lightbox="./media/luis-usage-tiers/alerts-4.png":::

6. Para enviar notificaciones o invocar acciones cuando se desencadene la regla de alerta, vaya a la sección **Acciones** y agregue el grupo de acciones.

    :::image type="content" source="./media/luis-usage-tiers/alerts-5.png" alt-text="Captura de pantalla que muestra la página Acciones para las alertas." lightbox="./media/luis-usage-tiers/alerts-5.png":::

### <a name="reset-an-authoring-key"></a>Restablecimiento de las claves de creación

En el caso de aplicaciones [migradas a recursos de creación](luis-migration-authoring.md): si la clave de creación se ve comprometida, restablézcala en Azure Portal, en la página **Claves** de ese recurso de creación.

En el caso de las aplicaciones que no se han migrado: la clave se restablece en todas las aplicaciones en el portal de LUIS. Si crea sus aplicaciones a través de las API de creación, debe cambiar el valor de `Ocp-Apim-Subscription-Key` por la clave nueva.

### <a name="regenerate-an-azure-key"></a>Regeneración de una clave de Azure

Puede volver a generar una clave de Azure desde la página **Claves** de Azure Portal.

<a name="securing-the-endpoint"></a>

## <a name="app-ownership-access-and-security"></a>Propiedad, acceso y seguridad de la aplicación

Una aplicación se define mediante sus recursos de Azure, que se determinan en función de la suscripción del propietario.

Puede mover la aplicación de LUIS. Use los siguientes recursos como ayuda para moverla mediante Azure Portal o la CLI de Azure:

* [Traslado de las aplicaciones entre recursos de creación de LUIS](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-move-app-to-another-luis-authoring-azure-resource)
* [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../../azure-resource-manager/management/move-resource-group-and-subscription.md)
* [Traslado de los recursos dentro de la misma suscripción o entre suscripciones](../../azure-resource-manager/management/move-limitations/app-service-move-limitations.md)


## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre [cómo usar versiones](luis-how-to-manage-versions.md) para controlar el ciclo de vida de la aplicación.


