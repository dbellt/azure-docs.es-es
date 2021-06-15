---
title: P+F sobre bases de datos MySql ClearDB con Azure App Service
description: Respuestas a preguntas habituales acerca del uso de las bases de datos MySQL ClearDB con Azure App Service.
documentationcenter: php
services: mysql
author: sunbuild
manager: yochayk
tags: mysql
ms.service: multiple
ms.topic: article
ms.date: 10/27/2016
ms.author: sumuth
ROBOTS: NOINDEX,NOFOLLOW
ms.custom: references_regions
ms.openlocfilehash: 51120f70977a1ff09a30ae6418dc622c77ff5797
ms.sourcegitcommit: 070122ad3aba7c602bf004fbcf1c70419b48f29e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111437953"
---
# <a name="faq-for-cleardb-mysql-databases-with-azure-app-service"></a>P+F sobre bases de datos MySql ClearDB con Azure App Service
Estas P+F responden a preguntas comunes sobre el uso y la adquisición de bases de datos MySQL ClearDB para Azure Web Apps.

> [!IMPORTANT]
> A partir del 13 de junio de 2018, ClearDB ha pasado a los clientes basados en Azure que actualmente son facturados por Microsoft a un modelo de facturación directa con ClearDB. La información de este artículo está ahora obsoleta. Ya no podrá crear ni actualizar una base de datos de ClearDB que se creara en Azure.
>
> Para más información y los siguientes pasos, consulte [Changes to ClearDB service plans](https://www.navisite.com/blog/news/update-change-of-billing-notice-for-all-azure-cleardb-service-plans/) (Cambios de los planes de servicio de ClearDB).

## <a name="what-options-do-i-have-for-mysql-on-azure"></a>¿Qué opciones tengo para MySQL en Azure?
Consulte [ClearDB](https://w2.cleardb.net/) para obtener la información más reciente sobre ese servicio. ClearDB es un servicio de hospedaje de MySQL que administra la infraestructura de MySQL en lugar del usuario. 

Tiene algunas otras opciones para el hospedaje de MySQL en Azure:
* [Azure Database for MySQL](https://azure.microsoft.com/services/mysql/)
* [Clúster de MySQL en ejecución en una máquina virtual de Azure](https://github.com/azure/azure-quickstart-templates/tree/master/mysql-replication)
* [Instancia única de MySQL en ejecución en una máquina virtual de Azure](/previous-versions/azure/virtual-machines/windows/classic/mysql-2008r2?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)


## <a name="do-i-need-a-credit-card-for-the-web-app--mysql-template-in-the-azure-marketplace"></a>¿Necesito una tarjeta de crédito para la aplicación web + plantilla MySQL en Azure Marketplace?
Depende del tipo de suscripción que esté utilizando. Estos son algunos tipos de suscripción de uso frecuente:

* [Pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/): requiere una tarjeta de crédito. Al adquirir una base de datos MySQL de pago, se cargará el importe correspondiente en su tarjeta de crédito.
* [Prueba gratuita](https://azure.microsoft.com/pricing/free-trial/): incluye créditos para usarlos con los servicios de Microsoft Azure pero no permite la adquisición de recursos de terceros. Para adquirir servicios de terceros o una base de datos MySQL de pago, tiene que usar una suscripción con tarjeta de crédito habilitada. Para Web Apps, puede crear una base de datos MySQL ClearDB GRATUITA.
* [Suscripción a MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits/) y **pruebas de desarrollo MSDN de pago por uso**: Al igual que la versión de prueba gratuita, una suscripción a MSDN requiere una tarjeta de crédito para adquirir una solución de MySQL de pago de ClearDB.
* [Contrato Enterprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/): los clientes de EA reciben su facturación de EA cada trimestre para todas sus compras en Azure Marketplace (terceros) mediante una factura independiente y consolidada. Las compras en Marketplace se le facturarán fuera del pago por adelantado de Azure (anteriormente denominado compromiso monetario). Tenga en cuenta que, en este momento, la tienda de Azure no está disponible para los clientes inscritos en Azerbaiyán, Croacia, Noruega y Puerto Rico. 

## <a name="why-was-i-charged-350-for-a-web-app--mysql-from-the-azure-marketplace"></a>¿Por qué se me ha cobrado 3,50 USD por una aplicación web + MySQL desde Azure Marketplace?
La opción de base de datos predeterminada es Titan, que cuesta 3,50 USD. El costo no aparece durante la creación de la base de datos y, por error, puede adquirir una base de datos que no deseaba. Estamos intentando encontrar una manera de mejorar la experiencia pero, hasta entonces, debe comprobar todos los planes de tarifa seleccionados para la aplicación web y la base de datos antes de hacer clic en **Crear** e iniciar la implementación de los recursos.

## <a name="i-am-running-mysql-on-my-own-azure-virtual-machine-can-i-connect-my-azure-web-app-to-my-database"></a>Estoy ejecutando MySQL en mi propia máquina virtual de Azure. ¿Puedo conectar mi aplicación web de Azure a la base de datos?
Sí. Puede conectar la aplicación web a la base de datos siempre y cuando a la máquina virtual de Azure se le haya dado acceso remoto a la aplicación web. Para más información, consulte [Instalación de MySQL en una máquina virtual creada con el modelo de implementación clásico que disponga de Windows Server 2012 R2](/previous-versions/azure/virtual-machines/windows/classic/mysql-2008r2?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json).

## <a name="in-which-countriesregions-are-cleardb-premium-mysql-clusters-supported"></a>¿En qué países o regiones se pueden usar los clústeres de MySQL Premium de ClearDB?
Los clústeres MySQL ClearDB Premium están disponibles en todas las regiones de Azure de todo el mundo, excepto en la India, Australia, Sur de Brasil y China.

## <a name="can-i-create-a-new-cluster-prior-to-creating-a-database-with-cleardb-premium-cluster-solution"></a>¿Puedo crear un nuevo clúster antes de crear una base de datos con la solución de clúster ClearDB Premium?
No, no se pueden crear clústeres de ClearDB vacíos. Azure Portal permite crear bases de datos en un clúster, por lo que puede crear un nuevo clúster al mismo tiempo.

## <a name="will-i-be-warned-if-i-try-to-delete-a-cleardb-mysql-database-that-is-in-use-by-one-of-my-applications"></a>¿Se me avisará si intento eliminar una base de datos MySQL ClearDB que esté utilizando una de mis aplicaciones?
No, Azure no avisa si se elimina una compra de Marketplace de la que dependa una aplicación.

## <a name="which-regions-can-i-create-cleardb-databases-in"></a>¿En qué regiones puedo crear bases de datos ClearDB?
Azure Marketplace no está disponible para los clientes inscritos en Azerbaiyán, Croacia, Noruega y Puerto Rico. ClearDB no está disponible en estas regiones.

## <a name="what-pricing-tier-should-i-choose-for-a-production-web-app-and-database"></a>¿Qué plan de tarifa debo elegir para una base de datos y una aplicación web de producción?
Use el plan de tarifa Básico o uno de nivel superior para Web Apps. Para ClearDB, se recomienda un plan Júpiter o Saturno. Revise las características y limitaciones de cada plan de tarifa para [Web Apps](https://azure.microsoft.com/pricing/details/app-service/) y las [bases de datos MySQL ClearDB](https://www.navisite.com/blog/news/update-change-of-billing-notice-for-all-azure-cleardb-service-plans/) con objeto de elegir el que se adapte a sus necesidades.

## <a name="how-do-i-upgrade-my-cleardb-database-from-one-plan-to-another"></a>¿Cómo actualizo mi base de datos ClearDB de un plan a otro?
En [Azure Portal](https://portal.azure.com), se puede escalar verticalmente una base de datos de hospedaje compartido de ClearDB. Lea este [artículo](https://blogs.msdn.microsoft.com/appserviceteam/2016/10/06/upgrade-your-cleardb-mysql-database-in-azure-portal/) para más información. Actualmente no se admite actualización para clústeres de ClearDB Premium en Azure Portal.

## <a name="i-cant-see-my-cleardb-database-in-azure-portal"></a>¿Por qué no veo la base de datos ClearDB en Azure Portal?
Si creó una base de datos de ClearDB en el portal clásico, no la verá en [Azure Portal](https://portal.azure.com). No hay ninguna solución alternativa para esta situación.

## <a name="who-do-i-contact-for-support-when-my-database-is-down"></a>¿Con quién debo ponerme en contacto para obtener soporte técnico cuando tenga problemas con la base de datos?
Si tiene algún problema relacionado con la base de datos, póngase en contacto con el [soporte técnico de ClearDB](https://www.cleardb.com/developers/help/support) . Tenga a mano la información de suscripción de Azure para proporcionarla.

## <a name="can-i-create-additional-users-for-my-cleardb-mysql-database-cluster-solution"></a>¿Puedo crear usuarios adicionales para la solución de clúster de base de datos MySQL ClearDB?
No. No se pueden crear usuarios adicionales, pero puede crear bases de datos adicionales en el clúster de base de datos de ClearDB.  

## <a name="can-basicpro-series-databases-be-upgraded-in-place-similar-to-planetary-plans-today-on-cleardb-portal"></a>¿Las bases de datos de las series Basic y Pro se pueden actualizar localmente de forma similar a los planes de la serie Planetary en el portal de ClearDB?
Sí, las bases de datos de la serie Basic se pueden actualizar localmente (de Basic 60 a Basic 500). La serie Pro se puede actualizar localmente (de Pro 125 a Pro 1000) con la excepción de Pro 60. En estos momentos, no se admite la actualización de la base de datos Pro 60. 

## <a name="when-i-migrate-my-resources-from-one-subscription-to-another-does-my-cleardb-mysql-database-get-migrated-as-well"></a>Al migrar mis recursos de una suscripción a otra, ¿se migra también la base de datos MySQL ClearDB?
Al realizar una migración de recursos entre suscripciones, se aplican algunas [limitaciones](azure-resource-manager/management/move-support-resources.md) . Una base de datos MySQL ClearDB es un servicio de terceros, de ahí que no se migre durante la migración de la suscripción de Azure. Si no administra la migración de su base de datos de MySQL antes de migrar recursos de Azure, se pueden deshabilitar las bases de datos MySQL ClearDB. En primer lugar migre manualmente las bases de datos y luego realice la migración de la suscripción de Azure de la aplicación web. 

## <a name="i-hit-the-spending-limit-on-my-subscription-i-removed-the-limit-and-my-app-service-is-online-however-the-database-is-not-accessible-how-do-i-re-enable-the-cleardb-database"></a>Alcancé el límite de gasto en mi suscripción. Quité el límite y mi App Service está en línea. Sin embargo, no puedo obtener acceso a la base de datos. ¿Cómo vuelvo a habilitar la base de datos ClearDB?
Póngase en contacto con el [soporte técnico de ClearDB](https://www.cleardb.com/developers/help/support) para volver a habilitar la base de datos. Proporcióneles información de suscripción de Azure y el nombre de la base de datos.

## <a name="can-i-transfer-a-cleardb-database-from-a-credit-card-subscription-to-an-ea-subscription"></a>¿Puedo transferir una base de datos ClearDB desde una suscripción de tarjeta de crédito hasta una suscripción de EA?
Las bases de datos existentes de ClearDB utilizan la tarjeta de crédito asociada a las suscripciones existentes. Para usar una suscripción de EA, deberá migrar los datos a una base de datos nueva:

* Adquiera una nueva base de datos de ClearDB con su suscripción de EA.
* Migre los datos a la base de datos nueva.
* Actualice la aplicación para que use la nueva base de datos.
* Elimine la base de datos antigua de ClearDB.

Al crear una nueva aplicación web con MySQL (ClearDB) o al crear una base de datos de MySQL (ClearDB), la suscripción que elija determinará cómo se paga por el servicio. Con una suscripción de EA, no se bloqueará la adquisición de servicios de terceros como ClearDB en Azure Portal. Las suscripciones de EA se facturan fuera del pago por adelantado de Azure y se facturan de forma trimestral a pago vencido. Los clientes de EA tendrán que establecer un método de pago, como una tarjeta de crédito, para pagar los servicios de Marketplace de terceros.

## <a name="where-can-i-see-the-charges-for-cleardb-resources-in-an-ea-subscription"></a>¿Dónde puedo ver los cargos para los recursos de ClearDB en una suscripción de EA?
Para los clientes directos de EA, los gastos de Azure Marketplace están visibles en Enterprise Portal. Tenga en cuenta que todas las compras y los consumos de Marketplace se facturan fuera del pago por adelantado de Azure y se facturan trimestralmente a pago vencido. Los clientes de EA tienen que pagar directamente a los proveedores de servicios de terceros. Para ello, pueden habilitar un método de pago, como una tarjeta de crédito, con su cuenta de EA.

Los clientes indirectos de EA pueden encontrar sus suscripciones de Azure Marketplace en la página **Administrar suscripciones** de Enterprise Portal, pero no se muestran los precios. Los clientes deben ponerse en contacto con su LSP para obtener información sobre los gastos en Marketplace.

El acceso a Azure Marketplace para los servicios de terceros puede administrarse mediante los administradores de inscripción de EA Azure. Estos pueden deshabilitar o volver a habilitar el acceso a las compras de terceros desde la Tienda en Administrar cuentas y Administrar suscripciones, en la sección Cuentas en Enterprise Portal.

## <a name="who-do-i-contact-for-questions-about-my-bill-for-cleardb-services-in-my-ea-subscription"></a>¿Con quién puedo ponerme en contacto si tengo dudas sobre la factura de los servicios de ClearDB en mi suscripción EA?
Póngase en contacto con el servicio de [atención al cliente de Enterprise](https://aka.ms/AzureEntSupport) para resolver las dudas sobre la facturación de su inscripción EA. El equipo de soporte técnico del Portal EA responderá a sus preguntas o resolverá el problema.

## <a name="more-information"></a>Más información
[P+F sobre Azure Marketplace](https://azure.microsoft.com/marketplace/faq/)
