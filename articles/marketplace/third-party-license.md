---
title: 'Administración de licencias de aplicaciones de terceros a través de Microsoft: Azure y Microsoft AppSource'
description: Obtenga información sobre cómo administrar licencias de aplicaciones de terceros a través de Microsoft.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: mingshen-ms
ms.author: mingshen
ms.reviewer: dannyevers
ms.date: 04/30/2021
ms.openlocfilehash: 8535d124596b2ce5bd80df0d95f0fb5af5bb68f9
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108759498"
---
# <a name="third-party-app-license-management-through-microsoft"></a>Administración de licencias de aplicaciones de terceros a través de Microsoft: Azure

> [!IMPORTANT]
> Esta funcionalidad se encuentra actualmente en versión preliminar.

Se aplica al siguiente tipo de oferta:

- Dynamics 365 for Customer Engagement & Power Apps

La _administración de licencias de aplicaciones de terceros a través de Microsoft_ permite a los proveedores de software independientes (ISV) que compilan soluciones con el conjunto de productos de Dynamics 365 administrar y aplicar licencias para sus soluciones mediante sistemas proporcionados por Microsoft. Al adoptar este enfoque, puede:

- Permitir que los clientes asignen y desasigne las licencias de la solución mediante herramientas conocidas, como el Centro de administración de Microsoft 365, que usan para administrar licencias de Office y Dynamics.
- Hacer que Power Platform aplique las licencias en tiempo de ejecución para asegurarse de que solo los usuarios con licencia puedan acceder a la solución.
- Ahorrarse el esfuerzo de compilar y mantener su propio sistema de administración y cumplimiento de licencias.


> [!NOTE]
> La administración de licencias de aplicaciones de terceros a través de Microsoft solo está disponible para los ISV que participan en el programa Connect para ISV. Microsoft no participa en la venta de licencias.

## <a name="prerequisites"></a>Requisitos previos

Para administrar las licencias de aplicaciones a través de Microsoft, debe cumplir los siguientes requisitos previos.

1. Tener una [cuenta de Microsoft Partner Network](/partner-center/mpn-create-a-partner-center-account) válida.
1. Estar registrado en el programa de marketplace comercial. Para más información, consulte [Creación de una cuenta de marketplace comercial en el Centro de partners](create-account.md).
1. Estar registrado en el [programa Connect para ISV](https://partner.microsoft.com/solutions/business-applications/isv-overview). Para más información, consulte [Guía de incorporación del programa Connect para fabricantes de software independientes (ISV) de Microsoft Business Applications](business-applications-isv-program.md).
1. El equipo de desarrolladores tiene los entornos de desarrollo y las herramientas necesarias para crear soluciones de Dataverse. La solución de Dataverse debe incluir aplicaciones controladas por modelos (actualmente son el único tipo de componentes de solución que se admite a través de la característica de administración de licencias).

## <a name="high-level-process"></a>Proceso de alto nivel

En esta tabla se muestra el proceso de alto nivel para administrar aplicaciones de terceros a través de Microsoft:

| Paso | Detalles |
| ------------ | ------------- |
| Paso 1: crear oferta | El ISV crea una oferta en el Centro de partners y decide administrar las licencias de esta oferta a través de Microsoft. Esto incluye la definición de uno o varios planes de licencia para la oferta. Para más información, vea [Creación de una oferta de Dynamics 365 for Customer Engagement & Power Apps en Microsoft AppSource](dynamics-365-customer-engage-offer-setup.md). |
| Paso 2: actualizar paquete | El ISV crea un paquete de solución para la oferta que incluye información del plan de licencia como metadatos y la carga en el Centro de partners para su publicación en Microsoft AppSource. Para más información, consulte [Incorporación de metadatos de licencia a la solución](/powerapps/developer/data-platform/appendix-add-license-information-to-your-solution). |
| Paso 3: comprar licencias | Los clientes detectan la oferta del ISV en AppSource o directamente en el sitio web del ISV. Los clientes compran licencias para los planes que desean directamente desde el ISV (estas ofertas no se pueden comprar a través de AppSource en este momento). |
| Paso 4: registrar oferta | El ISV registra la compra con Microsoft en el Centro de partners. Como parte del [registro de la oferta](/partner-center/csp-commercial-marketplace-licensing#register-isv-connect-deal-in-deal-registration), el ISV especificará el tipo y la cantidad de cada plan de licencias adquirido por el cliente. |
| Paso 5: administrar licencias | Los planes de licencia aparecerán en el Centro de administración de Microsoft 365 para que el cliente los [asigne a los usuarios o grupos](/microsoft-365/commerce/licenses/manage-third-party-app-licenses) de su organización. El cliente también puede instalar la aplicación en su inquilino a través del Centro de administración de Power Platform. |
| Paso 6: realizar la comprobación de la licencia | Cuando un usuario de la organización del cliente intenta ejecutar una aplicación, Microsoft comprueba que el usuario tiene una licencia antes de permitir que la ejecute. Si no tiene una licencia, el usuario ve un mensaje que explica que necesita ponerse en contacto con un administrador para obtener una licencia. |
| Paso 7: ver informes | Los ISV pueden ver información sobre las licencias aprovisionadas y asignadas durante un período de tiempo y por geografía. |
|||

## <a name="enabling-app-license-management-through-microsoft"></a>Habilitación de la administración de licencias de aplicaciones a través de Microsoft

Al crear una oferta, hay dos casillas en la pestaña Configuración de la oferta que se usan para habilitar la administración de licencias de aplicaciones en una oferta.

### <a name="enable-app-license-management-through-microsoft-check-box"></a>Casilla Habilitar la administración de licencias de aplicaciones a través de Microsoft

Funcionamiento:

- Después de seleccionar el cuadro **Habilitar la administración de licencias de aplicaciones a través de Microsoft**, puede definir planes de licencias para su oferta.
- Los clientes verán un botón **Obtenerla ahora** en la página de descripción de la oferta en AppSource. Los clientes pueden seleccionar este botón para ponerse en contacto con usted para comprar licencias para la aplicación.

### <a name="allow-customers-to-install-my-app-even-if-licenses-are-not-assigned-check-box"></a>Casilla Permitir que los clientes instalen mi aplicación incluso si no se han asignado licencias

Después de seleccionar el primer cuadro, aparece la casilla **Permitir que los clientes instalen mi aplicación incluso si no se han asignado licencias**. Esta opción es útil si está empleando una estrategia de licencias "freemium" por la que quiere ofrecer algunas características básicas de la solución de forma gratuita a todos los usuarios y cobrar por las características prémium. Por el contrario, si desea asegurarse de que solo los inquilinos que poseen actualmente licencias para el producto puedan descargarlo desde AppSource, no seleccione esta opción.

> [!NOTE]
> Si activa esta opción, debe configurar el paquete de la solución para que no exija una licencia.

Funcionamiento:

- Todos los usuarios de AppSource verán el botón **Obtenerla ahora** en la página de descripción de la oferta junto con el botón **Ponerse en contacto conmigo** y se les permitirá descargar e instalar la oferta.
- Si no selecciona esta opción, AppSource comprueba que el inquilino del usuario tiene al menos una licencia para la solución antes de mostrar el botón **Obtenerla ahora**. Si no hay ninguna licencia en el inquilino del usuario, solo se muestra el botón **Ponerse en contacto conmigo**.

Para más información sobre cómo configurar una oferta, consulte [Creación de una oferta de Dynamics 365 for Customer Engagement & Power Apps](dynamics-365-customer-engage-offer-setup.md).

## <a name="offer-listing-page-on-appsource"></a>Página de descripción de la oferta en AppSource

Una vez publicada la oferta, las opciones que elija accionarán los botones que se mostrarán a un usuario. En esta captura de pantalla se muestra una página de descripción de la oferta en AppSource con los botones **Obtenerla ahora** y **Ponerse en contacto conmigo**.

:::image type="content" source="./media/third-party-license/f365.png" alt-text="Captura de pantalla de una página de descripción de la oferta en AppSource. Se muestran los botones Obtenerla ahora y Ponerse en contacto conmigo":::.

***Ilustración 1: página de descripción de la oferta en Microsoft AppSource***

## <a name="next-steps"></a>Pasos siguientes

- [Planificación de una oferta de Dynamics 365](marketplace-dynamics-365.md)
- [Creación de una oferta de Dynamics 365 for Customer Engagement & Power Apps](dynamics-365-customer-engage-offer-setup.md)
