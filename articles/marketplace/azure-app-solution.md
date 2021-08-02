---
title: Configuración de un plan de plantilla de solución
description: Configure un plan de plantilla de solución para su oferta de aplicación de Azure en el Centro de partners (Azure Marketplace).
author: aarathin
ms.author: aarathin
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 06/01/2021
ms.openlocfilehash: 22e5bc0aa411fd4561094fc41d5d3e55a5dcdeb8
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111542823"
---
# <a name="configure-a-solution-template-plan"></a>Configuración de un plan de plantilla de solución

Este artículo solo se aplica a los planes de plantilla de solución de una oferta de Aplicación de Azure. Si va a configurar un plan de aplicación administrada, vaya a [Configuración de un plan de aplicación administrada](azure-app-managed.md).

## <a name="re-use-technical-configuration-optional"></a>Reutilización de la configuración técnica (opcional)

Si ha creado más de un plan del mismo tipo en esta oferta y la configuración técnica es idéntica entre ellos, puede volver a usar la configuración técnica de otro plan. Esta configuración no se puede modificar una vez publicado este plan.

Para reutilizar una configuración técnica:

1. Seleccione la casilla **This plan reuses the technical configuration from another plan of the same type** (Este plan reutiliza la configuración técnica de otro plan del mismo tipo).
1. En la lista que aparece, seleccione el plan base que desee.

> [!NOTE]
> Si un plan está reutilizando o ha reutilizado la configuración técnica de otro plan del mismo tipo, vaya al plan base para ver el historial de paquetes publicados anteriormente.

## <a name="choose-who-can-see-your-plan"></a>Selección de quién puede ver el plan

Tiene la opción de configurar cada plan para que lo pueda ver todo el mundo o solo un público concreto. Puede conceder acceso a un público privado con los identificadores de la suscripción de Azure con la opción de incluir una descripción de cada identificador de suscripción que asigne. Puede agregar hasta 10 identificadores de suscripción manualmente o hasta 10 000 mediante un archivo .csv. Los identificadores de suscripción de Azure se representan como GUID y las letras deben estar en minúsculas.

> [!NOTE]
> Si publica un plan privado, puede cambiar su visibilidad a público más adelante. Sin embargo, una vez que se publica un plan público, no se puede cambiar su visibilidad a privado.

En la pestaña **Disponibilidad**, en **Plan visibility** (Visibilidad del plan), haga lo siguiente:

- Para que el plan sea público, seleccione el botón de opción (también conocido como _botón de radio_) **Público**.
- Para que el plan sea privado, seleccione el botón de opción **Privado** y, luego, agregue los identificadores de la suscripción de Azure manualmente o con un archivo .csv.

    > [!NOTE]
    > Una audiencia privada o restringida es diferente del público preliminar que definió en la pestaña **Versión preliminar**. Una audiencia preliminar puede acceder a la oferta antes de que se publique en el marketplace. En tanto que la designación de una audiencia privada solo se aplica a un plan específico, el público preliminar puede ver todos los planes (privados o no) con fines de validación.

### <a name="manually-add-azure-subscription-ids-for-a-private-plan"></a>Adición manual de los identificadores de la suscripción de Azure a un plan privado

1. En **Plan visibility** (Visibilidad del plan), seleccione el botón de opción **Privado**.
1. En el cuadro **Identificador de la suscripción de Azure** que aparece, escriba el identificador de la suscripción de Azure del público al que quiere conceder acceso a este plan privado. Se requiere un identificador de suscripción como mínimo.
1. (Opcional) Escriba una descripción de este público en el cuadro **Descripción**.
1. Para agregar otro identificador de suscripción, seleccione el vínculo **Add ID (Max 10)** (Agregar identificador [10 como máximo]) y repita los pasos 2 y 3.

## <a name="use-a-csv-file-to-add-azure-subscription-ids-for-a-private-plan"></a>Uso de un archivo .csv para agregar los identificadores de la suscripción de Azure a un plan privado

1. En **Plan visibility** (Visibilidad del plan), seleccione el botón de opción **Privado**.
1. Seleccione el vínculo **Exportar público (CSV)** .
1. Abra el archivo .csv y, en la columna **Id.** , agregue los identificadores de la suscripción de Azure a los que quiere conceder acceso a la oferta privada.
1. Opcionalmente, escriba una descripción para cada público en la columna **Descripción**.
1. Agregue "SubscriptionId" en la columna **Tipo** para cada fila con un identificador de suscripción.
1. Guarde el archivo .CSV.
1. En la pestaña **Disponibilidad**, en **Plan visibility** (Visibilidad del plan), seleccione el vínculo **Import Audience (csv)** (Importar audiencia [csv]).
1. En el cuadro de diálogo que aparece, seleccione **Sí**.
1. Seleccione el archivo .CSV y, después, elija **Abrir**. Aparece un mensaje que indica que el archivo .csv se importó correctamente.

### <a name="hide-your-plan"></a>Ocultar el plan

Si su plantilla de solución está diseñada para implementarse solo indirectamente cuando se hace referencia a ella mediante otra plantilla de solución o aplicación administrada, active la casilla **Hide plan** (Ocultar plan) para publicar la plantilla de solución, pero ocultarla para los clientes que la busquen directamente.

Seleccione **Guardar borrador** antes de pasar a la siguiente sección: Defina la configuración técnica.

## <a name="define-the-technical-configuration"></a>Definición de la configuración técnica

En la pestaña **Technical configuration** (Configuración técnica), cargará el paquete de implementación que permite a los clientes implementar el plan y proporcionar un número de versión para el paquete.

### <a name="assign-a-version-number-for-the-package"></a>Asignación de un número de versión al paquete

En el cuadro **Versión**, proporcione la versión actual de la configuración técnica. Incremente esta versión cada vez que publique un cambio en esta página. El número de versión debe tener el formato: número entero.número entero.número entero. Por ejemplo, `1.0.2`.

### <a name="upload-a-package-file"></a>Carga de un archivo de paquete

En **Archivo de paquete (.zip)** , arrastre el archivo de paquete al cuadro gris o seleccione el vínculo para **buscar los archivos**.

> [!NOTE]
> Si tiene un problema al cargar archivos, asegúrese de que la red local no bloquee el servicio `https://upload.xboxlive.com` que usa el Centro de partners.

### <a name="previously-published-packages"></a>Paquetes publicados anteriormente

Después de publicar la oferta, aparece la subpestaña **Previously published packages** (Paquetes anteriormente publicados) en la página **Technical configuration** (Configuración técnica). En esta pestaña se enumeran todas las versiones publicadas anteriormente de la configuración técnica.

## <a name="view-your-plans"></a>Visualización de los planes

- Seleccione **Guardar borrador** y, luego, en la parte superior izquierda de la página, seleccione **Plan overview** (Información general del plan) para volver a la página **Plan overview** (Información general del plan).

Después de crear uno o varios planes, verá el nombre, el identificador y el tipo del plan, la disponibilidad (pública o privada), el estado de publicación actual y las acciones disponibles en la pestaña **Plan overview** (Información general del plan).

Las acciones que están disponibles en la columna **Acción** de la pestaña **Información general del plan** varían en función del estado del plan y pueden incluir lo siguiente:

- Si el estado del plan es **Borrador**, el vínculo de la columna **Acción** indicará **Eliminar borrador**.
- Si el estado del plan es **Activo**, el vínculo de la columna **Acción** será **Stop sell plan** (Dejar de vender el plan) o **Sync private audience** (Sincronizar audiencia privada). El vínculo **Sync private audience** (Sincronizar público privado) publicará solo los cambios para el público privado, sin publicar ninguna otra actualización que haya podido realizar en la oferta.
- Para crear otro plan para esta oferta, en la parte superior de la pestaña **Información general del plan**, seleccione **+ Crear nuevo plan**. Después, repita los pasos descritos en [Creación de planes para la oferta de Aplicación de Azure](azure-app-plans.md). En caso contrario, si ha terminado de crear planes, vaya a la sección siguiente: Pasos siguientes.

## <a name="next-steps"></a>Pasos siguientes

- [Probar y publicar esta oferta](azure-app-test-publish.md)
- [Vender esta oferta](azure-app-marketing.md) mediante los programas **Venta conjunta con Microsoft** o **Reventa a través de CSP**
