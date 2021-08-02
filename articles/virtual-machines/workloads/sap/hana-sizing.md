---
title: Ajuste de tamaño de SAP HANA en Azure (instancias grandes) | Microsoft Docs
description: Aprenda sobre el dimensionamiento de SAP HANA en Azure (instancias grandes).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-sap
ms.subservice: baremetal-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/14/2021
ms.author: madhukan
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 5b93e7febc478452d713d115162280f0fd2657a3
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110579469"
---
# <a name="sizing"></a>Ajuste de tamaño

En este artículo, veremos información útil sobre el dimensionamiento de HANA (instancias grandes). En general, el dimensionamiento de HANA (instancias grandes) es igual que para HANA. 

## <a name="moving-an-existing-system-to-sap-hana-large-instances"></a>Movimiento de un sistema existente a SAP HANA (instancias grandes)

Supongamos que quiere mover a HANA un sistema implementado existente desde otro sistema de administración de bases de datos relacionales (RDBMS). SAP proporciona informes para ejecutarse en el sistema SAP existente. Si la base de datos se traslada a HANA, estos informes comprueban los datos y calculan los requisitos de memoria para la instancia de HANA. 

Consulte las siguientes notas de SAP para más información sobre cómo ejecutar estos informes y cómo obtener las revisiones o versiones más recientes:

- [SAP Note #1793345 - Sizing for SAP Suite on HANA](https://launchpad.support.sap.com/#/notes/1793345) (Nota de SAP 1793345: Ajuste de tamaño para SAP Suite en HANA)
- [SAP Note #1872170 - Suite on HANA and S/4 HANA sizing report](https://launchpad.support.sap.com/#/notes/1872170) (Nota de SAP 1872170: Informe de ajuste de tamaño de Suite en HANA y S/4 HANA)
- [Nota de SAP 2121330 - FAQ: SAP BW on HANA Sizing Report](https://launchpad.support.sap.com/#/notes/2121330) (Preguntas más frecuentes: informe de ajuste de tamaño de SAP BW en HANA)
- [SAP Note #1736976 - Sizing Report for BW on HANA](https://launchpad.support.sap.com/#/notes/1736976) (Nota de SAP 1736976: Informe de ajuste de tamaño para BW en HANA)
- [SAP Note #2296290 - New Sizing Report for BW on HANA](https://launchpad.support.sap.com/#/notes/2296290) (Nota de SAP 2296290: Nuevo informe de ajuste de tamaño para BW en HANA)

## <a name="sizing-greenfield-implementations"></a>Dimensionamiento de implementaciones greenfield

Si va a iniciar la implementación desde cero, SAP Quick Sizer calculará los requisitos de memoria de la implementación de software de SAP sobre HANA.

## <a name="memory-requirements"></a>Requisitos de memoria

Los requisitos de memoria de HANA aumentan a medida que aumenta el volumen de datos. Tenga en cuenta el consumo de memoria actual, que le ayudará a predecir lo que va a ocurrir en el futuro. En función de los requisitos de memoria, puede asignar la demanda a una de las SKU de HANA (instancias grandes).

## <a name="next-steps"></a>Pasos siguientes

Conozca los requisitos de incorporación de HANA (instancias grandes).

> [!div class="nextstepaction"]
> [Requisitos de incorporación](hana-onboarding-requirements.md)
