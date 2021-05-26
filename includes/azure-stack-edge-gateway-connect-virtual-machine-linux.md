---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 05/13/2021
ms.author: alkohli
ms.openlocfilehash: 45a82d85510bd432efa93211ffc9dba8b911c557
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110070784"
---
Conéctese a la máquina virtual mediante la dirección IP privada que pasó durante la creación de la máquina virtual.

1. Abra una sesión de SSH para conectarse con la dirección IP.

   `ssh -l <username> <ip address>`

1. En el símbolo del sistema, escriba la contraseña que usó al crear la máquina virtual.

   Si tiene que proporcionar la clave SSH, use este comando.

   `ssh -i c:/users/Administrator/.ssh/id_rsa Administrator@5.5.41.236`

   A continuación se muestra un ejemplo de salida cuando se conecta a la máquina virtual:

    ```powershell
    PS C:\07-30-2020\linux> ssh -l Administrator 10.126.68.186
    Administrator@10.126.68.186's password:
    Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 5.0.0-1027-azure x86_64)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
    
      System information as of Thu Jul 30 22:56:11 UTC 2020
    
      System load:  0.0               Processes:           105
      Usage of /:   5.6% of 28.90GB   Users logged in:     0
      Memory usage: 12%               IP address for eth0: 10.126.68.186
      Swap usage:   0%
    
     * Are you ready for Kubernetes 1.19? It's nearly here! Try RC3 with
       sudo snap install microk8s --channel=1.19/candidate --classic
    
       https://www.microk8s.io/ has docs and details.
    
    68 packages can be updated.
    0 updates are security updates.
    
    
    *** System restart required ***
    
    The programs included with the Ubuntu system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.
    
    Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
    applicable law.
    
    To run a command as administrator (user "root"), use "sudo <command>".
    See "man sudo_root" for details.
    
    Administrator@mylinuxvm:
    ```
