---
title: Migración de las soluciones de venta conjunta de OCP GTM al Centro de partners para Microsoft AppSource
description: Obtenga información sobre cómo migrar soluciones de venta conjunta de OCP GTM al Centro de partners para Microsoft AppSource.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: vamahtan
ms.author: vamahtan
ms.reviewer: stmummer
ms.date: 05/18/2021
ms.openlocfilehash: d4eb677b0f55a8ccd2cda33e6026e01238deb9b3
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111971528"
---
# <a name="migration-of-co-sell-solutions-from-ocp-gtm-to-the-commercial-marketplace"></a>Migración de soluciones de venta conjunta de OCP GTM al marketplace comercial

Microsoft está migrando la experiencia de publicación. El [marketplace comercial](overview.md) proporciona una experiencia de publicación de ofertas simplificada para realizar ventas conjuntas a través de los tres canales de Microsoft, gracias a la centralización de la creación y administración de ofertas en el Centro de partners, donde ya está administrando su relación con Microsoft.

Como asociado de Microsoft inscrito en el marketplace comercial, puede realizar las siguientes acciones:

- Publicar sus ofertas de forma centralizada y realizar ventas conjuntas entre los canales de clientes directos, asociados y vendedores de Microsoft.
- Asegurarse de que sus ofertas estén en la tienda en línea correcta ([Microsoft AppSource](https://appsource.microsoft.com) o [Azure Marketplace](https://azure.microsoft.com)) para llegar a los millones de clientes en la nube que necesiten las funcionalidades que proporcionan su oferta.
- Mejorar su propia experiencia de publicación para realizar ventas conjuntas con las ofertas que se ajusten a sus objetivos empresariales.
- Alinear la publicación de ofertas en el Centro de partners, donde ya está administrando su relación con Microsoft y oportunidades de venta conjunta.
- Desbloquear [Marketplace Rewards](marketplace-rewards.md).

## <a name="prerequisites-to-continue-co-selling-with-microsoft"></a>Requisitos previos para continuar realizando ventas conjuntas con Microsoft

Asegúrese de que tiene una pertenencia de Microsoft Partner Network activa y que esté inscrito en el marketplace comercial, en el Centro de partners.

- Únase a Microsoft Partner Network [de forma gratuita](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership). Como asociado, tendrá acceso a recursos, programas, herramientas y conexiones exclusivos para hacer crecer su negocio.
- Si no tiene una cuenta en el marketplace comercial, [inscríbase ahora](create-account.md) para seguir realizando ventas conjuntas con Microsoft y disfrutar de la experiencia de publicación completa.

## <a name="publishing-updates-for-attaining-co-sell-ready-status"></a>Publicación de actualizaciones para alcanzar el estado de listo para la venta conjunta

Para que los vendedores y asociados de Microsoft puedan descubrir su solución, debe cumplir los [requisitos de preparación para realizar ventas conjuntas](./co-sell-overview.md). Para que un vendedor de Microsoft reciba incentivos, la solución debe cumplir los [requisitos pertinentes](./co-sell-overview.md). Complete estos requisitos en la pestaña Venta conjunta del Centro de partners (vea [esta imagen](#action-2-merge) que aparece más abajo en este artículo).

> [!NOTE]
> En el marketplace comercial, las soluciones se conocen como "ofertas" a lo largo de la experiencia de publicación.

Después de inscribirse en el marketplace comercial, prepárese para migrar las soluciones de OCP GTM. Hemos creado una interfaz de migración digital que proporciona una vista única de las soluciones de OCP GTM para su organización. Desde esta interfaz, podrá combinar la solución con una oferta similar ya publicada y activa en marketplace comercial, o descartarla.

Siga estos pasos antes de importar las soluciones de OCP GTM:

1. Visite la [lista de publicadores](https://partner.microsoft.com/dashboard/account/v3/publishers/list) de su empresa. Esta lista incluye el propietario de la cuenta, los administradores y los desarrolladores que tienen acceso de publicación. Obtenga más información sobre los [roles de usuario del Centro de partners](user-roles.md).
2. Pida a uno de los contactos indicados que [agregue usuarios](https://partner.microsoft.com/dashboard/account/usermanagement) al marketplace comercial como *administradores* o *desarrolladores*, ya que solo estos roles pueden editar y publicar soluciones.
3. Colabore sus desarrolladores para migrar sus soluciones de la cuenta de OCP GTM al marketplace comercial.
4. Decida cuál de las siguientes opciones desea realizar:
    1. Si tiene una solución en OCP GTM que desea migrar al Centro de partners, *para conservar la canalización de referencia, el material adjunto, el estado de la venta conjunta y los incentivos*, hay dos escenarios entre los que puede elegir:
        - Si no hay ninguna oferta similar en marketplace comercial
            - Paso 1. [Creación de una oferta](#step-1-similar-offer-does-not-exist-in-commercial-marketplace-please-follow-these-steps)
            - Paso 2. [Publicación de la oferta en el marketplace comercial](#scenario-2-similar-offer-exists-in-commercial-marketplace-please-follow-these-steps)
            - Paso 3. [Solicitud de combinación de la solución de OCP GTM con la oferta de marketplace recién publicada](#begin-the-migration-of-your-solutions-from-ocp-gtm).
        - Si ya hay una oferta similar en marketplace comercial
            - Paso 1. [Solicitud de combinación de la solución de OCP GTM con la oferta de marketplace publicada](#begin-the-migration-of-your-solutions-from-ocp-gtm)
    1. Si tiene una solución en OCP GTM que ya no es importante, descarte esta solución.

> [!IMPORTANT]
> El 17 de mayo de 2021, se eliminó la siguiente opción de migración.  Si anteriormente usó la opción Migración y tiene una oferta que se migró al marketplace comercial como un borrador de oferta, es importante que complete el proceso de publicación en el marketplace comercial.  
> - Para más información, consulte [Procedimiento para revisar y publicar una oferta en el marketplace comercial de Microsoft | Microsoft Docs](./review-publish-offer.md)
> - [Publicación en el conjunto de diapositivas de entrenamiento de marketplace comercial](https://assetsprod.microsoft.com/mpn/en-us/guidance-and-permissions-needed-to-publish-an-offer-and-co-sell-with-ms.pptm)
:::image type="content" source="media/co-sell-migrate/ready-to-migrate.png" alt-text="Quiero migrar esta solución al marketplace comercial":::

## <a name="begin-the-migration-of-your-solutions-from-ocp-gtm"></a>Inicio de la migración de las soluciones desde OCP GTM

1. Comience con la migración [aquí](https://partner.microsoft.com/solutions/migration#).
2. Seleccione la página **Overview** (Información general) y, después, en **Click here to get started** (Haga clic aquí para empezar).

    :::image type="content" source="media/co-sell-migrate/welcome-overveiw.png" alt-text="Muestra la página de información general":::

3. Para empezar la migración, seleccione la pestaña **Solutions** (Soluciones), que muestra todas las soluciones asociadas a los identificadores de MPN.

    :::image type="content" source="media/co-sell-migrate/solutions-tab.png" alt-text="Página Overview (Información general) del Centro de partners; pestaña Solutions (Soluciones).":::

    > [!NOTE]
    > Esta pestaña mostrará un mensaje en caso de que no haya ninguna solución pendiente de migrar al marketplace comercial. Para seguir realizando ventas conjuntas con Microsoft, asegúrese de que las soluciones migradas están publicadas en el marketplace comercial.

    Para obtener más información sobre el estado de la solución, revise la información sobre herramientas. Todas las soluciones pendientes de acción aparecen en **Acción** (Acción).<a name="beginmigration"></a>

4. Seleccione **Begin migration** (Iniciar migración) (vea la imagen anterior) de la solución que desea migrar y, a continuación, seleccione una de las siguientes opciones:

    :::image type="content" source="media/co-sell-migrate/migration-options.png" alt-text="Las tres opciones de migración.":::

### <a name="merge-solution"></a>Combinación de la solución

¿Cuál es la importancia de combinar la solución de OCP GTM en una oferta publicada y activa del marketplace comercial?

- A los asociados que venden actualmente de forma conjunta ofertas en OCP GTM, le recomendamos que migren sus soluciones al marketplace comercial para aprovechar sus funcionalidades y simplificar la experiencia de publicación.

Si tiene una solución en OCP GTM que desea migrar al Centro de partners puede elegir entre dos escenarios. Al final de este proceso de combinación, la oferta conservará la canalización de referencia, el material adjunto, el estado de la venta conjunta y los incentivos.

#### <a name="step-1-similar-offer-does-not-exist-in-commercial-marketplace-please-follow-these-steps"></a>Paso 1: No existe una oferta similar en marketplace comercial. Siga estos pasos.

Si aún no tiene una oferta en el marketplace comercial para combinar una solución en OCP GTM, primero deberá crearla Y PUBLICARLA el marketplace comercial (esto permitirá conservar el estado de la venta conjunta, los incentivos y la canalización de referencia).

1. Cree un borrador de la oferta en el marketplace comercial.

   1. Seleccione **+Nueva oferta**.

        :::image type="content" source="media/co-sell-migrate/new-offer.png" alt-text="Presentación de la nueva oferta":::

   2. Complete la información necesaria en cada pestaña. • Los vínculos de **Obtener más información** y la información sobre herramientas le guiarán a través de los requisitos y los detalles.
         • Si lo desea, puede completar la página **Resell through CSP** (Revender a través de CSP) (en el menú de navegación a la izquierda de abajo) para revender a través del programa Proveedor de soluciones en la nube (CSP).

        :::image type="content" source="media/co-sell-migrate/offer-setup-nav.png" alt-text="Muestra la página Configuración de la oferta con las opciones de información general resaltadas.":::
   3. Seleccione **Guardar borrador**.
        - Para obtener instrucciones detalladas sobre la información que debe proporcionar antes de que se pueda publicar su oferta, lea la [guía de publicación](./publisher-guide-by-offer-type.md) correspondiente.
        - Revise los requisitos de idoneidad en el artículo correspondientes para el tipo de oferta para finalizar la selección y la configuración de la oferta.
        - Revise los patrones de publicación por tienda en línea para ver ejemplos de cómo la solución se asigna a un tipo de oferta y configuración.
        - [Procedimientos recomendados para la publicación de ofertas: marketplace comercial de Microsoft | Microsoft Docs](./gtm-offer-listing-best-practices.md)

    > [!TIP]
    > Se recomienda *no rellenar* los datos de la pestaña **Venta conjunta con Microsoft**. Para ahorrarle tiempo, nos encargaremos de rellenar estos datos en su lugar con la documentación y materiales adjuntos existentes en OCP GTM durante el proceso de combinación.

    Una vez completada la combinación, puede volver a la pestaña Venta conjunta con Microsoft y realizar actualizaciones si es necesario. Para más información, consulte [Configuración de la venta conjunta para una oferta del marketplace comercial](./co-sell-configure.md).
1. Cuando haya terminado, seleccione **Review and publish** (Revisar y publicar).

     :::image type="content" source="media/co-sell-migrate/co-sell-with-ms.png" alt-text="Aparece la página Venta conjunta con Microsoft con las opciones resaltadas":::
1. Después de revisar toda la información enviada, seleccione **Publish** (Publicar) para enviar el borrador de oferta a la fase de revisión de certificación. [Obtenga más información sobre la fase de certificación](./review-publish-offer.md).:::image type="content" source="media/co-sell-migrate/review-and-publish.png" alt-text="Muestra la página Revisar y publicar.":::
1. Realice un seguimiento del estado de su envío en la pestaña Overview (Información general).

    :::image type="content" source="media/co-sell-migrate/offer-overview-tab.png" alt-text="Aparece la pestaña Información general":::
1. Se le notificará cuando se complete la revisión de certificación. Si proporcionamos comentarios útiles, haga clic en ellos y, a continuación, seleccione **Publish** (Publicar) para iniciar una recertificación.

1. Una vez que la oferta pase la certificación, obtenga una vista previa de la oferta con el vínculo proporcionado y realice los ajustes finales que desee. Cuando lo tengo todo listo, seleccione **Go Live** (Activar) (vea el botón arriba) para publicar la oferta en los escaparates del marketplace comercial pertinentes.

1. **Continúe con el escenario 2 a continuación para completar el proceso de combinación.**

#### <a name="scenario-2-similar-offer-exists-in-commercial-marketplace-please-follow-these-steps"></a>Escenario 2: Existe una oferta similar en el marketplace comercial. Siga estos pasos.

Seleccione esta opción si la solución ya está publicada y activa en el marketplace comercial, y la solución de OCP GTM y la oferta del marketplace comercial se deben combinar en una sola oferta. Esto evitará que se creen ofertas duplicadas.

1. Identifique la oferta que ya existe.
   1. Seleccione **This is similar to an existing offer in commercial marketplace** (Esto es similar a una oferta existente del marketplace comercial). Consulte la imagen **Begin your migration** (Comienzo de la migración) [más arriba](#beginmigration).
   1. En la pestaña **Action 1** (Acción 1) se muestran las ofertas del marketplace comercial activas a las que se puede asociar la solución de OCP GTM. Seleccione la oferta activa de la lista si tiene una. Si no hay ninguna lista de ofertas entre las que elegir, escriba la dirección orientada al cliente (URL) de Microsoft AppSource o Azure Marketplace.
   1. Seleccione **Continuar**.
        [![La pestaña Action 1 (Acción 1) del proceso de combinación.](media/co-sell-migrate/action-1-merge.png)](media/co-sell-migrate/action-1-merge.png#lightbox)
1. Solicite la combinación.
    1. La pestaña **Action 2** (Acción 2) muestra instrucciones para solicitar la combinación de la solución de OCP GTM con la que identificó. Para solicitar la combinación, seleccione **Save & Contact support** (Guardar y ponerse en contacto con el equipo de soporte técnico), que abre una página de soporte técnico del asociado en un explorador. 
    <a id="action-2-merge"></a>
        [![La pestaña Action 2 (Acción 2) del proceso de combinación.](media/co-sell-migrate/action-2-merge.png)](media/co-sell-migrate/action-2-merge.png#lightbox)
    1. Seleccione **Provide Issue Details** (Proporcionar detalles del problema) y escriba lo siguiente:
        1. Tipo de oferta
        1. Nombre de la oferta
        1. Identificador de la solución OCP
        1. Dirección URL de la publicación existente
    1. Seleccione **Submit** (Enviar). El equipo de soporte técnico del asociado se pondrá en contacto con usted en un plazo de dos días laborables.
    1. Ellos lo ayudarán a asegurarse de que la combinación de esta oferta se realiza correctamente para que se publique como una oferta activa.

### <a name="discard-this-solution"></a>Descarte de la solución

Se recomienda dedicar un tiempo a evaluar la viabilidad y relevancia de las soluciones para optimizar su presencia y detectabilidad en el marketplace comercial con ofertas de alta calidad que satisfagan la demanda de los clientes.

Seleccione esta opción cuando ya no sea relevante una solución de las de OCP GTM. Se le pedirá que confirme la acción de descarte; podrá deshacerla más adelante.

1. Seleccione **I want to discard this solution** (Deseo descartar esta solución) (vea la imagen de **Action required** [Acción necesaria] [anterior](#beginmigration)). A continuación, seleccione **Continue** (Continuar).
2. Seleccione **Discard** (Descartar).

    :::image type="content" source="media/co-sell-migrate/migration-discard.png" alt-text="Confirme la acción de descarte.":::

3. Para deshacer la acción de descarte, seleccione **Undo Discard** (Deshacer la acción de descarte).

    :::image type="content" source="media/co-sell-migrate/migration-discard-undo.png" alt-text="El vínculo Undo Discard (Deshacer la acción de descarte).":::

### <a name="additional-help"></a>Ayuda adicional

 Si necesita más ayuda, seleccione la pestaña **Get Help** (Obtener ayuda) para ponerse en contacto con el equipo de soporte técnico del asociado.

:::image type="content" source="media/co-sell-migrate/get-support-link.png" alt-text="El vínculo de soporte técnico de la pestaña Get Help (Obtener ayuda).":::

## <a name="next-steps"></a>Pasos siguientes

- [Reventa a través de asociados de CSP](cloud-solution-providers.md)
- [Configuración de la venta conjunta para una oferta del marketplace comercial](./co-sell-configure.md)
- Revisión de estas [preguntas más frecuentes](https://partner.microsoft.com/resources/detail/co-sell-requirements-publish-commercial-marketplace-faq-pdf) (PDF)