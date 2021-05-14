---
title: Cómo crear la instancia de Azure Web PubSub
description: Información general sobre las opciones para crear la instancia de Azure Web PubSub y cómo hacerlo
author: yjin81
ms.author: yajin1
ms.service: azure-web-pubsub
ms.topic: how-to
ms.date: 03/17/2021
ms.openlocfilehash: 9f13f0f62a9bd3b26b4c855110fbc97971ecfea8
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108167121"
---
# <a name="how-to-create-azure-web-pubsub-instance"></a>Cómo crear la instancia de Azure Web PubSub

Para compilar una aplicación con el servicio Azure Web PubSub, debe crear la instancia de Web PubSub y, después, conectar sus clientes y servidores. En esta guía paso a paso se muestran las opciones para crear la instancia de Azure Web PubSub.

## <a name="create-azure-web-pubsub-instance-with-azure-portal"></a>Creación de la instancia de Azure Web PubSub con Azure Portal 

[Azure Portal](https://docs.microsoft.com/azure/azure-portal/) es una consola unificada basada en web que proporciona una alternativa a las herramientas de línea de comandos. Puede administrar su suscripción de Azure con Azure Portal. Compile, administre y supervise todo, desde aplicaciones web sencillas hasta complejas implementaciones en la nube. También puede crear la instancia del servicio Azure Web PubSub con Azure Portal.

1. Seleccione el botón Nuevo de la esquina superior izquierda en Azure Portal. En la pantalla "Nuevo", escriba *Web PubSub* en el cuadro de búsqueda y presione Entrar.

1. Seleccione **Web PubSub** en los resultados de la búsqueda y, después, haga clic en **Crear**.

1. Escriba la siguiente configuración.

    | Configuración      | Descripción                                        |
    | ------------ | -------------------------------------------------- |
    | **Nombre del recurso** | Nombre único global que identifica la nueva instancia del servicio Web PubSub. Los caracteres válidos son `a-z`, `0-9` y `-`.  | 
    | **Suscripción** | Suscripción de Azure en la que se va a crear esta instancia del servicio Web PubSub. | 
    | **[Grupo de recursos](../azure-resource-manager/management/overview.md)** |  Nombre del grupo de recursos nuevo o existente en el que se va a crear la instancia del servicio Web PubSub. | 
    | **Ubicación** | Seleccione una [región](https://azure.microsoft.com/regions/) cerca de usted. |
    | **Plan de tarifa** | Obtenga más información sobre los [planes de tarifa del servicio Azure Web PubSub](https://azure.microsoft.com/pricing/details/web-[pubsub]). |
    | **Recuento de unidades** |  El recuento de unidades especifica cuántas conexiones puede aceptar la instancia del servicio Web PubSub. Cada unidad admite 1000 conexiones simultáneas como máximo. Solo es configurable en el nivel Estándar. |

1. Seleccione **Crear** para empezar a implementar la instancia del servicio Web PubSub.

## <a name="create-azure-web-pubsub-instance-with-azure-cli"></a>Creación de la instancia de Azure Web PubSub con la CLI de Azure

La [Interfaz de la línea de comandos (CLI) de Azure](https://docs.microsoft.com/cli/azure) es un conjunto de comandos que se usa para crear y administrar recursos de Azure. La CLI de Azure está disponible en los servicios de Azure y está diseñada para ayudarle a trabajar rápidamente con Azure, con especial atención a la automatización. También puede crear la instancia del servicio Azure Web PubSub con la CLI de Azure tras el anuncio de disponibilidad general.


