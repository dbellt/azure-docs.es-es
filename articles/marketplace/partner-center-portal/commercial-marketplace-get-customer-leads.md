---
title: Administración de clientes potenciales en el marketplace comercial de Microsoft
description: Aprenda a generar y recibir clientes potenciales a partir de sus ofertas de Microsoft AppSource y Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: trkeya
ms.author: trkeya
ms.date: 05/20/2021
ms.openlocfilehash: bfb900dd2be2d83c46663b21773bb14585129be9
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110453097"
---
# <a name="customer-leads-from-your-commercial-marketplace-offer"></a>Clientes potenciales a partir de la oferta en el marketplace comercial

Los clientes potenciales son clientes interesados en sus ofertas o que las implementan desde [Microsoft AppSource](https://appsource.microsoft.com) y [Azure Marketplace](https://azuremarketplace.microsoft.com). Puede recibir clientes potenciales después de publicar la oferta en el marketplace comercial. En este artículo se explican los siguientes conceptos relacionados con la administración de clientes potenciales:

* El modo en que su oferta de marketplace comercial genera clientes potenciales para garantizar que no pierde oportunidades de negocio. 
* Cómo conectar su sistema de administración de relaciones con clientes (CRM) a su oferta para que pueda administrar los clientes potenciales en una ubicación central.
* Los datos de clientes potenciales que le enviamos para que pueda hacer un seguimiento de los clientes que se comunican con usted.

## <a name="generate-customer-leads"></a>Generar clientes potenciales

Estos son los lugares en los que se genera un cliente potencial:

- Un cliente acepta compartir su información tras seleccionar **Ponerse en contacto conmigo** en el marketplace comercial. Este es un cliente potencial de *interés inicial*. Compartimos información con usted sobre el cliente que ha expresado interés en obtener su producto. El cliente potencial se encuentra en la parte superior del embudo que desemboca en la adquisición.

    ![Contacto de Dynamics 365](./media/commercial-marketplace-get-customer-leads/dynamics-365-contact-me.png)

- Un cliente selecciona **Obténgalo ahora** (o selecciona **Crear** en [Azure Portal](https://portal.azure.com/)) para conseguir la oferta. Este es un cliente potencial *activo*. Compartimos información con usted sobre el cliente que ha empezado a implementar su producto.

    ![Botón Obténgalo ahora de SQL](./media/commercial-marketplace-get-customer-leads/sql-get-it-now.png)

    ![Botón Crear de Windows Server](./media/commercial-marketplace-get-customer-leads/windows-server-create.png)

- Un cliente selecciona **Versión de prueba** o **Prueba gratuita** para probar su oferta. Las versiones de prueba o evaluaciones gratuitas representan vías rápidas para compartir su negocio al instante con clientes potenciales sin barreras de entrada.

    ![Botón Versión de prueba de Dynamics 365](./media/commercial-marketplace-get-customer-leads/dynamics-365-test-drive.png)

    ![Botón Prueba gratuita de Dynamics 365](./media/commercial-marketplace-get-customer-leads/dynamics-365-free-trial.png)

## <a name="connect-to-your-crm-system"></a>Conectarse a su sistema CRM

[!INCLUDE [Connect lead management](../includes/customer-leads.md)]

## <a name="understand-lead-data"></a>Descripción de los datos de clientes potenciales

Todos los clientes potenciales que recibe durante el proceso de adquisición de clientes tienen datos en campos específicos. El primer campo donde buscar es `LeadSource`, que sigue este formato: **Origen-Acción** | **Oferta**.

**Orígenes**: El valor de este campo se rellena en función del Marketplace que generó el cliente potencial. Los valores posibles son `"AzureMarketplace"`, `"AzurePortal"` y `"AppSource (SPZA)"`.

**Acciones**: El valor de este campo se rellena en función de la acción realizada por el cliente en el marketplace que generó el cliente potencial.

Los valores posibles son:

- **"INS"** : significa *instalación*. Esta acción se encuentra en Azure Marketplace o AppSource cuando un cliente adquiere su producto.
- **"PLT"** : significa *prueba dirigida por asociado*. Esta acción se encuentra en AppSource cuando un cliente selecciona la opción **Ponerse en contacto conmigo**.
- **"DNC"** : significa *no ponerse en contacto conmigo*. Esta acción se encuentra en AppSource cuando se solicita a un asociado al que se hace referencia cruzada en la página de la aplicación el establecimiento de contacto. Compartimos una notifican que indica que este cliente tenía una referencia cruzada en la aplicación, pero no es necesario establecer contacto con él.
- **"Create"** : esta acción aparece solo en Azure Portal y se genera cada vez que un cliente adquiere su oferta para su cuenta.
- **"StartTestDrive"** : esta acción solo está destinada a la opción **Versión de prueba** y se genera cuando un cliente inicia su versión de prueba.

**Ofertas**: Es posible que tenga varias ofertas en el marketplace comercial. El valor de este campo se rellena en función de la oferta que generó el cliente potencial. El identificador del publicador y el de la oferta se envían en este campo y son los valores que usted proporcionó al publicar la oferta en el marketplace.

A continuación se muestran los valores en el formato esperado `publisherid.offerid`: 

- `checkpoint.check-point-r77-10sg-byol`
- `bitnami.openedxcypress`
- `docusign.3701c77e-1cfa-4c56-91e6-3ed0b622145`

## <a name="customer-information"></a>Información del usuario

La información del cliente se envía a través de varios campos. En el ejemplo siguiente se muestra la información del cliente que se encuentra en un cliente potencial:

- Nombre: John
- Apellidos: Smith
- Correo electrónico: jsmith\@microsoft.com
- Teléfono: 1234567890
- País: US
- Compañía: Microsoft
- Título: CTO

>[!NOTE]
>No todos los datos del ejemplo anterior están siempre disponibles para cada cliente potencial. Puesto que obtendrá clientes potenciales a partir de distintos pasos, tal como se ha mencionado en la sección "Generar clientes potenciales", la mejor manera de administrarlos es desduplicar los registros y personalizar los seguimientos. De este modo, cada cliente obtiene un mensaje adecuado y se entabla una relación única.

## <a name="best-practices-for-lead-management"></a>Procedimientos recomendados para la administración de clientes potenciales

Aquí le presentamos algunas recomendaciones para generar clientes potenciales a lo largo del ciclo de ventas:

- **Proceso**: defina un proceso de ventas claro, con hitos, análisis y una clara propiedad del equipo.
- **Calificación**: defina los requisitos previos, que indican si un cliente potencial se ha calificado por completo. Asegúrese de que los representantes de ventas o marketing califiquen a los clientes potenciales con cuidado antes de hacerlos pasar por el proceso de ventas completo.
- **Seguimiento**: no olvide hacer un seguimiento en 24 horas. Incluya al cliente potencial en la herramienta CRM de su elección inmediatamente después de que el cliente implemente la versión de prueba. Envíele un correo electrónico mientras aún tenga el recuerdo reciente. Solicite la programación de una llamada telefónica para comprender mejor si el producto es una buena solución para su problema. Cabe esperar que la transacción típica requiera muchas llamadas de seguimiento.
- **Consolidación**: consolide a los clientes potenciales para encaminarse hacia un aumento del margen de ganancias. Póngase en contacto con ellos, pero no los bombardee. Le recomendamos que les envíe un correo electrónico varias veces antes de cerrarlos: no abandone al primer intento. Recuerde que estos clientes conocieron directamente el producto y emplearon un tiempo en una evaluación gratuita. Son posibles interesados.

Una vez implementada la configuración técnica, incorpore estos clientes potenciales a la estrategia actual de ventas y marketing, así como a los procesos operativos. Estamos interesados en comprender mejor el proceso de ventas global, y queremos trabajar en estrecha colaboración con usted para conseguir clientes potenciales de alta calidad y una cantidad suficiente de datos que le permitan tener éxito. Agradecemos sus comentarios acerca de cómo podemos optimizar y mejorar los clientes potenciales que le enviamos con datos adicionales para obtener los mejores resultados de estos clientes. Díganos si está interesado en [ofrecer comentarios](mailto:AzureMarketOnboard@microsoft.com) y sugerencias para ayudar a que su equipo de ventas tenga mayor éxito con los clientes potenciales del marketplace comercial.

## <a name="next-steps"></a>Pasos siguientes

- [Preguntas más frecuentes y solución de problemas de administración de clientes potenciales](../lead-management-faq.md)