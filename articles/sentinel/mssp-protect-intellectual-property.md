---
title: Protección de la propiedad intelectual del proveedor de servicios de seguridad administrada (MSSP) en Azure Sentinel | Microsoft Docs
description: Obtenga información sobre cómo pueden proteger los proveedores de servicios de seguridad administrada (MSSP) la propiedad intelectual que han creado en Azure Sentinel.
services: sentinel
documentationcenter: na
author: batamig
manager: rkarlin
editor: ''
ms.assetid: 10cce91a-421b-4959-acdf-7177d261f6f2
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/12/2021
ms.author: bagol
ms.openlocfilehash: b6ada209d556ae981360661cea4db71219dbba37
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108124534"
---
# <a name="protecting-mssp-intellectual-property-in-azure-sentinel"></a>Protección de la propiedad intelectual del MSSP en Azure Sentinel

En este artículo se describen los métodos que pueden usar los proveedores de servicios de seguridad administrada (MSSP) para proteger la propiedad intelectual que han desarrollado en Azure Sentinel, como reglas de análisis de Azure Sentinel, consultas de búsqueda, cuadernos de estrategias y libros.

El método que elija dependerá de cómo compre los servicios de Azure cada uno de sus clientes; tanto si actúa como proveedor de soluciones en la nube [(CSP)](#cloud-solutions-providers-csp) como si el cliente tiene una cuenta de [Contrato Enterprise (EA) o Pago por uso (PAYG)](#enterprise-agreements-ea--pay-as-you-go-payg). En las secciones siguientes se describe cada uno de estos métodos por separado.

## <a name="cloud-solutions-providers-csp"></a>Proveedores de soluciones en la nube (CSP)

Si va a revender Azure como proveedor de soluciones en la nube (CSP), administrará la suscripción de Azure del cliente. Gracias a la [administración delegada (AOBO)](/partner-center/azure-plan-manage), se concede a los usuarios del grupo de agentes de administración del inquilino del MSSP acceso de propietario a la suscripción de Azure del cliente y el cliente no tiene acceso de manera predeterminada.

Si otros usuarios del inquilino del MSSP que no pertenecen al grupo de agentes de administración necesitan acceder al entorno del cliente, se recomienda usar [Azure Lighthouse](multiple-tenants-service-providers.md). Azure Lighthouse permite conceder a los usuarios o grupos acceso a un ámbito específico, como un grupo de recursos o una suscripción, mediante uno de los roles integrados.

Si necesita proporcionar a los usuarios del cliente acceso al entorno de Azure, se recomienda concederles acceso en el nivel de *grupo de recursos*, en lugar de en toda la suscripción, para que pueda mostrar u ocultar partes del entorno según sea necesario.

Por ejemplo:

- Puede conceder al cliente acceso a varios grupos de recursos donde se encuentran sus aplicaciones, pero mantener el área de trabajo de Azure Sentinel en un grupo de recursos independiente al que el cliente no tenga acceso.

- Use este método para permitir que los clientes puedan ver libros y cuadernos de estrategias seleccionados, que son recursos independientes que pueden residir en su propio grupo de recursos.

Incluso con la concesión de acceso en el nivel de grupo de recursos, los clientes seguirán teniendo acceso a los datos de registro de los recursos a los que pueden acceder, como los registros de una máquina virtual, incluso sin acceso a Azure Sentinel. Para más información, consulte [Administración del acceso a los datos de Azure Sentinel por recurso](resource-context-rbac.md).

> [!TIP]
> Si necesita proporcionar a los clientes acceso a toda la suscripción, es posible que quiera ver las instrucciones que se describen en [Contrato Enterprise (EA) y Pago por uso (PAYG)](#enterprise-agreements-ea--pay-as-you-go-payg).
>

### <a name="sample-azure-sentinel-csp-architecture"></a>Arquitectura de ejemplo del CSP para Azure Sentinel

En la imagen siguiente se describe cómo podrían funcionar los permisos descritos en la [sección anterior](#cloud-solutions-providers-csp) al proporcionar acceso a los clientes del CSP:

:::image type="content" source="media/mssp-protect-intellectual-property/csp-customers.png" alt-text="Protección de la propiedad intelectual de Azure Sentinel con los clientes del CSP.":::

En esta imagen:

- Los usuarios a los que se ha concedido acceso de **Propietario** a la suscripción del CSP son los usuarios del grupo de agentes de administración, en el inquilino de Azure AD del MSSP.
- Otros grupos del MSSP obtienen acceso al entorno del cliente mediante Azure Lighthouse.
- RBAC de Azure administra el acceso de los clientes a los recursos de Azure en el nivel de grupo de recursos.

    Esto permite a los MSSP ocultar componentes de Azure Sentinel según sea necesario, como las reglas de análisis y las consultas de búsqueda.

Para más información, consulte también [Azure Lighthouse y el programa Proveedor de soluciones en la nube](../lighthouse/concepts/cloud-solution-provider.md).

## <a name="enterprise-agreements-ea--pay-as-you-go-payg"></a>Contratos Enterprise (EA) y Pago por uso (PAYG)

Si el cliente compra directamente a Microsoft, el cliente ya tiene acceso total al entorno de Azure y no puede ocultar nada que esté en la suscripción de Azure del cliente.

En su lugar, proteja la propiedad intelectual que ha desarrollado en Azure Sentinel como se muestra a continuación, en función del tipo de recurso que necesite proteger:

### <a name="analytics-rules-and-hunting-queries"></a>Reglas de análisis y consultas de búsqueda

Las reglas de análisis y las consultas de búsqueda están contenidas en Azure Sentinel y, por tanto, no se pueden separar del área de trabajo de Azure Sentinel.

Incluso si un usuario solo tiene permisos de lector de Azure Sentinel, todavía podrá ver la consulta. En este caso, se recomienda hospedar las reglas de análisis y las consultas de búsqueda en el propio inquilino del MSSP, en lugar del inquilino del cliente.

Para ello, necesitará un área de trabajo en su propio inquilino con Azure Sentinel habilitado y también necesitará ver el área de trabajo del cliente mediante [Azure Lighthouse](multiple-tenants-service-providers.md).

Para crear una regla de análisis o una consulta de búsqueda en el inquilino del MSSP que haga referencia a los datos del inquilino del cliente, debe usar la instrucción `workspace` de la siguiente manera:

```kql
workspace('<customer-workspace>').SecurityEvent
| where EventID == ‘4625’
```

Al agregar una instrucción `workspace` a las reglas de análisis, tenga en cuenta lo siguiente:

- **No hay alertas en el área de trabajo del cliente**. Las reglas creadas de esta manera no crearán alertas ni incidentes en el área de trabajo del cliente. Las alertas y los incidentes solo existirán en el área de trabajo del MSSP.

- **Cree alertas independientes para cada cliente.** Al usar este método, también se recomienda usar reglas de alertas independientes para cada cliente y detección, ya que la instrucción workspace será diferente en cada caso.

    Puede agregar el nombre del cliente al nombre de la regla de alertas para identificar fácilmente el cliente en el que se desencadena la alerta. Las alertas independientes pueden dar lugar a un gran número de reglas, que es posible que desee administrar mediante scripting o mediante [Azure Sentinel como código](https://techcommunity.microsoft.com/t5/azure-sentinel/deploying-and-managing-azure-sentinel-as-code/ba-p/1131928).

    Por ejemplo:

    :::image type="content" source="media/mssp-protect-intellectual-property/mssp-rules-per-customer.png" alt-text="Cree reglas independientes en el área de trabajo del MSSP para cada cliente.":::

- **Cree áreas de trabajo del MSSP independientes para cada cliente.** La creación de reglas independientes para cada cliente y detección puede hacer que alcance el número máximo de reglas de análisis del área de trabajo (512). Si tiene muchos clientes y espera alcanzar este límite, puede crear un área de trabajo del MSSP independiente para cada cliente.

    Por ejemplo:

    :::image type="content" source="media/mssp-protect-intellectual-property/mssp-rules-and-workspace-per-customer.png" alt-text="Cree un área de trabajo y reglas en el inquilino del MSSP para cada cliente.":::

> [!IMPORTANT]
> La clave para usar este método correctamente es usar la automatización para administrar un gran conjunto de reglas en las áreas de trabajo.
>
> Para más información consulte [Reglas de análisis entre áreas de trabajo](https://techcommunity.microsoft.com/t5/azure-sentinel/what-s-new-cross-workspace-analytics-rules/ba-p/1664211).
>

### <a name="workbooks"></a>Libros

Si ha desarrollado un libro de Azure Sentinel que no desea que el cliente copie, hospede el libro en el inquilino del MSSP. Asegúrese de que tiene acceso a las áreas de trabajo del cliente mediante Azure Lighthouse y, a continuación, asegúrese de modificar el libro para usar esas áreas de trabajo del cliente.

Por ejemplo:

:::image type="content" source="media/mssp-protect-intellectual-property/cross-workspace-workbook.png" alt-text="Libros entre áreas de trabajo":::

Para más información, consulte [Uso de libros entre áreas de trabajo](extend-sentinel-across-workspaces-tenants.md#cross-workspace-workbooks).

Si desea que el cliente pueda ver las visualizaciones del libro y a la vez mantener el código en secreto, se recomienda exportar el libro a Power BI.

La exportación del libro a Power BI:

- **Hace que las visualizaciones del libro sean más fáciles de compartir.** Puede enviar al cliente un vínculo al panel de Power BI, donde puede ver los datos notificados sin necesidad de requerir permisos de acceso de Azure.
- **Habilita la programación**. Configure Power BI para enviar correos electrónicos periódicamente que contengan una instantánea del panel en ese momento.

Para más información, consulte [Importación de los datos del registro de Azure Monitor en Power BI](../azure-monitor/visualize/powerbi.md).

### <a name="playbooks"></a>Playbooks

Puede proteger los cuadernos de estrategias como se muestra a continuación, en función de dónde se hayan creado las reglas de análisis que desencadenan el cuaderno de estrategias:

- **Reglas de análisis creadas en el área de trabajo del MSSP**.  Asegúrese de crear los cuadernos de estrategias en el inquilino del MSSP y de obtener todos los datos de incidentes y alertas del área de trabajo del MSSP. Puede adjuntar los cuadernos de estrategias cada vez que cree una nueva regla en el área de trabajo.

    Por ejemplo:

    :::image type="content" source="media/mssp-protect-intellectual-property/rules-in-mssp-workspace.png" alt-text="Reglas creadas en el área de trabajo del MSSP.":::

- **Reglas de análisis creadas en el área de trabajo del cliente**. Use Azure Lighthouse para adjuntar las reglas de análisis desde el área de trabajo del cliente a un cuaderno de estrategias hospedado en el área de trabajo del MSSP. En este caso, el cuaderno de estrategias obtiene los datos de alertas e incidentes, así como cualquier otra información del cliente, del área de trabajo del cliente.

    Por ejemplo:

    :::image type="content" source="media/mssp-protect-intellectual-property/rules-in-customer-workspace.png" alt-text="Reglas creadas en el área de trabajo del cliente.":::

En ambos casos, si el cuaderno de estrategias necesita acceder al entorno de Azure del cliente, utilice un usuario o una entidad de servicio que tenga ese acceso mediante Lighthouse.

Sin embargo, si el cuaderno de estrategias necesita acceder a recursos que no son de Azure en el inquilino del cliente, como Azure AD, Office 365 o Microsoft 365 Defender, deberá crear una entidad de servicio con los permisos adecuados en el inquilino del cliente y, a continuación, agregar esa identidad en el cuaderno de estrategias.

> [!NOTE]
> Si usa reglas de automatización junto con los cuadernos de estrategias, debe establecer los permisos de las reglas de automatización en el grupo de recursos donde se encuentra el cuaderno de estrategias.
> Para más información, consulte [Permisos para que las reglas de automatización ejecuten cuadernos de estrategias](automate-incident-handling-with-automation-rules.md#permissions-for-automation-rules-to-run-playbooks).
>

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte:

- [Cuaderno de estrategias técnico de Azure Sentinel para los MSSP](https://cloudpartners.transform.microsoft.com/download?assetname=assets/Azure-Sentinel-Technical-Playbook-for-MSSPs.pdf&download=1)
- [Administración de varios inquilinos en Azure Sentinel como un MSSP](multiple-tenants-service-providers.md)
- [Extender Azure Sentinel por áreas de trabajo e inquilinos](extend-sentinel-across-workspaces-tenants.md)
- [Tutorial: Visualizar y supervisar los datos](tutorial-monitor-your-data.md)
- [Tutorial: Configuración de respuestas automatizadas frente a amenazas en Azure Sentinel](tutorial-respond-threats-playbook.md)