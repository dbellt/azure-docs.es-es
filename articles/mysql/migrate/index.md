---
title: Introducción a la guía para la migración de MySQL en el entorno local a Azure Database for MySQL
description: Guía de migración de MySQL local a Azure Database for MySQL
ms.service: mysql
ms.subservice: migration-guide
ms.topic: how-to
author: arunkumarthiags
ms.author: arthiaga
ms.reviewer: maghan
ms.custom: ''
ms.date: 05/25/2021
ms.openlocfilehash: 1a65469b38a82b4dabeeb2be1f2ea8db269b760d
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111958635"
---
# <a name="mysql-on-premises-to-azure-database-for-mysql-migration-guide-introduction"></a>Introducción a la guía para la migración de MySQL en el entorno local a Azure Database for MySQL

Esta guía de migración está diseñada para proporcionar información apilable y práctica para aquellos los clientes de MySQL e integradores de software que buscan migrar cargas de trabajo de MySQL a [Azure Database for MySQL.](../overview.md) En esta guía se proporcionan conocimientos vigentes que se aplican a la mayoría de los casos y se proporcionan instrucciones que permiten planear y ejecutar correctamente una migración de MySQL a Azure.

El proceso de mover bases de datos y cargas de trabajo de MySQL existentes a la nube puede presentar desafíos que afecten a la funcionalidad de las cargas de trabajo y la conectividad de las aplicaciones existentes. En esta guía se presentan vínculos y recomendaciones útiles que se centran en una migración correcta y garantizan que las cargas de trabajo y las aplicaciones siguen funcionando según lo previsto originalmente.

La información proporcionada se centra en un recorrido del cliente mediante Microsoft [Cloud Adoption Framework](/azure/cloud-adoption-framework/get-started/) para realizar actividades de valoración, migración y posterior optimización en un entorno de Azure Database for MySQL.

## <a name="mysql"></a>MySQL

MySQL tiene un amplio historial en la comunidad de código abierto y se ha hecho muy popular entre las empresas de todo el mundo para la creación de sitios web y otras aplicaciones críticas para la empresa. Esta guía ayuda a los administradores a los que se les ha pedido que indiquen el ámbito, planee y ejecuten la migración. Los administradores que nunca han usado MySQL también pueden consultar la [documentación de MySQL](https://dev.mysql.com/doc/) para obtener información más profunda sobre el funcionamiento interno de MySQL. Además, en esta guía también se incluyen vínculos a varios artículos de referencia en todas las secciones que le dirigen a información y tutoriales útiles.

## <a name="azure-database-for-mysql"></a>Azure Database for MySQL

[Azure Database for MySQL](../overview.md) es una oferta de plataforma como servicio (PaaS) de Microsoft en la que el entorno de MySQL está totalmente administrado. En este entorno totalmente administrado, se aplican automáticamente las actualizaciones tanto del software como del sistema operativo, así como la implementación de la alta disponibilidad y la protección de los datos.

Además de la oferta de PaaS, en las máquinas virtuales de Azure también se puede ejecutar MySQL. Consulte el artículo [Selección de la opción adecuada de MySQL Server en Azure](../select-right-deployment-type.md) para decidir qué tipo de implementación es más adecuada para la carga de trabajo de datos de destino.

![Comparación de entornos MySQL](./media/image3.jpg)

**Comparación de entornos MySQL.**

Esta guía se centra completamente en la migración de las cargas de trabajo de MySQL locales a la oferta de plataforma como servicio Azure Database for MySQL debido a sus diversas ventajas con respecto a las características de infraestructura como servicio (IaaS), como escalabilidad vertical y horizontal, el pago por uso, la alta disponibilidad, la seguridad y las características de administración.  

## <a name="summary"></a>Resumen

Una vez que haya completado esta guía, puede intentar [implementar una aplicación de ejemplo y revisar los parámetros del servidor](summary.md#next-steps) por su cuenta.

> [!div class="nextstepaction"]
> [Caso de uso representativo](./representative-use-case.md)