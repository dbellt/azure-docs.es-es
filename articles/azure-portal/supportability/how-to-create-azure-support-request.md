---
title: Creación de una solicitud de soporte técnico de Azure
description: Los clientes que necesiten asistencia pueden usar Azure Portal para buscar soluciones de autoservicio y para crear y administrar solicitudes de soporte técnico.
ms.assetid: fd6841ea-c1d5-4bb7-86bd-0c708d193b89
ms.topic: how-to
ms.date: 05/25/2021
ms.openlocfilehash: 3b38a010fa23605019f1969a80e9f9a3cda467c8
ms.sourcegitcommit: bb9a6c6e9e07e6011bb6c386003573db5c1a4810
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110496584"
---
# <a name="create-an-azure-support-request"></a>Creación de una solicitud de soporte técnico de Azure

Azure le permite crear y administrar solicitudes de soporte técnico, también conocidas como incidencias de soporte técnico. Puede crear y administrar las solicitudes en [Azure Portal](https://portal.azure.com), tal y como se describe en este artículo. También puede crear y administrar solicitudes mediante programación con la [API REST de incidencia de soporte técnico de Azure](/rest/api/support) o con la [CLI de Azure](/cli/azure/azure-cli-support-request).

> [!NOTE]
> La dirección URL de Azure Portal es específica de la nube de Azure donde está implementada la organización.
>
>* Azure Portal para uso comercial: [https://portal.azure.com](https://portal.azure.com)
>* Azure Portal para Alemania: [https://portal.microsoftazure.de](https://portal.microsoftazure.de)
>* Azure Portal para el gobierno de los Estados Unidos: [https://portal.azure.us](https://portal.azure.us)

La experiencia de solicitud de soporte técnico se centra en tres objetivos principales:

* **Optimizada**: hacer que el soporte técnico y la solución de problemas sean fáciles de encontrar y simplificar el envío de una solicitud de soporte técnico.
* **Integrada**: puede abrir con facilidad una solicitud de soporte técnico para un problema con un recurso de Azure, sin necesidad de cambiar de contexto.
* **Eficaz**: recopile la información clave que necesita el ingeniero de soporte técnico para resolver el problema de forma eficaz.

Azure proporciona soporte ilimitado para la administración de suscripciones, lo que incluye facturación, ajustes de cuota y transferencias de cuentas. Para obtener soporte técnico, necesita un plan de soporte técnico. Para más información, consulte [Comparar planes de soporte técnico](https://azure.microsoft.com/support/plans).

## <a name="getting-started"></a>Introducción

Puede ir a **Ayuda y soporte técnico** en Azure Portal. Está disponible en el menú de Azure Portal, en el encabezado global o en el menú de recursos de un servicio. Debe tener los permisos adecuados para enviar una solicitud de soporte técnico.

### <a name="azure-role-based-access-control"></a>Control de acceso basado en roles de Azure

Para crear una solicitud de soporte técnico, debe ser un [propietario](../../role-based-access-control/built-in-roles.md#owner), un[colaborador](../../role-based-access-control/built-in-roles.md#contributor) o debe tener asignado el rol de [colaborador de la solicitud de soporte técnico](../../role-based-access-control/built-in-roles.md#support-request-contributor) en el nivel de suscripción. Para crear una solicitud de soporte técnico sin una suscripción, por ejemplo, en un escenario de Azure Active Directory, debe ser [administrador](../../active-directory/roles/permissions-reference.md).

### <a name="go-to-help--support-from-the-global-header"></a>Ir a Ayuda y soporte técnico en el encabezado global

Para iniciar una solicitud de soporte técnico desde cualquier lugar de Azure Portal:

1. Seleccione el signo **?** en el encabezado global, seleccione **Ayuda y soporte técnico**.

   ![Ayuda y soporte técnico](./media/how-to-create-azure-support-request/helpandsupportnewlower.png)

1. Seleccione **Nueva solicitud de soporte técnico**. Siga las indicaciones para proporcionar información sobre el problema. Sugeriremos algunas soluciones posibles, recopilaremos los detalles del problema y le ayudaremos a enviar y realizar el seguimiento de la solicitud de soporte técnico.

   ![Nueva solicitud de soporte técnico](./media/how-to-create-azure-support-request/newsupportrequest2lower.png)

### <a name="go-to-help--support-from-a-resource-menu"></a>Ir a Ayuda y soporte técnico en un menú de recursos

Para iniciar una solicitud de soporte técnico en el contexto del recurso con el que está trabajando actualmente:

1. En el menú de recursos, en la sección **Soporte técnico y solución de problemas**, seleccione **Nueva solicitud de soporte técnico**.

   ![En contexto](./media/how-to-create-azure-support-request/incontext2lower.png)

1. Siga las indicaciones para proporcionarnos la información sobre el problema que tiene. Al iniciar el proceso de solicitud de soporte técnico desde el recurso, algunas opciones se preseleccionan automáticamente.

## <a name="create-a-support-request"></a>Crear una solicitud de soporte

Le guiaremos a través de algunos pasos para recopilar información sobre el problema y ayudarle a resolverlo. Este paso se describe en las secciones siguientes.

### <a name="basics"></a>Aspectos básicos

En el primer paso del proceso de solicitud de soporte técnico se recopila información básica sobre el problema y el plan de soporte técnico.

En la pestaña **Datos básicos** de **Nueva solicitud de soporte técnico**, use los selectores para empezar a contarnos el problema. En primer lugar, identificará algunas categorías generales para el tipo de problema y elegirá la suscripción relacionada. Seleccione el servicio; por ejemplo, **Máquina virtual que ejecuta Windows**. Seleccione el recurso, por ejemplo, el nombre de la máquina virtual. Describa el problema con sus propias palabras y, a continuación, seleccione **Tipo de problema** y **Subtipo de problema** para proporcionar información más específica.

![Hoja Básico](./media/how-to-create-azure-support-request/basics2lower.png)

### <a name="solutions"></a>Soluciones

Después de recopilar la información básica, le mostraremos soluciones para que pueda probar por su cuenta. En algunos casos, es posible que incluso ejecutemos un diagnóstico rápido. Las soluciones están escritas por ingenieros de Azure y resolverán la mayoría de los problemas comunes.

### <a name="details"></a>Detalles

A continuación, recopilaremos información adicional sobre el problema. Proporcionar información exhaustiva y detallada en este paso nos ayuda a enrutar la solicitud de soporte técnico al ingeniero adecuado.

1. Si es posible, indíquenos cuándo se inició el problema y los pasos para reproducirlo. Puede cargar un archivo, como un archivo de registro o una salida de diagnósticos. Para obtener más información sobre las cargas de archivos, consulte [Instrucciones de carga de archivos](how-to-manage-azure-support-request.md#file-upload-guidelines).

1. Una vez que tenemos toda la información sobre el problema, puede elegir cómo obtener soporte técnico. En la sección **Método de soporte técnico** de **Detalles**, seleccione la gravedad del impacto. El nivel de gravedad máximo depende de su [plan de soporte técnico](https://azure.microsoft.com/support/plans).

    La opción **Compartir información de diagnóstico** está seleccionada de forma predeterminada. Esto permite al soporte técnico de Azure recopilar [información de diagnóstico](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/) de los recursos de Azure. Puede desactivar esta opción si prefiere no compartir información de diagnóstico. En algunos casos, hay una segunda pregunta que no está seleccionada de forma predeterminada, como solicitar acceso a la memoria de una máquina virtual.

1. Proporcione su método de contacto preferido, un buen momento para ponernos en contacto con usted y su idioma de soporte técnico.

1. A continuación, complete la sección **Información de contacto** para que sepamos cómo ponernos en contacto con usted.

### <a name="review--create"></a>Revisar y crear

Complete toda la información necesaria en cada pestaña y, a continuación, seleccione **Revisar y crear**. Compruebe los detalles que va a enviar al servicio de soporte técnico. Vuelva a cualquier pestaña para realizar un cambio si es necesario. Cuando esté satisfecho con la solicitud de soporte técnico, seleccione **Crear**.

Un ingeniero de soporte técnico se pondrá en contacto con usted mediante el método que haya indicado. Consulte [Ámbito de soporte técnico y capacidad de respuesta](https://azure.microsoft.com/support/plans/response/) para obtener información sobre los tiempos de respuesta inicial.


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre las opciones de soporte técnico de autoayuda de Azure, vea este vídeo:

> [!VIDEO https://www.youtube.com/embed/gNhzR5FE9DY]

Siga estos vínculos para obtener más información:

* [Administración de una solicitud de soporte técnico de Azure](how-to-manage-azure-support-request.md)
* [API REST de incidencia de soporte técnico de Azure](/rest/api/support)
* [Enviarnos sus comentarios y sugerencias](https://feedback.azure.com/forums/266794-support-feedback)
* Ponerse en contacto con nosotros en [Twitter](https://twitter.com/azuresupport)
* Obtener ayuda de sus compañeros en la [página de preguntas y respuestas de Microsoft](/answers/products/azure)
* Obtener más información en [Preguntas más frecuentes de soporte técnico de Azure](https://azure.microsoft.com/support/faq)