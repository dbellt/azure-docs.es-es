---
title: Creación y administración de servicios de Azure con la CLI de Azure
description: Use la CLI de Azure para crear y administrar servicios de Azure para Azure Backup.
ms.topic: conceptual
ms.date: 05/21/2021
ms.openlocfilehash: 3581be20082d1c579caad4cf82ce6b2408f01a92
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110483340"
---
# <a name="create-and-manage-azure-backup-services-using-azure-command-line-interface-azure-cli"></a>Creación y administración de servicios de Azure Backup mediante la interfaz de la línea de comandos de Azure (CLI de Azure)

La interfaz de la línea de comandos de Azure (CLI de Azure) es un conjunto de comandos que se usa para crear y administrar recursos de Azure. La CLI de Azure ayuda con la automatización y en el trabajo con Azure Backup.

En este artículo se proporciona información sobre los servicios que admite la CLI de Azure para Azure Backup y los vínculos correspondientes a los artículos.

## <a name="document-references-for-cli-supported-azure-services"></a>Referencias a documentos sobre los servicios de Azure compatibles con la CLI

En la tabla siguiente se muestran las referencias a documentos de la CLI de Azure disponibles para servicios de Azure admitidos.

Servicios de Azure | Referencias a documentos de la CLI
-------------------------- | ---------------------------------
Azure Vault | [Eliminación de un almacén de Recovery Services](backup-azure-delete-vault.md#delete-the-recovery-services-vault-by-using-cli)
Máquina virtual (VM) de Azure | <li>[Copia de seguridad de una máquina virtual de Azure](quick-backup-vm-cli.md)</li><li>[Restauración de máquinas virtuales de Azure](tutorial-restore-disk.md)</li><li>[Restauración de archivos a partir de copias de seguridad de máquinas virtuales de Azure](tutorial-restore-files.md)</li><li>[Actualización de la directiva de copia de seguridad de máquinas virtuales existente](modify-vm-policy-cli.md)</li><li>[Copia de seguridad y restauración de un disco selectivo para máquinas virtuales de Azure](selective-disk-backup-restore.md#using-azure-cli)</li>
Recurso compartido de archivos de Azure | <li>[Copia de seguridad de recursos compartidos de archivos de Azure](backup-afs-cli.md)</li><li>[Restauración de recursos compartidos de archivos de Azure](restore-afs-cli.md)</li><li>[Administración de copias de seguridad de recursos compartidos de archivos de Azure](manage-afs-backup-cli.md)</li>
SAP HANA | <li>[Copia de seguridad de bases de datos de SAP HANA en una máquina virtual de Azure](tutorial-sap-hana-backup-cli.md)</li><li>[Restauración de bases de datos de SAP HANA en máquinas virtuales de Azure](tutorial-sap-hana-restore-cli.md)</li><li>[Administración de bases de datos de SAP HANA en máquinas virtuales de Azure](tutorial-sap-hana-manage-cli.md)</li>

