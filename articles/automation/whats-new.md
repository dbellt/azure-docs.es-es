---
title: Novedades de Azure Automation
description: Se realizan actualizaciones significativas de la documentación de Azure Automation todos los meses.
services: automation
ms.subservice: ''
ms.topic: overview
ms.date: 06/09/2021
ms.custom: references_regions
ms.openlocfilehash: 6eae4a65412ad922724d4c35b7628b15698ba841
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111982941"
---
# <a name="whats-new-in-azure-automation"></a>Novedades de Azure Automation

Azure Automation recibe mejoras de forma continua. Para mantenerse al día de los avances más recientes, este artículo proporciona información acerca de los elementos siguientes:

- Versiones más recientes
- Problemas conocidos
- Corrección de errores

Esta página se actualiza mensualmente, por lo que se recomienda visitarla con frecuencia.

## <a name="june-2021"></a>Junio de 2021

### <a name="support-for-automation-and-state-configuration-available-in-west-us-3"></a>Compatibilidad con Automation y State Configuration disponible en Oeste de EE. UU. 3

**Tipo:** Nueva característica

Para más información, consulte [Residencia de datos en Azure](https://azure.microsoft.com/global-infrastructure/data-residency/) y seleccione su ubicación geográfica de la lista desplegable.

## <a name="may-2021"></a>Mayo de 2021

### <a name="startstop-vms-during-off-hours-v1"></a>Start/Stop VMs during off-hours (v1)

**Tipo:** Plan de cambio

Start/Stop VMs during off-hours (v1) caerá en desuso el 21 de mayo de 2022. Los clientes deben evaluar y planificar la migración a Start/Stop VMs v2 (versión preliminar). Para más información, consulte [Información general sobre Start/Stop VMs v2 (versión preliminar)](../azure-functions/start-stop-vms/overview.md).

## <a name="april-2021"></a>Abril de 2021

### <a name="support-for-update-management-and-change-tracking"></a>Compatibilidad con Update Management y Change Tracking

**Tipo:** Nueva característica

La asignación de regiones se ha actualizado para admitir Update Management y Change Tracking Este de Noruega, Norte de Emiratos Árabes Unidos, Centro-norte de EE. UU., Sur de Brasil y Centro de Corea del Sur. Para más información, consulte [Asignaciones admitidas](./how-to/region-mappings.md#supported-mappings).

### <a name="support-for-system-assigned-managed-identities"></a>Compatibilidad para identidades administradas asignadas por el sistema

**Tipo:** Nueva característica

Azure Automation ahora admite [identidades administradas asignadas por el sistema](./automation-security-overview.md#managed-identities-preview) para trabajos híbridos y en la nube en las regiones públicas y Gov de Azure. Para más información, lea este [anuncio](https://azure.microsoft.com/updates/azure-automation-system-assigned-managed-identities/).

## <a name="march-2021"></a>Marzo de 2021

### <a name="new-azure-automation-built-in-policies"></a>Nuevas directivas integradas de Azure Automation

**Tipo:** Nueva característica

Azure Automation ha agregado cinco nuevas directivas integradas:

- Las cuentas de Automation deben deshabilitar el acceso a la red pública.
- Las cuentas de Azure Automation deben usar claves administradas por el cliente para cifrar los datos en reposo.
- Configurar cuentas de Azure Automation para deshabilitar el acceso a la red pública
- Configurar conexiones de punto de conexión privado en cuentas de Azure Automation
- Las conexiones de punto de conexión privado en cuentas de Automation deben estar habilitadas.

Para más información, consulte la [referencia sobre Policy](./policy-reference.md).

### <a name="support-for-automation-and-state-configuration-declared-ga-in-south-india"></a>Compatibilidad con la disponibilidad general declarada de Automation y State Configuration en Sur de la India

**Tipo:** Nueva característica

Use las funcionalidades de automatización de procesos y de configuración de estados en la región Sur de la India. Para más información, lea este [anuncio](https://azure.microsoft.com/updates/azure-automation-in-south-india-region/).

### <a name="support-for-automation-and-state-configuration-declared-ga-in-uk-west"></a>Compatibilidad con la disponibilidad general declarada de Automation y State Configuration en Oeste de Reino Unido

**Tipo:** Nueva característica

Use las funcionalidades de automatización de procesos y de configuración de estados en la región Oeste de Reino Unido. Para más información, lea el [anuncio](https://azure.microsoft.com/updates/azure-automation-in-uk-west-region/).

### <a name="support-for-automation-and-state-configuration-declared-ga-in-uae-central"></a>Compatibilidad con la disponibilidad general declarada de Automation y State Configuration en Centro de Emiratos Árabes Unidos

**Tipo:** Nueva característica

Use las funcionalidades de automatización de procesos y de configuración de estados en la región Centro de Emiratos Árabes Unidos. Para más información, lea este [anuncio](https://azure.microsoft.com/updates/azure-automation-in-uae-central-region/).

### <a name="support-for-automation-and-state-configuration-available-in-australia-central-2-norway-west-and-france-south"></a>Compatibilidad con Automation y State Configuration disponible en Centro de Australia 2, Oeste de Noruega y Sur de Francia

**Tipo:** Nueva característica

Para más información sobre la [página Residencia de datos](https://azure.microsoft.com/global-infrastructure/data-residency/), seleccione la ubicación geográfica de cada región.

### <a name="new-scripts-added-for-installing-hybrid-worker-on-windows-and-linux"></a>Nuevos scripts agregados para instalar Hybrid Worker en Windows y Linux

**Tipo:** Nueva característica

Se han agregado dos nuevos scripts al [repositorio de GitHub](https://github.com/azureautomation) de Azure Automation que permiten abordar uno de los escenarios clave de Azure Automation para configurar una instancia de Hybrid Runbook Worker en una máquina Windows o Linux. El script crea una máquina virtual o usa una ya existente, crea un área de trabajo de Log Analytics si es necesario, instala el agente de Log Analytics para Windows o para Linux y registra la máquina en el área de trabajo de Log Analytics. El script de Windows se denomina **Create Automation Windows HybridWorker** y el de Linux es **Create Automation Linux HybridWorker**.

### <a name="invoke-runbook-through-an-azure-resource-manager-template-webhook"></a>Invocación de runbook mediante un webhook de una plantilla de Resource Manager

**Tipo:** Nueva característica

Para más información, consulte [Uso de un webhook de una plantilla de ARM](./automation-webhooks.md#use-a-webhook-from-an-arm-template).

### <a name="azure-update-management-now-supports-centos-8x-red-hat-enterprise-linux-server-8x-and-suse-linux-enterprise-server-15"></a>Azure Update Management ahora admite Centos 8.x, Red Hat Enterprise Linux Server 8.x y SUSE Linux Enterprise Server 15

**Tipo:** Nueva característica

Consulte la [lista completa](./update-management/operating-system-requirements.md) de sistemas operativos Linux compatibles para más información.

### <a name="in-region-data-residency-support-for-brazil-south-and-south-east-asia"></a>Compatibilidad con la residencia de datos en la región para las regiones Sur de Brasil y Sudeste de Asia 

**Tipo:** Nueva característica

En todas las regiones excepto Sur de Brasil y Sudeste de Asia, los datos de Azure Automation se almacenan en una región diferente (región emparejada de Azure) para proporcionar continuidad empresarial y recuperación ante desastres (BCDR). En el caso de las regiones Sur de Brasil y Sudeste de Asia, los datos de Azure Automation se almacenan en la misma región para adaptarse a los requisitos de residencia de datos de estas regiones. Para más información, consulte [Replicación geográfica en Azure Automation](./automation-managing-data.md#geo-replication-in-azure-automation).

## <a name="february-2021"></a>Febrero de 2021

### <a name="support-for-automation-and-state-configuration-declared-ga-in-japan-west"></a>Compatibilidad con la disponibilidad general declarada de Automation y State Configuration en Japón Occidental

**Tipo:** Nueva característica

Disponibilidad de la cuenta de Automation y de State Configuration en la región Japón Occidental. Para más información, lea el [anuncio](https://azure.microsoft.com/updates/azure-automation-in-japan-west-region/).

### <a name="introduced-custom-azure-policy-compliance-to-enforce-runbook-execution-on-hybrid-worker"></a>Se ha presentado el cumplimiento de Azure Policy personalizado para aplicar la ejecución de runbooks en Hybrid Worker

**Tipo:** nueva característica

Puede usar la nueva regla de cumplimiento de Azure Policy para permitir la creación de trabajos, webhooks y programaciones de trabajos para que se ejecuten solo en grupos de Hybrid Worker.

### <a name="update-management-availability-in-east-us-france-central-and-north-europe-regions"></a>Disponibilidad de Update Management en las regiones Este de EE. UU., Centro de Francia y Norte de Europa

**Tipo:** Nueva característica

La característica Update Management de Automation está disponible en las regiones Este de EE. UU., Centro de Francia y Norte de Europa. Vea [Asignación de regiones admitidas](how-to/region-mappings.md) para obtener actualizaciones de la documentación en la que se refleja este cambio.

## <a name="january-2021"></a>Enero de 2021

### <a name="support-for-automation-and-state-configuration-declared-ga-in-switzerland-west"></a>Compatibilidad con la disponibilidad general declarada de Automation y State Configuration en Oeste de Suiza

**Tipo:** Nueva característica

Disponibilidad de la cuenta de Automation y de State Configuration en la región Oeste de Suiza. Para más información, lea el [anuncio](https://azure.microsoft.com/updates/azure-automation-in-switzerland-west-region/).

### <a name="added-python-3-script-to-import-module-with-multiple-dependencies"></a>Se ha agregado el script Python 3 para importar módulos con varias dependencias

**Tipo:** Nueva característica

El script está disponible para su descarga desde nuestro [repositorio de GitHub](https://github.com/azureautomation/runbooks/blob/master/Utility/Python/import_py3package_from_pypi.py). 
 
### <a name="hybrid-runbook-worker-role-support-for-centos-8xrhel-8xsles-15"></a>Compatibilidad con roles de Hybrid Runbook Worker para Centos 8.x, RHEL 8.x y SLES 15

**Tipo.** Nueva característica

La característica Hybrid Runbook Worker admite las distribuciones CentOS 8.x, REHL 8.x y SLES 15 solo para la automatización de procesos en instancias de Hybrid Runbook Worker. Consulte el apartado sobre [sistemas operativos compatibles ](automation-linux-hrw-install.md#supported-linux-operating-systems) para conocer las actualizaciones de la documentación que refleja estos cambios.

### <a name="update-management-and-change-tracking-availability-in-australia-east-east-asia-west-us-and-central-us-regions"></a>Disponibilidad de Update Management y Change Tracking en las regiones Este de Australia, Este de Asia, Oeste de EE. UU. y Centro de EE. UU.

**Tipo:** Nueva característica

Cuenta de Automation, Seguimiento de cambios e inventario, y Update Management están disponibles en las regiones Este de Australia, Este de Asia, Oeste de EE. UU. y Centro de EE. UU. 

### <a name="introduced-public-preview-of-python-3-runbooks-in-us-government-cloud"></a>Se ha presentado la versión preliminar pública de runbooks de Python 3 en la nube del gobierno de EE. UU.

**Tipo:** Nueva característica Azure Automation presenta la compatibilidad con la versión preliminar pública de la nube de Python 3 y la ejecución de runbooks híbrido en las regiones de la nube del gobierno de EE. UU. Para obtener más información, consulte el [anuncio](https://azure.microsoft.com/updates/azure-automation-python-3-public-preview/).

### <a name="azure-automation-runbooks-moved-from-technet-script-center-to-github"></a>Runbooks de Azure Automation migrados desde el Centro de scripts de TechNet a GitHub

**Tipo:** Plan de cambio

El Centro de scripts de TechNet se está retirando y todos los runbooks hospedados en la Galería de runbooks se han migrado a la [organización GitHub de Automation](https://github.com/azureautomation). Para más información, consulte el artículo sobre la [migración de runbooks de Azure Automation a GitHub](https://techcommunity.microsoft.com/t5/azure-governance-and-management/azure-automation-runbooks-moving-to-github/ba-p/2039337).

## <a name="december-2020"></a>Diciembre de 2020

### <a name="azure-automation-and-update-management-private-link-ga"></a>Disponibilidad general de Azure Automation y Update Management Private Link

**Tipo:** Nueva característica

La compatibilidad de Azure Automation y Update Management se ha anunciado como GA para las nubes de Azure Global y del gobierno. Azure Automation ha habilitado la compatibilidad con Private Link para proteger la ejecución de un runbook en un rol de trabajo híbrido, mediante el uso de Update Management para aplicar revisiones a las máquinas, la invocación de un runbook a través de un webhook y el uso del servicio State Configuration para mantener la compatibilidad de las máquinas. Para más información, consulte el artículo sobre la [compatibilidad de Private Link de Azure Automation](https://azure.microsoft.com/updates/azure-automation-private-link).

### <a name="azure-automation-classified-as-grade-c-certified-on-accessibility"></a>Azure Automation ha recibido la certificación de Grado C en accesibilidad

**Tipo:** Nueva característica

Las características de accesibilidad de los productos de Microsoft ayudan a las agencias a satisfacer los requisitos globales de accesibilidad. En la página del [anuncio del blog](https://cloudblogs.microsoft.com/industry-blog/government/2018/09/11/accessibility-conformance-reports/), busque **Azure Automation** para leer el informe de conformidad de accesibilidad del servicio Automation.

### <a name="support-for-automation-and-state-configuration-ga-in-uae-north"></a>Compatibilidad con GA de Automation y State Configuration en Norte de Emiratos Árabes Unidos

**Tipo:** Nueva característica

Disponibilidad de la cuenta de Automation y de State Configuration en la región de Norte de Emiratos Árabes Unidos. Para más información, lea el [anuncio](https://azure.microsoft.com/updates/azure-automation-in-uae-north-region/).

### <a name="support-for-automation-and-state-configuration-ga-in-germany-west-central"></a>Compatibilidad con GA de Automation y State Configuration en Centro-oeste de Alemania

**Tipo:** Nueva característica

Disponibilidad de la cuenta de Automation y de State Configuration en la región de Oeste de Alemania. Para más información, lea el [anuncio](https://azure.microsoft.com/updates/azure-automation-in-germany-west-central-region/).

### <a name="dsc-support-for-oracle-6-and-7"></a>Compatibilidad de DSC con Oracle 6 y 7

**Tipo:** Nueva característica

Administre máquinas con Oracle Linux 6 y 7 con Automation y State Configuration. Consulte el apartado de [distribuciones de Linux compatibles](https://github.com/Azure/azure-linux-extensions/tree/master/DSC#4-supported-linux-distributions) para conocer las actualizaciones de la documentación para reflejar estos cambios.

### <a name="public-preview-for-python3-runbooks-in-automation"></a>Versión preliminar pública de runbooks de Python3 en Automation

**Tipo:** Nueva característica

Azure Automation ahora admite la ejecución híbrida y en la nube de Python 3 en la versión preliminar pública en todas las regiones de la nube de Azure Global. Para más información, consulte el anuncio (https://azure.microsoft.com/updates/azure-automation-python-3-public-preview/).

## <a name="november-2020"></a>Noviembre de 2020

### <a name="dsc-support-for-ubuntu-1804"></a>Compatibilidad de DSC con Ubuntu 18.04.

**Tipo:** Nueva característica

Consulte el apartado de [distribuciones de Linux compatibles](https://github.com/Azure/azure-linux-extensions/tree/master/DSC#4-supported-linux-distributions) para conocer las actualizaciones de la documentación que refleja estos cambios.

## <a name="october-2020"></a>Octubre de 2020

### <a name="support-for-automation-and-state-configuration-ga-in-switzerland-north"></a>Compatibilidad con GA de Automation y State Configuration en Norte de Suiza

**Tipo:** Nueva característica

Disponibilidad de la cuenta de Automation y de State Configuration en la región de Norte de Suiza. Para más información, lea el [anuncio](https://azure.microsoft.com/updates/azure-automation-in-switzerland-north-region/).

### <a name="support-for-automation-and-state-configuration-ga-in-brazil-south-east"></a>Compatibilidad con GA de Automation y State Configuration en Sureste de Brasil

**Tipo:** Nueva característica

Disponibilidad de la cuenta de Automation y de State Configuration en la región de Sureste de Brasil. Para más información, lea el [anuncio](https://azure.microsoft.com/updates/azure-automation-in-brazil-southeast-region/).

### <a name="update-management-availability-in-south-central-us"></a>Disponibilidad de Update Management en Centro-sur de EE. UU.

**Tipo:** Nueva característica

Se ha actualizado la asignación de regiones de Azure Automation para que la característica Update Management se pueda usar en la región Centro-sur de EE. UU. Consulte el apartado sobre la [asignación de regiones admitidas](how-to/region-mappings.md#supported-mappings) para conocer las actualizaciones de la documentación para reflejar este cambio.

## <a name="september-2020"></a>Septiembre de 2020

### <a name="startstop-vms-during-off-hours-runbooks-updated-to-use-azure-az-modules"></a>Se han actualizado los runbooks con Start/Stop VMs during off-hours para usar módulos de Azure Az

**Tipo:** Nueva característica

Se han actualizado los runbooks con Start/Stop VM para usar módulos de Az, en lugar de los módulos de Azure Resource Manager. Consulte la información general sobre [Start/Stop VMs during off-hours](automation-solution-vm-management.md) para ver las actualizaciones de la documentación que refleja estos cambios.

## <a name="august-2020"></a>Agosto de 2020

### <a name="published-the-dsc-extension-to-support-azure-arc"></a>Se publicó la extensión DSC para admitir Azure Arc

**Tipo:** Nueva característica

Use Azure Automation State Configuration para almacenar de forma centralizada las configuraciones y mantener el estado deseado de las máquinas conectadas híbridas que se hayan habilitado a través de la extensión de máquina virtual DSC de servidores habilitados para Azure Arc. Para más información, lea el artículo [Extensiones de máquina virtual con servidores habilitados para Azure Arc](../azure-arc/servers/manage-vm-extensions.md).

### <a name="july-2020"></a>Julio de 2020

### <a name="introduced-public-preview-of-private-link-support-in-automation"></a>Se presentó la versión preliminar pública de la compatibilidad con Private Link en Automation

**Tipo:** Nueva característica

Use Azure Private Link para conectar de forma segura redes virtuales a Azure Automation de mediante puntos de conexión privados. Para más información, lea el [anuncio](https://azure.microsoft.com/updates/public-preview-private-link-azure-automation-is-now-available/).

### <a name="hybrid-runbook-worker-support-for-windows-server-2008-r2"></a>Compatibilidad de Hybrid Runbook Worker con Windows Server 2008 R2

**Tipo:** Nueva característica

Automation Hybrid Runbook Worker es compatible con el sistema operativo Windows Server 2008 R2. Consulte el apartado sobre [sistemas operativos compatibles ](automation-windows-hrw-install.md#supported-windows-operating-system) para conocer las actualizaciones de la documentación que refleja estos cambios.

### <a name="update-management-support-for-windows-server-2008-r2"></a>Compatibilidad de Update Management con Windows Server 2008 R2

**Tipo:** Nueva característica

Update Management admite la evaluación y aplicación de revisiones en el sistema operativo Windows Server 2008 R2. Consulte el apartado sobre [sistemas operativos compatibles ](update-management/operating-system-requirements.md) para conocer las actualizaciones de la documentación que refleja estos cambios.

### <a name="automation-diagnostic-logs-schema-update"></a>Actualización del esquema de registros de diagnóstico de Automation

**Tipo:** Nueva característica

Se ha cambiado el esquema de datos de registro de Azure Automation en el servicio Log Analytics. Para más información, consulte [Reenvío de datos de un trabajo de Azure Automation a registros de Azure Monitor](automation-manage-send-joblogs-log-analytics.md#filter-job-status-output-converted-into-a-json-object).

### <a name="azure-lighthouse-supports-automation-update-management"></a>Azure Lighthouse admite Automation Update Management

**Tipo:** Nueva característica

Azure Lighthouse permite la administración de recursos delegados con Update Management para proveedores de servicios y clientes. Obtenga más información [aquí](https://azure.microsoft.com/blog/how-azure-lighthouse-enables-management-at-scale-for-service-providers/).

## <a name="june-2020"></a>Junio de 2020

### <a name="automation-and-update-management-availability-in-the-us-gov-arizona-region"></a>Disponibilidad de Automation y Update Management en la región de US Gov Arizona

**Tipo:** Nueva característica

La cuenta de Automation y Update Management están disponibles en US Gov Arizona. Para obtener más información, consulte el [anuncio](https://azure.microsoft.com/updates/azure-automation-generally-available-in-usgov-arizona-region/).

### <a name="hybrid-runbook-worker-onboarding-script-updated-to-use-az-modules"></a>Script de incorporación de Hybrid Runbook Worker para usar módulos Az

**Tipo:** Nueva característica

El runbook New-OnPremiseHybridWorker se ha actualizado para que admita módulos Az. Para más información, vea el paquete en la [Galería de PowerShell](https://www.powershellgallery.com/packages/New-OnPremiseHybridWorker/1.7).

### <a name="update-management-availability-in-china-east-2"></a>Disponibilidad de Update Management en Este de China 2

**Tipo:** Nueva característica

Se ha actualizado la asignación de regiones de Azure Automation para que la característica Update Management se pueda usar en la región Este de China 2. Consulte el apartado sobre la [asignación de regiones admitidas](how-to/region-mappings.md#supported-mappings) para conocer las actualizaciones de la documentación para reflejar este cambio.

## <a name="may-2020"></a>Mayo de 2020

### <a name="updated-automation-service-dns-records-from-region-specific-to-automation-account-specific-urls"></a>Registros DNS del servicio Automation actualizados desde direcciones URL específicas de la región a específicas de la cuenta de Automation

**Tipo:** Nueva característica

Los registros DNS de Azure Automation se han actualizado para admitir Private Link. Para más información, lea el [anuncio](https://azure.microsoft.com/updates/azure-automation-updateddns-records/).

### <a name="added-capability-to-keep-automation-runbooks-and-dsc-scripts-encrypted-by-default"></a>Funcionalidad agregada para mantener los runbooks de Automation y los scripts de DSC cifrados de forma predeterminada

**Tipo:** Nueva característica

Además de mejorar la seguridad de los recursos, los runbooks y DSC, los scripts también se cifran para mejorar la seguridad de Azure Automation.

## <a name="april-2020"></a>Abril de 2020

### <a name="retirement-of-the-automation-watcher-task"></a>Retirada de la tarea de monitor de Automation

**Tipo:** Plan de cambio

Azure Logic Apps es la forma recomendada y admitida de supervisar eventos, programar tareas periódicas y desencadenar acciones. No habrá más inversiones en la funcionalidad de la tarea de monitor. Para más información, consulte [Programación y ejecución de tareas, procesos y flujos de trabajo automatizados y periódicos con Azure Logic Apps](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md).

## <a name="march-2020"></a>Marzo de 2020

### <a name="support-for-impact-level-5-il5-compute-isolation-in-azure-commercial-and-government-cloud"></a>Compatibilidad con el aislamiento de proceso de nivel de impacto 5 (IL5) en la nube gubernamental y comercial de Azure

**Tipo**:

Hybrid Runbook Worker de Azure Automation se puede usar en Azure Government para admitir cargas de trabajo de nivel 5 de impacto. Para más información, consulte la [documentación](automation-hybrid-runbook-worker.md#support-for-impact-level-5-il5).

## <a name="february-2020"></a>Febrero de 2020

### <a name="introduced-support-for-azure-virtual-network-service-tags"></a>Incorporación de compatibilidad con las etiquetas de servicio de Azure Virtual Network

**Tipo:** Nueva característica

La compatibilidad de Automation con las etiquetas de servicio permite o deniega el tráfico para el servicio Automation en un subconjunto de escenarios. Para obtener más información, consulte la [documentación](automation-hybrid-runbook-worker.md#service-tags).

### <a name="enable-tls-12-support-for-azure-automation-service"></a>Habilitar la compatibilidad con TLS 1.2 con el servicio de Azure Automation

**Tipo:** Plan de cambio

Azure Automation es totalmente compatible con TLS 1.2 y todas las llamadas del cliente (a través de webhooks, nodos de DSC y Hybrid Worker). TLS 1.1 y TLS 1.0 aún se admiten para la compatibilidad con versiones anteriores con clientes anteriores hasta que los clientes se estandaricen y migren por completo a TLS 1.2.

## <a name="january-2020"></a>Enero de 2020

### <a name="introduced-public-preview-of-customer-managed-keys-for-azure-automation"></a>Se presentó la versión preliminar pública de las claves administradas por clientes para Azure Automation

**Tipo:** Nueva característica

Los clientes pueden administrar y proteger el cifrado de los recursos de Azure Automation mediante sus propias claves administradas. Para más información, consulte [Uso de claves administradas por el cliente](automation-secure-asset-encryption.md#use-of-customer-managed-keys-for-an-automation-account).

### <a name="retirement-of-azure-service-management-asm-rest-apis-for-azure-automation"></a>Retirada de las API REST de Azure Service Management (ASM) para Azure Automation

**Tipo:** Retirada

Las API REST de Azure Service Management (ASM) para Azure Automation se retirarán y dejarán de tener soporte técnico el 30 de enero de 2020. Para más información, vea el [anuncio](https://azure.microsoft.com/updates/azure-automation-service-management-rest-apis-are-being-retired-april-30-2019/).

## <a name="next-steps"></a>Pasos siguientes

Para colaborar en la documentación de Azure Automation, consulte la [Guía para colaboradores de Microsoft Docs](/contribute/).
