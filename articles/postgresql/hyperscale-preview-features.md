---
title: Características en versión preliminar de Hiperescala (Citus) de Azure Database for PostgreSQL
description: Lista actualizada de características que se encuentran actualmente en versión preliminar
author: jonels-msft
ms.author: jonels
ms.custom: mvc
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: overview
ms.date: 04/07/2021
ms.openlocfilehash: 5f0552e09464a6b571a665cfe3895f48c3aa8c41
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107258459"
---
# <a name="preview-features-for-postgresql---hyperscale-citus"></a>Características en versión preliminar de Hiperescala (Citus) de PostgreSQL

Hiperescala (Citus) de Azure Database for PostgreSQL ofrece versiones preliminares de características no publicadas. Las características en versión preliminar se ofrecen sin Acuerdo de Nivel de Servicio y no se recomiendan para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.  Para más información, consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="features-currently-in-preview"></a>Características actualmente en versión preliminar

Estas son las características disponibles actualmente para la versión preliminar:

* **[Nivel Básico](concepts-hyperscale-tiers.md)** . Ejecute un grupo de servidores solo con un nodo de coordinación y sin nodos de trabajo. Una manera económica de realizar pruebas y desarrollos iniciales, y de controlar pequeñas cargas de trabajo de producción.
* **[PostgreSQL 12 y 13](concepts-hyperscale-versions.md)** .
  Use la versión más reciente de la base de datos en el grupo de servidores.
* **[Citus 10](concepts-hyperscale-versions.md#citus-and-other-extension-versions)** .
  Se instala automáticamente en los grupos de servidores que ejecutan PostgreSQL 13.
* **[Almacenamiento en columnas](concepts-hyperscale-columnar.md)** .
  Almacene las columnas de las tablas seleccionadas (en lugar de las filas) de forma contigua en el disco. Se admite la compresión en disco. Adecuado para cargas de trabajo de análisis y almacenamiento de datos.
* **[Réplicas de lectura](howto-hyperscale-read-replicas-portal.md)** (actualmente solo en la misma región). Los cambios que se realicen en el grupo de servidores principal se reflejarán en su réplica y las consultas en la réplica no provocarán ninguna carga adicional en el original.
  Las réplicas son una herramienta útil para mejorar el rendimiento de las cargas de trabajo de solo lectura.
* **[PgBouncer administrado](concepts-hyperscale-limits.md#managed-pgbouncer-preview)** .
  Un agrupador de conexiones que permite a muchos clientes conectarse al grupo de servidores a la vez, a la vez que limita el número de conexiones activas. Satisface las solicitudes de conexión y mantiene el nodo de coordinación con una ejecución sin problemas.
* **[pgAudit](concepts-hyperscale-audit.md)** . Proporciona un registro detallado de auditoría de objetos y sesiones mediante la utilidad de registro estándar de PostgreSQL. Genera los registros de auditoría necesarios para pasar determinadas auditorías de certificación gubernamentales, financieras o ISO.

### <a name="available-regions-for-preview-features"></a>Regiones disponibles para las características en versión preliminar

La extensión pgAudit está disponible en todas las [regiones admitidas por Hiperescala (Citus)](concepts-hyperscale-configuration-options.md#regions).
Las otras características en versión preliminar solo están disponibles en la región **Este de EE. UU.** .

## <a name="does-my-server-group-have-access-to-preview-features"></a>¿Mi grupo de servidores tiene acceso a las características en versión preliminar?

Para determinar si el grupo de servidores Hiperescala (Citus) tiene habilitadas las características en versión preliminar, vaya a la página **Información general** del grupo de servidores en Azure Portal.
Si ve la propiedad **Nivel: Básico (versión preliminar)** o **Nivel: Estándar (versión preliminar)** , el grupo de servidores tiene acceso a las características en versión preliminar.

### <a name="how-to-get-access"></a>Cómo obtener acceso

Al crear un nuevo grupo de servidores Hiperescala (Citus), active la casilla **Habilitar características de vista previa**.

## <a name="contact-us"></a>Ponerse en contacto con nosotros

Háganos saber sobre su experiencia con las características en versión preliminar. Para ello, envíe un correo electrónico a [Preguntas de Azure Database for PostgreSQL](mailto:AskAzureDBforPostgreSQL@service.microsoft.com).
(Esta dirección de correo electrónico no es un alias de soporte técnico. Para problemas técnicos, abra una [solicitud de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)).
