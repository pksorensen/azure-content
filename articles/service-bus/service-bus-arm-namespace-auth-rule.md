<properties
    pageTitle="Create a Service Bus authorization rule using an ARM template | Microsoft Azure"
    description="Create a Service Bus authorization rule for namespace and queue using ARM template"
    services="service-bus"
    documentationCenter=".net"
    authors="sethmanheim"
    manager="timlt"
    editor=""/>

<tags
    ms.service="service-bus"
    ms.devlang="tbd"
    ms.topic="article"
    ms.tgt_pltfrm="dotnet"
    ms.workload="na"
    ms.date="04/15/2016"
    ms.author="sethm;shvija"/>

# Create a Service Bus authorization rule for namespace and queue using an ARM template

This article shows how to use an Azure Resource Manager (ARM) template that creates an authorization rule for a Service Bus namespace and queue. You will learn how to define which resources are deployed and how to define parameters that are specified when the deployment is executed. You can use this template for your own deployments, or customize it to meet your requirements.

For more information about creating templates, please see [Authoring Azure Resource Manager Templates][].

For the complete template, see the [Service Bus auth rule template][] on GitHub.

>[AZURE.NOTE] The following ARM templates are available for download and deployment.
>
>-    [Create a Service Bus namespace with an Event Hub and consumer group](service-bus-arm-namespace-event-hub.md)
>-    [Create a Service Bus namespace with queue](service-bus-arm-namespace-queue.md)
>-    [Create a Service Bus namespace with topic and subscription](service-bus-arm-namespace-topic.md)
>-    [Create a Service Bus namespace](service-bus-arm-namespace.md)
>
>To check for the latest templates, see the [Azure Quickstart Templates][] and search for Service Bus.

## What will you deploy?

With this template, you will deploy a Service Bus authorization rule for a namespace and messaging entity (a queue, in this case).

This template uses Shared Access Signature (SAS) for authentication. SAS enables applications to authenticate to Service Bus using an access key configured on the namespace, or on the messaging entity (queue or topic) with which specific rights are associated. You can then use this key to generate a SAS token that clients can in turn use to authenticate to Service Bus.

To run the deployment automatically, click the following button:

[![Deploy to Azure](./media/service-bus-arm-namespace-auth-rule/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F301-servicebus-create-authrule-namespace-and-queue%2Fazuredeploy.json)

## Parameters

With Azure Resource Manager, you define parameters for values you want to specify when the template is deployed. The template includes a section called `Parameters` that contains all of the parameter values. You should define a parameter for those values that will vary based on the project you are deploying or based on the environment you are deploying to. Do not define parameters for values that will always stay the same. Each parameter value is used in the template to define the resources that are deployed.

We will describe each parameter in the template.

### serviceBusNamespaceName

The name of the Service Bus namespace to create.

```
"serviceBusNamespaceName": {
"type": "string"
}
```

### namespaceAuthorizationRuleName 

The name of the authorization rule for the namespace.

```
"namespaceAuthorizationRuleName ": {
"type": "string"
}
```

### serviceBusQueueName

The name of the queue in the Service Bus namespace.

```
"serviceBusQueueName": {
"type": "string"
}
```

### serviceBusApiVersion

The Service Bus API version of the template.

```
"serviceBusApiVersion": {
"type": "string"
}
```

## Resources to deploy

Creates a Service Bus authorization rule for namespace and entity.

```
"resources": [
        {
            "apiVersion": "[variables('sbVersion')]",
            "name": "[parameters('serviceBusNamespaceName')]",
            "type": "Microsoft.ServiceBus/namespaces",
            "location": "[variables('location')]",
            "kind": "Messaging",
            "sku": {
                "name": "StandardSku",
                "tier": "Standard"
            },
            "resources": [
                {
                    "apiVersion": "[variables('sbVersion')]",
                    "name": "[parameters('serviceBusQueueName')]",
                    "type": "Queues",
                    "dependsOn": [
                        "[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"
                    ],
                    "properties": {
                        "path": "[parameters('serviceBusQueueName')]"
                    },
                    "resources": [
                        {
                            "apiVersion": "[variables('sbVersion')]",
                            "name": "[parameters('queueAuthorizationRuleName')]",
                            "type": "authorizationRules",
                            "dependsOn": [
                                "[parameters('serviceBusQueueName')]"
                            ],
                            "properties": {
                                "Rights": ["Listen"]
                            }
                        }
                    ]
                }
            ]
        }, {
            "apiVersion": "[variables('sbVersion')]",
            "name": "[variables('namespaceAuthRuleName')]",
            "type": "Microsoft.ServiceBus/namespaces/authorizationRules",
            "dependsOn": ["[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"],
            "location": "[resourceGroup().location]",
            "properties": {
                "Rights": ["Send"]
            }
        }
    ]
```

## Commands to run deployment

[AZURE.INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

### PowerShell

```
New-AzureRmResourceGroupDeployment -ResourceGroupName \<resource-group-name\> -TemplateFile <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/301-servicebus-create-authrule-namespace-and-queue/azuredeploy.json>
```

## Azure CLI

```
azure config mode arm

azure group deployment create \<my-resource-group\> \<my-deployment-name\> --template-uri <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/301-servicebus-create-authrule-namespace-and-queue/azuredeploy.json>
```

## Next steps

Now that you've created and deployed resources using ARM, learn how to manage these resources by viewing these articles:

- [Manage Azure Service Bus using Azure Automation](service-bus-automation-manage.md)
- [Manage Service Bus with PowerShell](service-bus-powershell-how-to-provision.md)
- [Manage Service Bus resources with the Service Bus Explorer](https://code.msdn.microsoft.com/Service-Bus-Explorer-f2abca5a)

Learn more about SAS here:

- [Service Bus authentication and authorization](service-bus-authentication-and-authorization.md)
- [Shared Access Signatures](service-bus-sas-overview.md)
- [AuthorizationRule class](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.authorizationrule.aspx)

  [Authoring Azure Resource Manager Templates]: ../resource-group-authoring-templates.md
  [Azure Quickstart Templates]: https://azure.microsoft.com/documentation/templates/
  [Using Azure PowerShell with Azure Resource Manager]: ../powershell-azure-resource-manager.md
  [Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Management]: ../xplat-cli-azure-resource-manager.md
  [Service Bus auth rule template]: https://github.com/Azure/azure-quickstart-templates/blob/master/301-servicebus-create-authrule-namespace-and-queue/
