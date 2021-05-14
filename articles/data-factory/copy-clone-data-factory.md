---
title: Copia o clonación de una factoría de datos en Azure Data Factory
description: Obtenga información sobre cómo copiar o clonar una factoría de datos en Azure Data Factory
ms.service: data-factory
author: minhe-msft
ms.author: hemin
ms.reviewer: jburchel
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 39ec94a63d7a4318a0774508ebb1c1c2abc3a927
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107903837"
---
# <a name="copy-or-clone-a-data-factory-in-azure-data-factory"></a>Copia o clonación de una factoría de datos en Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este artículo se describe cómo copiar o clonar una factoría de datos en Azure Data Factory.

## <a name="use-cases-for-cloning-a-data-factory"></a>Casos de uso para la clonación de una factoría de datos

Estas son algunas de las circunstancias en las que podría resultar útil copiar o clonar una factoría de datos:

- **Mover Data Factory** a una región nueva. Si quiere mover Data Factory a otra región, la mejor manera de hacerlo es crear una copia en la región de destino y eliminar la existente.

- **Cambiar el nombre de Data Factory**. Azure no permite cambiar el nombre de los recursos. Si quiere cambiar el nombre de una factoría de datos, puede clonarla con otro nombre y eliminar la existente.

- **Depurar los cambios** cuando las características de depuración no sean suficientes. En la mayoría de los escenarios, puede usar [Depurar](iterative-development-debugging.md). En otros, resulta más conveniente probar los cambios en un entorno de espacio aislado clonado. Por ejemplo, cómo se comportarán las canalizaciones de ETL con parámetros cuando un desencadenador se active tras la llegada de archivos frente a la ventana de saltos de tamaño constante, es posible que no se pueda probar fácilmente solo a través de la depuración. En estos casos, puede que quiera clonar un entorno de espacio aislado para experimentación. Como Azure Data Factory cobra principalmente por la cantidad de ejecuciones, una segunda factoría no genera ningún cobro adicional.

## <a name="how-to-clone-a-data-factory"></a>Clonación de una factoría de datos

1. Como requisito previo, primero debe crear la factoría de datos de destino desde Azure Portal.

1. Si está en modo GIT:
    1. Cada vez que publique desde el portal, la plantilla de Resource Manager de la factoría se guarda en GIT en la rama adf\_publish.
    1. Conecte la factoría nueva al _mismo_ repositorio y compile desde la rama adf\_publish. Se transportarán recursos como las canalizaciones, los conjuntos de datos y los desencadenadores.

1. Si está en modo real:
    1. La UI de Data Factory le permite exportar toda la carga de la factoría de datos a un archivo de plantilla de Resource Manager y un archivo de parámetro. Se puede acceder a ellos desde el botón **Plantilla de ARM \ Exportación de plantillas de Resource Manager** en el portal.
    1. Puede realizar los cambios correspondientes en el archivo de parámetros e intercambiar valores nuevos de la factoría nueva.
    1. A continuación, puede implementarla a través de los métodos de implementación de plantilla de Resource Manager estándar.

1. Si tiene una instancia de Integration Runtime autohospedada en la factoría de origen, deberá crearla previamente con el mismo nombre en el generador de destino. Si quiere compartir la instancia de Integration Runtime autohospedada entre distintas factorías, puede usar el patrón publicado [aquí](create-shared-self-hosted-integration-runtime-powershell.md) para compartir la instancia de IR autohospedada.

1. Por motivos de seguridad, la plantilla de Resource Manager generada no contendrá información secreta, como contraseñas para los servicios vinculados. Por lo tanto, debe proporcionar las credenciales como parámetros de implementación. Si no quiere escribir manualmente las credenciales para la configuración, considere en su lugar la posibilidad de recuperar las cadenas de conexión y las contraseñas de Azure Key Vault. [Más información](store-credentials-in-key-vault.md)

## <a name="next-steps"></a>Pasos siguientes

Revise la guía para crear una factoría de datos en Azure Portal en [Creación de una factoría de datos con la interfaz de usuario de Azure Data Factory](quickstart-create-data-factory-portal.md).
