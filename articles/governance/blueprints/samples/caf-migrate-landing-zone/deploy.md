---
title: Implementación del ejemplo de plano técnico de la zona de aterrizaje de la migración de CAF
description: Pasos de implementación del ejemplo de plano técnico de la zona de aterrizaje de migración de CAF, incluidos los detalles de los parámetros del artefacto de plano técnico.
ms.date: 03/12/2021
ms.topic: sample
ms.openlocfilehash: 9c953c5ec7f6c8a9a9ae7a1ff784b5c0d0f8f877
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108757824"
---
# <a name="deploy-the-microsoft-cloud-adoption-framework-for-azure-migrate-landing-zone-blueprint-sample"></a>Implementación del ejemplo de plano técnico de la zona de aterrizaje de migración de Microsoft Cloud Adoption Framework para Azure

Para implementar el ejemplo de plano técnico de la zona de aterrizaje de migración de CAF de Azure Blueprints, debe realizar los pasos siguientes:

> [!div class="checklist1"]
> - Recomendado para implementar el ejemplo de plano técnico de [fundamentos de CAF](../caf-foundation/index.md).

> [!div class="checklist2"]
> - Crear un plano técnico a partir del ejemplo
> - Marcar la copia del ejemplo como **publicada**
> - Asignar la copia del plano técnico a una suscripción existente

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

## <a name="create-blueprint-from-sample"></a>Creación de un plano técnico a partir del ejemplo

En primer lugar, implemente el ejemplo de plano técnico mediante la creación de un plano técnico en su entorno tomando el ejemplo como punto de partida.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la página **Introducción** de la izquierda, seleccione el botón **Crear** en _Crear un plano técnico_.

1. Busque el ejemplo de plano técnico de la **zona de aterrizaje de migración de CAF** en _Otras muestras_ y seleccione **Usar esta muestra**.

1. Escriba los _Aspectos básicos_ del ejemplo de plano técnico:
   - **Blueprint name** (Nombre del plano técnico): proporcione un nombre para su copia del ejemplo de plano técnico de la zona de aterrizaje de migración de CAF.
   - **Ubicación de definición**: use los puntos suspensivos y seleccione el grupo de administración donde guardar la copia del ejemplo.

1. Seleccione la pestaña _Artefactos_ en la parte superior de la página **Siguiente: Artefactos** en la parte inferior de la página.

1. Revise la lista de artefactos que componen el ejemplo de plano técnico. Muchos de los artefactos tienen parámetros que se definirán más tarde. Seleccione **Guardar borrador** cuando haya terminado de revisar el ejemplo de plano técnico.

## <a name="publish-the-sample-copy"></a>Publicación de la copia del ejemplo

La copia del ejemplo de plano técnico ahora se ha creado en el entorno. Se crea en el modo **Borrador** y debe **publicarse** antes de que se pueda asignar e implementar. La copia del ejemplo de plano técnico se puede personalizar para adaptarla a su entorno y sus necesidades, pero esa modificación puede desviarla de la guía para la zona de aterrizaje de migración de CAF.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Publicar plano técnico** en la parte superior de la página. En la nueva página de la derecha, especifique una **versión** para la copia del ejemplo de plano técnico. Esta propiedad es útil si realiza una modificación posteriormente. En **Notas de cambios**, escriba "Primera versión publicada del ejemplo de plano técnico de la zona de aterrizaje de migración de CAF". A continuación, seleccione **Publicar** en la parte inferior de la página.

## <a name="assign-the-sample-copy"></a>Asignación de la copia de ejemplo

Una vez que la copia del ejemplo de plano técnico se haya **publicado** correctamente, se podrá asignar a una suscripción dentro del grupo de administración donde se guardó. En este paso se proporcionan los parámetros para hacer que cada implementación de la copia del ejemplo de plano técnico sea única.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Asignar plano técnico** en la parte superior de la página de definición del plano técnico.

1. Proporcione los valores de parámetro para la asignación de plano técnico:

   - Aspectos básicos
     - **Suscripciones**: seleccione una o varias de las suscripciones que están en el grupo de administración donde guardó la copia del ejemplo de plano técnico. Si selecciona más de una suscripción, se creará una asignación para cada una mediante los parámetros especificados.
     - **Nombre de asignación**: el nombre se rellena de antemano de forma automática en función del nombre del plano técnico.
       Cámbielo si fuera necesario o déjelo tal cual.
     - **Ubicación**: seleccione una región para la identidad administrada en la que se va a crear.
     - Azure Blueprint usa esta identidad administrada para implementar todos los artefactos del plano técnico asignado.
       Para más información, consulte [Identidades administradas para recursos de Azure](../../../../active-directory/managed-identities-azure-resources/overview.md).
     - **Versión de definición de Blueprint**: Elija una versión **publicada** de la copia del ejemplo de plano técnico.

   - Asignación de bloqueo

     Seleccione el valor de bloqueo de plano técnico para el entorno. Para más información, consulte [Bloqueo de recursos en planos técnicos](../../concepts/resource-locking.md).

   - Identidad administrada

     Elija la opción de identidad administrada predeterminada _asignada por el sistema_ o la opción de identidad _asignada por el usuario_.

   - Parámetros de plano técnico

     Muchos de los artefactos de la definición de plano técnico usan los parámetros definidos en esta sección para proporcionar coherencia.

     - **Organización**: escriba el nombre de la organización, como Contoso o Fabrikam. El nombre debe ser único.
     - **AzureRegion**: seleccione una región de Azure para la implementación.

   - Parámetros de artefacto

     Los parámetros definidos en esta sección se aplican al artefacto en el que se define. Estos parámetros son [parámetros dinámicos](../../concepts/parameters.md#dynamic-parameters), ya que se definen durante la asignación del plano técnico. Para obtener una lista completa de los parámetros de los artefactos y sus descripciones, consulte la [tabla de parámetros de los artefactos](#artifact-parameters-table).

1. Una vez que se hayan especificado todos los parámetros, seleccione **Asignar** en la parte inferior de la página. Se crea la asignación del plano técnico y comienza la implementación del artefacto. La implementación tarda aproximadamente cinco minutos. Para comprobar el estado de implementación, abra la asignación del plano técnico.

> [!WARNING]
> El servicio Azure Blueprints y los ejemplos de plano técnico incorporados son **gratuitos**. El precio de los recursos de Azure se [calcula por producto](https://azure.microsoft.com/pricing/). Use la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) para estimar el costo de la ejecución de los recursos implementados en este ejemplo de plano técnico.

## <a name="artifact-parameters-table"></a>Tabla de parámetros de los artefactos

En la tabla siguiente se proporciona una lista de los parámetros del artefacto de plano técnico:

|Nombre del artefacto|Tipo de artefacto|Nombre de parámetro|Descripción|
|-|-|-|-|
|Implementación de la zona de aterrizaje de red virtual|Plantilla de Resource Manager|IPAddress_Space|**Bloqueado**: proporcione el ejemplo de los dos primeros octetos, 10.0.|
|Implementación de Key Vault|Plantilla de Resource Manager|KV-AccessPolicy|**Bloqueado**: grupo o identificador de objeto de usuario al que conceder permisos en Key Vault.|
|Implementación de Log Analytics|Plantilla de Resource Manager|LogAnalytics_DataRetention|**Bloqueado**: número de días que se conservarán los datos en Log Analytics.|
|Implementación de Log Analytics|Plantilla de Resource Manager|LogAnalytics_Location|**Bloqueado**: región que se usa al establecer el área de trabajo.|
|Implementación de Azure Migrate|Plantilla de Resource Manager|Azure_Migrate_Location|**Bloqueado** : seleccione la región para implementar Azure Migrate.|

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha revisado los pasos para implementar el ejemplo de plano técnico de la zona de aterrizaje de migración de CAF, visite los siguientes artículos para obtener información sobre la arquitectura:

> [!div class="nextstepaction"]
> [Introducción al plano técnico de la zona de aterrizaje de migración de CAF](./index.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).
