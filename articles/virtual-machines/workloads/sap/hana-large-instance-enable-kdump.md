---
title: Script para habilitar Kdump en SAP HANA (instancias grandes) | Microsoft Docs
description: Script para habilitar Kdump en SAP HANA (instancias grandes), HLI tipo I, HLI tipo II
services: virtual-machines-linux
documentationcenter: ''
author: prtyag
manager: hrushib
editor: ''
ms.service: virtual-machines-sap
ms.subservice: baremetal-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/30/2020
ms.author: prtyag
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c33288f80a6375b6a0c5e77d928e4314147efe0a
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110577316"
---
# <a name="kdump-for-sap-hana-on-azure-large-instances-hli"></a>Kdump para SAP HANA en Azure (instancias grandes)

La configuración y habilitación de Kdump es un paso necesario para solucionar los problemas de bloqueos del sistema que no tienen una causa evidente.
Hay ocasiones en las que un sistema se bloquea inesperadamente, pero no se puede explicar con un problema de hardware o infraestructura.
En estos casos, puede tratarse de un problema del sistema operativo o de la aplicación, y Kdump permitirá a SUSE determinar por qué se bloqueó un sistema.

## <a name="enable-kdump-service"></a>Habilitar el servicio Kdump

En este documento se describen los detalles sobre cómo habilitar el servicio Kdump en una instancia grande de HANA en Azure (**tipo I y tipo II**).

## <a name="supported-skus"></a>SKU compatibles

|  Tipo de instancia grande de HANA   |  Proveedor del SO   |  Versión del paquete del SO   |  SKU |
|-----------------------------|--------------|-----------------------|-------------|
|   Tipo I                    |  SuSE        |   SLES 12 SP3         |  S224 m      |
|   Tipo I                    |  SuSE        |   SLES 12 SP4         |  S224 m      |
|   Tipo I                    |  SuSE        |   SLES 12 SP2         |  S72        |
|   Tipo I                    |  SuSE        |   SLES 12 SP2         |  S72m       |
|   Tipo I                    |  SuSE        |   SLES 12 SP3         |  S72m       |
|   Tipo I                    |  SuSE        |   SLES 12 SP2         |  S96        |
|   Tipo I                    |  SuSE        |   SLES 12 SP3         |  S96        |
|   Tipo I                    |  SuSE        |   SLES 12 SP2         |  S192       |
|   Tipo I                    |  SuSE        |   SLES 12 SP3         |  S192       |
|   Tipo I                    |  SuSE        |   SLES 12 SP4         |  S192       |
|   Tipo I                    |  SuSE        |   SLES 12 SP2         |  S192m      |
|   Tipo I                    |  SuSE        |   SLES 12 SP3         |  S192m      |
|   Tipo I                    |  SuSE        |   SLES 12 SP4         |  S192m      |
|   Tipo I                    |  SuSE        |   SLES 12 SP2         |  S144       |
|   Tipo I                    |  SuSE        |   SLES 12 SP3         |  S144       |
|   Tipo I                    |  SuSE        |   SLES 12 SP2         |  S144m      |
|   Tipo I                    |  SuSE        |   SLES 12 SP3         |  S144m      |
|   Tipo II                   |  SuSE        |   SLES 12 SP2         |  S384       |
|   Tipo II                   |  SuSE        |   SLES 12 SP3         |  S384       |
|   Tipo II                   |  SuSE        |   SLES 12 SP4         |  S384       |
|   Tipo II                   |  SuSE        |   SLES 12 SP2         |  S384xm     |
|   Tipo II                   |  SuSE        |   SLES 12 SP3         |  S384xm     |
|   Tipo II                   |  SuSE        |   SLES 12 SP4         |  S384xm     |
|   Tipo II                   |  SuSE        |   SLES 12 SP2         |  S576m      |
|   Tipo II                   |  SuSE        |   SLES 12 SP3         |  S576m      |
|   Tipo II                   |  SuSE        |   SLES 12 SP4         |  S576m      |

## <a name="prerequisites"></a>Requisitos previos

- El servicio Kdump usa el directorio `/var/crash` para escribir volcados; asegúrese de que la partición correspondiente a este directorio tenga espacio suficiente para alojar los volcados.

## <a name="setup-details"></a>Detalles de la configuración

- El script para habilitar Kdump se puede encontrar [aquí](https://github.com/Azure/sap-hana-tools/blob/master/tools/enable-kdump.sh).
> [!NOTE]
> Este script está basado en la configuración del laboratorio y se espera que el cliente se ponga en contacto con el proveedor del sistema operativo para cualquier optimización adicional.
> Se va a aprovisionar un número de unidad lógica independiente para los servidores nuevos y existentes con el fin de guardar los volcados de memoria, y el script se encargará de configurar el sistema de archivos a partir del número de unidad lógica.
> Microsoft no será responsable del análisis del volcado de memoria. El cliente tiene que abrir un vale con el proveedor del sistema operativo para que lo analice.

- Ejecute este script en la instancia grande de HAN con el domando siguiente.

    > [!NOTE]
    > Se necesita el privilegio sudo para ejecutar este comando.

    ```bash
    sudo bash enable-kdump.sh
    ```

- Si el comando que genera Kdump está habilitado correctamente, asegúrese de reiniciar el sistema para aplicar los cambios correctamente.

- Si el resultado del comando no puede realizar una operación determinada, como Exiting!!!!, el servicio Kdump no está habilitado. Consulte la sección [Problema de soporte técnico](#support-issue).

## <a name="test-kdump"></a>Prueba de Kdump

> [!NOTE]
>  La operación siguiente desencadenará un bloqueo del kernel y el reinicio del sistema.

- Desencadenar un bloqueo del kernel

    ```bash
    echo c > /proc/sysrq-trigger
    ```

- Una vez que el sistema se reinicie correctamente, compruebe el directorio `/var/crash` para buscar los registros de bloqueo del kernel.

- Si `/var/crash` tiene un directorio con la fecha actual, el Kdump se habilita correctamente.

## <a name="support-issue"></a>Problema de soporte técnico

Si el script produce un error o el Kdump no está habilitado, envíe una solicitud de servicio al equipo de soporte técnico de Microsoft con los detalles siguientes.

* Identificador de la suscripción de HLI

* Nombre de servidor

* Proveedor del SO

* Versión del SO

* Versión del kernel

## <a name="related-documents"></a>Documentos relacionados
- Para obtener más información sobre cómo [configurar Kdump](https://www.suse.com/support/kb/doc/?id=3374462)
