---
title: Conexión de datos de Microsoft 365 Defender con Azure Sentinel | Microsoft Docs
description: Obtenga información sobre cómo ingerir incidentes, alertas y datos de eventos sin procesar desde Microsoft 365 Defender en Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/13/2019
ms.author: yelevin
ms.openlocfilehash: 16cf1b89c2660d2505685fa931cc8b97ccb42a9b
ms.sourcegitcommit: 18cd3c1c8cc47258c6a1a04e0e03d6248c52ef24
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107992297"
---
# <a name="connect-data-from-microsoft-365-defender-to-azure-sentinel"></a>Conexión de datos de Microsoft 365 Defender con Azure Sentinel

> [!IMPORTANT]
>
> **Microsoft 365 Defender** se conocía antes como **Microsoft Threat Protection** o **MTP**.
>
> **Microsoft Defender para punto de conexión** se conocía antes como **Protección contra amenazas avanzada de Microsoft Defender** o **MDATP**.
>
> Es posible que vea que los nombres antiguos todavía están en uso durante un tiempo.

## <a name="background"></a>Fondo

El conector [Microsoft 365 Defender (M365D)](/microsoft-365/security/mtp/microsoft-threat-protection) de Azure Sentinel con la integración de incidentes le permite transmitir todos los incidentes y alertas de M365D a Azure Sentinel y mantiene los incidentes sincronizados entre ambos portales. Los incidentes de M365D incluyen todas sus alertas, entidades y otra información relevante, y se completan y agrupan las alertas de los servicios de componentes de M365D **Microsoft Defender para punto de conexión**, **Microsoft Defender for Identity**, **Microsoft Defender para Office 365** y **Microsoft Cloud App Security**.

El conector también permite transmitir eventos de **búsqueda avanzada** desde Microsoft Defender para punto de conexión a Azure Sentinel, lo que le permite copiar consultas de búsqueda avanzada de Defender para punto de conexión en Azure Sentinel, enriquecer las alertas de Sentinel con datos de eventos sin formato de Defender para punto de conexión para proporcionar información adicional y almacenar los registros con una mayor retención en Log Analytics.

Para obtener más información sobre la integración de incidentes y la recopilación de eventos de búsqueda avanzada, vea [Integración de Microsoft 365 Defender con Azure Sentinel](microsoft-365-defender-sentinel-integration.md).

> [!IMPORTANT]
>
> El conector Microsoft 365 Defender está actualmente en **VERSIÓN PRELIMINAR**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

## <a name="prerequisites"></a>Requisitos previos

- Debe tener una licencia válida de Microsoft 365 Defender, como se describe en [Requisitos previos de Microsoft 365 Defender](/microsoft-365/security/mtp/prerequisites). 

- Debe ser **administrador global** o un **administrador de seguridad** en Azure Active Directory.

## <a name="connect-to-microsoft-365-defender"></a>Conexión a Microsoft 365 Defender

1. En Azure Sentinel, seleccione **Data connectors** (Conectores de datos), elija **Microsoft 365 Defender (Preview)** (Microsoft 365 Defender [versión preliminar]) en la galería y seleccione **Open connector page** (Abrir página del conector).

1. En la sección **Connect incidents & alerts** (Conectar incidentes y alertas) de **Configuration** (Configuración), haga clic en el botón **Connect incidents & alerts** (Conectar incidentes y alertas).

1. Para evitar la duplicación de incidentes, se recomienda marcar la casilla **Turn off all Microsoft incident creation rules for these products** (Desactivar todas las reglas de creación de incidentes de Microsoft para estos productos).

    > [!NOTE]
    > Cuando se habilita el conector de Microsoft 365 Defender, todos los conectores de los componentes de M365D (los que se mencionan al principio de este artículo) se conectan automáticamente en segundo plano. Para desconectar uno de los conectores de los componentes, primero debe desconectar el conector de Microsoft 365 Defender.

1. Para consultar los datos de incidentes de Microsoft 365 Defender, use la instrucción siguiente en la ventana de consulta:
    ```kusto
    SecurityIncident
    | where ProviderName == "Microsoft 365 Defender"
    ```

1. Si quiere recopilar eventos de búsqueda avanzada de Microsoft Defender para punto de conexión, se pueden recopilar los siguientes tipos de eventos de sus tablas de búsqueda avanzada correspondientes.

    1. Marque las casillas de las tablas con los tipos de evento que quiera recopilar:

       | Nombre de la tabla | Tipo de eventos |
       |-|-|
       | DeviceInfo | Información de la máquina (incluida la información del sistema operativo) |
       | DeviceNetworkInfo | Propiedades de red de las máquinas |
       | DeviceProcessEvents | Creación de procesos y eventos relacionados |
       | DeviceNetworkEvents | Conexión de red y eventos relacionados |
       | DeviceFileEvents | Creación y modificación de archivos, y otros eventos del sistema de archivos |
       | DeviceRegistryEvents | Creación y modificación de entradas del Registro |
       | DeviceLogonEvents | Inicios de sesión y otros eventos de autenticación |
       | DeviceImageLoadEvents | Eventos de carga de DLL |
       | DeviceEvents | Tipos de eventos adicionales |
       | DeviceFileCertificateInfo | Información de certificado de archivos firmados |
       |

    1. Haga clic en **Aplicar cambios**.

    1. Para consultar las tablas de búsqueda avanzada en Log Analytics, escriba el nombre de la tabla de la lista anterior en la ventana de consulta.

## <a name="verify-data-ingestion"></a>Comprobación de la ingesta de datos

El grafo de datos de la página del conector indica que se están ingiriendo datos. Observará que se muestra una línea para incidentes, alertas y eventos, y la línea de eventos es una agregación del volumen de eventos en todas las tablas habilitadas. Una vez que haya habilitado el conector, puede usar las siguientes consultas de KQL para generar grafos más específicos.

Use la siguiente consulta de KQL para obtener un grafo de los incidentes de Microsoft 365 Defender entrantes:

```kusto
let Now = now(); 
(range TimeGenerated from ago(14d) to Now-1d step 1d 
| extend Count = 0 
| union isfuzzy=true ( 
    SecurityIncident
    | where ProviderName == "Microsoft 365 Defender"
    | summarize Count = count() by bin_at(TimeGenerated, 1d, Now) 
) 
| summarize Count=max(Count) by bin_at(TimeGenerated, 1d, Now) 
| sort by TimeGenerated 
| project Value = iff(isnull(Count), 0, Count), Time = TimeGenerated, Legend = "Events") 
| render timechart 
```

Use la siguiente consulta de KQL para generar un grafo de volumen de eventos para una sola tabla (cambie la tabla *DeviceEvents* por la tabla necesaria que prefiera):

```kusto
let Now = now();
(range TimeGenerated from ago(14d) to Now-1d step 1d
| extend Count = 0
| union isfuzzy=true (
    DeviceEvents
    | summarize Count = count() by bin_at(TimeGenerated, 1d, Now)
)
| summarize Count=max(Count) by bin_at(TimeGenerated, 1d, Now)
| sort by TimeGenerated
| project Value = iff(isnull(Count), 0, Count), Time = TimeGenerated, Legend = "Events")
| render timechart
```

En la pestaña **Pasos siguientes** encontrará algunos libros útiles, consultas de ejemplo y plantillas de reglas de análisis que se han incluido. Puede ejecutarlas en el acto o modificarlas y guardarlas.

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha obtenido información sobre cómo integrar incidentes de Microsoft 365 Defender y datos de eventos de búsqueda avanzada de Microsoft Defender para punto de conexión en Azure Sentinel mediante el conector Microsoft 365 Defender. Para más información sobre Azure Sentinel, consulte los siguientes artículos:

- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](./tutorial-detect-threats-built-in.md).
