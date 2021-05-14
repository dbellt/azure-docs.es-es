---
title: 'Configuración de un servicio QnA Maker: QnA Maker'
description: En este documento se describen las configuraciones avanzadas de los recursos de QnA Maker.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 02/18/2021
ms.openlocfilehash: 48a56500646e4bd5e24cdcf3e076f46f1de94741
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108743910"
---
# <a name="configure-qna-maker-resources"></a>Configuración de recursos de QnA Maker

El usuario puede configurar QnA Maker para usar distintos recursos de Cognitive Search. También puede configurar las opciones de App Service si usa QnA Maker GA.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

## <a name="configure-qna-maker-to-use-different-cognitive-search-resource"></a>Configuración de QnA Maker para usar distintos recursos de Cognitive Search

Si crea un servicio QnA y sus dependencias (como, Search) en el portal, se creará el servicio Search de inmediato y se vinculará al servicio QnA Maker. Después de crear estos recursos puede actualizar el valor de App Service para usar un servicio Search existente y eliminar el que acaba de crear.

El recurso **App Service** de QnA Maker utiliza el recurso Cognitive Search. Para cambiar el recurso de Cognitive Search que usa QnA Maker, debe cambiar la configuración en Azure Portal.

1. Obtenga la **clave de administración** y el **nombre** del recurso de Cognitive Search que desea que utilice QnA Maker.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y busque el recurso de **App Service** asociado al recurso de QnA Maker. Ambos tienen el mismo nombre.

1. Seleccione **Configuración** y, después, **Configuración**. Se mostrarán todos los valores existentes de la instancia de App Service de QnA Maker.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de Azure Portal que muestra los valores de configuración de App Service](../media/qnamaker-how-to-upgrade-qnamaker/change-search-service-app-service-configuration.png)

1. Cambie los valores de las siguientes claves:

    * **AzureSearchAdminKey**
    * **AzureSearchName**

1. Para usar la nueva configuración, debe reiniciar App Service. Seleccione **Información general** y, después, **Restaurar**.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de Azure Portal con el reinicio de App Service después de cambiar la configuración](../media/qnamaker-how-to-upgrade-qnamaker/screenshot-azure-portal-restart-app-service.png)

Si crea un servicio QnA mediante plantillas de Azure Resource Manager, puede crear todos los recursos y controlar la creación de App Service para usar un servicio Search existente.

Aprenda más sobre cómo realizar la [configuración de la aplicación](../../../app-service/configure-common.md#configure-app-settings) de App Service.

## <a name="get-the-latest-runtime-updates"></a>Obtención de las últimas actualizaciones del runtime

El runtime de QnAMaker forma parte de la instancia de Azure App Service que se implementa al [crear un servicio QnAMaker](./set-up-qnamaker-service-azure.md) en Azure Portal. El sistema de tiempo de ejecución se actualiza periódicamente. La instancia de App Service en QnA Maker está en modo de actualización automática tras la versión de extensión de sitio de abril de 2019 (versión 5+). Esta actualización está diseñada para administrar el tiempo de inactividad CERO durante las actualizaciones.

Puede consultar su versión actual en https://www.qnamaker.ai/UserSettings. Si su versión es anterior a la 5.x, debe reiniciar App Service para aplicar las actualizaciones más recientes:

1. Vaya a su servicio QnAMaker (grupo de recursos) en [Azure Portal](https://portal.azure.com).

    > [!div class="mx-imgBorder"]
    > ![Grupo de recursos de QnAMaker en Azure](../media/qnamaker-how-to-troubleshoot/qnamaker-azure-resourcegroup.png)

1. Seleccione la instancia de App Service y abra la sección **Información general**.

    > [!div class="mx-imgBorder"]
    > ![Instancia de App Service en QnAMaker](../media/qnamaker-how-to-troubleshoot/qnamaker-azure-appservice.png)


1. Reinicie App Service. El proceso de actualización debería finalizar en un par de segundos. Todas las aplicaciones o bots dependientes que usen este servicio QnAMaker dejarán de estar disponibles para los usuarios finales durante el periodo de reinicio.

    ![Reinicie la instancia de App Service en QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-appservice-restart.png)

## <a name="configure-app-service-idle-setting-to-avoid-timeout"></a>Configuración del valor de inactividad de App Service para evitar el tiempo de espera

El servicio de aplicaciones, que proporciona el entorno de ejecución de predicción de QnA Maker para una base de conocimiento publicada, tiene una configuración de tiempo de espera de inactividad que acepta automáticamente como predeterminado el tiempo de espera si el servicio está inactivo. En QnA Maker, esto significa que la API generateAnswer del entorno de ejecución de predicción agota a veces el tiempo de espera después de períodos sin tráfico.

Para mantener cargada la aplicación del punto de conexión de predicción incluso cuando no hay tráfico, establezca el tiempo de inactividad en siempre activo.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque y seleccione la instancia de App Service del recurso de QnA Maker. Tendrá el mismo nombre que el recurso de QnA Maker pero un **tipo** diferente de App Service.
1. Busque **Configuración** y seleccione **Configuración**.
1. En el panel Configuración, seleccione **Configuración general**, busque **Siempre activado** y elija **Activado** como valor.

    > [!div class="mx-imgBorder"]
    > ![En el panel Configuración, seleccionar **Configuración general** luego buscar **Siempre activado** y elegir **Activado** como valor.](../media/qnamaker-how-to-upgrade-qnamaker/configure-app-service-idle-timeout.png)

1. Para guardar la configuración, seleccione **Guardar**.
1. Se le preguntará si desea reiniciar la aplicación para usar la nueva configuración. Seleccione **Continuar**.

Aprenda más sobre cómo realizar la [configuración general](../../../app-service/configure-common.md#configure-general-settings) de App Service.

## <a name="business-continuity-with-traffic-manager"></a>Continuidad del negocio con Traffic Manager

El objetivo principal del plan de continuidad empresarial consiste en crear un punto de conexión de base de conocimiento resistente, que garantizaría que el bot o la aplicación que lo consume no tengan tiempos de inactividad.

> [!div class="mx-imgBorder"]
> ![Plan bcp de QnA Maker](../media/qnamaker-how-to-bcp-plan/qnamaker-bcp-plan.png)

La idea de alto nivel como se representa anteriormente es la siguiente:

1. Configure dos [servicios QnA Maker](set-up-qnamaker-service-azure.md) paralelos en [Regiones emparejadas de Azure](../../../best-practices-availability-paired-regions.md).

1. [Realice una copia de seguridad](../../../app-service/manage-backup.md) del servicio de aplicaciones de QnA Maker principal y [restáurela](../../../app-service/web-sites-restore.md) en la configuración secundaria. De este modo, se asegurará de que ambas configuraciones funcionan con el mismo nombre de host y las mismas claves.

1. Mantenga sincronizados los índices principal y secundario de Azure Search. Use el ejemplo de GitHub [aquí](https://github.com/pchoudhari/QnAMakerBackupRestore) para ver cómo realizar una copia de seguridad de los índices de Azure y cómo restaurarlos.

1. Realice una copia de seguridad de Application Insights con la [exportación continua](../../../azure-monitor/app/export-telemetry.md).

1. Una vez configuradas las pilas principal y secundaria, use [Traffic Manager](../../../traffic-manager/traffic-manager-overview.md) para configurar los dos puntos de conexión y establecer un método de enrutamiento.

1. Debe crear un certificado de Seguridad de la capa de transporte (TLS), conocido anteriormente como Capa de sockets seguros (SSL), para el punto de conexión de Traffic Manager. [Enlace el certificado TLS/SSL](../../../app-service/configure-ssl-bindings.md) en los servicios de aplicaciones.

1. Por último, use el punto de conexión de Traffic Manager del bot o de la aplicación.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker administrado (versión preliminar)](#tab/v2)

## <a name="configure-qna-maker-managed-preview-service-to-use-different-cognitive-search-resource"></a>Configuración de QnA Maker administrado (versión preliminar) para usar distintos recursos de Cognitive Search

Si crea un servicio QnA administrado (versión preliminar) y sus dependencias (como Search) en el portal, se creará el servicio Search de inmediato y se vinculará al servicio QnA Maker administrado (versión preliminar). Después de crear estos recursos, puede actualizar el servicio Search en la pestaña **Configuración**.

1. Vaya al servicio QnA Maker administrado (versión preliminar) en Azure Portal.

1. Seleccione **Configuración** y, después, el servicio de Azure Cognitive Search que desea vincular con el servicio QnA Maker administrado (versión preliminar).

    ![Captura de pantalla de la página de configuración de QnA Maker administrado (versión preliminar)](../media/qnamaker-how-to-upgrade-qnamaker/change-search-service-configuration.png)

1. Haga clic en **Guardar**.

> [!NOTE]
> Si cambia el servicio de Azure Search asociado a QnA Maker, perderá el acceso a todas las knowledge bases que existen en él. Asegúrese de exportar las knowledge bases existentes antes de cambiar el servicio de Azure Search.

---
