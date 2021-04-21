---
title: Procedimientos recomendados del registro
description: Obtenga información sobre cómo de forma eficaz Azure Container Registry mediante los siguientes procedimientos recomendados.
ms.topic: article
ms.date: 01/07/2021
ms.openlocfilehash: 0811cc4a5bffc21ffba19e64a3887eab6bc36fbb
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107784144"
---
# <a name="best-practices-for-azure-container-registry"></a>Procedimientos recomendados para Azure Container Registry

Estos procedimientos recomendados pueden servir de ayuda para maximizar el rendimiento y rentabilizar el uso de un registro privado en Azure para almacenar e implementar imágenes de contenedor y otros artefactos.

Para obtener información sobre el fondo de los conceptos del registro, consulte [Acerca de los registros, los repositorios y las imágenes](container-registry-concepts.md). Vea también [Recomendaciones para el etiquetado y el control de versiones de las imágenes de contenedor](container-registry-image-tag-version.md) para las estrategias de etiquetado y versión de las imágenes en el registro. 

## <a name="network-close-deployment"></a>Implementación cercana a la red

Cree el registro de contenedor en la misma región de Azure en la que implementa los contenedores. Colocar el registro en una región que esté cercana a la red para los hosts del contenedor puede ayudar a reducir tanto la latencia como el costo.

La implementación cercana a la red es una de las razones principales para utilizar un registro de contenedor privado. Las imágenes de Docker tienen una [construcción en capas](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/) eficaz que permite realizar implementaciones incrementales. Sin embargo, los nuevos nodos tienen que extraer todas las capas necesarias para una imagen determinada. Este `docker pull` inicial puede agregar rápidamente hasta varios gigabytes. Tener un registro privado próximo a la implementación minimiza la latencia de red.
Además, todas las nubes públicas, Azure incluido, implementan cuotas de salida de red. La extracción de imágenes desde un centro de datos a otro agrega cuotas de salida de red, además de latencia.

## <a name="geo-replicate-multi-region-deployments"></a>Implementación de la replicación geográfica en varias regiones

Utilice la característica de [replicación geográfica](container-registry-geo-replication.md) de Azure Container Registry si está implementando contenedores en varias regiones. Si se está prestando servicio a clientes globales de centros de datos locales o su equipo de desarrollo se encuentra en distintas ubicaciones, puede simplificar la administración del registro y minimizar la latencia mediante la replicación geográfica del registro. También puede configurar [webhooks](container-registry-webhook.md) regionales para que le envíen notificaciones cuando se produzcan eventos en réplicas concretas, como cuando se insertan imágenes.

La replicación geográfica está disponible para los registros [Premium](container-registry-skus.md). Para aprender a usar la replicación geográfica, vea el tutorial de tres partes [Geo-replication in Azure Container Registry](container-registry-tutorial-prepare-registry.md) (Replicación geográfica en Azure Container Registry).

## <a name="maximize-pull-performance"></a>Maximización del rendimiento de las extracciones

Además de colocar imágenes próximas a las implementaciones, las características de las propias imágenes pueden afectar al rendimiento de las extracciones.

* **Tamaño de la imagen**: minimice los tamaños de las imágenes quitando los [niveles](container-registry-concepts.md#manifest) innecesarios o reduciendo el tamaño de las capas. Una manera de reducir el tamaño de la imagen es usar el método de [compilación de Docker en varias fases](https://docs.docker.com/develop/develop-images/multistage-build/), que incluye solo los componentes necesarios en tiempo de ejecución. 

  Compruebe también si la imagen puede incluir una imagen de sistema operativo base más ligera. Además, si usa un entorno de implementación, como Azure Container Instances, que almacene en caché determinadas imágenes base, compruebe si puede cambiar una capa de imagen en una de las imágenes almacenadas en caché. 
* **Número de capas**: equilibre el número de capas utilizadas. Si tiene muy pocas, no se beneficiará de la reutilización de capas ni del almacenamiento en caché en el host. Sin embargo, si tiene demasiadas, el entorno de implementación invierte más tiempo en extraer y descomprimir. Lo óptimo es tener entre cinco y diez capas.

Elija también un [nivel de servicio](container-registry-skus.md) de Azure Container Registry que cubra sus necesidades de rendimiento. El nivel Premium proporciona el mayor ancho de banda y la máxima tasa de operaciones simultáneas de lectura y escritura cuando se tiene un gran volumen de implementaciones.

## <a name="repository-namespaces"></a>Espacios de nombres del repositorio

Al usar los espacios de nombres del repositorio, puede permitir el uso compartido de un registro único en varios grupos de su organización. Los registros se pueden compartir por las implementaciones y los equipos. Azure Container Registry admite espacios de nombres anidados, lo que permite el aislamiento del grupo. Sin embargo, el registro administra todos los repositorios de forma independiente, no como una jerarquía.

Por ejemplo, considere las siguientes etiquetas de imagen de contenedor. Las imágenes que se usan en toda la organización, como `aspnetcore`, se colocan en el espacio de nombres raíz, mientras que las imágenes de contenedor que pertenecen a los grupos de producción y marketing usan sus propios espacios de nombres.

- *contoso.azurecr.io/aspnetcore:2.0*
- *contoso.azurecr.io/products/widget/web:1*
- *contoso.azurecr.io/products/bettermousetrap/refundapi:12.3*
- *contoso.azurecr.io/marketing/2017-fall/concertpromotions/campaign:218.42*

## <a name="dedicated-resource-group"></a>Grupo de recursos dedicado

Dado que los registros de contenedor son recursos que se utilizan en varios hosts de contenedor, un registro debe residir en su propio grupo de recursos.

Aunque puede experimentar con un tipo de host específico, como [Azure Container Instances](../container-instances/container-instances-overview.md), es probable que desee eliminar la instancia de contenedor al terminar. Sin embargo, también puede mantener la colección de imágenes que ha insertado en Azure Container Registry. Al colocar el registro en su propio grupo de recursos, se minimiza el riesgo de eliminar de manera accidental la colección de imágenes del registro cuando se elimina el grupo de recursos de la instancia de contenedor.

## <a name="authentication-and-authorization"></a>Autenticación y autorización

Al autenticar con un registro de contenedor de Azure, hay dos escenarios principales: la autenticación individual y la autenticación de servicio (o "sin periféricos"). En la tabla siguiente se proporciona una breve descripción de estos escenarios y el método recomendado de autenticación para cada uno.

| Tipo | Escenario de ejemplo | Método recomendado |
|---|---|---|
| Identidad individual | Un desarrollador que extrae imágenes de su máquina de desarrollo, o las inserta en ella. | [az acr login](/cli/azure/acr#az_acr_login) |
| Identidad de servicio o sin periféricos | Canalizaciones de compilación e implementación donde el usuario no está directamente implicado. | [Entidad de servicio](container-registry-authentication.md#service-principal) |

Para obtener información en profundidad sobre este y otros escenarios de autenticación de Azure Container Registry, consulte [Autenticación con un registro de contenedor de Azure](container-registry-authentication.md).

Azure Container Registry admite prácticas de seguridad en su organización para distribuir deberes y privilegios a diferentes identidades. Utilice el [control de acceso basado en roles](container-registry-roles.md) para asignar los permisos adecuados a diferentes usuarios, entidades de servicio u otras identidades que realicen diferentes operaciones del registro. Por ejemplo, asigne permisos de inserción a las entidades de servicio que se usen en una canalización de compilación y asigne permisos de extracción a otras identidades utilizadas para la implementación. Cree [tokens](container-registry-repository-scoped-permissions.md) para el acceso pormenorizado y limitado en el tiempo a repositorios concretos.

## <a name="manage-registry-size"></a>Administración del tamaño del registro      

Las restricciones de almacenamiento de cada [nivel de servicio del registro de contenedor][container-registry-skus] están diseñadas para adaptarse a un escenario típico: **Básico** para los primeros pasos, **Estándar** para la mayoría de las aplicaciones de producción y **Premium** para un rendimiento a gran escala y para la [replicación geográfica][container-registry-geo-replication]. Durante todo el ciclo de vida del registro, debe administrar su tamaño mediante la eliminación periódica del contenido sin usar.

Use el comando [az acr show-usage][az-acr-show-usage] de la CLI de Azure para mostrar el tamaño actual del registro:

```azurecli
az acr show-usage --resource-group myResourceGroup --name myregistry --output table
```

```output
NAME      LIMIT         CURRENT VALUE    UNIT
--------  ------------  ---------------  ------
Size      536870912000  185444288        Bytes
Webhooks  100                            Count
```

También puede encontrar el almacenamiento actual usado en **Información general** del registro en Azure Portal:

![Información del uso del registro en Azure Portal][registry-overview-quotas]

### <a name="delete-image-data"></a>Eliminación de los datos de la imagen

Azure Container Registry admite varios métodos para eliminar datos de imagen del registro de contenedor. Puede eliminar imágenes por etiqueta o síntesis de manifiesto o eliminar un repositorio entero.

Para más información sobre cómo eliminar datos de imagen del registro, como imágenes no etiquetadas (a veces llamadas "pendientes" o "huérfanas"), consulte [Eliminación de imágenes de contenedor en Azure Container Registry](container-registry-delete.md).

## <a name="next-steps"></a>Pasos siguientes

Azure Container Registry está disponible en varios niveles (también denominados SKU) que proporcionan funcionalidades diferentes. Para más información sobre los niveles de servicio disponibles, consulte [SKU de Azure Container Registry](container-registry-skus.md).

Para obtener recomendaciones sobre cómo mejorar la postura de seguridad de los registros de contenedor, consulte [Base de referencia de Azure Security para Azure Container Registry](security-baseline.md).

<!-- IMAGES -->
[delete-repository-portal]: ./media/container-registry-best-practices/delete-repository-portal.png
[registry-overview-quotas]: ./media/container-registry-best-practices/registry-overview-quotas.png

<!-- LINKS - Internal -->
[az-acr-repository-delete]: /cli/azure/acr/repository#az_acr_repository_delete
[az-acr-show-usage]: /cli/azure/acr#az_acr_show_usage
[azure-cli]: /cli/azure
[azure-portal]: https://portal.azure.com
[container-registry-geo-replication]: container-registry-geo-replication.md
[container-registry-skus]: container-registry-skus.md