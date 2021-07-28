---
title: Uso de Azure Defender para registros de contenedor
description: Obtenga información sobre el uso de Azure Defender para registros de contenedor para examinar imágenes de Linux en los registros hospedados en Linux.
author: memildin
ms.author: memildin
ms.date: 10/21/2020
ms.topic: how-to
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 286ce9b628eff0975cff1cfa006f914ed43c3c46
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110469685"
---
# <a name="use-azure-defender-for-container-registries-to-scan-your-images-for-vulnerabilities"></a>Uso de Azure Defender para registros de contenedor para examinar las imágenes en busca de vulnerabilidades

En esta página se explica cómo usar el analizador de vulnerabilidades integrado para examinar las imágenes de contenedor almacenadas en la instancia de Azure Container Registry basada en Azure Resource Manager.

Cuando **Azure Defender para registros de contenedor** está habilitado, cualquier imagen que inserte en el registro se examinará de inmediato. Además, también se analizan las imágenes extraídas en los últimos 30 días. 

Cuando en detector notifica vulnerabilidades a Security Center, este presenta los resultados y la información relacionada como recomendaciones. Además, los resultados incluyen información relacionada, como pasos de corrección, CVE pertinentes, puntuaciones de CVSS, etc. Se pueden ver las vulnerabilidades identificadas para una o varias suscripciones, o para un registro específico.

> [!TIP]
> También puede examinar las imágenes de contenedor en busca de vulnerabilidades a medida que las imágenes se integran en los flujos de trabajo de CI/CD de GitHub. Obtenga más información en [Identificación de imágenes de contenedor vulnerables en los flujos de trabajo de CI/CD](defender-for-container-registries-cicd.md).


## <a name="identify-vulnerabilities-in-images-in-azure-container-registries"></a>Identificación de vulnerabilidades en las imágenes de los registros de contenedor de Azure 

Para habilitar exámenes de vulnerabilidades de las imágenes almacenadas en la instancia de Azure Container Registry basada en Azure Resource Manager:

1. Habilite **Azure Defender para registros de contenedor** para su suscripción. Security Center ya está listo para examinar imágenes en los registros.

    >[!NOTE]
    > Esta característica se cobra por imagen.

1. Los recorridos de imagen se desencadenan en cada una de las inserciones o importaciones, y si la imagen se ha extraído en los últimos 30 días. 

    Cuando finaliza el examen (al cabo de 2 minutos aproximadamente, aunque puede tardar hasta 15 minutos), los resultados están disponibles en recomendaciones de Security Center.

1. [Vea y corrija los resultados tal y como se explica a continuación](#view-and-remediate-findings).

## <a name="identify-vulnerabilities-in-images-in-other-container-registries"></a>Identificación de vulnerabilidades en imágenes en otros registros de contenedor 

1. Use las herramientas de ACR para incorporar imágenes al registro desde Docker Hub o Microsoft Container Registry.  Una vez finalizada la importación, Azure Defender examina las imágenes importadas. 

    Obtenga más información en [Importación de imágenes de contenedor en un registro de contenedor](../container-registry/container-registry-import-images.md).

    Cuando finaliza el examen (al cabo de 2 minutos aproximadamente, aunque puede tardar hasta 15 minutos), los resultados están disponibles en recomendaciones de Security Center.

1. [Vea y corrija los resultados tal y como se explica a continuación](#view-and-remediate-findings).


## <a name="view-and-remediate-findings"></a>Visualización y corrección de resultados

1. Para ver los resultados, vaya a la página **Recomendaciones**. Si se encontraron problemas, verá la recomendación **Las vulnerabilidades de las imágenes de Azure Container Registry deben corregirse**.

    ![Recomendación para corregir problemas ](media/monitor-container-security/acr-finding.png)

1. Seleccione la recomendación. 

    Se abre la página de detalles de la recomendación con información adicional. Esta información incluye la lista de registros con imágenes vulnerables ("recursos afectados") y los pasos de corrección. 

1. Seleccione un registro específico para ver los repositorios que contienen repositorios vulnerables.

    ![Seleccione un registro.](media/monitor-container-security/acr-finding-select-registry.png)

    Se abre la página Detalles del registro con la lista de repositorios afectados.

1. Seleccione un repositorio específico para ver los repositorios que contienen imágenes vulnerables.

    ![Selección de un repositorio](media/monitor-container-security/acr-finding-select-repository.png)

    Se abre la página Detalles del repositorio. Muestra las imágenes vulnerables junto con una evaluación de la gravedad de las conclusiones.

1. Seleccione una imagen específica para ver las vulnerabilidades.

    ![Selección de imágenes](media/monitor-container-security/acr-finding-select-image.png)

    Se abre la lista de resultados de la imagen seleccionada.

    ![Lista de resultados](media/monitor-container-security/acr-findings.png)

1. Para más información sobre un resultado, selecciónelo. 

    Se abre el panel de detalles de los resultados.

    [![Panel de detalles de los resultados](media/monitor-container-security/acr-finding-details-pane.png)](media/monitor-container-security/acr-finding-details-pane.png#lightbox)

    En este panel se incluye una descripción detallada del problema y vínculos a recursos externos para ayudar a mitigar las amenazas.

1. Siga los pasos de la sección de corrección de este panel.

1. Cuando haya realizado los pasos necesarios para corregir el problema de seguridad, reemplace la imagen en el registro:

    1. Inserte la imagen actualizada. Esto desencadenará un examen. 
    
    1. Consulte en la página de recomendaciones la recomendación "Las vulnerabilidades de las imágenes de Azure Container Registry deben corregirse". 
    
        Si la recomendación sigue apareciendo y la imagen que ha corregido todavía aparece en la lista de imágenes vulnerables, vuelva a comprobar los pasos de corrección.

    1. Cuando esté seguro de que la imagen actualizada se ha insertado y examinado y que ya no aparece en la recomendación, elimine la imagen vulnerable "antigua" del registro.


## <a name="disable-specific-findings-preview"></a>Deshabilitación de resultados específicos (versión preliminar)

> [!NOTE]
> [!INCLUDE [Legalese](../../includes/security-center-preview-legal-text.md)]

Si tiene la necesidad organizativa de omitir un resultado, en lugar de corregirlo, tiene la opción de deshabilitarlo. Los resultados deshabilitados no afectan a la puntuación de seguridad ni generan ruido no deseado.

Cuando un resultado coincide con los criterios que ha definido en las reglas de deshabilitación, no aparecerá en la lista de resultados. Entre los escenarios típicos se incluyen:

- Deshabilitar resultados con gravedad inferior a media
- Deshabilitar resultados que no se pueden revisar
- Deshabilitar resultados con una puntuación de CVSS por debajo de 6,5
- Deshabilitar resultados con texto específico en la categoría o la comprobación de seguridad (por ejemplo, "RedHat", "actualización de seguridad de CentOS para sudo")

> [!IMPORTANT]
> Para crear una regla, necesita permisos para editar directivas en Azure Policy.
>
> Obtenga más información en [Permisos de Azure RBAC en Azure Policy](../governance/policy/overview.md#azure-rbac-permissions-in-azure-policy).

Puede utilizar cualquiera de los criterios siguientes: 

- Identificador del resultado 
- Category
- Comprobación de seguridad 
- Puntuaciones de CVSS v3
- Gravedad 
- Estado revisable 

Para crear una regla:

1. En la página de detalles de recomendaciones de **Las vulnerabilidades de las imágenes de Azure Container Registry deben corregirse**, seleccione **Deshabilitar regla**.
1. Seleccione el ámbito pertinente.
1. Defina los criterios.
1. Seleccione **Aplicar regla**.

    :::image type="content" source="./media/defender-for-container-registries-usage/new-disable-rule-for-registry-finding.png" alt-text="Creación de una regla de deshabilitación para los resultados de VA en el registro":::

1. Para ver, invalidar o eliminar una regla: 
    1. Seleccione **Deshabilitar regla**.
    1. En la lista de ámbitos, las suscripciones con reglas activas se muestran como **Regla aplicada**.
        :::image type="content" source="./media/remediate-vulnerability-findings-vm/modify-rule.png" alt-text="Modificación o eliminación de una regla existente":::
    1. Para ver o eliminar la regla, seleccione el menú de puntos suspensivos ("...").


## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre Azure Defender](azure-defender.md)