---
title: Migración de las soluciones de venta conjunta de OCP GTM al Centro de partners para Microsoft AppSource
description: Obtenga información sobre cómo migrar soluciones de venta conjunta de OCP GTM al Centro de partners para Microsoft AppSource.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: vamahtan
ms.author: vamahtan
ms.reviewer: stmummer
ms.date: 3/04/2021
ms.openlocfilehash: bb954a32a4e598c45578d4b06606bc133babb50d
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107107159"
---
# <a name="migration-of-co-sell-solutions-from-ocp-gtm-to-the-commercial-marketplace"></a>Migración de soluciones de venta conjunta de OCP GTM al marketplace comercial

Microsoft está migrando la experiencia de publicación. El [marketplace comercial](overview.md) proporciona una experiencia de publicación de ofertas simplificada para realizar ventas conjuntas a través de los tres canales de Microsoft, gracias a la centralización de la creación y administración de ofertas en el Centro de partners, donde ya está administrando su relación con Microsoft.

Como asociado de Microsoft inscrito en el marketplace comercial, puede realizar las siguientes acciones:

- Publicar sus ofertas de forma centralizada y realizar ventas conjuntas entre los canales de clientes directos, asociados y vendedores de Microsoft.
- Asegurarse de que sus ofertas estén en la tienda en línea correcta ([Microsoft AppSource](https://appsource.microsoft.com) o [Azure Marketplace](https://azure.microsoft.com)) para llegar a los millones de clientes en la nube que necesiten las funcionalidades que proporcionan su oferta.
- Mejorar su propia experiencia de publicación para realizar ventas conjuntas con las ofertas que se ajusten a sus objetivos empresariales.
- Alinear la publicación de ofertas en el Centro de partners, donde ya está administrando su relación con Microsoft y oportunidades de venta conjunta.
- Desbloquear [Marketplace Rewards](partner-center-portal/marketplace-rewards.md).

## <a name="prerequisites-to-continue-co-selling-with-microsoft"></a>Requisitos previos para continuar realizando ventas conjuntas con Microsoft

Asegúrese de que tiene una pertenencia de Microsoft Partner Network activa y que esté inscrito en el marketplace comercial, en el Centro de partners.

- Únase a Microsoft Partner Network [de forma gratuita](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership). Como asociado, tendrá acceso a recursos, programas, herramientas y conexiones exclusivos para hacer crecer su negocio.
- Si no tiene una cuenta en el marketplace comercial, [inscríbase ahora](create-account.md) para seguir realizando ventas conjuntas con Microsoft y disfrutar de la experiencia de publicación completa.

## <a name="publishing-updates-for-attaining-co-sell-ready-status"></a>Publicación de actualizaciones para alcanzar el estado de listo para la venta conjunta

Para que los vendedores y asociados de Microsoft puedan descubrir su solución, debe cumplir los [requisitos de preparación para realizar ventas conjuntas](marketplace-co-sell.md). Para que un vendedor de Microsoft reciba incentivos, la solución debe cumplir los [requisitos pertinentes](marketplace-co-sell.md). Complete estos requisitos en la pestaña Venta conjunta del Centro de partners (vea [esta imagen](#cosell-tab) que aparece más abajo en este artículo).

> [!NOTE]
> En el marketplace comercial, las soluciones se conocen como "ofertas" a lo largo de la experiencia de publicación.

Después de inscribirse en el marketplace comercial, prepárese para migrar las soluciones de OCP GTM.

Siga estos pasos antes de importar las soluciones de OCP GTM:

1. Visite la [lista de publicadores](https://partner.microsoft.com/dashboard/account/v3/publishers/list) de su empresa. Esta lista incluye el propietario de la cuenta, los administradores y los desarrolladores que tienen acceso de publicación. Obtenga más información sobre los [roles de usuario del Centro de partners](user-roles.md).
2. Pida a uno de los contactos indicados que [agregue usuarios](https://partner.microsoft.com/dashboard/account/usermanagement) al marketplace comercial como *administradores* o *desarrolladores*, ya que solo estos roles pueden editar y publicar soluciones.
3. Colabore sus desarrolladores para migrar sus soluciones de la cuenta de OCP GTM al marketplace comercial.
4. Decida cuál de las siguientes opciones desea realizar:
    1. Combinar esta solución con una oferta similar del marketplace comercial
    1. Migrar esta solución desde OCP GTM al marketplace comercial
    1. Descartar esta solución

## <a name="migrate-your-solutions-from-ocp-gtm"></a>Migración de soluciones de OCP GTM

1. Comience con la migración [aquí](https://partner.microsoft.com/solutions/migration#).
2. Seleccione la página **Overview** (Información general) y, después, en **Click here to get started** (Haga clic aquí para empezar).

    :::image type="content" source="media/co-sell-migrate/migration-overview.png" alt-text="Página Overview (Información general) del Centro de partners; pestaña Overview (Información general).":::

3. Para empezar la migración, seleccione la pestaña **Solutions** (Soluciones), que muestra todas las soluciones asociadas a los identificadores de MPN.

    :::image type="content" source="media/co-sell-migrate/solutions-tab.png" alt-text="Página Overview (Información general) del Centro de partners; pestaña Solutions (Soluciones).":::

    > [!NOTE]
    > Esta pestaña mostrará un mensaje en caso de que no haya ninguna solución pendiente de migrar al marketplace comercial. Para seguir realizando ventas conjuntas con Microsoft, asegúrese de que las soluciones migradas están publicadas en el marketplace comercial.

    Para obtener más información sobre el estado de la solución, revise la información sobre herramientas. Todas las soluciones pendientes de acción aparecen en **Acción** (Acción).<a name="beginmigration"></a>

4. Seleccione **Begin migration** (Iniciar migración) (vea la imagen anterior) de la solución que desea migrar y, a continuación, seleccione una de las siguientes opciones:

    :::image type="content" source="media/co-sell-migrate/migration-options.png" alt-text="Las tres opciones de migración.":::

### <a name="merge-solution-with-a-similar-offer"></a>Combinación de la solución con una oferta similar

Seleccione esta opción si la solución ya está publicada en el marketplace comercial y si ambas deben combinarse en una única oferta. Esto evitará que se creen ofertas duplicadas.

1. Identifique la oferta que ya existe.
    1. Seleccione **I want to merge this solution with a similar offer in the commercial marketplace** (Quiero combinar esta solución con una oferta similar del marketplace comercial) (vea la imagen de **Action required** [Acción necesaria] [anterior](#beginmigration)).
    1. En la pestaña **Action 1** (Acción 1) se muestran las ofertas del marketplace comercial activas a las que se puede asociar la solución de OCP GTM. Seleccione la oferta activa de la lista si tiene una. Si no hay ninguna lista de ofertas entre las que elegir, escriba la dirección orientada al cliente (URL) de Microsoft AppSource o Azure Marketplace.
        [![La pestaña Action 1 (Acción 1) del proceso de combinación.](media/co-sell-migrate/action-1-merge.png)](media/co-sell-migrate/action-1-merge.png#lightbox)
    1. Seleccione **Continuar**.
1. Solicite la combinación.
    1. La pestaña **Action 2** (Acción 2) muestra instrucciones para solicitar la combinación de la solución de OCP GTM con la que identificó. Para solicitar la combinación, seleccione **Save & Contact support** (Guardar y ponerse en contacto con el equipo de soporte técnico), que abre una página de soporte técnico del asociado en un explorador.
    1. Seleccione **Provide Issue Details** (Proporcionar detalles del problema) y escriba lo siguiente:      [![La pestaña Action 2 (Acción 2) del proceso de combinación.](media/co-sell-migrate/action-2-merge.png)](media/co-sell-migrate/action-2-merge.png#lightbox)
    1. Seleccione **Submit** (Enviar). El equipo de soporte técnico del asociado se pondrá en contacto con usted en un plazo de dos días laborables.
    1. Ellos lo ayudarán a asegurarse de que la combinación de esta oferta se realiza correctamente para que se publique como una oferta activa.

### <a name="migrate-this-solution-from-ocp-gtm"></a>Migración de la solución de OCP GTM

Seleccione esta opción cuando tenga una solución de OCP GTM que no cuente aún con una oferta correspondiente publicada en el marketplace comercial. Tendrá que publicar esta solución en el marketplace comercial para seguir realizando ventas conjuntas con Microsoft. Al migrar esta solución, ahorrará tiempo conservando la información y la lista de materiales de OCP GTM. Esta opción requiere que seleccione un tipo de oferta.

1. Seleccione **I want to migrate this solution from OCP GTM to the commercial marketplace** (Quiero migrar esta solución de OCP GTM al marketplace comercial) (consulte la imagen de **Action required** [Acción necesaria] [anterior](#beginmigration)). A continuación, seleccione **Continue** (Continuar).
1. En la pestaña **Action 1** (Acción 1), seleccione el [tipo de oferta](publisher-guide-by-offer-type.md) y **Continue** (Continuar).

    [![La pestaña Action 1 (Acción 1) del proceso de migración.](media/co-sell-migrate/action-1-migrate.png)](media/co-sell-migrate/action-1-migrate.png#lightbox)

1. En la pestaña **Action 2** (Acción 2), seleccione el [perfil de publicador](create-account.md) de la lista que se proporciona. Si no tiene una cuenta de publicador, cree una en el [Centro de partners](https://partner.microsoft.com/solutions/migration) y selecciónela en esta pantalla.

    [![La pestaña Action 2 (Acción 2) del proceso de migración.](media/co-sell-migrate/action-2-migrate.png)](media/co-sell-migrate/action-2-migrate.png#lightbox)

1. Seleccione **Create a draft offer** (Crear un borrador de oferta) para migrar la solución al marketplace comercial como borrador. Todavía no estará activa.
1. Continúe el proceso de publicación en el Centro de partners. Para obtener ayuda con la publicación en el Centro de partners, consulte [Publicación de la oferta en el marketplace](#make-your-offer-live-in-the-marketplace) a continuación.

### <a name="discard-this-solution"></a>Descarte de la solución

Seleccione esta opción cuando ya no sea relevante una solución de las de OCP GTM. Se le pedirá que confirme la acción de descarte; podrá deshacerla más adelante.

1. Seleccione **I want to discard this solution** (Deseo descartar esta solución) (vea la imagen de **Action required** [Acción necesaria] [anterior](#beginmigration)). A continuación, seleccione **Continue** (Continuar).
2. Seleccione **Discard** (Descartar).

    :::image type="content" source="media/co-sell-migrate/migration-discard.png" alt-text="Confirme la acción de descarte.":::

3. Para deshacer la acción de descarte, seleccione **Undo Discard** (Deshacer la acción de descarte).

    :::image type="content" source="media/co-sell-migrate/migration-discard-undo.png" alt-text="El vínculo Undo Discard (Deshacer la acción de descarte).":::

4. Si necesita más ayuda, seleccione la pestaña **Get Help** (Obtener ayuda) para ponerse en contacto con el equipo de soporte técnico del asociado.

    :::image type="content" source="media/co-sell-migrate/get-support-link.png" alt-text="El vínculo de soporte técnico de la pestaña Get Help (Obtener ayuda).":::

## <a name="make-your-offer-live-in-the-marketplace"></a>Publicación de la oferta en el marketplace

Si ha seleccionado migrar su oferta al marketplace comercial, se migrará como borrador. Todavía tiene que publicar su oferta para que se active en el marketplace comercial. De esta forma, se conservará su estado de venta conjunta, incentivos y la canalización de referencia.

Para obtener instrucciones detalladas sobre la información que debe proporcionar antes de que se pueda publicar su oferta, lea la [guía de publicación](publisher-guide-by-offer-type.md) correspondiente. Para ver un resumen, siga leyendo a continuación.

1. En la página **Overview** (Información general) del Centro de partners, seleccione el nombre del borrador de oferta.

    :::image type="content" source="media/co-sell-migrate/offer-overview.png" alt-text="La página Overview (Información general) de la oferta del Centro de partners.":::

<a name="cosell-tab"></a>

2. Complete la información requerida en cada pestaña. Si lo desea, puede completar la página **Resell through CSP** (Revender a través de CSP) (en el menú de navegación a la izquierda de abajo) para revender a través del programa Proveedor de soluciones en la nube (CSP). Los vínculos **Learn more** (Más información) y la información sobre herramientas lo guiarán a través de los requisitos y los detalles.

    :::image type="content" source="media/co-sell-migrate/offer-setup.png" alt-text="Las pestañas de configuración e información general de ofertas del Centro de partners.":::

3. Algunos de los detalles visibles para los vendedores de Microsoft se copiaron de la solución de OCP GTM. Complete la información necesaria restante en la pestaña **Co-sell with Microsoft** (Realizar venta conjunta con Microsoft) para hacer que la oferta de venta conjunta esté lista. Cuando haya terminado, seleccione **Review and publish** (Revisar y publicar). Para más información, consulte [Configuración de la venta conjunta para una oferta del marketplace comercial](commercial-marketplace-co-sell.md).

    :::image type="content" source="media/co-sell-migrate/co-sell-page.png" alt-text="La pestaña Co-sell with Microsoft (Realizar venta conjunta con Microsoft) del Centro de partners.":::

4. Después de revisar toda la información enviada, seleccione **Publish** (Publicar) para enviar el borrador de oferta a la fase de revisión de certificación.

    :::image type="content" source="media/co-sell-migrate/co-sell-review-publish.png" alt-text="La pestaña Review and publish (Revisar y publicar) y el botón Publish (Publicar) del Centro de partners.":::

5. Realice un seguimiento del estado de su envío en la pestaña **Overview** (Información general).

    :::image type="content" source="media/publish-status-publisher-signoff.png" alt-text="La barra de estado de publicación de la oferta en la pestaña Overview (Información general) del Centro de partners.":::

6. Se le notificará cuando se complete la revisión de certificación. Si proporcionamos comentarios útiles, haga clic en ellos y, a continuación, seleccione **Publish** (Publicar) para iniciar una recertificación.
7. Una vez que la oferta pase la certificación, obtenga una vista previa de la oferta con el vínculo proporcionado y realice los ajustes finales que desee. Cuando lo tengo todo listo, seleccione **Go Live** (Activar) (vea el botón arriba) para publicar la oferta en los escaparates del marketplace comercial pertinentes. Una vez que lo haya hecho, su oferta conservará su estado de venta conjunta de la solución de OCP GTM original.

## <a name="next-steps"></a>Pasos siguientes

- [Reventa a través de asociados de CSP](cloud-solution-providers.md)
- [Configuración de la venta conjunta para una oferta del marketplace comercial](commercial-marketplace-co-sell.md)
- Revisión de estas [preguntas más frecuentes](https://partner.microsoft.com/resources/detail/co-sell-requirements-publish-commercial-marketplace-faq-pdf) (PDF)
