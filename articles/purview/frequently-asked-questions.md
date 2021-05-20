---
title: Preguntas más frecuentes
description: En este artículo se responden las preguntas más frecuentes sobre Azure Purview.
author: SunetraVirdi
ms.author: suvirdi
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 05/08/2021
ms.openlocfilehash: eb835fad0f051098b1689ed6d165dc712a73f908
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109655271"
---
# <a name="frequently-asked-questions-faq-about-azure-purview"></a>Preguntas más frecuentes sobre Azure Purview

## <a name="overview"></a>Información general

Muchas organizaciones carecen de una comprensión holística de sus datos. Es difícil comprender qué datos existen, dónde se encuentran, y cómo buscar y acceder a datos pertinentes. Los datos carecen de contexto, como el linaje, la clasificación y metadatos completos, lo que dificulta a los usuarios de negocios la búsqueda de los datos correctos y el uso adecuado de estos. Como resultado, solo se utiliza una pequeña parte de los datos recopilados para la toma de decisiones empresariales informadas. Por último, la identificación de los problemas de seguridad de los datos y la protección de la información confidencial es incoherente. Requiere tiempo y esfuerzo constantemente, sobre todo al mantener la agilidad de los datos.

Azure Purview es una solución de gobernanza de datos que ayuda a los clientes a obtener un conocimiento minucioso de todos sus datos y a mantener el control sobre su uso. Azure Purview permite a las organizaciones descubrir y mantener los datos. Obtienen información sobre su patrimonio de datos y controlan el acceso a los datos de forma centralizada.

## <a name="purpose-of-this-faq"></a>Propósito de estas P+F

Estas P+F responden a cuestiones comunes que los clientes y equipos de campo suelen preguntar. Están diseñadas para aclarar preguntas sobre Azure Purview y soluciones asociadas, como Azure Data Catalog (ADC) Gen 2 (en desuso) y Azure Information Protection.

### <a name="what-are-the-source-types-available-for-metadata-scanning-and-classification"></a>¿Cuáles son los tipos de origen disponibles para el examen y la clasificación de metadatos?

|Azure|Ajeno a Azure|
|---------|---------|
|Azure Blob Storage|Power BI|
|Azure Synapse Analytics (SQL DW)|SQL Server |
|Azure Cosmos DB|Teradata (disponible a finales de 2020)|
|Instancia administrada de Azure SQL|SAP ECC (disponible a finales de 2020)|
|Explorador de datos de Azure|SAP S/4 HANA (disponible a finales de 2020)|
|Azure Data Lake Storage Gen1|Hive Metastore (disponible a finales de 2020)|
|Azure Data Lake Storage Gen2|Amazon S3|
|Azure Files|--|
|Azure SQL Database|--|

### <a name="what-data-systemsprocessors-can-we-connect-and-get-lineage"></a>¿Qué sistemas/procesadores de datos se pueden conectar y sirven para obtener linaje?

|Sistema/procesador de datos 
|---------
|Azure Data Factory: actividad de copia, actividad de flujo de datos 
|Linaje personalizado   
|Azure Data Share   
|Power BI    |
|SQL Server Integration Services  

### <a name="how-are-adc-gen-2-azure-information-protection-and-azure-purview-related"></a>¿Cómo están relacionados ADC Gen 2, Azure Information Protection y Azure Purview?

Azure Purview comenzó originalmente como ADC Gen 2, pero su ámbito se ha ampliado. Ahora adopta las funcionalidades de catálogo avanzadas de ADC Gen 2 en combinación con las funcionalidades de clasificación de datos, de etiquetado y de aplicación de directivas de cumplimiento de Azure Information Protection. Actualmente se adapta mejor a la definición más amplia del sector de la gobernanza de datos.

### <a name="what-happens-to-customers-using-adc-gen-1"></a>¿Qué ocurre con los clientes que usan ADC Gen 1?

Azure Purview es el corazón de la innovación de productos del espacio de soluciones de catálogo para Microsoft. ADC Gen 1 se seguirá admitiendo.

### <a name="can-customers-have-multiple-azure-purview-accounts-in-the-same-subscription"></a>¿Pueden los clientes tener varias cuentas de Azure Purview en la misma suscripción?

Sí, se admiten varias cuentas de Azure Purview por suscripción y por inquilino.

### <a name="can-i-run-adc-gen-1-and-azure-purview-in-parallel"></a>¿Puedo ejecutar ADC Gen 1 y Azure Purview en paralelo?

Sí. Son servicios independientes.

### <a name="how-do-i-migrate-existing-adc-gen-1-data-assets-to-azure-purview"></a>¿Cómo se migran recursos de datos de ADC Gen 1 existentes a Azure Purview?

Use las API de Azure Purview para extraer de ADC Gen 1 e ingerir en Azure Purview. En el caso de los glosarios, se admiten herramientas en bloque basadas en CSV.

### <a name="how-do-i-encrypt-sensitive-data-for-sql-tables-using-azure-purview"></a>¿Cómo se cifra la información confidencial para las tablas SQL con Azure Purview?

El cifrado de datos se realiza en el nivel de origen de datos. Azure Purview solo almacena los metadatos. No obtiene una vista previa de los datos.

### <a name="will-all-the-capabilities-of-adc-gen-2-exist-in-azure-purview"></a>¿Existen todas las funcionalidades de ADC Gen 2 en Azure Purview?

Sí.

<!--## Is the data lineage feature available in Azure Purview?

Yes, but it's limited to the Azure Data Factory connector.

<!-- ## How can I scan SQL Server on-premises? 

Use the self-host integration runtime capability. !-->

<!--### What is the difference between classification in Azure SQL Database and classification in Azure Purview?

|Azure SQL DB classification  |Azure Purview classification  |
|---------|---------|
|Classification is based on SQL metadata from system catalogs. |Classification is based on Azure Purview's sampling technique by using the system-defined or custom-defined regex pattern.|
|Custom classification is supported.     |Custom classification is supported.         |
|Doesn't use Microsoft 365 system classifiers out of the box.    | Uses Microsoft 365 system classifiers out of the box.        |
-->

### <a name="whats-the-difference-between-a-glossary-and-classification"></a>¿Cuál es la diferencia entre un glosario y una clasificación?

Un glosario utiliza una convención de nomenclatura que siguen los usuarios no técnicos y empresariales de los datos, que también se conocen como consumidores de datos. Estos tipos de personas son analistas empresariales o científicos de datos que usan Azure Purview para buscar determinados tipos de datos en función del uso que hace de ellos la empresa. Por ejemplo, los analistas de la cadena de suministro pueden necesitar buscar los términos como *tipos de SKU* y *detalles de envío*. Estos términos se encuentran en el glosario y sirven para buscar datos relevantes.
La clasificación es una etiqueta que se aplica a un recurso de datos en el nivel de tabla, columna o archivo, e identifica qué datos existen en el recurso. La clasificación se puede aplicar de forma automática o manual, según el tipo de datos que se busquen. Normalmente se usan etiquetas de clasificación para identificar si un recurso contiene datos confidenciales y el tipo de datos confidenciales que pueden ser.

### <a name="does-azure-purview-scan-and-classify-emails-pdfs-etc-in-my-sharepoint-and-onedrive"></a>Azure Purview ¿examina y clasifica correos electrónicos, archivos PDF, etc. en mi SharePoint y OneDrive?

El examen de las bibliotecas y los sitios de SharePoint locales se proporciona mediante Azure Information Protection. El examen está disponible para su uso mediante la suscripción a Microsoft 365 de un cliente con las SKU siguientes: AIP P1, EMS E3 y M365 E3. Si tiene alguna de estas SKU, debe tener los derechos adecuados para empezar a usar el examen de Azure Information Protection.

<!--### What is the difference between classifications and sensitivity labels in Azure Purview?

Azure Purview's data governance solution is based on the Apache Atlas framework. As defined by Atlas, classification is a way to identify the contents of an asset (table or file) or an entity (table column or structured file). This classification becomes a metadata property that allows Azure Purview to understand the data within each asset and govern and protect them.

Sensitivity labels are a Microsoft 365 concept that resembles classification at the asset level. You create a label with a collection of classifications applied at the asset or entity level.

Atlas-centric customers will see no real distinction between classifications and labels. To these customers, everything is a classification and labels aren't needed.

Security-focused customers will see a distinction between classification and labeling, but only because in Microsoft 365 the classifications aren't exposed directly to the user; only labels are visible. So, similar to Atlas, Office 365 security customers don't need to deal with both entities.
-->

### <a name="what-is-the-compute-used-for-the-scan"></a>¿Cuál es el proceso que se usa para el examen?
Existe una infraestructura de examen administrada por Microsoft. Para la mayoría de los recursos de Azure/AWS admitidos, no es necesario implementar una infraestructura de examen.

### <a name="is-there-a-way-to-provision-azure-purview-via-azure-resource-manager-arm-template--cli--powershell"></a>¿Hay alguna manera de aprovisionar Azure Purview mediante la plantilla de Azure Resource Manager (ARM)/la CLI/PowerShell?

Sí, la plantilla de ARM está disponible.

<!--### Does Azure Purview support guest users in AAD?-->

### <a name="im-already-using-atlas-can-i-easily-move-to-azure-purview"></a>Ya estoy usando Atlas, ¿puedo cambiar fácilmente a Azure Purview?

Azure Purview es compatible con API de Atlas. Si va a migrar desde Atlas, se recomienda primero examinar los orígenes de datos con Azure Purview. Una vez que los recursos estén disponibles en su cuenta, puede usar API de Atlas similares para integrarlos, actualizarlos o agregar un linaje personalizado. Azure Purview modifica Search API para usar Azure Search, por lo que debería poder usar la búsqueda avanzada.

### <a name="can-i-create-multiple-catalogs-in-my-tenant"></a>¿Puedo crear varios catálogos en mi inquilino?

Sí, puede crear varias cuentas de Azure Purview por suscripción y por inquilino. Puede revisar la página de límites [Administración y aumento de las cuotas para los recursos con Azure Purview](how-to-manage-quotas.md).

En nuestros [procedimientos recomendados de implementación de Azure Purview](deployment-best-practices.md) se incluyen otras recomendaciones sobre cuándo se debe o no tener varias cuentas.

### <a name="can-i-register-multiple-tenants-within-a-single-azure-purview-account"></a>¿Puedo registrar varios inquilinos en una sola cuenta de Azure Purview?

No. Actualmente, para examinar el origen de datos de otro inquilino, debe crear una cuenta de Azure Purview independiente en ese inquilino.

### <a name="does-azure-purview-support-column-level-lineage"></a>¿Admite Azure Purview el linaje de nivel de columna?

Sí, Azure Purview admite el linaje de nivel de columna.

### <a name="does-azure-purview-support-soft-delete"></a>¿Admite Azure Purview la eliminación temporal?

Sí, Azure Purview admite la eliminación temporal para la perspectiva de administración del estado de la suscripción de Azure. Purview puede leer los estados de la suscripción (deshabilitada, con advertencias, etc.) y poner la cuenta en estado de eliminación temporal hasta que la cuenta se restaure o elimine. Todas las llamadas de API del plano de datos se bloquearán cuando la cuenta esté en estado de eliminación temporal y solo se permitirán las llamadas de API del plano de control GET y DELETE. Puede encontrar información adicional en la página [Estado de la suscripción de Azure](../cost-management-billing/manage/subscription-states.md).

### <a name="does-azure-purview-currently-support-data-loss-prevention-capabilities"></a>¿Azure Purview admite actualmente funcionalidades de prevención de pérdida de datos?

No, Azure Purview no proporciona funcionalidades de prevención de pérdida de datos en este momento. 

Consulte [Prevenir la pérdida de datos](https://docs.microsoft.com/microsoft-365/compliance/information-protection?view=o365-worldwide#prevent-data-loss) si está interesado en las características de prevención de pérdida de datos dentro de Microsoft 365.
