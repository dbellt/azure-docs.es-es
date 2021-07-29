---
title: Cambio de SKU en Azure AD Domain Services | Microsoft Docs
description: Aprenda a cambiar el nivel de SKU de un dominio administrado de Azure AD Domain Services si los requisitos empresariales así lo exigen.
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: 2bdf660d57f4fa8cb3a804ff55028dc442f96b8b
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110786142"
---
# <a name="change-the-sku-for-an-existing-azure-active-directory-domain-services-managed-domain"></a>Cambio de SKU en un dominio administrado de Azure Active Directory Domain Services

En Azure Active Directory Domain Services (Azure AD DS), el rendimiento y las características disponibles dependen del tipo de SKU. Las características difieren, por ejemplo, en la frecuencia con que se realizan copias de seguridad o en el número máximo de relaciones de confianza de salida unidireccionales que se establecen entre los bosques.

Puede seleccionar una SKU al crear el dominio administrado y ampliarla o reducirla después si las necesidades del negocio varían una vez que ha implementado el dominio administrado. Las necesidades del negocio pueden variar, por ejemplo, porque sea necesario aumentar la frecuencia de las copias de seguridad o crear nuevas relaciones de confianza entre los bosques. Para más información sobre los límites y los precios de las distintas SKU, consulte estas páginas sobre los [conceptos relacionados con SKU en Azure AD DS][concepts-sku] y los [precios de Azure AD DS][pricing].

En este artículo, se explica cómo se crea la SKU de un dominio administrado de Azure AD DS en Azure Portal.

## <a name="before-you-begin"></a>Antes de empezar

Para completar este artículo, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
* Un dominio administrado de Azure Active Directory Domain Services habilitado y configurado en su inquilino de Azure AD.
    * Si es necesario, complete el tutorial para [crear y configurar un dominio administrado][create-azure-ad-ds-instance].

## <a name="sku-change-limitations"></a>Limitaciones de los cambios de SKU

Puede aumentar o reducir las SKU una vez implementado el dominio administrado. Sin embargo, si utiliza un bosque de recursos y se han creado relaciones de confianza de bosques de salida unidireccionales entre Azure AD DS y un entorno local de AD DS, los cambios de SKU estarán sujetos a algunas limitaciones. Las SKU *Premium* y *Enterprise* establecen un límite en el número de relaciones de confianza que se pueden crear. No se puede cambiar a una SKU cuyo límite máximo sea inferior a la configuración actual.

Por ejemplo:

* No se puede cambiar a la *SKU* estándar. El bosque de recursos de Azure AD DS no admite la SKU *Estándar*. 
* De igual modo, si ha creado siete relaciones de confianza en la SKU *Premium*, no podrá cambiar a la SKU *Enterprise*. La SKU *Enterprise* admite un máximo de cinco relaciones de confianza.

Para más información sobre estos límites, consulte este artículo sobre las [características y límites de las SKU de Azure AD DS][concepts-sku].

## <a name="select-a-new-sku"></a>Selección de una nueva SKU

Para cambiar la SKU de un dominio administrado en Azure Portal, siga estos pasos:

1. En la parte superior de Azure Portal, busque y seleccione **Azure AD Domain Services**. Elija un dominio administrado de la lista; por ejemplo, *aaddscontoso.com*.
1. En el menú situado a la izquierda de la página de Azure AD DS, seleccione **Configuración > SKU**.

    ![Seleccione la opción SKU del menú del dominio administrado de Azure AD DS en Azure Portal](media/change-sku/overview-change-sku.png)

1. En el menú desplegable, seleccione la SKU que quiera usar con el dominio administrado. Si tiene un bosque de recursos, no podrá seleccionar la SKU *Estándar*, ya que las relaciones de confianza entre bosques solo están disponibles en la SKU *Enterprise* o superiores.

    Elija la SKU que desee en el menú desplegable y seleccione **Guardar**.

    ![Elija la SKU necesaria en el menú desplegable de Azure Portal](media/change-sku/change-sku-selection.png)

El cambio de SKU puede tardar un minuto o dos.

## <a name="next-steps"></a>Pasos siguientes

Si tiene un bosque de recursos y desea crear nuevas relaciones de confianza después de cambiar de SKU, consulte [Creación de una relación de confianza entre bosques de salida en un dominio local de Azure Active Directory Domain Services][create-trust].

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[concepts-sku]: administration-concepts.md#azure-ad-ds-skus
[create-trust]: tutorial-create-forest-trust.md

<!-- EXTERNAL LINKS -->
[pricing]: https://azure.microsoft.com/pricing/details/active-directory-ds/
