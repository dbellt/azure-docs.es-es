---
title: Solución de problemas de Application Change Analysis - Azure Monitor
description: Obtenga información sobre cómo solucionar problemas de Application Change Analysis.
ms.topic: conceptual
author: cawams
ms.author: cawa
ms.date: 02/11/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 91a7cc4b96dcff7a20afccfe5995574b89a8e87b
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108315712"
---
# <a name="troubleshoot-application-change-analysis-preview"></a>Solución de problemas de Application Change Analysis (versión preliminar)

## <a name="having-trouble-registering-microsoft-change-analysis-resource-provider-from-change-history-tab"></a>Problemas al registrar el proveedor de recursos de análisis Microsoft.Change Analysis desde la pestaña Historial de cambios

Si es la primera vez que ve el historial de cambios después de su integración en Application Change Analysis, verá que registra automáticamente un proveedor de recursos **Microsoft.ChangeAnalysis**. En contadas ocasiones, podría producirse un error por las razones siguientes:

- **No tiene permisos suficientes para registrar el proveedor de recursos Microsoft.ChangeAnalysis**. Este mensaje de error significa que el rol de la suscripción actual no tiene el ámbito **Microsoft.support/register/action** asociado a ella. Esto puede ocurrir si no es el propietario de una suscripción y obtuvo permisos de acceso compartido mediante un compañero de trabajo (es decir, ver el acceso a un grupo de recursos). Para corregirlo, puede ponerse en contacto con el propietario de la suscripción para registrar el proveedor de recursos **Microsoft.ChangeAnalysis**. Esto puede hacerse en Azure Portal a través de **Suscripciones | Proveedores de recursos** y buscar ```Microsoft.ChangeAnalysis``` y registrarse en la interfaz de usuario, o a través de Azure PowerShell o la CLI de Azure.

    Registre el proveedor de recursos a través de PowerShell:
    ```PowerShell
    # Register resource provider
    Register-AzResourceProvider -ProviderNamespace "Microsoft.ChangeAnalysis"
    ```

- **No se pudo registrar el proveedor de recursos Microsoft.ChangeAnalysis**. Este mensaje indica que ha habido un error inmediatamente después de que la interfaz de usuario enviara la solicitud para registrar el proveedor de recursos, y que no está relacionada con un problema de permisos. Lo más probable es que se deba a un problema temporal de conectividad a Internet. Intente actualizar la página y comprobar la conexión a Internet. Si el error continúa, póngase en contacto con changeanalysishelp@microsoft.com.

- **Está tardando más de lo esperado**. Este mensaje significa que el registro está tardando más de 2 minutos. Esto es poco habitual, pero no significa necesariamente que haya algo mal. Puede ir a **Suscripciones | Proveedor de recursos** para comprobar el estado de registro del proveedor de recursos **Microsoft.ChangeAnalysis**. Puede intentar usar la interfaz de usuario para anular el registro, volver a registrarse o actualizar para ver si esto ayuda. Si el problema persiste, póngase en contacto con changeanalysishelp@microsoft.com para obtener soporte.
    ![La solución de problemas de registro del proveedor de recursos está tardando demasiado tiempo](./media/change-analysis/troubleshoot-registration-taking-too-long.png)

![Captura de pantalla de la herramienta Diagnosticar y resolver problemas de una máquina virtual con Herramientas para la solución de problemas seleccionado.](./media/change-analysis/vm-dnsp-troubleshootingtools.png)

![Captura de pantalla del mosaico Analizar cambios recientes de la herramienta para la solución de problemas para una máquina virtual.](./media/change-analysis/analyze-recent-changes.png)

## <a name="azure-lighthouse-subscription-is-not-supported"></a>No se admite la suscripción de Azure Lighthouse.

- **Error al consultar al proveedor de recursos Microsoft.ChangeAnalysis** con el mensaje *No se admite la suscripción de Azure Lighthouse; los cambios solo están disponibles en el inquilino principal de la suscripción*. Ahora existe una limitación para que el proveedor de recursos de Change Analysis se registre a través de la suscripción de Azure Lighthouse para los usuarios que no están en el inquilino principal. Estamos trabajando actualmente para poder resolver este incidente. Si se trata de un problema de bloqueo, hay una solución alternativa que implica la creación de una entidad de servicio y la asignación explícita del rol para permitir el acceso.  Póngase en contacto con changeanalysishelp@microsoft.com para más información al respecto.

## <a name="an-error-occurred-while-getting-changes-please-refresh-this-page-or-come-back-later-to-view-changes"></a>Error al obtener los cambios. Actualice esta página o vuelva más tarde para ver los cambios

Este es el mensaje de error general que presenta el servicio Application Change Analysis cuando los cambios no se han podido cargar. Algunas de las causas conocidas son:

- Error de conectividad a Internet desde el dispositivo cliente.
- El servicio Change Analysis no está disponible temporalmente. Con la actualización de la página al cabo de unos minutos se suele corregir este problema. Si el error continúa, póngase en contacto con changeanalysishelp@micorosoft.com.

## <a name="you-dont-have-enough-permissions-to-view-some-changes-contact-your-azure-subscription-administrator"></a>No tiene permisos suficientes para ver algunos cambios. Póngase en contacto con su administrador de suscripciones de Azure

Este es el mensaje de error general de autorización, que explica que el usuario actual no tiene permisos suficientes para ver el cambio. Se requiere al menos acceso de lectura en el recurso para ver los cambios de infraestructura devueltos por Azure Resource Graph y Azure Resource Manager. En el caso de los cambios en el archivo en el invitado de la aplicación web y los cambios de configuración, se requiere un rol de colaborador como mínimo.

## <a name="failed-to-register-microsoftchangeanalysis-resource-provider"></a>No se pudo registrar el proveedor de recursos Microsoft.ChangeAnalysis

Este mensaje indica que ha habido un error inmediatamente después de que la interfaz de usuario enviara la solicitud para registrar el proveedor de recursos, y que no está relacionada con un problema de permisos. Lo más probable es que se deba a un problema temporal de conectividad a Internet. Intente actualizar la página y comprobar la conexión a Internet. Si el error continúa, póngase en contacto con changeanalysishelp@microsoft.com.

## <a name="you-dont-have-enough-permissions-to-register-microsoftchangeanalysis-resource-provider-contact-your-azure-subscription-administrator"></a>No tiene permisos suficientes para registrar el proveedor de recursos Microsoft.ChangeAnalysis. Póngase en contacto con su administrador de suscripciones de Azure

Este mensaje de error significa que el rol de la suscripción actual no tiene el ámbito **Microsoft.support/register/action** asociado a ella. Esto puede ocurrir si no es el propietario de una suscripción y obtuvo permisos de acceso compartido mediante un compañero de trabajo (es decir, ver el acceso a un grupo de recursos). Para corregirlo, puede ponerse en contacto con el propietario de la suscripción para registrar el proveedor de recursos **Microsoft.ChangeAnalysis**. Esto puede hacerse en Azure Portal a través de **Suscripciones | Proveedores de recursos** y buscar ```Microsoft.ChangeAnalysis``` y registrarse en la interfaz de usuario, o a través de Azure PowerShell o la CLI de Azure.

Registre el proveedor de recursos a través de PowerShell:

```PowerShell
# Register resource provider
Register-AzResourceProvider -ProviderNamespace "Microsoft.ChangeAnalysis"
```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre [Azure Resource Graph](../../governance/resource-graph/overview.md), que ayuda a la eficacia de Change Analysis.
