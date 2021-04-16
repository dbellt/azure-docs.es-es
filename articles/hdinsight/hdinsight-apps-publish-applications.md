---
title: Publicación de aplicaciones de Azure HDInsight
description: Aprenda a crear una aplicación de HDInsight y luego a publicarla en Azure Marketplace.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 05/14/2018
ms.openlocfilehash: d432d20660cd1b7ee59c9ef72f9db04bc0009481
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107306592"
---
# <a name="publish-an-hdinsight-application-in-the-azure-marketplace"></a>Publicación de una aplicación de HDInsight en Azure Marketplace
Puede instalar una aplicación de Azure HDInsight en un clúster de HDInsight basado en Linux. En este artículo, aprenderá a publicar una aplicación de HDInsight en Azure Marketplace. Para obtener información general sobre cómo publicar en Azure Marketplace, consulte [Publicación de una oferta en Azure Marketplace](../marketplace/overview.md).

Las aplicaciones de HDInsight usan el modelo *Traiga su propia licencia (BYOL)*. En un escenario BYOL, un proveedor de aplicaciones es responsable de conceder a los usuarios licencia para una aplicación. Los usuarios de la aplicación solo pagan por los recursos de Azure que crean, como el clúster de HDInsight y las máquinas virtuales y los nodos del clúster. Actualmente, la facturación de la aplicación no se realiza en Azure.

Para más información, consulte estos artículos relacionados con la aplicación de HDInsight:

* [Instalación de aplicaciones de HDInsight](hdinsight-apps-install-applications.md) Aprenda a instalar una aplicación de HDInsight en sus clústeres.
* [Instalación de aplicaciones de HDInsight personalizadas](hdinsight-apps-install-custom-applications.md) Aprenda a instalar y probar aplicaciones de HDInsight personalizadas.

## <a name="prerequisites"></a>Requisitos previos
Para enviar la aplicación personalizada a Marketplace, primero [cree y pruebe su aplicación personalizada](hdinsight-apps-install-custom-applications.md).

También debe registrar la cuenta de desarrollador. Para más información, consulte [Publicación de una oferta en Azure Marketplace](../marketplace/overview.md) y [Crear una cuenta de desarrollador de Microsoft](../marketplace/overview.md).

## <a name="define-the-application"></a>Definición de la aplicación
La publicación de aplicaciones en Marketplace consta de dos pasos. En primer lugar, defina un archivo *createUiDef.json*. El archivo createUiDef.json indica los clústeres con los que es compatible su aplicación. A continuación, publique la plantilla desde Azure Portal. Este es un archivo createUiDef.json de ejemplo:

```json
{
    "handler": "Microsoft.HDInsight",
    "version": "0.0.1-preview",
    "clusterFilters": {
        "types": ["Hadoop", "HBase", "Storm", "Spark"],
        "versions": ["3.6"]
    }
}
```

| Campo | Descripción | Valores posibles |
| --- | --- | --- |
| types |Los tipos de clúster con los que es compatible la aplicación. |Hadoop, HBase, Storm y Spark (o cualquier combinación de estos) |
| versions |Los tipos de clúster de HDInsight con los que es compatible la aplicación. |3.4 |

## <a name="application-installation-script"></a>Script de instalación de aplicaciones
Cuando se instala una aplicación en un clúster (ya sea en un clúster existente o en uno nuevo), se crea un nodo perimetral. El script de instalación de la aplicación se ejecuta en el nodo perimetral.

  > [!IMPORTANT]  
  > El nombre del script de instalación de la aplicación debe ser único para un clúster específico. El nombre del script debe tener el formato siguiente:
  > 
  > "name": "[concat('hue-install-v0','-' ,uniquestring(‘applicationName’)]"
  > 
  > El nombre del script tiene tres partes:
  > 
  > * El prefijo de nombre del script, que debe incluir el nombre de la aplicación o un nombre pertinente para la aplicación.
  > * Un guion, para mejorar la legibilidad.
  > * Una función de cadena única con el nombre de la aplicación como parámetro.
  > 
  > En la lista de acciones de script persistentes, el ejemplo anterior se muestra como **hue-install-v0-4wkahss55hlas**. Vea una [carga JSON de ejemplo](https://raw.githubusercontent.com/hdinsight/Iaas-Applications/master/Hue/azuredeploy.json).
  > 

El script de instalación debe tener las siguientes características:
* El script es idempotente. Las distintas llamadas al script producen el mismo resultado.
* El script tiene la versión adecuada. Use una ubicación diferente para el script cuando vaya a actualizar o probar los cambios. De esta forma se garantiza que los clientes que instalan la aplicación no resultan afectados por sus actualizaciones o pruebas. 
* El script tiene el registro adecuado en cada punto. Normalmente los registros de script son la única manera de depurar problemas de instalación de aplicaciones.
* Las llamadas a servicios o recursos externos tienen los reintentos adecuados para que la instalación no se vea afectada por problemas de red transitorios.
* Si el script inicia servicios en los nodos, los servicios se supervisan y configuran para iniciarse automáticamente si se produce el reinicio de un nodo.

## <a name="package-the-application"></a>Empaquetado de la aplicación
Cree un archivo .zip que contenga todos los archivos que son necesarios para instalar su aplicación de HDInsight. Usará el archivo .zip para publicar la aplicación. Este archivo incluye los siguientes archivos:

* createUiDefinition.json
* mainTemplate.json (para ver un ejemplo, consulte [Instalación de aplicaciones personalizadas de HDInsight](hdinsight-apps-install-custom-applications.md)).
* Todos los scripts requeridos

> [!NOTE]  
> Puede hospedar los archivos de aplicación (incluidos los archivos de aplicación web) en cualquier punto de conexión accesible públicamente.

## <a name="publish-the-application"></a>Publicación de la aplicación
Para publicar una aplicación de HDInsight:

1. Inicie sesión en Publicación de Azure.

2. En el menú de la izquierda, seleccione **Plantillas de solución**.
3. Escriba un título y luego seleccione **Create a new solution template** ///(Crear una plantilla de solución).
4. Si todavía no ha registrado su organización, seleccione **Create Dev Center account and join the Azure program** ///(Crear cuenta de Centro de desarrollo y unirse al programa de Azure).  Para más información, consulte [Crear una cuenta de desarrollador de Microsoft](../marketplace/overview.md).
5. Seleccione **Define some Topologies to get Started** ///(Definir algunas topologías para comenzar). Una plantilla de solución es una "matriz" para todas sus topologías. Puede definir varias topologías en una oferta o plantilla de solución. Cuando se inserta una oferta en un entorno de ensayo, se inserta con todas sus topologías. 
6. Escriba un nombre de topología y luego seleccione **+** .
7. Escriba una nueva versión y luego seleccione **+** .
8. Cargue el archivo .zip que creó cuando empaquetó la aplicación.  
9. Seleccione **Request Certification** ///(Solicitar certificación). El equipo de certificación de Microsoft revisa los archivos y certifica la topología.

## <a name="next-steps"></a>Pasos siguientes
* Aprenda a [instalar aplicaciones de HDInsight](hdinsight-apps-install-applications.md) en sus clústeres.
* Aprenda a [instalar aplicaciones de HDInsight personalizadas](hdinsight-apps-install-custom-applications.md) y a implementar una aplicación de HDInsight sin publicar en HDInsight.
* Aprenda a [usar acciones de script para personalizar clústeres de HDInsight basados en Linux](hdinsight-hadoop-customize-cluster-linux.md) y a agregar más aplicaciones. 
* Aprenda a [crear clústeres de Apache Hadoop basados en Linux en HDInsight mediante plantillas de Azure Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md).
* Aprenda a [usar un nodo perimetral vacío en HDInsight](hdinsight-apps-use-edge-node.md) para acceder a clústeres de HDInsight y probar y hospedar aplicaciones de HDInsight.
