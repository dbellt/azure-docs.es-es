---
title: 'Inicio rápido: Incorporación en Azure Sentinel'
description: En este inicio rápido, aprenderá cómo incorporar Azure Sentinel habilitándolo primero y, a continuación, conectando los orígenes de datos.
services: sentinel
author: yelevin
ms.author: yelevin
ms.assetid: d5750b3e-bfbd-4fa0-b888-ebfab7d9c9ae
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: quickstart
ms.date: 10/14/2020
ms.custom: references_regions
ms.openlocfilehash: b30fee5d269a633173b8524ed6c902517063b2a2
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111986001"
---
# <a name="quickstart-on-board-azure-sentinel"></a>Inicio rápido: Incorporación de Azure Sentinel

En este tutorial de inicio rápido, aprenderá cómo incorporar Azure Sentinel. 

Para incorporarse a Azure Sentinel, primero debe habilitarlo y, después, conectar sus orígenes de datos. Azure Sentinel incluye varios conectores para soluciones de Microsoft, que están disponibles inmediatamente y proporcionan integración en tiempo real, entre las que se incluyen las soluciones de Microsoft 365 Defender (anteriormente, Protección contra amenazas de Microsoft), orígenes de Microsoft 365 (como Office 365), Azure AD, Microsoft Defender for Identity (anteriormente, Azure ATP), Microsoft Cloud App Security, alertas de Azure Defender desde Azure Security Center y más. Además, hay conectores integrados al amplio ecosistema de seguridad para soluciones que no son de Microsoft. También puede usar el formato de evento común (CEF), Syslog o la API de REST para conectar los orígenes de datos con Azure Sentinel. 

Después de conectar los orígenes de datos, puede elegir de una galería de libros creados de forma experta que exponen información basada en los datos. Estos libros se pueden personalizar fácilmente en función de sus necesidades.

>[!IMPORTANT] 
> Para obtener información sobre los cargos que se producen al usar Azure Sentinel, consulte los [precios de Azure Sentinel](https://azure.microsoft.com/pricing/details/azure-sentinel/).

## <a name="global-prerequisites"></a>Requisitos previos globales

- Suscripción activa de Azure. Si no tiene una, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de comenzar.

- Área de trabajo de Log Analytics. Aprenda a [crear un área de trabajo de Log Analytics](../azure-monitor/logs/quick-create-workspace.md). Para obtener más información sobre las áreas de trabajo de Log Analytics, consulte [Diseño de su implementación de Azure Monitor Logs](../azure-monitor/logs/design-logs-deployment.md).

- Para habilitar Azure Sentinel, necesita permisos de colaborador en la suscripción en la que reside el área de trabajo de Azure Sentinel. 
- Para usar Azure Sentinel, necesita permisos de colaborador o lector en el grupo de recursos al que pertenece el área de trabajo.
- Es posible que se necesiten permisos adicionales para conectarse a orígenes de datos específicos.
- Azure Sentinel es un servicio de pago. Para más información sobre precios, consulte [Acerca de Azure Sentinel](https://go.microsoft.com/fwlink/?linkid=2104058).

### <a name="geographical-availability-and-data-residency"></a>Disponibilidad geográfica y residencia de datos

- Azure Sentinel se puede ejecutar en áreas de trabajo de las [regiones con disponibilidad general de Log Analytics](https://azure.microsoft.com/global-infrastructure/services/?products=monitor), excepto en China y Alemania (soberana). A veces, las regiones nuevas de Log Analytics pueden tardar algún tiempo en incorporarse al servicio Azure Sentinel. 

- Los datos que genera Azure Sentinel, como incidentes, marcadores y reglas de análisis, pueden contener datos del cliente procedentes de las áreas de trabajo de Log Analytics del cliente. Estos datos generados por Azure Sentinel se guardan en la geografía o región que se muestran en la tabla siguiente, según la geografía o región en que se encuentra el área de trabajo:

    | Geografía/región del área de trabajo | Geografía/región de datos generados por Azure Sentinel |
    | --- | --- |
    | Estados Unidos<br>India<br>Brasil<br>África<br>Corea<br>Emiratos Árabes Unidos | Estados Unidos |
    | Europa<br>Francia<br>Suiza | Europa |
    | Australia | Australia |
    | Reino Unido | Reino Unido |
    | Canadá | Canadá |
    | Japón | Japón |
    | Sudeste Asiático (Singapur) | Sudeste Asiático (Singapur)* |
    |
    
    \* No hay ninguna región emparejada para Sudeste Asiático.

    > [!IMPORTANT]
    > - Al habilitar ciertas reglas que usan el motor de aprendizaje automático (ML), **concede a Microsoft permiso para copiar los datos ingeridos pertinentes fuera de la geografía del área de trabajo de Azure Sentinel**, según sea necesario para que el motor de aprendizaje automático procese estas reglas.

## <a name="enable-azure-sentinel"></a>Habilitar Azure Sentinel <a name="enable"></a>

1. Inicie sesión en Azure Portal. Asegúrese de que la suscripción en la que se crea Azure Sentinel está seleccionada.

1. Busque y seleccione **Azure Sentinel**.

   ![Búsqueda de servicios](./media/quickstart-onboard/search-product.png)

1. Seleccione **Agregar**.

1. Seleccione el área de trabajo que quiere usar o cree una nueva. Puede ejecutar Azure Sentinel en más de un área de trabajo, pero los datos se aíslan en un área de trabajo.

   ![Elegir un área de trabajo](./media/quickstart-onboard/choose-workspace.png)

   >[!NOTE] 
   > - Las áreas de trabajo predeterminadas creadas por Azure Security Center no aparecerán en la lista; no puede instalar Azure Sentinel en ellas.
   >

   >[!IMPORTANT]
   >
   > - Una vez implementado en un área de trabajo, Azure Sentinel **no admite actualmente** el movimiento de esa área de trabajo a otros grupos de recursos o suscripciones. 
   >
   >   Si ya ha movido el área de trabajo, deshabilite todas las reglas activas en **Análisis** y vuelva a habilitarlas después de cinco minutos. Esto debe ser efectivo en la mayoría de los casos; sin embargo, cabe reiterar que no se admite y se realiza bajo su responsabilidad.

1. Seleccione **Agregar Azure Sentinel**.

## <a name="connect-data-sources"></a>Conexión con orígenes de datos

Azure Sentinel ingiere datos de servicios y aplicaciones mediante la conexión y el reenvío de los eventos y registros a Azure Sentinel. Tanto en las máquinas físicas como en las virtuales, puede instalar el agente de Log Analytics que recopila los registros y los reenvía a Azure Sentinel. En el caso de los firewalls y servidores proxy, Azure Sentinel instala el agente de Log Analytics en un servidor de Syslog de Linux, desde el que el agente recopila los archivos de registro y los reenvía a Azure Sentinel. 
 
1. En el menú principal, seleccione **Data connectors** (Conectores de datos). Se abre la galería de conectores de datos.

1. La galería de es una lista de todos los orígenes de datos que se pueden conectar. Seleccione un origen de datos y, después, el botón **Open connector page** (Abrir página del conector).

1. En la página del conector encontrará instrucciones para configurar el conector, así como otras instrucciones adicionales que pueda necesitar.<br>
Por ejemplo, si selecciona el origen de datos **Azure Active Directory**, que permite transmitir registros de Azure AD a Azure Sentinel, puede seleccionar el tipo de registros en los que desea obtener (registros de inicio de sesión o registros de auditoría). <br> Siga las instrucciones de instalación o [consulte la guía de conexión relevante](connect-data-sources.md) para obtener más información. Para obtener información acerca de los conectores de datos, vea [Conexión de servicios de Microsoft](connect-data-sources.md).

1. En la pestaña **Pasos siguientes** de la página del conector se muestran los libros integrados, las consultas de ejemplo y las plantillas de reglas de análisis pertinentes que acompañan al conector de datos. Puede usarlos tal cual o modificarlos (de cualquiera de las dos formas puede obtener información interesante en los datos). <br>

Una vez conectados los orígenes de datos, los datos comienzan a transmitirse a Azure Sentinel y podrá comenzar a trabajar con ellos. Puede ver los registros en los [libros integrados](quickstart-get-visibility.md) y comenzar a crear consultas en Log Analytics para [investigar los datos](tutorial-investigate-cases.md).

## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido a incorporar y conectar orígenes de datos en Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).
- Transmita datos desde [dispositivos de formato de evento común](connect-common-event-format.md) a Azure Sentinel.
