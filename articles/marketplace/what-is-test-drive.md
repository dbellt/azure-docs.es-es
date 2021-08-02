---
title: ¿Qué es una versión de prueba en Microsoft AppSource?
description: Explicación de la característica de versión de prueba de Microsoft AppSource
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: trkeya
ms.author: trkeya
ms.date: 12/08/2020
ms.openlocfilehash: 5fba64beff10f59be42776cc87cf6f37922295dd
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111949065"
---
# <a name="what-is-a-test-drive"></a>¿Qué es una versión de prueba?

Una versión de prueba es una excelente manera de presentar la oferta a clientes potenciales, ya que se les brinda la oportunidad de probar antes de comprar, lo que aumenta la conversión y la generación de clientes potenciales altamente cualificados. Una versión de prueba hará que su producto cobre vida en un escenario de implementación real. Los clientes que prueben su producto demostrarán una clara intención de comprar una solución similar. Use las ventajas que le ofrece esta opción realizando el seguimiento de los clientes potenciales más avanzados.

Asimismo, los clientes también podrán usar una versión de prueba. Al permitirles probar el producto en primer lugar, se reduce la fricción del proceso de compra. Además, la versión de prueba está aprovisionada previamente; es decir, los clientes no tienen que descargar, instalar o configurar el producto.

## <a name="how-does-it-work"></a>¿Cómo funciona?

Las versiones de prueba son instancias administradas que inician la solución o aplicación a petición para los clientes que la solicitan. Una vez asignada una instancia de la versión de prueba, el cliente podrá usarla durante un período establecido. Una vez finalizado ese período, se elimina para que otro cliente pueda usarla.

Como publicador, puede administrar y configurar las opciones de la versión de prueba en el Centro de partners. Los detalles de la configuración técnica varían según el tipo de oferta. Para obtener instrucciones detalladas, consulte la [Configuración técnica de la versión de prueba](./test-drive-technical-configuration.md).

Los clientes potenciales detectan la versión de prueba como una CTA de la oferta en [AppSource](https://appsource.microsoft.com/en-US/). Proporcionan su información de contacto y aceptan los términos y la directiva de privacidad de la oferta; a continuación, obtienen acceso al entorno preconfigurado para probarlo durante un período de tiempo fijo. Los clientes reciben una evaluación gratuita práctica y guiada de las principales características y ventajas del producto y, a cambio, usted obtiene un valioso cliente potencial.

## <a name="types-of-test-drives"></a>Tipos de versiones de prueba

Hay diferentes versiones de prueba disponibles en el marketplace comercial para ofertas concretas según el tipo de producto, el escenario y el marketplace en el que se encuentre:

- Azure Resource Manager
    - Aplicaciones de Azure
    - SaaS
    - Virtual Machines
- Versión de prueba hospedada
    - Dynamics 365 for Business central (actualmente no se admite)
    - Dynamics 365 for Customer Engagement
    - Dynamics 365 for Operations
- Aplicación lógica (solo en modo de soporte técnico)
- Power BI

Para obtener más información sobre la configuración de una de estas versiones de prueba, consulte [Configuración técnica de la versión de prueba](./test-drive-technical-configuration.md). 

### <a name="azure-resource-manager-test-drive"></a>Versión de prueba de Azure Resource Manager

Esta plantilla de implementación contiene todos los recursos de Azure que componen la solución. Los productos que se ajustan a este escenario usan solo recursos de Azure. La versión de prueba de Azure Resource Manager está disponible para estos tipos de oferta: 

- Aplicaciones de Azure
- SaaS
- Máquinas virtuales

>[!NOTE]
>Es la única opción de versión de prueba para las ofertas de máquina virtual o de aplicación de Azure.

### <a name="hosted-test-drive-recommended"></a>Versión de prueba hospedada (recomendada)

Una versión de prueba hospedada elimina la complejidad de la configuración ya que deja que Microsoft hospede y mantenga el servicio que realiza las tareas de aprovisionamiento y desaprovisionamiento de los usuarios de la versión de prueba. Si tiene una oferta en Microsoft AppSource, compile la versión de prueba para conectarse con una instancia de Dynamics AX/CRM. Puede usar los siguientes tipos de oferta de AppSource:

- Use [Dynamics 365 for Customer Engagement y Power Apps](dynamics-365-customer-engage-offer-setup.md) para un sistema de involucración del cliente como los de ventas, servicio, servicio de proyecto y servicio de campo.
- Use [Dynamics 365 for Operations](./dynamics-365-operations-offer-setup.md) para un sistema de planeamiento de recursos empresariales de finanzas y operaciones, como finanzas, operaciones y fabricación, cadena de suministro.

### <a name="logic-app-test-drive"></a>Versión de prueba de aplicación lógica

Este tipo de versión de prueba no lo hospeda Microsoft y usa plantillas Azure Resource Manager (ARM) para los tipos de oferta de Dynamics AX/CRM. Tendrá que ejecutar la plantilla de ARM para crear los recursos necesarios en su suscripción de Azure. La versión de prueba de la aplicación lógica solo está disponible en el modo de compatibilidad y Microsoft no la recomienda. Para obtener más información sobre la configuración de la versión de prueba de una aplicación lógica, consulte [Configuración técnica de la versión de prueba](./test-drive-technical-configuration.md).

### <a name="power-bi-test-drive"></a>Versión de prueba de Power BI

Se trata simplemente de un vínculo insertado en un panel personalizado. Este tipo de versión de prueba lo deben usar los productos que solo demuestren un objeto visual interactivo de Power BI.

## <a name="transforming-examples"></a>Ejemplos de transformación

El proceso de conversión de una arquitectura de recursos en una versión de prueba puede ser abrumador. Consulte estos ejemplos de cómo transformar mejor las arquitecturas actuales.

[Transformación de una plantilla de sitio web en una versión de prueba](https://github.com/Azure/AzureTestDrive/wiki/Transforming-Website-Deployment-Template-for-Test-Drive)

[Transformación de una plantilla de máquina virtual en una versión de prueba](https://github.com/Azure/AzureTestDrive/wiki/Transforming-Virtual-Machine-Deployment-Template-for-Test-Drive)

[Transformación de una plantilla de Resource Manager existente en una versión de prueba](https://github.com/Azure/AzureTestDrive/wiki/Deploying-Existing-Solutions)

## <a name="generate-leads-from-your-test-drive"></a>Generación de clientes potenciales a partir de la versión de prueba

Una versión de prueba del marketplace comercial es una herramienta excelente para los vendedores. Le recomendamos que la incorpore durante al lanzamiento a sus esfuerzos de comercialización para generar más clientes potenciales para el negocio. Para obtener instrucciones detalladas, vea [Clientes potenciales a partir de la oferta en el marketplace comercial](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/marketplace/partner-center-portal/commercial-marketplace-get-customer-leads.md).

Si cierra un trato con un cliente potencial de una versión de prueba, no olvide registrarlo en [Microsoft Partner Sales Connect](https://support.microsoft.com/help/3155788/getting-started-with-microsoft-partner-sales-connect). Además, nos encantaría conocer sus éxitos con los clientes en los que la versión de prueba ha desempeñado un papel importante.

## <a name="other-resources"></a>Otros recursos

Recursos adicionales de la versión de prueba:

- [Prácticas recomendadas de la versión de prueba](https://github.com/Azure/AzureTestDrive/wiki/Test-Drive-Best-Practices)
- [Información general](https://assetsprod.microsoft.com/mpn/azure-marketplace-appsource-test-drives.pdf) (PDF, asegúrese de que el bloqueador de elementos emergentes está desactivado)

## <a name="next-steps"></a>Pasos siguientes

- [Configuración técnica de la versión de prueba](test-drive-technical-configuration.md)