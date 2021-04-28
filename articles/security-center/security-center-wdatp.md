---
title: Uso de la licencia de Microsoft Defender para punto de conexión incluida con Azure Security Center
description: Obtenga información acerca de Microsoft Defender para punto de conexión e implemente esta solución desde Azure Security Center.
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 04/19/2021
ms.author: memildin
ms.openlocfilehash: e12578fa6da679587d41fb25b17b00eb1645299a
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107718420"
---
# <a name="protect-your-endpoints-with-security-centers-integrated-edr-solution-microsoft-defender-for-endpoint"></a>Proteja los puntos de conexión con la solución EDR integrada de Security Center: Microsoft Defender para punto de conexión

Microsoft Defender para punto de conexión es una solución integral de seguridad de punto de conexión que se entrega en la nube. Sus principales características son:

- Evaluación y administración de vulnerabilidades basadas en riesgos. 
- Reducción de la superficie expuesta a ataques
- Protección basada en el comportamiento y con tecnología de nube.
- Detección y respuesta de puntos de conexión (EDR).
- Investigación y corrección automatizadas.
- Servicios de búsqueda administrados.

> [!TIP]
> Originalmente se inició como **ATP de Windows Defender**, pero se cambió el nombre de este producto de Detección y respuesta de puntos de conexión (EDR) en 2019 a **ATP de Microsoft Defender**.
>
> En Ignite 2020, lanzamos el [conjunto Microsoft Defender XDR](https://www.microsoft.com/security/business/threat-protection) y se cambió el nombre del componente EDR a **Microsoft Defender para punto de conexión**.


## <a name="availability"></a>Disponibilidad

| Aspecto                          | Detalles                                                                                                                                                                                                                                                                                                       |
|---------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Estado de la versión:                  | Disponible con carácter general                                                                                                                                                                                                                                                                                      |
| Precios:                        | Requiere [Azure Defender para servidores](defender-for-servers-introduction.md).                                                                                                                                                                                                                                             |
| Plataformas compatibles:            |  • Máquinas de Azure que ejecutan Windows.<br> • Máquinas de Azure Arc que ejecutan Windows.|
| Versiones compatibles de Windows para la detección:  |  • Windows Server 2019, 2016, 2012 R2 y 2008 R2 SP1<br> • [Windows Virtual Desktop (WVD)](../virtual-desktop/overview.md)<br> • [Windows 10 Enterprise multi-session](../virtual-desktop/windows-10-multisession-faq.yml) (antes Enterprise for Virtual Desktops (EVD)|
| Sistemas operativos no admitidos:  |  • Windows 10 (excepto EVD o WVD)<br> • Linux|
| Roles y permisos necesarios: | Para habilitar o deshabilitar la integración: **Administrador de seguridad** o **Propietario**<br>Para ver las alertas de MDATP en Security Center: **Lector de seguridad**, **Lector**, **Colaborador del grupo de recursos**, **Propietario del grupo de recursos**, **Administrador de seguridad**, **Propietario de la suscripción** o **Colaborador de la suscripción**|
| Nubes:                         | ![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) US Gov<br>![No](./media/icons/no-icon.png) China Gov, otros gobiernos                                                        |
|                                 |                                                                                                                                                                                                                                                                                                               |

## <a name="microsoft-defender-for-endpoint-features-in-security-center"></a>Características de Microsoft Defender para punto de conexión en Security Center

Microsoft Defender para punto de conexión proporciona lo siguiente:

- **Sensores de detección avanzados posteriores a las brechas**. Los sensores de Defender para punto de conexión para máquinas de Windows recopilan una amplia gama de señales de comportamiento.

- **Detección posterior a las brechas basada en el análisis y con tecnología de nube**. Defender para punto de conexión se adapta rápidamente a las amenazas cambiantes. Utiliza análisis avanzados y macrodatos. Asimismo, se amplía gracias a la eficacia de Intelligent Security Graph con señales a través de Windows, Azure y Office para detectar amenazas desconocidas. Proporciona alertas que requieren intervención y le permite responder rápidamente.

- **Información sobre amenazas**. Defender para punto de conexión crea alertas cuando identifica las herramientas, las técnicas y los procedimientos de los atacantes. Usa datos generados por los equipos de seguridad y los buscadores de amenazas de Microsoft, con la ayuda de mecanismos de inteligencia que aportan los partners.

Mediante la integración de Defender para punto de conexión con Security Center, también puede beneficiarse de las siguientes funcionalidades adicionales:

- **Incorporación automatizada**. Security Center habilita automáticamente el sensor de Microsoft Defender para punto de conexión en todos los servidores Windows que supervisa Security Center.

- **Panel único**. la consola de Security Center muestra las alertas de Microsoft Defender para punto de conexión. Para obtener más detalles, use las páginas del portal de Microsoft Defender para punto de conexión, donde verá información adicional, como el árbol de procesos de alerta y el gráfico de incidentes. También puede ver una escala de tiempo de la máquina detallada que muestra cada comportamiento durante un período histórico de hasta seis meses.

    :::image type="content" source="./media/security-center-wdatp/microsoft-defender-security-center.png" alt-text="Security Center en Microsoft Defender para punto de conexión" lightbox="./media/security-center-wdatp/microsoft-defender-security-center.png":::

## <a name="microsoft-defender-for-endpoint-tenant-location"></a>Ubicación del inquilino de Microsoft Defender para punto de conexión

Al usar Azure Security Center para supervisar los servidores, se crea automáticamente un inquilino Microsoft Defender para punto de conexión. Los datos que recopila Defender para punto de conexión se almacenan en la ubicación geográfica del inquilino identificada durante el aprovisionamiento. Los datos de cliente en formato seudonimizado también se pueden almacenar en los sistemas de procesamiento y almacenamiento central en el Estados Unidos. 

Una vez configurada la ubicación, no se puede cambiar. Si tiene su propia licencia de Microsoft Defender para el punto de conexión y necesita trasladar los datos a otra ubicación, póngase en contacto con Soporte técnico de Microsoft para restablecer el inquilino.


## <a name="enable-the-microsoft-defender-for-endpoint-integration"></a>Habilitación de la integración de Microsoft Defender para punto de conexión

### <a name="prerequisites"></a>Requisitos previos

Compruebe que el equipo cumple los requisitos necesarios para Defender para punto de conexión:

1. Configure las opciones de red descritas en [Configurar el proxy de dispositivo y la configuración de conectividad a Internet](/windows/security/threat-protection/microsoft-defender-atp/configure-proxy-internet).
1. Si va a implementar Defender para punto de conexión en máquinas locales, conéctelo a Azure Arc como se explica en [Conexión de una máquina híbrida con servidores habilitados para Azure Arc](../azure-arc/servers/learn/quick-enable-hybrid-vm.md).
1. En el caso de las máquinas con Windows Server 2019 únicamente, confirme que ejecuten un agente válido y que tienen la extensión MicrosoftMonitoringAgent.
1. Habilite **Azure Defender para los servidores**. Consulte [Inicio rápido: Habilitación de Azure Defender](enable-azure-defender.md).
1. Si ya ha obtenido una licencia e implementado Microsoft defender para punto de conexión en sus servidores, quítelo mediante el procedimiento descrito en la [Retirada de servidores de Windows](/windows/security/threat-protection/microsoft-defender-atp/configure-server-endpoints#offboard-windows-servers).
1. Si ha movido la suscripción entre inquilinos de Azure, también se requieren algunos pasos de preparación manuales. Para obtener más información, [póngase en contacto con el soporte técnico de Microsoft](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).


### <a name="enable-the-integration"></a>Habilitación de la integración
1. En el menú de Security Center, seleccione **Precios y configuración**, y seleccione la suscripción que quiere cambiar.
1. Seleccione **Detección de amenazas**.
1. Seleccione **Permitir que Microsoft Defender for Endpoint acceda a mis datos** y luego **Guardar**.

    :::image type="content" source="./media/security-center-wdatp/enable-integration-with-edr.png" alt-text="Habilitar la integración entre Azure Security Center y la solución EDR de Microsoft, Microsoft Defender para punto de conexión":::

    Azure Security Center incorporará automáticamente los servidores en Microsoft Defender para punto de conexión. Este proceso de incorporación puede tardar hasta 24 horas.


## <a name="access-the-microsoft-defender-for-endpoint-portal"></a>Acceso al portal de Microsoft Defender para punto de conexión

1. Asegúrese de que la cuenta de usuario tenga los permisos necesarios. Obtenga más información en [Asignar acceso de usuario al Centro de seguridad de Microsoft Defender](/windows/security/threat-protection/microsoft-defender-atp/assign-portal-access).

1. Compruebe si tiene un proxy o firewall que esté bloqueando el tráfico anónimo. El sensor de Defender para punto de conexión se conecta desde el contexto del sistema, por lo que se debe permitir el tráfico anónimo. Para garantizar un acceso sin obstáculos al portal de Defender para punto de conexión, siga las instrucciones de [Habilitar el acceso a las direcciones URL de servicio en el servidor proxy](/windows/security/threat-protection/microsoft-defender-atp/configure-proxy-internet#enable-access-to-microsoft-defender-atp-service-urls-in-the-proxy-server).

1. Abra el [portal del Centro de seguridad de Microsoft Defender](https://securitycenter.windows.com/). Obtenga más información sobre las características e iconos del portal, en [Información general del portal del Centro de seguridad de Microsoft Defender](/windows/security/threat-protection/microsoft-defender-atp/portal-overview). 

## <a name="send-a-test-alert"></a>Envío de una alerta de prueba

Para generar una alerta de prueba benigna de Microsoft Defender para punto de conexión:

1. Cree una carpeta "C:\test-MDATP-test".
1. Use el Escritorio remoto para acceder a su máquina.
1. Abra una ventana de línea de comandos.
1. En el símbolo del sistema, copie el siguiente comando y ejecútelo. La ventana del símbolo del sistema se cerrará automáticamente.

    ```powershell
    powershell.exe -NoExit -ExecutionPolicy Bypass -WindowStyle Hidden (New-Object System.Net.WebClient).DownloadFile('http://127.0.0.1/1.exe', 'C:\\test-MDATP-test\\invoice.exe'); Start-Process 'C:\\test-MDATP-test\\invoice.exe'
    ```
    :::image type="content" source="./media/security-center-wdatp/generate-edr-alert.png" alt-text="Ventana del símbolo del sistema con el comando para generar una alerta de prueba.":::

1. Si el comando se ejecuta correctamente, verá una nueva alerta en el panel de Azure Security Center y el portal de Microsoft Defender para punto de conexión. Esta alerta puede tardar unos minutos en aparecer.
1. Para revisar la alerta en Security Center, vaya a **Security Alerts (Alertas de seguridad)**  > **Suspicious Powershell CommandLine (Línea de comandos de PowerShell sospechosa)** .
1. En la ventana de la investigación, seleccione el vínculo para ir al portal de Microsoft Defender para punto de conexión.

    > [!TIP]
    > La alerta se desencadena con gravedad **Información**.

## <a name="faq-for-security-centers-integrated-microsoft-defender-for-endpoint"></a>Preguntas más frecuentes sobre la instancia de Microsoft Defender para punto de conexión integrada en Security Center

- [¿Cuáles son los requisitos de licencia de Microsoft Defender para punto de conexión?](#what-are-the-licensing-requirements-for-microsoft-defender-for-endpoint)
- [Si tengo una licencia de Microsoft Defender para punto de conexión, ¿puedo obtener un descuento en Azure Defender?](#if-i-already-have-a-license-for-microsoft-defender-for-endpoint-can-i-get-a-discount-for-azure-defender)
- [¿Cómo puedo cambiar una herramienta de EDR de terceros?](#how-do-i-switch-from-a-third-party-edr-tool)

### <a name="what-are-the-licensing-requirements-for-microsoft-defender-for-endpoint"></a>¿Cuáles son los requisitos de licencia de Microsoft Defender para punto de conexión?
Defender para punto de conexión se incluye sin costo adicional en los **servidores de Azure Defender**. Como alternativa, se puede adquirir por separado para 50 máquinas o más.

### <a name="if-i-already-have-a-license-for-microsoft-defender-for-endpoint-can-i-get-a-discount-for-azure-defender"></a>Si tengo una licencia de Microsoft Defender para punto de conexión, ¿puedo obtener un descuento en Azure Defender?
Si ya tiene una licencia de Microsoft Defender para punto de conexión, no tendrá que pagar esa parte de la licencia de Azure Defender.

Para confirmar su descuento, póngase en contacto con el equipo de soporte técnico de Security Center y especifique el identificador de área de trabajo, la región y la información de licencia pertinentes de cada licencia.

### <a name="how-do-i-switch-from-a-third-party-edr-tool"></a>¿Cómo puedo cambiar una herramienta de EDR de terceros?
Las instrucciones completas para cambiarse de una solución de punto de conexión que no es de Microsoft están disponibles en la documentación de Microsoft Defender para punto de conexión: [Información general sobre la migración](/windows/security/threat-protection/microsoft-defender-atp/switch-to-microsoft-defender-migration).
  


## <a name="next-steps"></a>Pasos siguientes

- [Características y plataformas compatibles con Azure Security Center](security-center-os-coverage.md)
- [Administración de recomendaciones de seguridad en el Centro de seguridad de Azure](security-center-recommendations.md): aprenda cómo las recomendaciones le ayudarán a proteger los recursos de Azure.