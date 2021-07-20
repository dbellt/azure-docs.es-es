---
title: Habilitación de las protecciones de carga de trabajo integradas de Azure Security Center
description: Obtenga información sobre cómo habilitar Azure Defender para ampliar las protecciones de Azure Security Center a los recursos híbridos y de varias nubes.
author: memildin
ms.author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: quickstart
ms.date: 06/07/2021
ms.openlocfilehash: bc8449950134ba8c247c57df06b7495a48558def
ms.sourcegitcommit: f3b930eeacdaebe5a5f25471bc10014a36e52e5e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/16/2021
ms.locfileid: "112238593"
---
# <a name="quickstart-enable-azure-defender"></a>Inicio rápido: Habilitación de Azure Defender

Para obtener información sobre las ventajas de Azure Defender, consulte [Azure Security Center gratuito frente a Azure Defender habilitado](security-center-pricing.md).

## <a name="prerequisites"></a>Prerrequisitos

Para seguir los inicios rápidos y tutoriales de Security Center, debe habilitar Azure Defender. 

Puede proteger toda una suscripción de Azure con Azure Defender y todos los recursos de la suscripción heredarán las protecciones.

Hay disponible una versión de evaluación gratuita de 30 días. Para obtener información sobre los precios en la moneda de su elección y según su región, consulte [Precios de Security Center](https://azure.microsoft.com/pricing/details/security-center/).

## <a name="enable-azure-defender-from-the-azure-portal"></a>Habilitación de Azure Defender desde Azure Portal

Para habilitar todas las características de Security Center, incluidas las funcionalidades de protección contra amenazas, debe habilitar Azure Defender en la suscripción que contenga las cargas de trabajo aplicables. La habilitación en el nivel de área de trabajo no habilita el acceso a VM Just-In-Time, los controles de aplicación adaptables ni las detecciones de red para los recursos de Azure. Además, los únicos planes de Azure Defender disponibles en el nivel de área de trabajo son Azure Defender para servidores y Azure Defender para servidores SQL en máquinas.

- Puede habilitar **Azure Defender para cuentas de Storage** en el nivel de suscripción o de recurso.
- Puede habilitar **Azure Defender para SQL** en el nivel de suscripción o de recurso.
- Solo puede habilitar **Azure Defender para bases de datos relacionales de código abierto** en el nivel de recurso.

### <a name="to-enable-azure-defender-on-your-subscriptions-and-workspaces"></a>Para habilitar Azure Defender en las suscripciones y áreas de trabajo:

- Para habilitar Azure Defender en una suscripción:

    1. En el menú principal de Security Center, seleccione **Precios y configuración**.
    1. Seleccione la suscripción o el área de trabajo que desea proteger.
    1. Seleccione **Azure Defender está activado** para actualizar.
    1. Seleccione **Guardar**.

    > [!TIP]
    > Observará que cada plan de Azure Defender tiene un precio independiente y se puede activar o desactivar de forma independiente. Por ejemplo, puede que desee desactivar Azure Defender para App Service en suscripciones que no tengan un plan de Azure App Service asociado. 

    :::image type="content" source="./media/security-center-pricing/pricing-tier-page.png" alt-text="Página de precios de Security Center en el portal":::

- Para habilitar Azure Defender en varias suscripciones o áreas de trabajo:

    1. En la barra lateral de Security Center, seleccione **Introducción**.

        La pestaña **Actualizar** muestra las suscripciones y áreas de trabajo aptas para la incorporación.

        :::image type="content" source="./media/enable-azure-defender/get-started-upgrade-tab.png" alt-text="Pestaña Actualizar de la página de introducción."::: 

    1. En la lista **Select subscriptions and workspaces to enable Azure Defender on** (Seleccionar las suscripciones y áreas de trabajo en las que habilitar Azure Defender), seleccione las suscripciones y áreas de trabajo que desea actualizar y seleccione **Upgrade** (Actualizar) para habilitar Azure Defender.

       - Si selecciona suscripciones y áreas de trabajo que no sean válidas para la evaluación, el paso siguiente las actualizará y se iniciarán los cargos.
       - Si selecciona un área de trabajo apta para una evaluación gratuita, el siguiente paso comenzará una evaluación gratuita.

        :::image type="content" source="./media/enable-azure-defender/upgrade-selected-workspaces-and-subscriptions.png" alt-text="Actualizar todas las áreas de trabajo y suscripciones seleccionadas desde la página de introducción.":::


## <a name="disable-azure-defender"></a>Deshabilitación de Azure Defender

Si tiene que deshabilitar Azure Defender para una suscripción, el procedimiento es el mismo, pero tiene que **desactivar Azure Defender**:
 
1. En el menú de Security Center, seleccione **Precios y configuración**.
1. Seleccione la suscripción correspondiente.
1. Si su suscripción tiene habilitado Azure Defender, abra los **planes de Azure Defender** y seleccione **Azure Defender off** (Azure Defender desactivado).

    :::image type="content" source="./media/enable-azure-defender/disable-plans.png" alt-text="Habilitación o deshabilitación de Azure Defender.":::

1. Seleccione **Guardar**.

> [!NOTE]
> Después de deshabilitar Azure Defender, tanto si deshabilita un plan único como todos los planes a la vez, la recopilación de datos podrá continuar durante un breve período de tiempo. 

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha habilitado Azure Defender, habilite la recopilación automática de datos mediante los agentes y las extensiones necesarios descritos en [Configuración del aprovisionamiento automático de agentes y extensiones](security-center-enable-data-collection.md).