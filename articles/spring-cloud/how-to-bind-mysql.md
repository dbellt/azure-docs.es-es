---
title: Enlace de una instancia de Azure Database for MySQL con una aplicación de Azure Spring Cloud
description: Aprenda a enlazar una instancia de Azure Database for MySQL a una aplicación de Azure Spring Cloud.
author: bmitchell287
ms.service: spring-cloud
ms.topic: how-to
ms.date: 11/04/2019
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 71408513e18dc506f2c596d4e05db4fcc98ccea2
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112004158"
---
# <a name="bind-an-azure-database-for-mysql-instance-to-your-azure-spring-cloud-application"></a>Enlace de una instancia de Azure Database for MySQL con una aplicación de Azure Spring Cloud 

**Este artículo se aplica a:** ✔️ Java

Con Azure Spring Cloud puede enlazar determinados servicios de Azure con sus aplicaciones automáticamente, en lugar de tener que configurar manualmente la aplicación Spring Boot. En este artículo se muestra cómo enlazar una aplicación a una instancia de Azure Database for MySQL.

## <a name="prerequisites"></a>Requisitos previos

* Una instancia de Azure Spring Cloud implementada
* Una cuenta de Azure Database for MySQL
* Azure CLI

Si no tiene ninguna instancia de Azure Spring Cloud implementado, siga las instrucciones de [Inicio rápido: Inicio de una aplicación de Azure Spring Cloud desde Azure Portal](./quickstart.md) para implementar la primera aplicación de Spring Cloud.

## <a name="prepare-your-java-project"></a>Preparación del proyecto de Java

1. En el archivo *pom.xml* del proyecto, agregue la siguiente dependencia:

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    ```
1. En el archivo *application.properties*, quite todas las propiedades de `spring.datasource.*`.

1. Actualice la aplicación actual mediante la ejecución de `az spring-cloud app deploy` o cree una implementación para este cambio mediante la ejecución de `az spring-cloud app deployment create`.

## <a name="bind-your-app-to-the-azure-database-for-mysql-instance"></a>Enlace de una aplicación a una instancia de Azure Database for MySQL

#### <a name="service-binding"></a>[enlace del servicio](#tab/Service-Binding)
1. Anote el nombre de usuario y la contraseña de administrador de su cuenta de Azure Database for MySQL. 

1. Conéctese al servidor, cree una base de datos llamada **testdb** desde un cliente de MySQL y, después cree una cuenta que no sea de administrador.

1. En Azure Portal, en la página del servicio **Azure Spring Cloud**, busque el **Panel de la aplicación** y seleccione la aplicación que va a enlazar a la instancia de Azure Database for MySQL.  Se trata de la misma aplicación que se actualizó o implementó en el paso anterior. 

1. Seleccione **Service binding** (Enlace de servicio) y, después, seleccione el botón **Create service binding** (Crear enlace de servicio). 

1. Rellene el formulario, seleccione **Azure MySQL** en **Tipo de enlace** y use el mismo nombre de base de datos que usó anteriormente, y el nombre de usuario y contraseña que anotó en el primer paso.

1. Tras reiniciar la aplicación el enlace debería funcionar.

1. Para asegurarse de que el enlace del servicio es correcto, seleccione el nombre del enlace y compruebe sus detalles. El campo `property` debería ser similar a este:
    ```
    spring.datasource.url=jdbc:mysql://some-server.mysql.database.azure.com:3306/testdb?useSSL=true&requireSSL=false&useLegacyDatetimeCode=false&serverTimezone=UTC
    spring.datasource.username=admin@some-server
    spring.datasource.password=abc******
    spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
    ```

#### <a name="terraform"></a>[Terraform](#tab/Terraform)

El siguiente script de Terraform muestra cómo configurar una aplicación Azure Spring Cloud con Azure Database for MySQL.

```terraform
provider "azurerm" {
  features {}
}

variable "application_name" {
  type        = string
  description = "The name of your application"
  default     = "demo-abc"
}

variable "administrator_login" {
  type        = string
  description = "The MySQL administrator login"
  default     = "myadmin"
}

resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "West Europe"
}

resource "random_password" "password" {
  length           = 32
  special          = true
  override_special = "_%@"
}

resource "azurerm_mysql_server" "database" {
  name                = "mysql-${var.application_name}-001"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location

  administrator_login          = var.administrator_login
  administrator_login_password = random_password.password.result

  sku_name                          = "B_Gen5_1"
  storage_mb                        = 5120
  version                           = "5.7"
  auto_grow_enabled                 = true
  backup_retention_days             = 7
  geo_redundant_backup_enabled      = false
  infrastructure_encryption_enabled = false
  public_network_access_enabled     = true
  ssl_enforcement_enabled           = true
  ssl_minimal_tls_version_enforced  = "TLS1_2"
}

resource "azurerm_mysql_database" "database" {
  name                = "mysqldb-${var.application_name}-001"
  resource_group_name = azurerm_resource_group.example.name
  server_name         = azurerm_mysql_server.database.name
  charset             = "utf8"
  collation           = "utf8_unicode_ci"
}

# This rule is to enable the 'Allow access to Azure services' checkbox
resource "azurerm_mysql_firewall_rule" "database" {
  name                = "mysqlfw-${var.application_name}-001"
  resource_group_name = azurerm_resource_group.example.name
  server_name         = azurerm_mysql_server.database.name
  start_ip_address    = "0.0.0.0"
  end_ip_address      = "0.0.0.0"
}

resource "azurerm_spring_cloud_service" "example" {
  name                = "example-springcloud"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
}

resource "azurerm_spring_cloud_app" "example" {
  name                = "example-springcloudapp"
  resource_group_name = azurerm_resource_group.example.name
  service_name        = azurerm_spring_cloud_service.example.name
  is_public           = true
  https_only          = true
}

resource "azurerm_spring_cloud_java_deployment" "example" {
  name                = "default"
  spring_cloud_app_id = azurerm_spring_cloud_app.example.id
  cpu                 = 2
  memory_in_gb        = 4
  instance_count      = 2
  jvm_options         = "-XX:+PrintGC"
  runtime_version     = "Java_11"

  environment_variables = {
    "spring.datasource.url" : "jdbc:mysql://${azurerm_mysql_server.database.fqdn}:3306/${azurerm_mysql_database.database.name}?useSSL=true&requireSSL=false&useLegacyDatetimeCode=false&serverTimezone=UTC"
    "spring.datasource.username" : "${var.administrator_login}@${azurerm_mysql_server.database.name}"
    "spring.datasource.password" : random_password.password.result
    "spring.jpa.properties.hibernate.dialect" : "org.hibernate.dialect.MySQL5InnoDBDialect"
  }
}

resource "azurerm_spring_cloud_active_deployment" "example" {
  spring_cloud_app_id = azurerm_spring_cloud_app.example.id
  deployment_name     = azurerm_spring_cloud_java_deployment.example.name
}
```
---

## <a name="next-steps"></a>Pasos siguientes

En este artículo ha aprendido a enlazar una aplicación de Azure Spring Cloud a una instancia de Azure Database for MySQL. Para más información sobre los servicios de enlace a una aplicación, consulte [Enlace de una base de datos de Azure Cosmos DB a una aplicación de Azure Spring Cloud](./how-to-bind-cosmos.md).
