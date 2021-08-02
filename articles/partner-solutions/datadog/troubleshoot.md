---
title: 'Solución de problemas de Datadog: soluciones para asociados de Azure'
description: Este artículo contiene información sobre la solución de problemas de Datadog en Azure.
ms.service: partner-services
ms.topic: conceptual
ms.date: 05/28/2021
author: tfitzmac
ms.author: tomfitz
ms.openlocfilehash: 1c23851dc76cb0e012ad8796d361e5c293bc8d99
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110655159"
---
# <a name="fix-common-errors-for-datadog-on-azure"></a>Corrección de errores comunes de Datadog en Azure

Este documento contiene información sobre cómo solucionar los problemas de las soluciones que usan Datadog.

## <a name="purchase-errors"></a>Errores de compra

* Los errores de compra se producen porque una tarjeta de crédito válida no está conectada a la suscripción de Azure o un método de pago no está asociado a la suscripción.

  Use una suscripción de Azure diferente. O bien, agregue o actualice la tarjeta de crédito o el método de pago de la suscripción. Para obtener más información, consulte cómo [actualizar el método de pago y la tarjeta de crédito](../../cost-management-billing/manage/change-credit-card.md).

* Una suscripción de Contrato Enterprise (EA) no permite realizar compras en Marketplace.

  Use otra suscripción. O bien, compruebe si su suscripción de EA está habilitada para la compra en Marketplace. Para obtener más información, consulte la sección [Habilitación de compras en Azure Marketplace](../../cost-management-billing/manage/ea-azure-marketplace.md#enabling-azure-marketplace-purchases). Si esas opciones no solucionan el problema, póngase en contacto con el [servicio de soporte técnico de Datadog](https://www.datadoghq.com/support).

## <a name="unable-to-create-datadog-resource"></a>No se pueden crear recursos de Datadog

Para configurar la integración de Azure Datadog, debe tener acceso de **propietario** en la suscripción de Azure. Asegúrese de que tiene el acceso adecuado antes de iniciar el programa de instalación.

## <a name="single-sign-on-errors"></a>Errores en el inicio de sesión único

**No se puede guardar la configuración de inicio de sesión único**: este error se produce cuando hay otra aplicación empresarial que usa el identificador SAML de Datadog. Para buscar dicha aplicación, seleccione **Edit** (Editar) en la sección Configuración básica de SAML.

Para resolver este problema, deshabilite la otra aplicación o úsela como aplicación empresarial para configurar el inicio de sesión único de SAML con Datadog. Si decide usar la otra aplicación, asegúrese de que tiene la [configuración necesaria](create.md#configure-single-sign-on).

**La aplicación no se muestra en la página Configuración de inicio de sesión único**: en primer lugar, busque el identificador de la aplicación. Si no se muestra ningún resultado, compruebe la configuración de SAML de la aplicación. La cuadrícula solo muestra las aplicaciones con la configuración de SAML correcta. 

La dirección URL del identificador debe ser `https://us3.datadoghq.com/account/saml/metadata.xml`.

La dirección URL de respuesta debe ser `https://us3.datadoghq.com/account/saml/assertion`.

En la imagen siguiente se muestran los valores correctos.
  
:::image type="content" source="media/troubleshoot/troubleshooting.png" alt-text="Compruebe la configuración de SAML para la aplicación Datadog en AAD." border="true":::

**Los usuarios invitados al inquilino no pueden acceder al inicio de sesión único**: algunos usuarios tienen dos direcciones de correo electrónico en Azure Portal. Normalmente, un correo electrónico es el nombre principal de usuario (UPN) y el otro es un correo electrónico alternativo.

Cuando invite a cualquier usuario, use el UPN de inquilino del inicio. Al usar el UPN, la dirección de correo electrónico se mantiene sincronizada durante el proceso de inicio de sesión único. Para encontrar el nombre principal de usuario, busque la dirección de correo electrónico en la esquina superior derecha de Azure Portal del usuario.
  
## <a name="logs-not-being-emitted"></a>No se emiten registros

Los recursos enumerados en las categorías de registro de recursos de Azure Monitor son los únicos que emiten registros a Datadog. Para comprobar si el recurso emitiendo registros a Datadog, vaya a la configuración de diagnóstico de Azure del recurso específico. Compruebe si hay una configuración de diagnóstico de Datadog.

:::image type="content" source="media/troubleshoot/diagnostic-setting.png" alt-text="Configuración de diagnóstico de Datadog en el recurso de Azure" border="true":::

## <a name="metrics-not-being-emitted"></a>No se emiten métricas

Al recurso de Datadog se le asigna el rol **Lector de supervisión** en la suscripción de Azure adecuada. Este rol permite al recurso de Datadog recopilar métricas y enviarlas a Datadog.

Para comprobar que el recurso tiene la asignación de roles correcta, abra Azure Portal y seleccione la suscripción. En el panel izquierdo, seleccione **Access Control (IAM)** . Busque el nombre del recurso de Datadog. Confirme que el recurso de Datadog tiene la asignación de roles **Lector de supervisión**, como se muestra a continuación.

:::image type="content" source="media/troubleshoot/datadog-role-assignment.png" alt-text="Asignación de roles de Datadog en la suscripción de Azure" border="true":::

## <a name="datadog-agent-installation-fails"></a>Error al instalar el agente de Datadog

La integración de Azure Datadog ofrece la posibilidad de instalar el agente de Datadog en una máquina virtual o en una instancia de App Service. Para configurar el agente de Datadog, se usa la clave de API seleccionada como **clave predeterminada** en la pantalla Claves de API. Si no se selecciona una clave predeterminada, se producirá un error en la instalación del agente de Datadog.

Si el agente de Datadog se ha configurado con una clave incorrecta, vaya a la pantalla de claves de API y cambie la **clave predeterminada**. Tendrá que desinstalar el agente de Datadog y volver a instalarlo para configurar la máquina virtual con las nuevas claves de API.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre [cómo administrar una instancia](manage.md) de Datadog.
