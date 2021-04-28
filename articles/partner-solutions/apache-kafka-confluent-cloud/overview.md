---
title: 'Información general de Apache Kafka en Confluent Cloud: soluciones de partners de Azure'
description: Obtenga información sobre el uso de Apache Kafka en Confluent Cloud en Azure Marketplace.
author: tfitzmac
ms.topic: conceptual
ms.service: partner-services
ms.date: 01/15/2021
ms.author: tomfitz
ms.openlocfilehash: eea96f9a445fcfd1af5d23db1e01d1a7440fceea
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108137082"
---
# <a name="what-is-apache-kafka-for-confluent-cloud"></a>¿Qué es Apache Kafka para Confluent Cloud?

Apache Kafka para Confluent Cloud es una oferta de Azure Marketplace que proporciona Apache Kafka como servicio. Está totalmente administrado para que pueda centrarse en la creación de aplicaciones en lugar de en la administración de los clústeres.

Para reducir la carga de la administración multiplataforma, Microsoft se ha asociado con Confluent Cloud para crear una capa de aprovisionamiento integrada desde Azure a Confluent Cloud. Proporciona una experiencia consolidada para usar Confluent Cloud en Azure. Puede integrar y administrar fácilmente Confluent Cloud con sus aplicaciones de Azure.

Anteriormente, tenía que adquirir la oferta de Confluent Cloud en Marketplace y configurar por separado la cuenta en Confluent Cloud. Para administrar las configuraciones y los recursos, tenía que navegar entre los portales de Azure y Confluent Cloud.

Ahora se aprovisionan los recursos de Confluent Cloud a través de un proveedor de recursos denominado **Microsoft.Confluent**. Puede crear y administrar recursos de la organización Confluent Cloud a través de [Azure Portal](https://portal.azure.com/), la [CLI de Azure](/cli/azure/) o los [SDK de Azure](/azure/#languages-and-tools). Confluent Cloud posee y ejecuta la aplicación de software como servicio (SaaS), incluidos los entornos, los clústeres, los temas, las claves de API y los conectores administrados.

## <a name="capabilities"></a>Funcionalidades

La integración profunda entre Confluent Cloud y Azure permite las siguientes funcionalidades:

- Aprovisionar un nuevo recurso de la organización de Confluent Cloud desde Azure Portal con una infraestructura totalmente administrada.
- Optimizar el inicio de sesión único (SSO) desde Azure a Confluent Cloud con Azure Active Directory (Azure AD). No se necesita ninguna autenticación independiente desde el portal de Confluent Cloud.
- Obtenga una facturación unificada del consumo de Confluent Cloud mediante la facturación de suscripciones de Azure.
- Administre los recursos de Confluent Cloud desde Azure Portal y realice su seguimiento en la página **Todos los recursos** con los demás recursos de Azure.

## <a name="confluent-organization"></a>Organización de Confluent

Una organización de Confluent es un recurso que proporciona la asignación entre los recursos de Azure y Confluent Cloud. Es el recurso primario para otros recursos de Confluent Cloud.

Cada suscripción de Azure puede contener varios planes de Confluent. Cada plan de Confluent se asigna a una cuenta de usuario y una organización en el portal de Confluent. Dentro de cada organización de Confluent, puede crear varios entornos, clústeres, temas y conectores.

Al aprovisionar un recurso de Confluent Cloud en Azure, obtiene un identificador de organización, un entorno predeterminado y una cuenta de usuario de Confluent. Para más información, consulte [Inicio rápido: Introducción a Confluent Cloud en Azure](create.md).

Para la facturación, cada oferta de Confluent Cloud comprada en Marketplace se asigna a una organización de Confluent única.

## <a name="single-sign-on"></a>Inicio de sesión único

Al iniciar sesión en Azure Portal, las credenciales también se usan para iniciar sesión en el portal de SaaS de Confluent Cloud. La experiencia usa [Azure AD](../../active-directory/fundamentals/active-directory-whatis.md) y [SSO de Azure AD](../../active-directory/manage-apps/what-is-single-sign-on.md) para proporcionar una manera segura y cómoda de iniciar sesión.

Para más información, consulte [Inicio de sesión único](manage.md#single-sign-on).

## <a name="billing"></a>Facturación

Hay dos opciones de facturación disponibles: plan mensual de pago por uso y plan de compromiso.

- Con el **plan mensual de pago por uso**, recibirá los cargos de consumo de Confluent Cloud en la factura mensual de Azure.
- Con un **plan de compromiso**, se inscribe por un importe mínimo de gasto y obtiene un descuento en su uso comprometido de Confluent Cloud.

Decida qué opción de facturación usar al crear el servicio.

## <a name="connector-to-azure-cosmos-db"></a>Conector de Azure Cosmos DB

Desde dentro del cliente del concentrador de complementos, instale el conector de Cosmos DB como se recomienda en la [lista del centro de complementos](https://www.confluent.io/hub/microsoftcorporation/kafka-connect-cosmos). 

Para instalar el conector manualmente, descargue primero un archivo JAR de uber desde la [Página de versiones de Cosmos DB](https://github.com/microsoft/kafka-connect-cosmosdb/releases). Como alternativa, puede [crear su propio archivo JAR de uber directamente desde el código fuente](https://github.com/microsoft/kafka-connect-cosmosdb/blob/dev/doc/README_Sink.md#install-sink-connector). Para completar la instalación, siga las instrucciones descritas en la documentación del complemento para la [instalación manual de los conectores](https://docs.confluent.io/home/connect/install.html#install-connector-manually).  

## <a name="confluent-links"></a>Vínculos de Confluent

Para obtener ayuda adicional con el uso de Apache Kafka para Confluent Cloud, consulte los vínculos siguientes en el [sitio de Confluent](https://docs.confluent.io/home/overview.html).

Para obtener información sobre las opciones de facturación, consulte:

* [Azure Marketplace con pago por uso](https://docs.confluent.io/cloud/current/billing/ccloud-azure-payg.html)
* [Azure Marketplace con compromisos](https://docs.confluent.io/cloud/current/billing/ccloud-azure-ubb.html)

Para obtener información acerca de cómo administrar las soluciones, consulte:

* [Creación de un clúster en Confluent Cloud](https://docs.confluent.io/cloud/current/clusters/create-cluster.html)
* [Entornos de Confluent Cloud](https://docs.confluent.io/current/cloud/using/environments.html)
* [Conceptos básicos sobre Confluent Cloud](https://docs.confluent.io/current/cloud/using/cloud-basics.html)

Para obtener soporte técnico y conocer las condiciones, consulte:

* [Soporte técnico de Confluent](https://support.confluent.io)
* [Términos del servicio](https://www.confluent.io/confluent-cloud-tos).

## <a name="next-steps"></a>Pasos siguientes

Para crear una instancia de Apache Kafka para Confluent Cloud, consulte [Inicio rápido: Introducción a Confluent Cloud en Azure](create.md).