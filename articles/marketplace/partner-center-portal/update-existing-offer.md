---
title: 'Actualización de una oferta existente: Marketplace comercial de Microsoft'
description: Obtenga información sobre cómo actualizar una oferta o plan existente de Marketplace comercial, sincronizar audiencias privadas y quitar una oferta.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: trkeya
ms.author: trkeya
ms.date: 04/14/2021
ms.custom: contperf-fy21q2
ms.openlocfilehash: e4ffb01b6fcd9fe3e487abcf306470dd9529eec0
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109635218"
---
# <a name="update-existing-offers-in-the-commercial-marketplace"></a>Actualización de ofertas existentes en el Marketplace comercial

En este artículo se explica cómo hacer actualizaciones de ofertas y planes existentes y también cómo quitar una oferta del Marketplace comercial. Puede ver las ofertas en la pestaña **Información general** del [portal de Marketplace comercial](https://partner.microsoft.com/dashboard/commercial-marketplace/offers) en el Centro de partners.

## <a name="update-a-published-offer"></a>Actualización de una oferta publicada

Use estos pasos para actualizar una oferta que se ha publicado correctamente en estado Versión preliminar o En directo.

1. Seleccione el nombre de la oferta que desea actualizar. El estado de la oferta puede aparecer como **Versión preliminar**, **Activa**, **Publicación en curso**, **Borrador**, **Atención necesaria** o **No disponible** (si anteriormente ha elegido desusar, antiguamente dejar de vender, la oferta). Una vez seleccionado, se abrirá la página **Información general de la oferta** para dicha oferta.
1. Seleccione la página de la oferta que quiere actualizar, como **Propiedades**, **Descripción de la oferta** o **Versión preliminar** (o seleccione **Actualizar** en la tarjeta correspondiente de la página **Información general de la oferta**).
1. Realice los cambios y, después seleccione **Save draft** (Guardar borrador). Repita este proceso hasta que completar todos los cambios.
1. Revise los cambios en la página **[Comparar](#compare-changes-to-your-offer)** .
1. Cuando esté listo para publicar la oferta actualizada, seleccione **Review and publish** (Revisar y publicar) en cualquier página. Se abrirá la página **Review and publish** (Revisar y publicar). En esta página verá el estado de finalización de las secciones de la oferta que ha actualizado: 
    - **Cambios sin publicar**: La sección se ha actualizado y completado. Se han proporcionado todos los datos necesarios y no ha habido errores en las actualizaciones.
    - **Incompletos**: Las actualizaciones realizadas en la sección han introducido errores que tienen que corregirse o es necesario proporcionar más información.
2. Seleccione **Publicar** para publicar la oferta actualizada. Luego, la oferta pasará por los [pasos de validación y publicación](../review-publish-offer.md#validation-and-publishing-steps) estándar.

## <a name="changing-offer-type"></a>Cambio del tipo de oferta

[!INCLUDE [change-offer-type](../includes/change-offer-type.md)]

> [!IMPORTANT]
> Debe revisar la versión preliminar de la oferta una vez que esté disponible y seleccionar **Transmitir** para publicar su oferta actualizada para los destinatarios previstos (públicos o privados).

## <a name="add-a-plan-to-an-existing-offer"></a>Agregar un plan a una oferta existente

Siga estos pasos para agregar un plan nuevo a una oferta que ya ha publicado.

1. Con la página de **Información general de la oferta** de la oferta existente abierta, vaya a la página de **Información general del plan** y, a continuación, seleccione **Crear un nuevo plan**.
1. Cree un nuevo plan de acuerdo con las [directrices](../plans-pricing.md) mediante el **modelo de precios de planes existentes**.
1. Seleccione **Guardar borrador** después de cambiar el nombre del plan.
1. Seleccione **Publicar** cuando esté listo para publicar las actualizaciones. Se abrirá la página **[Revisar y publicar](../review-publish-offer.md)** proporcionando un estado de finalización para las actualizaciones.

## <a name="update-a-plan-for-an-existing-offer"></a>Actualización de un plan para una oferta existente

Siga estos pasos para realizar cambios en un plan para una oferta que ya haya publicado.

1. Con la página de **Información general de la oferta** de la oferta existente abierta, elija el plan que quiera cambiar. Si el plan no es accesible desde la lista de **Información general del plan**, seleccione **Ver todos los planes**.
1. Seleccione los valores de **Name** (Nombre), **Pricing model** (Modelo de precios) o **Availability** (Disponibilidad) del plan. *En la actualidad, los planes solo están disponibles en inglés (Estados Unidos)* .
1. Seleccione **Save draft** (Guardar borrador) después de realizar los cambios necesarios en el nombre, la descripción, el público o la disponibilidad del plan.
1. Si está listo para publicar sus actualizaciones, seleccione **Review and publish** (Revisar y publicar). Se abrirá la página **[Revisar y publicar](../review-publish-offer.md)** proporcionando un estado de finalización para las actualizaciones.
1. Seleccione **Publicar** para publicar el plan actualizado. Le enviaremos un correo electrónico cuando haya alguna versión preliminar de la oferta actualizada disponible para su revisión y aprobación.

## <a name="offer-a-virtual-machine-plan-at-a-new-price"></a>Oferta de un plan de máquina virtual a un nuevo precio

Una vez publicado un plan de máquina virtual, no se puede cambiar su precio. Para ofrecer el mismo plan a un precio diferente, debe ocultar el plan y crear otro con el precio actualizado. En primer lugar, oculte el plan con el precio que quiere cambiar:

1. Con la página de **Información general de la oferta** de la oferta existente abierta, elija el plan que quiera cambiar. Si el plan no es accesible desde la lista de **Información general del plan**, seleccione **Ver todos los planes**.
1. Active la casilla **Ocultar plan**. Guarde el borrador antes de continuar.

Ahora que ha ocultado el plan con el precio anterior, cree una copia de ese plan con el precio actualizado:

1. En el Centro de partners, vuelva a **Plan overview** (Información general del plan).
2. Seleccione **Crear nuevo plan**. Rellene los campos **Id. de plan** y **Nombre del plan** y, luego, seleccione **Crear**.
1. Para volver a usar la configuración técnica del plan que ha ocultado, active la casilla **Reuse technical configuration** (Reutilizar la configuración técnica). Lea [Creación de planes para una oferta de máquina virtual](../azure-vm-create-plans.md) para obtener más información.
    > [!IMPORTANT]
    > Si selecciona **This plan reuses technical configuration from another plan** (Este plan reutiliza la configuración técnica de otro plan), no podrá desusar (antiguamente, dejar de vender) el plan principal más adelante. No use esta opción si quiere desusar (antiguamente, dejar de vender) el plan principal.
3. Complete todas las secciones necesarias del nuevo plan, incluido el nuevo precio.
1. Seleccione **Guardar borrador**.
1. Cuando haya completado todas las secciones necesarias del nuevo plan, seleccione **Review and publish** (Revisar y publicar). Esta acción envía la oferta para su revisión y publicación. Lea [Revisión y publicación de una oferta en el marketplace comercial](../review-publish-offer.md) para más información.

## <a name="sync-private-plan-audiences"></a>Sincronización de audiencias de plan privado

Si su oferta incluye uno o varios planes que están configurados para que solo estén disponibles a una audiencia privada restringida, puede actualizar solo la audiencia que tenga acceso a ese plan privado sin publicar otros cambios en la oferta. 

Para actualizar y sincronizar la audiencia privada de sus planes:

- Modifique la audiencia en uno o varios planes privados mediante el botón + **Agregar un ID** o **Importar clientes (csv)** y, a continuación, guarde los cambios.
- Seleccione **Sync private audience** (Sincronizar audiencia privada) desde la página **Información general del plan**.

**Sync private audience** (Sincronizar audiencia privada) publicará solo los cambios para sus audiencias privadas, sin publicar ninguna otra actualización que haya podido realizar en la oferta borrador.

## <a name="compare-changes-to-your-offer"></a>Comparación de los cambios en la oferta

Antes de publicar las actualizaciones de la oferta en directo o de [versión preliminar](#compare-changes-to-a-preview-offer), puede auditar los cambios guardados en la página **Comparar**. Acceda a la página **Comparar** en la esquina superior derecha de cualquier página de oferta, como las páginas **Propiedades** o **Descripción de la oferta**. La página **Comparar** muestra versiones en paralelo de los cambios guardados de esta oferta y de la oferta de Marketplace publicada.

- Puede usar **Compare** (Comparar) en cualquier momento del proceso de edición.
- Seleccione un campo en la página **Compare** (Comparar) para ir al valor que desea modificar.
- Para ver los valores de todos los campos, hasta de los que no están actualizados, seleccione el filtro **Todos los campos**. Para modificar los filtros de estos campos, seleccione **Modified fields** (Campos modificados) y, después, seleccione uno de los siguientes filtros:
    - El filtro **Removed values** (Valores eliminados) muestra los campos que ha publicado y que está eliminando completamente.
    - El filtro **Added values** (Valores agregados) muestra los campos que no publicó originalmente y agrega ahora.
    - El filtro **Edited values** (Valores editados) muestra campos que se habían publicado, pero ahora ha actualizado el contenido.

      >[!NOTE]
      > Si alguno de estos filtros no está disponible, indica que no ha realizado ninguna actualización de ese tipo.

- Para ver solo los valores que no se han actualizado, seleccione el campo **Unchanged fields** (Campos sin cambios). Los valores de los campos que se muestran para la versión de borrador y la publicada serán los mismos.

  ![Los filtros para comparar las actualizaciones de una oferta publicada o de versión preliminar](./media/compare-changes-marketplace.png)

>[!NOTE]
> Las siguientes páginas no admiten actualmente **Compare** (Comparar):
>- Audiencia de revendedores de CSP
>- Configuración técnica de la versión de prueba
>- Listas de Marketplace de la versión de prueba
>- Venta conjunta
>- Archivos complementarios

Recuerde volver a publicar la oferta después de realizar actualizaciones para que los cambios surtan efecto.

### <a name="compare-changes-to-a-preview-offer"></a>Comparación de cambios en una oferta de versión preliminar

Si tiene cambios en versión preliminar que no están activos, puede comparar los cambios nuevos con la versión preliminar de la oferta de Marketplace.

1. Seleccione **Compare** (Comparar) en la barra de comandos de la página.
2. Seleccione la lista desplegable **Con** y cambie la opción de **Live offer** (Oferta en directo) a **Preview offer** (Oferta en versión preliminar). Si la oferta todavía no se publica, no verá la opción **Oferta en directo**.
3. La página **Compare** (Comparar) proporciona versiones en paralelo que muestran los cambios.

Recuerde volver a publicar la oferta después de realizar actualizaciones para que los cambios surtan efecto.

## <a name="deprecate-an-offer-or-plan"></a>Desuso de una oferta o plan

> [!IMPORTANT]
> El nombre de la opción para detener la venta ha cambiado a desuso.

Puede quitar planes y descripciones de las ofertas del Marketplace comercial de Microsoft, lo que impedirá que los clientes nuevos los encuentren y los compren. Los clientes que hayan adquirido anteriormente la oferta o el plan todavía podrán usarlo y pueden descargarlo de nuevo si es necesario. Sin embargo, no recibirán actualizaciones si decide volver a publicar la oferta o el plan más adelante.

- Para desusar (antiguamente, dejar de vender) una oferta tras su publicación, seleccione **Desusar** desde la página **Información general de la oferta**. Pocas horas después de la confirmación, la oferta dejará de estar visible en el Marketplace comercial.

- Para desusar (antiguamente, detener la venta) de un plan, seleccione **Desusar** en la página **Información general del plan**. La opción para desusar (antiguamente, dejar de vender) un plan solo está disponible si tiene más de un plan en la oferta. Puede elegir desusar (antiguamente, dejar de vender) un plan sin que otros planes dentro de la oferta se vean afectados.
     >[!NOTE]
     > Una vez que confirme que quiere desusar (antiguamente, dejar de vender) el plan, debe volver a publicar la oferta para que el cambio surta efecto.

Cuando desuse (antiguamente, deje de vender) una oferta o un plan, seguirá viéndolos en el Centro de partners con el estado **No disponible**. Si decide volver a mostrar o vender esta oferta o este plan, siga las instrucciones para [actualizar una oferta publicada](#update-a-published-offer). No olvide que después de realizar cualquier cambio tiene que volver a **publicar** la oferta o el plan.

## <a name="remove-offers-from-existing-customers"></a>Eliminación de ofertas de clientes existentes

Para eliminar ofertas de los clientes existentes, [presente una solicitud de soporte técnico](https://aka.ms/marketplacepublishersupport). En la lista de temas de soporte técnico, seleccione **Marketplace comercial** > **Eliminación de ofertas o aplicaciones** y envíe la solicitud. El equipo de soporte técnico le guiará en el proceso de eliminación de la oferta.

## <a name="next-steps"></a>Pasos siguientes

- [Comprobación del estado de publicación de la oferta de marketplace comercial](../review-publish-offer.md)
