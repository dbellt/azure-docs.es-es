---
title: Ejecución de scripts en una máquina virtual Windows de Azure
description: En este tema se describe cómo ejecutar scripts dentro de una máquina virtual Windows
services: automation
ms.service: virtual-machines
ms.collection: windows
author: bobbytreed
ms.author: robreed
ms.date: 05/02/2018
ms.topic: how-to
manager: carmonm
ms.openlocfilehash: 7bf62eb2ab8d2ce82ce73e3e8ae26cf303b8ba67
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107765888"
---
# <a name="run-scripts-in-your-windows-vm"></a>Ejecución de scripts en una máquina virtual Windows

Para automatizar las tareas o solucionar los problemas, es posible que deba ejecutar comandos en una máquina virtual. El artículo siguiente ofrece una breve introducción a las características disponibles para ejecutar scripts y comandos dentro de las máquinas virtuales.

## <a name="custom-script-extension"></a>Custom Script Extension

La [extensión de scripts personalizados](../extensions/custom-script-windows.md) se usa principalmente para la instalación de software y la configuración después de la implementación.

* Descargue y ejecute scripts en máquinas virtuales de Azure.
* Se puede ejecutar mediante las plantillas de Azure Resource Manager, la CLI de Azure, la API de REST, PowerShell o Azure Portal.
* Los archivos de script se pueden descargar de Azure Storage o GitHub, o bien se pueden proporcionar desde el equipo si la ejecución se realiza desde Azure Portal.
* Ejecute un script de PowerShell en máquinas Windows y un script de Bash en máquinas Linux.
* Es útil para la configuración posterior a la implementación, la instalación de software y otras tareas de configuración o administración.

## <a name="run-command"></a>Comando Ejecutar

La característica [Comando Ejecutar](run-command.md) habilita la administración de máquinas virtuales y aplicaciones y la solución de problemas mediante scripts, y está disponible incluso cuando no se puede acceder a la máquina, por ejemplo, si el firewall invitado no tiene abierto el puerto RDP o SSH.

* Ejecute scripts en máquinas virtuales de Azure.
* Se puede ejecutar mediante [Azure Portal](run-command.md), la [API de REST](/rest/api/compute/virtual%20machines%20run%20commands/runcommand), la [CLI de Azure](/cli/azure/vm/run-command#az_vm_run_command_invoke) o [PowerShell](/powershell/module/az.compute/invoke-azvmruncommand).
* Ejecute un script, vea la salida rápidamente y repita según sea necesario en Azure Portal.
* El script se puede ejecutar directamente o puede ejecutar uno de los scripts integrados.
* Ejecute un script de PowerShell en máquinas Windows y un script de Bash en máquinas Linux.
* Es útil para la administración de máquinas virtuales y aplicaciones y para ejecutar scripts en máquinas virtuales a las que no se puede acceder.

## <a name="hybrid-runbook-worker"></a>Hybrid Runbook Worker

[Hybrid Runbook Worker](../../automation/automation-hybrid-runbook-worker.md) brinda administración general de máquinas, aplicaciones y entornos con scripts personalizados del usuario almacenados en una cuenta de Automation.

* Ejecute scripts en máquinas de Azure y en máquinas que no son de Azure.
* Se puede ejecutar mediante Azure Portal, la CLI de Azure, la API de REST, PowerShell, WebHook.
* Los scripts se almacenan y administran en una cuenta de Automation.
* Ejecute PowerShell, Flujo de trabajo de PowerShell, Python o runbooks gráficos.
* No hay límite de tiempo en el tiempo de ejecución de los scripts.
* Varios scripts se pueden ejecutar de manera simultánea.
* Se devuelve y almacena toda la salida de un script.
* Historial de trabajos disponible durante 90 días.
* Los scripts se pueden ejecutar como sistema local o con credenciales suministradas por el usuario.
* Requiere [instalación manual](../../automation/automation-windows-hrw-install.md).

## <a name="serial-console"></a>Consola de serie

La [consola serie](/troubleshoot/azure/virtual-machines/serial-console-windows) brinda acceso directo a una máquina virtual, igual que si tuviera un teclado conectado a la máquina virtual.

* Ejecute comandos en máquinas virtuales de Azure.
* Se puede ejecutar mediante una consola basada en texto en la máquina en Azure Portal.
* Inicie sesión en la máquina con una cuenta de usuario local.
* Es útil cuando se requiere acceso a la máquina virtual independientemente del estado del sistema operativo o de la red de la máquina.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las distintas características disponibles para ejecutar scripts y comandos dentro de las máquinas virtuales.

* [Extensión Custom Script](../extensions/custom-script-windows.md)
* [Comando Ejecutar](run-command.md)
* [Trabajo híbrido de runbook](../../automation/automation-hybrid-runbook-worker.md)
* [Consola serie](/troubleshoot/azure/virtual-machines/serial-console-windows)