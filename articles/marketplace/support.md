---
title: Obtención de soporte técnico para el programa de marketplace comercial en el Centro de partners
description: Obtenga información sobre las opciones de soporte técnico para el programa de marketplace comercial en el Centro de partners, incluida la forma de presentar una solicitud de soporte técnico.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: navits09
ms.author: navits
ms.date: 01/19/2020
ms.openlocfilehash: cfe365af868f01b868e737a409724e92f1d57f81
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107106996"
---
# <a name="support-for-the-commercial-marketplace-program-in-partner-center"></a>Soporte técnico para el programa de marketplace comercial en el Centro de partners

Microsoft ofrece soporte para una amplia variedad de productos y servicios. Encontrar el equipo de soporte técnico correcto es importante para garantizar una respuesta oportuna y adecuada. Considere los siguientes escenarios que le ayudarán a dirigir su consulta al equipo adecuado:

- Si es anunciante y tiene una pregunta de un cliente, indíquele que solicite soporte técnico a través de los vínculos de soporte técnico en  [Azure Portal](https://portal.azure.com/).
- Si es publicador y detectó una incidencia de seguridad con una aplicación que se ejecuta en Azure, consulte [Procedimientos para registrar una incidencia de soporte técnico de un evento de seguridad](../security/fundamentals/event-support-ticket.md). Los anunciantes deben notificar lo antes posible los eventos de seguridad sospechosos, incluidos incidentes y vulnerabilidades de seguridad de sus ofertas de software y servicio de Azure Marketplace.
- Si es anunciante y tiene una pregunta relacionada con la aplicación o el servicio, revise las siguientes opciones de soporte técnico.

## <a name="get-help-or-open-a-support-ticket"></a>Obtención de ayuda o apertura de una incidencia de soporte técnico

1. Inicia sesión con la cuenta profesional. Si aún no lo ha hecho, tendrá que [crear una cuenta del Centro de partners](create-account.md).

1. En el menú que se encuentra en la parte superior derecha de la página, seleccione el icono **Soporte técnico**. El panel **Ayuda y soporte técnico** aparece en el lado derecho de la página.

1. Para ayuda con el marketplace comercial, seleccione **Marketplace comercial**.

   ![Menú desplegable de soporte técnico](./media/support/commercial-marketplace-support-pane.png)

1. En el cuadro **Problem summary** (Resumen del problema), escriba una descripción breve de la incidencia.

1. En el cuadro **Tipo de problema**, haga una de las acciones siguientes:

    - **Opción 1**: Escriba palabras clave como: Marketplace, aplicación de Azure, oferta de SaaS, administración de cuentas, administración de clientes potenciales, problema de implementación, pago o migración de ofertas de venta conjunta. Luego, seleccione un tipo de problema en la lista de recomendaciones que aparece.

    - **Opción 2**: Seleccione **Examinar temas** en la lista **Categoría** y, luego, seleccione **Marketplace comercial**. Luego, seleccione el **Tema** y **Subtema** adecuados.

1. Una vez que haya encontrado el tema que desea, seleccione **Revisar soluciones**.

    ![Paso siguiente](./media/support/next-step.png)

Se muestran las opciones siguientes:

- Para seleccionar otro tema, haga clic en **Select a different issue** (Seleccionar otra incidencia).
- Para ayudar a resolver el problema, revise los pasos y documentos recomendados, si está disponible.

    ![Soluciones recomendadas](./media/support/recommended-solutions.png)

Si no encuentra la respuesta que busca en la autoayuda, seleccione **Provide issue details** (Proporcionar detalles del problema). Complete todos los campos obligatorios para acelerar el proceso de resolución y seleccione **Enviar**.

>[!Note]
>Si no ha iniciado sesión en el Centro de partners, puede que tenga que iniciar sesión para poder crear un vale.

## <a name="track-your-existing-support-requests"></a>Realizar un seguimiento de las solicitudes de soporte técnico existentes

Para revisar los vales abiertos y cerrados, seleccione **Marketplace comercial** > **Soporte técnico** en el menú de navegación de la izquierda.

## <a name="record-issue-details-with-a-har-file"></a>Registro de los detalles del problema con un archivo HAR

Para ayudar a los agentes de soporte técnico a solucionar el problema, considere la posibilidad de adjuntar un archivo con formato de almacenamiento HTTP (HAR) a la incidencia de soporte técnico. HAR files son registros de solicitudes de red en un explorador web.

> [!WARNING]
> Los archivos HAR pueden registrar datos confidenciales acerca de su cuenta del Centro de partners.

### <a name="microsoft-edge-and-google-chrome"></a>Microsoft Edge o Google Chrome

Para generar un archivo HAR mediante **Microsoft Edge** o **Google Chrome**:

1. Vaya a la página web donde está experimentando el problema.
2. En la esquina superior derecha de la ventana, seleccione el icono de puntos suspensivos y, a continuación, **Más herramientas** > **Herramientas de desarrollo**. Puede presionar F12 como método abreviado de teclado.
3. En el panel Herramientas de desarrollo, seleccione la pestaña **Red**.
4. Seleccione **Stop recording network log** (Dejar de grabar el registro de red) y **Borrar** para quitar los registros existentes. El icono de registro cambiará a gris.

    ![Cómo quitar registros existentes en Microsoft Edge o Google Chrome](media/support/chromium-stop-clear-session.png)

5. Seleccione **Record network log** (Grabar el registro de red) para iniciar la grabación. Al iniciar la grabación, el icono de registro se volverá rojo.

    ![Cómo iniciar la grabación en Microsoft Edge o Google Chrome](media/support/chromium-start-session.png)

6. Reproduzca el problema que desea solucionar.
7. Una vez que haya reproducido el problema, seleccione **Stop recording network log** (Dejar de grabar el registro de red).
8. Seleccione **Exportar HAR**, marcado con un icono de flecha hacia abajo y guarde el archivo.

    ![Cómo exportar un archivo HAR en Microsoft Edge o Google Chrome](media/support/chromium-network-export-har.png)

### <a name="mozilla-firefox"></a>Mozilla Firefox

Para generar un archivo HAR mediante **Mozilla Firefox**:

1. Vaya a la página web donde está experimentando el problema.
1. En la esquina superior derecha de la ventana, seleccione el icono de puntos suspensivos y, a continuación, **Desarrollador web** > **Alternar herramientas**. Puede presionar F12 como método abreviado de teclado.
1. Seleccione la pestaña **Red** y **Limpiar** para quitar los registros existentes.

    ![Cómo quitar los registros existentes en Mozilla Firefox](media/support/firefox-clear-session.png)

1. Reproduzca el problema que desea solucionar.
1. Una vez que haya reproducido el problema, seleccione la opción de **exportación o importación de HAR** > **Guardar todo como HAR**.

    ![Cómo exportar un archivo HAR en Mozilla Firefox](media/support/firefox-network-export-har.png)

### <a name="apple-safari"></a>Apple Safari

Para generar un archivo HAR mediante **Safari**:

1. Habilite las herramientas de desarrollo en Safari: seleccione **Safari** > **Preferencias**. Diríjase a la pestaña **Avanzadas** y, a continuación, seleccione **Mostrar menú Desarrollar en la barra de menús**.
1. Vaya a la página web donde está experimentando el problema.
1. Seleccione **Desarrollar** y, a continuación, seleccione **Mostrar inspector web**.
1. Seleccione la pestaña **Red** y **Borrar elementos de red** para quitar los registros existentes.

    ![Cómo quitar los registros existentes en Safari](media/support/safari-clear-session.png)

1. Reproduzca el problema que desea solucionar.
1. Una vez que haya reproducido el problema, seleccione **Exportar** y guarde el archivo.

    ![Cómo exportar un archivo HAR en Safari](media/support/safari-network-export-har.png)

## <a name="next-steps"></a>Pasos siguientes

- [Actualización de una oferta existente en Marketplace comercial](partner-center-portal/update-existing-offer.md)