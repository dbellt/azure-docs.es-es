---
title: Solución de problemas de Azure Bastion | Microsoft Docs
description: En este artículo aprenderá a solucionar problemas de Azure Bastion.
services: bastion
author: charwen
ms.service: bastion
ms.topic: troubleshooting
ms.date: 10/16/2019
ms.author: charwen
ms.openlocfilehash: bf6cde94374133e10b077ac799be0b85ae02bcb7
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108773402"
---
# <a name="troubleshoot-azure-bastion"></a>Solución de problemas de Azure Bastion

Este artículo le mostrará cómo solucionar problemas de Azure Bastion.

## <a name="unable-to-create-an-nsg-on-azurebastionsubnet"></a><a name="nsg"></a>No se puede crear un grupo de seguridad de red en AzureBastionSubnet

**P:** Cuando intento crear un NSG en la subred de Azure Bastion, aparece el siguiente error: *"El grupo de seguridad de red <NSG name> no tiene las reglas necesarias para la subred de Azure Bastion AzureBastionSubnet"* .

**R:** Si crea y aplica un grupo de seguridad de red a *AzureBastionSubnet*, asegúrese de que le ha agregado las reglas necesarias. Puede encontrar una lista de las reglas necesarias en [Trabajo con acceso a grupos de seguridad de red y Azure Bastion](./bastion-nsg.md). Si no agrega estas reglas, se producirá un error en la creación o actualización de los grupos de seguridad de red.

Hay un ejemplo de las reglas de los grupos de seguridad de red disponible como referencia en esta [plantilla de inicio rápido](https://azure.microsoft.com/resources/templates/101-azure-bastion-nsg/).
Para más información, consulte [Guía sobre los grupos de seguridad de red para Azure Bastion](bastion-nsg.md).

## <a name="unable-to-use-my-ssh-key-with-azure-bastion"></a><a name="sshkey"></a>No se puede usar mi clave SSH con Azure Bastion

**P:** Al intentar examinar el archivo de clave SSH, aparece el siguiente error: *"La clave privada de SSH debe empezar por -----BEGIN RSA PRIVATE KEY----- y acabar por -----END RSA PRIVATE KEY-----"* .

**R:** Por el momento, Azure Bastion solo admite claves SSH de RSA. Asegúrese de examinar un archivo de clave que sea una clave privada RSA para SSH, con la clave pública aprovisionada en la máquina virtual de destino. 

Por ejemplo, puede usar el siguiente comando para crear una nueva clave SSH de RSA:

**ssh-keygen -t rsa -b 4096 -C "email@domain.com"**

Salida:

```
ashishj@dreamcatcher:~$ ssh-keygen -t rsa -b 4096 -C "email@domain.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ashishj/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ashishj/.ssh/id_rsa.
Your public key has been saved in /home/ashishj/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:c+SBciKXnwceaNQ8Ms8C4h46BsNosYx+9d+AUxdazuE email@domain.com
The key's randomart image is:
+---[RSA 4096]----+
|      .o         |
| .. ..oo+. +     |
|=.o...B==.O o    |
|==o  =.*oO E     |
|++ .. ..S =      |
|oo..   + =       |
|...     o o      |
|         . .     |
|                 |
+----[SHA256]-----+
```

## <a name="unable-to-sign-in-to-my-windows-domain-joined-virtual-machine"></a><a name="domain"></a>No se puede iniciar sesión en mi máquina virtual de Windows unida a un dominio

**P:** No puedo conectarme a mi máquina virtual de Windows que está unida a un dominio.

**R:** Azure Bastion admite el inicio de sesión de máquina virtual unida a un dominio solo para el inicio de sesión de dominio basado en nombre de usuario y contraseña. Al especificar las credenciales de dominio en Azure Portal, use el formato UPN (username@domain) en lugar del formato de *dominio/nombre de usuario* para iniciar sesión. Esto es compatible con las máquinas virtuales unidas a un dominio o unidas de forma híbrida (tanto unidas a un dominio como a Azure AD). No es compatible para las máquinas virtuales unidas únicamente a Azure AD.

## <a name="file-transfer-issues"></a><a name="filetransfer"></a>Problemas de transferencia de archivos

**P:** ¿Azure Bastion es compatible con la transferencia de archivos?

**R:** Por el momento, la transferencia de archivos no es compatible. Estamos trabajando en agregar esta compatibilidad.

## <a name="black-screen-in-the-azure-portal"></a><a name="blackscreen"></a>Pantalla negra en Azure Portal

**P:** Cuando intento conectarme mediante Azure Bastion, no puedo conectar con la máquina virtual de destino y obtengo una pantalla negra en Azure Portal.

**R:** Esto sucede si hay un problema de conectividad de red entre el explorador web y Azure Bastion (el firewall de Internet del cliente puede estar bloqueando el tráfico de WebSockets o similar), o entre Azure Bastion y su máquina virtual de destino. La mayoría de los casos incluyen un grupo de seguridad de red aplicado a AzureBastionSubnet o a la subred de la máquina virtual de destino que bloquea el tráfico de RDP/SSH en la red virtual. Permita el tráfico de WebSockets en el firewall de Internet del cliente y compruebe el grupo de seguridad de red en la subred de la máquina virtual de destino.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte las [Preguntas frecuentes sobre Bastion](bastion-faq.md).
