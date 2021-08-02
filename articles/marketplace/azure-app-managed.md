---
title: Configuración de un plan de una aplicación administrada
description: Configuración de un plan de una aplicación administrada para su oferta de aplicación de Azure en el Centro de partners (Azure Marketplace).
author: aarathin
ms.author: aarathin
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 06/01/2021
ms.openlocfilehash: 9fb40048a57955fc6a8e4a4e73361f545380f84e
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111971558"
---
# <a name="configure-a-managed-application-plan"></a>Configuración de un plan de una aplicación administrada

Este artículo solo se aplica a los planes de aplicaciones administradas de una oferta de aplicación de Azure. Si va a configurar un plan de una plantilla de solución, vaya a [Configuración de un plan de plantilla de solución](azure-app-solution.md).

## <a name="re-use-technical-configuration-optional"></a>Reutilización de la configuración técnica (opcional)

Si ha creado más de un plan del mismo tipo en esta oferta y la configuración técnica es idéntica entre ellos, puede volver a usar la configuración técnica de otro plan. Esta configuración no se puede modificar una vez publicado este plan.

Para reutilizar una configuración técnica:

1. Seleccione la casilla **This plan reuses the technical configuration from another plan of the same type** (Este plan reutiliza la configuración técnica de otro plan del mismo tipo).
1. En la lista que aparece, seleccione el plan base que desee.

> [!NOTE]
> Si un plan está reutilizando o ha reutilizado la configuración técnica de otro plan del mismo tipo, vaya al plan base para ver el historial de paquetes publicados anteriormente.

## <a name="define-markets-pricing-and-availability"></a>Definición de mercados, precios y disponibilidad

Todos los planes debe estar disponible al menos en un mercado. En la pestaña **Precios y disponibilidad**, puede configurar los mercados en los que este plan estará disponible, el precio y si desea que el plan sea visible para todo el mundo o solo para clientes específicos (conocido también como un plan privado).

1. En **Mercados**, seleccione el vínculo **Editar mercados**.
1. En el cuadro de diálogo que aparece, seleccione las ubicaciones de mercado en las que desea que el plan esté disponible. Debe seleccionar un mínimo de uno y un máximo de 141 mercados.

    > [!NOTE]
    > En este cuadro de diálogo se incluyen un cuadro de búsqueda y una opción para filtrar los países con "Impuesto remitido", en los que Microsoft remite de los impuestos de uso y ventas en su nombre.

1. Seleccione **Guardar** para cerrar el cuadro de diálogo.

## <a name="define-pricing"></a>Definición de los precios

En el cuadro **Precio**, proporcione el precio mensual de este plan. El precio se suma a los costos de cualquier infraestructura de Azure o basados en el uso en los que incurren los recursos implementados por esta solución.

Además del precio por mes, también puede establecer precios por el consumo de unidades no estándar mediante la [facturación de uso medido](./azure-app-metered-billing.md). Si lo desea, puede establecer el precio por mes en cero y cobrar exclusivamente con la facturación de uso medido.

Los precios se establecen en USD (USD = dólar estadounidense) y se convierten en la moneda local de todos los mercados seleccionados con los tipos de cambio actuales en el momento de guardarlos. Valide estos precios antes de realizar la publicación. Para ello, debe exportar la hoja de cálculo de precios y examinar el precio de cada mercado. Si desea establecer precios personalizados en un mercado individual, modifique e importe la hoja de cálculo de precios.

### <a name="add-a-custom-meter-dimension-optional"></a>Adición de una dimensión de medidor personalizada (opcional)

1. En **Dimensiones del servicio de medición de Marketplace**, seleccione el vínculo **Adición de una dimensión de medidor personalizada (máx. 18)** .
1. En el cuadro **Id.** , escriba la referencia inmutable del identificador mientras emite eventos de uso.
1. En el cuadro **Nombre para mostrar**, escriba el nombre para mostrar asociado a la dimensión. Por ejemplo, "mensajes de texto enviados".
1. En el cuadro **Unidad de medida**, escriba la descripción de la unidad de facturación. Por ejemplo, "por mensaje de texto" o "por 100 correos electrónicos".
1. En el cuadro **Precio por unidad en USD**, indique el precio de una unidad de la dimensión.
1. En el cuadro **Cantidad mensual incluida en base**, escriba la cantidad (como entero) de la dimensión que se incluye cada mes para los clientes que pagan la cuota mensual recurrente. Para establecer una cantidad ilimitada, active la casilla en su lugar.
1. Para agregar otra dimensión de medidor personalizada, repita los pasos 1 a 7.

### <a name="set-custom-prices-optional"></a>Definición de precios personalizados (opcional)

Los precios establecidos en USD (USD = dólares de Estados Unidos) se convierten en la moneda local de todos los mercados seleccionados con los tipos de cambio actuales en el momento de guardarlos. Valide estos precios antes de realizar la publicación. Para ello, debe exportar la hoja de cálculo de precios y examinar el precio de cada mercado. Si desea establecer precios personalizados en un mercado individual, modifique e importe la hoja de cálculo de precios.

Revise los precios cuidadosamente antes de la publicación, ya que hay algunas restricciones en lo que se puede cambiar una vez publicado un plan.

> [!NOTE]
> Después de que se publica un precio para un mercado en el plan, no se puede cambiar más adelante.

Para establecer precios personalizados en un mercado individual, exporte, modifique e importe la hoja de cálculo de precios. El usuario es responsable de validar estos precios y de poseer estos valores. Para obtener información detallada, consulte [Precios personalizados](plans-pricing.md#custom-prices).

1. Primero debe guardar los cambios de precios para habilitar la exportación de los datos de precios. Cerca de la parte inferior de la pestaña **Precios y disponibilidad**, seleccione **Guardar borrador**.
1. En **Precio**, seleccione el vínculo **Export pricing data** (Exportar datos de precios).
1. Abra el archivo exportedPrice.xlsx en Microsoft Excel.
1. En la hoja de cálculo, realice las actualizaciones que desee en la información de los precios y, después, guarde el archivo.

   Para poder actualizar el archivo, es posible que tenga que habilitar la edición en Excel.

1. En la pestaña **Precios y disponibilidad**, en **Precios**, seleccione el vínculo **Import pricing data** (Importar datos de precios).
1. En el cuadro de diálogo que aparece, haga clic en **Sí**.
1. Seleccione el archivo exportedPrice.xlsx que ha actualizado y, después, haga clic en **Abrir**.

## <a name="choose-who-can-see-your-plan"></a>Selección de quién puede ver el plan

Tiene la opción de configurar cada plan para que lo pueda ver todo el mundo o solo un público concreto. Puede conceder acceso a un público privado con los identificadores de la suscripción de Azure con la opción de incluir una descripción de cada identificador de suscripción que asigne. Puede agregar hasta 10 identificadores de suscripción manualmente o hasta 10 000 mediante un archivo .csv. Los identificadores de suscripción de Azure se representan como GUID y las letras deben estar en minúsculas.

> [!NOTE]
> Si publica un plan privado, puede cambiar su visibilidad a público más adelante. Sin embargo, una vez que se publica un plan público, no se puede cambiar su visibilidad a privado.

En **Plan visibility** (Visibilidad del plan), haga lo siguiente:

- Para que el plan sea público, seleccione el botón de opción **Público** (también conocido como _botón de radio_).
- Para que el plan sea privado, seleccione el botón de opción **Privado** y, luego, agregue los identificadores de la suscripción de Azure manualmente o con un archivo .csv.

> [!NOTE]
> Una audiencia privada o restringida es diferente del público preliminar que definió en la pestaña **Versión preliminar**. Una audiencia preliminar puede acceder a la oferta antes de que se publique en el marketplace. En tanto que la designación de una audiencia privada solo se aplica a un plan específico, el público preliminar puede ver todos los planes (privados o no) con fines de validación.

### <a name="manually-add-azure-subscription-ids-for-a-private-plan"></a>Adición manual de los identificadores de la suscripción de Azure a un plan privado

1. En **Plan visibility** (Visibilidad del plan), seleccione el botón de opción **Privado**.
1. En el cuadro **Identificador de la suscripción de Azure** que aparece, escriba el identificador de la suscripción de Azure del público al que quiere conceder acceso a este plan privado. Se requiere un identificador de suscripción como mínimo.
1. (Opcional) Escriba una descripción de este público en el cuadro **Descripción**.
1. Para agregar otro identificador de suscripción, seleccione el vínculo **Add ID (Max 10)** (Agregar identificador [10 como máximo]) y repita los pasos 2 y 3.

### <a name="use-a-csv-file-to-add-azure-subscription-ids-for-a-private-plan"></a>Uso de un archivo .csv para agregar los identificadores de la suscripción de Azure a un plan privado

1. En **Plan visibility** (Visibilidad del plan), seleccione el botón de opción **Privado**.
1. Seleccione el vínculo **Exportar público (CSV)** .
1. Abra el archivo .csv y, en la columna **Id.** , agregue los identificadores de la suscripción de Azure a los que quiere conceder acceso a la oferta privada.
1. Opcionalmente, escriba una descripción para cada público en la columna **Descripción**.
1. Agregue "SubscriptionId" en la columna **Tipo** para cada fila con un identificador de suscripción.
1. Guarde el archivo .CSV.
1. En la pestaña **Disponibilidad**, en **Plan visibility** (Visibilidad del plan), seleccione el vínculo **Import Audience (csv)** (Importar audiencia [csv]).
1. En el cuadro de diálogo que aparece, seleccione **Sí**.
1. Seleccione el archivo .CSV y, después, elija **Abrir**. Aparece un mensaje que indica que el archivo .csv se importó correctamente.

## <a name="define-the-technical-configuration"></a>Definición de la configuración técnica

En la pestaña **Configuración técnica**, cargará el paquete de implementación que permite a los clientes implementar el plan y proporcionar un número de versión para el paquete. También podrá proporcionar información técnica adicional.

> [!NOTE]
> Esta pestaña no estará visible si ha elegido volver a usar paquetes de otro plan en la pestaña **Configuración del plan**. Si este es el caso, vaya a [Visualización de los planes](#view-your-plans).

### <a name="assign-a-version-number-for-the-package"></a>Asignación de un número de versión al paquete

En el cuadro **Versión**, proporcione la versión actual de la configuración técnica. Incremente esta versión cada vez que publique un cambio en esta página. El número de versión debe tener el formato: número entero.número entero.número entero. Por ejemplo, `1.0.2`.

### <a name="upload-a-package-file"></a>Carga de un archivo de paquete

En **Archivo de paquete (.zip)** , arrastre el archivo de paquete al cuadro gris o seleccione el vínculo para **buscar los archivos**.

> [!NOTE]
> Si tiene un problema al cargar archivos, asegúrese de que la red local no bloquee el servicio `https://upload.xboxlive.com` que usa el Centro de partners.

#### <a name="previously-published-packages"></a>Paquetes publicados anteriormente

La subpestaña **Paquetes publicados anteriormente** permite ver todas las versiones publicadas de la configuración técnica.

### <a name="enable-just-in-time-jit-access-optional"></a>Habilitar acceso Just-In-Time (JIT) (opcional)

Para habilitar el acceso JIT para este plan, seleccione la casilla **Habilitar el acceso Just-In-Time (JIT)** . Para exigir que los consumidores de la aplicación administrada concedan el acceso permanente a la cuenta, deje esta opción desactivada. Para más información sobre esta opción, vea [Acceso Just-In-Time (JIT)](plan-azure-app-managed-app.md#just-in-time-jit-access).

### <a name="select-a-deployment-mode"></a>Selección de un modo de implementación

Seleccione el modo de implementación **Completo** o **Incremental**.

- En el modo **Completo**, una reimplementación de la aplicación del cliente provocará la eliminación de recursos en el grupo de recursos administrados si los recursos no se definen en [mainTemplate.json](../azure-resource-manager/managed-applications/publish-service-catalog-app.md?tabs=azure-powershell#create-the-arm-template).
- En el modo **Incremental**, una reimplementación de la aplicación deja los recursos existentes sin cambios.

Para obtener más información sobre los modos de implementación, consulte [Modos de implementación de Azure Resource Manager](../azure-resource-manager/templates/deployment-modes.md).

### <a name="provide-a-notification-endpoint-url"></a>Definición de una dirección URL del punto de conexión de notificaciones

En el cuadro **URL del punto de conexión de notificación**, proporcione un punto de conexión de webhook de HTTPS para recibir notificaciones sobre todas las operaciones CRUD que están en las instancias de aplicación administradas de esta versión del plan.

### <a name="customize-allowed-customer-actions-optional"></a>Personalización de acciones de cliente permitidas (opcional)

1. Para especificar qué acciones pueden realizar los clientes en los recursos administrados, además de las acciones "`*/read`" que están disponibles de forma predeterminada, seleccione el cuadro **Personalizar acciones de cliente permitidas**.
1. En los cuadros que aparecen, proporcione las acciones de control adicionales y las acciones de datos permitidas que desea permitir que realice el cliente, separadas por punto y coma. Por ejemplo, para permitir que los consumidores reinicien las máquinas virtuales, agregue `Microsoft.Compute/virtualMachines/restart/action` al cuadro **Allowed control actions** (Acciones de control permitidas).

### <a name="choose-who-can-manage-the-application"></a>Elección de las personas que pueden administrar la aplicación

Indique quién debe tener acceso de administración a esta aplicación administrada en cada región de Azure seleccionada: _Azure global_ y _Nube de Azure Government_. Usará las identidades de Azure AD para identificar a los usuarios, grupos y aplicaciones a los que quiere conceder permiso para el grupo de recursos administrados. Para más información, vea [Planeamiento de una aplicación administrada de Azure para una oferta de aplicación de Azure](plan-azure-application-offer.md).

Complete los pasos siguientes para Azure Global y la nube de Azure Government, según corresponda.

1. En el cuadro **Id. de inquilino de Azure Active Directory**, escriba el identificador de inquilino de Azure AD (también conocido como identificador de directorio) que contiene las identidades de los usuarios, grupos o aplicaciones a los que quiere conceder permisos.
1. En el cuadro **Id. de entidad de seguridad**, proporcione el identificador de objeto de Azure AD del usuario, grupo o aplicación a los que quiere conceder permiso para el grupo de recursos administrados. Identifique al usuario por su identificador de entidad de seguridad, que encontrará en la [hoja Usuarios de Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/AllUsers), en Azure Portal.
1. En la lista **Definición de roles**, seleccione un rol integrado de Azure AD. El rol que seleccione describe los permisos que tendrá la entidad de seguridad en los recursos de la suscripción del cliente.
1. Para agregar otra autorización, seleccione el vínculo **Agregar autorización (máx. 100)** y repita los pasos 1 a 3.

### <a name="policy-settings-optional"></a>Configuración de directiva (opcional)

Puede configurar un máximo de cinco directivas y solo una instancia de cada opción Directivas. Algunas directivas requieren parámetros adicionales.

1. En **Configuración de directiva**, seleccione el vínculo **+ Agregar directiva (máx. 5)** .
1. En el cuadro **Nombre**, escriba el nombre de la asignación de directiva (límite de 50 caracteres).
1. En el cuadro de lista **Directivas**, seleccione la directiva de Azure que se aplicará a los recursos creados por la aplicación administrada en la suscripción del cliente.
1. En el cuadro **Parámetros de la directiva**, proporcione el parámetro en el que se deben aplicar las directivas de configuración de auditoría y diagnóstico.
1. En el cuadro de lista **SKU de directiva**, seleccione el tipo de SKU de directiva.

    > [!NOTE]
    > La SKU de _directiva Estándar_ es obligatoria para las directivas de auditoría.

## <a name="view-your-plans"></a>Visualización de los planes

- Seleccione **Guardar borrador** y, luego, en la parte superior izquierda de la página, seleccione **Plan overview** (Información general del plan) para volver a la página **Plan overview** (Información general del plan).

Después de crear uno o varios planes, verá el nombre, el identificador y el tipo del plan, la disponibilidad (pública o privada), el estado de publicación actual y las acciones disponibles en la pestaña **Plan overview** (Información general del plan).

Las acciones que están disponibles en la columna **Acción** de la pestaña **Información general del plan** varían en función del estado del plan y pueden incluir lo siguiente:

- Si el estado del plan es **Borrador**, el vínculo de la columna **Acción** indicará **Eliminar borrador**.
- Si el estado del plan es **Activo**, el vínculo de la columna **Acción** será **Stop sell plan** (Dejar de vender el plan) o **Sync private audience** (Sincronizar audiencia privada). El vínculo **Sync private audience** (Sincronizar público privado) publicará solo los cambios para el público privado, sin publicar ninguna otra actualización que haya podido realizar en la oferta.
- Para crear otro plan para esta oferta, en la parte superior de la pestaña **Información general del plan**, seleccione **+ Crear nuevo plan**. Después, repita los pasos descritos en [Creación de planes para la oferta de Aplicación de Azure](azure-app-plans.md). En caso contrario, si ha terminado de crear planes, vaya a la sección siguiente: Pasos siguientes.

## <a name="next-steps"></a>Pasos siguientes

- [Pruebe y publique una oferta de aplicación de Azure](azure-app-test-publish.md).
- [Venda una oferta de aplicación de Azure](azure-app-marketing.md) mediante los programas **Venta conjunta con Microsoft** y **Reventa a través de CSP**.