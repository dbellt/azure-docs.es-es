---
title: 'Integridad del código de plataforma: seguridad de Azure'
description: Obtenga información sobre cómo Microsoft garantiza que solo se ejecuta software autorizado.
author: yosharm
ms.service: security
ms.subservice: security-fundamentals
ms.topic: article
ms.author: terrylan
manager: rkarlin
ms.date: 06/10/2021
ms.openlocfilehash: e9a58bf83181352adb3894fb159fd212f0f1b7ca
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112063806"
---
# <a name="platform-code-integrity"></a>Integridad del código de plataforma

Un desafío importante para el funcionamiento de un sistema complejo como Microsoft Azure es garantizar que solo se ejecuta software autorizado en el sistema. El software no autorizado presenta varios riesgos para cualquier empresa:

- Riesgos de seguridad, como herramientas de ataque dedicadas, malware personalizado y software de terceros con vulnerabilidades conocidas
- Riesgos de cumplimiento cuando el proceso de administración de cambios aprobado no se usa para traer software nuevo
- Riesgo de calidad del software desarrollado externamente, que puede no cumplir los requisitos operativos de la empresa

En Azure, nos enfrentamos al mismo desafío y a una complejidad considerable. Tenemos miles de servidores que ejecutan software desarrollado y mantenido por miles de ingenieros. Esto representa una gran superficie de ataque que no se puede administrar solo a través de procesos empresariales.

## <a name="adding-an-authorization-gate"></a>Adición de una puerta de autorización

Azure usa un completo proceso de ingeniería que implementa puertas sobre la seguridad, el cumplimiento y la calidad del software que implementamos. Este proceso incluye el control de acceso al código fuente y la realización de revisiones de código del mismo nivel y de análisis estáticos de vulnerabilidades de seguridad, de acuerdo con el [Ciclo de vida de desarrollo de seguridad (SDL)](https://www.microsoft.com/securityengineering/sdl/) de Microsoft y la realización de pruebas funcionales y de calidad. Debemos garantizar que el software que implementamos ha fluido a lo largo de este proceso. La integridad del código nos permite ofrecer esa garantía.

## <a name="code-integrity-as-an-authorization-gate"></a>Integridad del código como una puerta de autorización

La integridad de código es un servicio de nivel de kernel que empezó a estar disponible a partir de Windows Server 2016. La integridad del código puede aplicar una directiva de control de ejecución estricta cuando se carga un controlador o una biblioteca vinculada dinámicamente (DLL), se ejecuta un archivo binario ejecutable o se ejecuta un script. Existen sistemas similares, como [DM-Verity,](https://www.kernel.org/doc/html/latest/admin-guide/device-mapper/verity.html) para Linux. Una directiva de integridad de código consta de un conjunto de indicadores de autorización, ya sea certificados de firma de código o hashes de archivo [SHA256](https://en.wikipedia.org/wiki/Secure_Hash_Algorithms), que el kernel hace coincidir antes de cargar o ejecutar un archivo binario o un script.

La integridad de código permite a un administrador del sistema definir una directiva que solo autoriza los archivos binarios y scripts firmados por certificados determinados o que coinciden con los hash SHA256 especificados. El kernel aplica esta directiva impidiendo que se ejecute todo aquello que no cumpla la directiva establecida.

Un problema con una directiva de integridad de código es que, a menos que la directiva sea perfectamente correcta, puede bloquear el software crítico en producción y provocar una interrupción. Dada esta preocupación, puede que se pregunte por qué no es suficiente usar la supervisión de seguridad para detectar cuándo se ha ejecutado software no autorizado. La integridad del código tiene un modo de auditoría que, en lugar de impedir la ejecución, puede alertar de cuando se ejecuta software no autorizado. Sin duda, las alertas pueden agregar mucho valor a la hora de abordar los riesgos de cumplimiento, pero, en el caso de los riesgos de seguridad como ransomware o malware personalizado, retrasar la respuesta incluso unos segundos puede ser la diferencia entre la protección y un adversario que obtiene un apoyo persistente en su conjunto de dispositivos. En Azure, hemos invertido significativamente en administrar cualquier riesgo de integridad de código que contribuya a una interrupción que afecta al cliente.

## <a name="build-process"></a>Proceso de compilación

Como se ha descrito anteriormente, el sistema de compilación de Azure tiene un amplio conjunto de pruebas para garantizar que los cambios de software son seguros y compatibles. Una vez que una compilación ha progresado a través de la validación, el sistema de compilación la firma mediante un certificado de compilación de Azure. El certificado indica que la compilación ha superado todo el proceso de administración de cambios. La prueba final en la que se somete la compilación se denomina Validación de firma de código (CSV). CSV confirma que los archivos binarios recién creados cumplen la directiva de integridad de código antes de implementarlos en producción. Esto nos proporciona confianza en el hecho de que no provocaremos ninguna interrupción del cliente debido a que los archivos binarios no están firmados correctamente. Si CSV encuentra un problema, la compilación se interrumpe y se llama a los ingenieros pertinentes para investigar y corregir el problema.

## <a name="safety-during-deployment"></a>Seguridad durante la implementación

Aunque realicemos CSV para cada compilación, todavía existe la posibilidad de que algún cambio o incoherencia en producción pueda provocar una interrupción relacionada con la integridad del código. Por ejemplo, una máquina puede ejecutar una versión anterior de la directiva de integridad de código o puede estar en un estado incorrecto que produzca falsos positivos en la integridad del código. (A escala de Azure, lo hemos visto todo). Por lo tanto,debemos continuar protegiéndonos contra el riesgo de una interrupción durante la implementación.

Todos los cambios en Azure se deben implementar mediante una serie de fases. La primera de ellas son instancias de prueba internas de Azure. La siguiente fase solo se usa para atender a otros equipos de productos de Microsoft. La fase final atiende a clientes de terceros. Cuando se implementa un cambio, se pasa a cada una de las fases correspondientes y el proceso se detiene para medir el estado de la fase. Si se encuentra que el cambio no tiene ningún impacto negativo, se pasa a la fase siguiente. Si se realiza un cambio no correcto en una directiva de integridad de código, el cambio se detecta durante esta implementación por fases y se revierte.

## <a name="incident-response"></a>Respuesta a los incidentes

Incluso con esta protección por capas, todavía es posible que algún servidor de la flota bloquee el software autorizado correctamente y cause algún problema a los clientes, lo que representa uno de nuestros peores escenarios. Nuestra capa final de defensa es la investigación humana. Cada vez que la integridad del código bloquea un archivo, genera una alerta para que los ingenieros de llamadas lo investiguen. La alerta nos permite iniciar investigaciones de seguridad e intervenir, si el problema es un indicador de un ataque real, un falso positivo u otra situación que afecta al cliente. De este modo, se minimiza el tiempo necesario para mitigar los problemas relacionados con la integridad del código.  

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo [Windows 10](/windows/security/threat-protection/device-guard/introduction-to-device-guard-virtualization-based-security-and-windows-defender-application-control) usa la integridad de código configurable.

Para más información sobre lo que se hace para fomentar la seguridad y la integridad de la plataforma, vea:

- [Seguridad de firmware](firmware.md)
- [Arranque seguro](secure-boot.md)
- [Atestación de host y de arranque medido](measured-boot-host-attestation.md)
- [Proyecto Cerberus](project-cerberus.md)
- [Cifrado en reposo](encryption-atrest.md)
- [Seguridad de Hypervisor](hypervisor.md)