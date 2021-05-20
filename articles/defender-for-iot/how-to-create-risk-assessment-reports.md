---
title: Creación de informes de evaluación de riesgos
description: Conozca los riesgos de red detectados por sensores individuales o una vista agregada de los riesgos detectados por todos los sensores.
ms.date: 12/17/2020
ms.topic: how-to
ms.openlocfilehash: 567660d1e1e55ee3dde0cbf5e5c09748cbb2cf0c
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109655343"
---
# <a name="risk-assessment-reporting"></a>Informes de evaluación de riesgos

## <a name="about-risk-assessment-reports"></a>Acerca de los informes de evaluación de riesgos

Los informes de evaluación de riesgos proporcionan:

- Una puntuación de seguridad general para los dispositivos detectados por los sensores de la organización.

- Una puntuación de seguridad para cada dispositivo de red detectado por un sensor individual.

- Un desglose del número de dispositivos vulnerables, dispositivos que necesitan mejoras y dispositivos seguros.

-  Información sobre problemas de seguridad y operativos:

    - Problemas de configuración

    - Vulnerabilidad del dispositivo priorizada por nivel de seguridad

    - Problemas de seguridad de red

    - Problemas operativos de red

    - Conexiones a redes ICS

    - Conexiones a Internet

    - Indicadores de malware industrial

    - Problemas de protocolo

    - Vectores de ataque

### <a name="risk-mitigation"></a>Mitigación de riesgos

Los informes ofrecen recomendaciones para ayudarle a mejorar la puntuación de seguridad. Por ejemplo, instale las actualizaciones de seguridad más recientes, actualice el firmware a la versión más reciente o realice un seguimiento de las alertas.

## <a name="about-security-scores"></a>Acerca de las puntuaciones de seguridad

La puntuación total de seguridad de red se genera en cada informe. La puntuación representa el porcentaje del 100 % de seguridad. Por ejemplo, una puntuación del 30 % indicaría que el 30 % de la red es seguro.

Las puntuaciones de evaluación de riesgos se basan en la información obtenida de la inspección de paquetes, los motores de modelado de comportamiento y un diseño de máquina de estados específico de SCADA.

**Dispositivos seguros** son aquellos dispositivos con una puntuación de seguridad superior al 90 %.

**Dispositivos que necesitan mejorar**: dispositivos con una puntuación de seguridad entre el 70 y el 89 %.

**Dispositivos vulnerables** son aquellos con una puntuación de seguridad inferior al 70 %.

### <a name="about-backup-and-anti-virus-servers"></a>Acerca de las copias de seguridad y los servidores de antivirus

La puntuación de evaluación de riesgos puede verse afectada si no define direcciones de copia de seguridad y servidor de antivirus en el sensor. Agregar estas direcciones mejora la puntuación. De manera predeterminada, estas direcciones no están definidas.
La portada del informe Evaluación de riesgos indicará si no están definidos los servidores de copia de seguridad y los servidores de antivirus.

**Para agregar los servidores:**

1. Seleccione **Configuración del sistema** y, luego, elija **Propiedades del sistema**.
1. Seleccione **Evaluación de vulnerabilidad** y agregue las direcciones a los campos **backup_servers** y **AV_addresses**. Si agregará varias direcciones,  escríbalas separadas por comas.  
1. Seleccione **Guardar**.
## <a name="create-risk-assessment-reports"></a>Creación de informes de evaluación de riesgos

Cree un informe de evaluación de riesgos en PDF. El nombre del informe se genera automáticamente como risk-assessment-report-1.pdf. El número se actualizará con cada nuevo informe que cree.  Se muestran la hora y el día de la creación.

### <a name="create-a-sensor-risk-assessment-report"></a>Creación de un informe de evaluación de riesgos de un sensor

Cree un informe de evaluación de riesgos basado en las detecciones realizadas por el sensor en el que ha iniciado sesión.

Para crear un informe:

1. Inicie sesión en la consola del sensor.
1. Seleccione **Evaluación de riesgos** en el menú lateral.
1. Seleccione **Generar informe**. El informe aparece en la sección de informes archivados.
1. Seleccione el informe desde la sección de informes archivados para descargarlo.

:::image type="content" source="media/how-to-generate-reports/risk-assessment.png" alt-text="Vista de la evaluación de riesgos.":::

Para importar un logotipo de empresa:

- Seleccione **Import Logo** (Importar logotipo).

### <a name="create-an-on-premises-management-console-risk-assessment-report"></a>Creación de un informe de evaluación de riesgos de la consola de administración local

Cree un informe de evaluación de riesgos basado en las detecciones realizadas por cualquiera de los sensores administrados por la consola de administración local. 

Para crear un informe:

1. Seleccione **Evaluación de riesgos** en el menú lateral.

2. Seleccione un sensor en la lista desplegable **Select Sensor** (Seleccionar sensor).

3. Seleccione **Generar informe**.

4. Seleccione **Descargar** desde la sección **Informes archivados**.

Para importar un logotipo de empresa:

- Seleccione **Import Logo** (Importar logotipo).

:::image type="content" source="media/how-to-generate-reports/import-logo-screenshot.png" alt-text="Importar un logotipo a través de la vista de evaluación de riesgos.":::

## <a name="see-also"></a>Consulte también

[Informe de vectores de ataque](how-to-create-attack-vector-reports.md)

