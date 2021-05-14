---
title: Advanced Threat Protection para Azure Cosmos DB
description: Obtenga información acerca de cómo Azure Cosmos DB proporciona cifrado de datos en reposo y cómo se implementa.
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/13/2019
ms.custom: seodec18
ms.author: memildin
author: memildin
manager: rkarlin
ms.openlocfilehash: 23d95e02c74918de711b18c7580c3774b19c6227
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108229457"
---
# <a name="advanced-threat-protection-for-azure-cosmos-db-preview"></a>Advanced Threat Protection para Azure Cosmos DB (versión preliminar)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Advanced Threat Protection para Azure Cosmos DB proporciona un nivel adicional de inteligencia de seguridad que detecta intentos poco habituales y potencialmente peligrosos de acceder a las cuentas de Azure Cosmos DB o vulnerarlas. Esta capa de protección le permite afrontar las amenazas, incluso sin necesidad de ser un experto en seguridad, e integrarlas con sistemas centrales de supervisión de seguridad.

Las alertas de seguridad se desencadenan cuando se producen anomalías en una actividad. Estas alertas de seguridad se integran en [Azure Security Center](https://azure.microsoft.com/services/security-center/) y también se envían por correo electrónico a los administradores de las suscripciones, con detalles de la actividad sospechosa y recomendaciones sobre cómo investigar y solucionar las amenazas.

> [!NOTE]
>
> * Advanced Threat Protection para Azure Cosmos DB está actualmente disponible solo para SQL API.
> * Advanced Threat Protection para Azure Cosmos DB no está disponible actualmente en Azure Government y en regiones de nube soberana.

Para una experiencia de investigación completa de las alertas de seguridad, se recomienda habilitar el [registro de diagnóstico en Azure Cosmos DB](./monitor-cosmos-db.md), que registra las operaciones en la base de datos, incluidas las operaciones CRUD en todos los documentos, contenedores y bases de datos.

## <a name="threat-types"></a>Tipos de amenaza

Advanced Threat Protection para Azure Cosmos DB detecta actividades anómalas que indican intentos inusuales y potencialmente maliciosos de acceso o ataque a las bases de datos. Actualmente, puede desencadenar las siguientes alertas:

- **Acceso desde una ubicación inusual**: Esta alerta se desencadena cuando se produce un cambio en el patrón de acceso a una cuenta de Azure Cosmos, donde alguien se ha conectado al punto de conexión de Azure Cosmos DB desde una ubicación geográfica inusual. En algunos casos, la alerta detecta una acción legítima (una nueva aplicación o una operación de mantenimiento de un desarrollador). En otros casos, la alerta detecta una acción malintencionada (por ejemplo, un antiguo empleado, un atacante externo, etc.).

- **Extracción de datos inusual**: esta alerta se desencadena cuando un cliente extrae una cantidad de datos inusual de una cuenta de Azure Cosmos DB. Esto puede ser el síntoma de la filtración de datos realizada para transferir todos los datos almacenados en la cuenta a un almacén de datos externos.



## <a name="configure-advanced-threat-protection"></a>Configuración de Advanced Threat Protection

Puede configurar Advanced Threat Protection de varias maneras, como se describe en las secciones siguientes.

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Inicie Azure Portal en [https://portal.azure.com](https://portal.azure.com/).

2. En la cuenta de Azure Cosmos DB, en el menú **Configuración**, seleccione **Seguridad avanzada**.

    :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db-atp.png" alt-text="Configuración de ATP":::

3. En la hoja de configuración de **Seguridad avanzada**:

    * Haga clic en la opción **Advanced Threat Protection** para configurarla en **ACTIVADA**.
    * Haga clic en **Guardar** para guardar la directiva de Protección contra amenazas avanzada nueva o actualizada.   

# <a name="rest-api"></a>[REST API](#tab/rest-api)

Use comandos de la API de REST para crear, actualizar u obtener la configuración de Advanced Threat Protection para una cuenta de Azure Cosmos DB específica.

* [Creación de Advanced Threat Protection](/rest/api/securitycenter/advancedthreatprotection/create)
* [Obtención de Advanced Threat Protection](/rest/api/securitycenter/advancedthreatprotection/get)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Use los siguientes cmdlets de PowerShell:

* [Habilitación de Advanced Threat Protection](/powershell/module/az.security/enable-azsecurityadvancedthreatprotection)
* [Obtención de Advanced Threat Protection](/powershell/module/az.security/get-azsecurityadvancedthreatprotection)
* [Deshabilitación de Advanced Threat Protection](/powershell/module/az.security/disable-azsecurityadvancedthreatprotection)

# <a name="arm-template"></a>[Plantilla ARM](#tab/arm-template)

Use una plantilla de Azure Resource Manager (ARM) para configurar Cosmos DB con Advanced Threat Protection habilitada.
Para más información, consulte [Creación de una cuenta de Cosmos DB con Advanced Threat Protection](https://azure.microsoft.com/resources/templates/201-cosmosdb-advanced-threat-protection-create-account/).

# <a name="azure-policy"></a>[Azure Policy](#tab/azure-policy)

Use una instancia de Azure Policy para habilitar Advanced Threat Protection para Cosmos DB.

1. Inicie la página **Policy - Definitions** (Directiva - Definiciones) de Azure y busque la directiva **Deploy Advanced Threat Protection for Cosmos DB** (Implementar Advanced Threat Protection para Cosmos DB).

    :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db.png" alt-text="Búsqueda de directiva"::: 

1. Haga clic en la directiva **Deploy Advanced Threat Protection for Cosmos DB** (Implementar Advanced Threat Protection para Cosmos DB) y, a continuación, haga clic en **Assign** (Asignar).

    :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db-atp-policy.png" alt-text="Selección de suscripción o grupo":::


1. En el campo **Scope** (Ámbito), haga clic en los tres puntos, seleccione una suscripción de Azure o un grupo de recursos y, después, haga clic en **Select** (Seleccionar).

    :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db-atp-details.png" alt-text="Página de definiciones de directiva":::


1. Escriba los demás parámetros y haga clic en **Assign** (Asignar).




## <a name="manage-atp-security-alerts"></a>Administración de alertas de seguridad de ATP

Cuando se producen anomalías en la actividad de Azure Cosmos DB, se desencadena una alerta de seguridad con información sobre el evento de seguridad sospechoso. 

 Desde Azure Security Center, se pueden revisar y administrar las [alertas de seguridad](../security-center/security-center-alerts-overview.md) actuales.  Haga clic en una alerta específica en [Security Center](https://ms.portal.azure.com/#blade/Microsoft_Azure_Security/SecurityMenuBlade/0) para ver las posibles causas y las medidas recomendadas para investigar y mitigar la potencial amenaza. En la imagen siguiente se muestra un ejemplo de los detalles de la alerta proporcionados en Security Center.

 :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db-alert-details.png" alt-text="Detalles de la amenaza":::

También se envía una notificación por correo electrónico con los detalles de la alerta y las medidas recomendadas. En la imagen siguiente se muestra un ejemplo de un correo electrónico de alerta.

 :::image type="content" source="./media/cosmos-db-advanced-threat-protection/cosmos-db-alert.png" alt-text="Detalles de alerta":::

## <a name="cosmos-db-atp-alerts"></a>Alertas de ATP de Cosmos DB

 Para ver una lista de las alertas generadas al supervisar cuentas de Azure Cosmos DB, consulte la sección [Alertas de Cosmos DB](../security-center/alerts-reference.md#alerts-azurecosmos) en la documentación de Azure Security Center.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre el [Registro de diagnóstico de Azure Cosmos DB](cosmosdb-monitor-resource-logs.md).
* Más información acerca de [Azure Security Center](../security-center/security-center-introduction.md)