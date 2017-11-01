---
title: "Create auto-number-attributes (Developer Guide for Dynamics 365 Customer Engagement) | MicrosoftDocs)"
description: "Learn about creating auto-number attribute in the same way you create a string attribute using the StringAttributeMetadata class except that you use the new AutoNumberFormat property. Use the AutoNumberFormat property to define a pattern that includes sequential numbers and random strings by composing placeholders, which indicates the length and type of values that are generated."
keywords: "Auto-number attributes"
ms.date: 10/31/2017
ms.service: crm-online
ms.topic: article
applies_to:
  - "CRM 2017"
  - "Dynamics 365"
ms.assetid: e97477d2-5509-9f5e-76e0-e0039b2e72c8
author: kabala123
ms.author: kabala
manager: kudesia
topic-status: Drafting
---

# Create auto-number attributes

[!INCLUDE[](../includes/cc_applies_to_update_9_0_0.md)]

With the [!INCLUDE [pn-crm-9-0-0-online](../includes/pn-crm-9-0-0-online.md)] release, you can add an auto-number attribute for any entity. Currently, you can add the attribute programmatically. There is no user interface to add this type of attribute. The topic explains how you can programmatically create an auto-number attribute and set a seed value for sequential elements. In addition, the topic shows how to set the sequence number for the next record if you need to reset the seed at any time later.
> [!NOTE]
>The setting of the seed is optional. There is no need to call the seed if you don’t need to reseed.

You can create an auto-number attribute in the same way you create a string attribute using the **StringAttributeMetadata** class except that you use the new **AutoNumberFormat** property. Use the **AutoNumberFormat** property to define a pattern that includes sequential numbers and random strings by composing placeholders, which indicates the length and type of values that are generated. The random strings help you to avoid duplicates or collisions, especially when offline clients trying to create auto-numbers.

When creating an auto-number attribute, the **StringAttributeMetadata.FormatName** property or the **StringAttributeMetadata.Format** property values must be Text. Since these are the default values you will typically not set this property. You cannot create an auto-number attribute that uses any other special kind of format such as Email, Phone, TextArea, Url or any other [existing formats](https://msdn.microsoft.com/en-us/library/microsoft.xrm.sdk.metadata.stringformatname.aspx).

> [!NOTE]
>You can modify an existing format text attribute to be an auto-number format.

When you add the auto-number attribute as a field to a form, the auto-number attribute field behaves as read-only in the form, and end-users cannot edit the field. The value is set only after you save the entity. You can see auto-number attribute as any other field value in views, grids and so on.

### Examples
The following examples show how to create a new auto-number attribute named **new\_SerialNumber** for a custom entity named **new\_Widget** which will have a value that looks like this: **WID-00001-AB7LSFG-20170913070240**.
Using the Organization service with SDK assemblies **CreateAttributeRequest** and **StringAttributeMetadata** classes:

```csharp
CreateAttributeRequest widgetSerialNumberAttributeRequest = new CreateAttributeRequest
            {
                EntityName = "newWidget",
                Attribute = new StringAttributeMetadata
                {
                    //Define the format of the attribute
                    AutoNumberFormat = "WID-{SEQNUM:5}-{RANDSTRING:6}-{DATETIMEUTC:yyyyMMddhhmmss}",
                    LogicalName = "new_serialnumber",
                    SchemaName = "new_SerialNumber",
                    RequiredLevel = new AttributeRequiredLevelManagedProperty(AttributeRequiredLevel.None),
                    MaxLength = 100, // The MaxLength defined for the string attribute must be greater than the length of the AutoNumberFormat value, that is, it should be able to fit in the generated value.
                    DisplayName = new Label("Serial Number", 1033),
                    Description = new Label("Serial Number of the widget.", 1033)
                }
            };
            _serviceProxy.Execute(widgetSerialNumberAttributeRequest);

```

## Use Web API

You can create and update entity definitions using the Web API.

More information: [Create and update entity definitions using the Web API > Create attributes](https://msdn.microsoft.com/en-us/library/mt593078.aspx#Anchor_3).

#### Request
```http
POST [Organization URI]/api/data/v9.0/EntityDefinitions(LogicalName='new_widget')/Attributes HTTP/1.1
Accept: application/json
Content-Type: application/json; charset=utf-8
OData-MaxVersion: 4.0
OData-Version: 4.0

{
 "AttributeType": "String",
 "AttributeTypeName": {
  "Value": "StringType"
 },
 "Description": {
  "@odata.type": "Microsoft.Dynamics.CRM.Label",
  "LocalizedLabels": [
   {
    "@odata.type": "Microsoft.Dynamics.CRM.LocalizedLabel",
    "Label": "Serial Number of the widget.",
    "LanguageCode": 1033
   }
  ]
 },
 "DisplayName": {
  "@odata.type": "Microsoft.Dynamics.CRM.Label",
  "LocalizedLabels": [
   {
    "@odata.type": "Microsoft.Dynamics.CRM.LocalizedLabel",
    "Label": "Serial Number",
    "LanguageCode": 1033
   }
  ]
 },
 "RequiredLevel": {
  "Value": "None",
  "CanBeChanged": true,
  "ManagedPropertyLogicalName": "canmodifyrequirementlevelsettings"
 },
 "SchemaName": "new_SerialNumber",
 "AutoNumberFormat": "WID-{SEQNUM:5}-{RANDSTRING:6}-{DATETIMEUTC:yyyyMMddhhmmss}",
 "@odata.type": "Microsoft.Dynamics.CRM.StringAttributeMetadata",
 "FormatName": {
  "Value": "Text"
 },
 "MaxLength": 100
}
```

#### Response

```http
HTTP/1.1 204 No Content
OData-Version: 4.0
OData-EntityId: [Organization URI]/api/data/v9.0/EntityDefinitions(402fa40f-287c-e511-80d2-00155d2a68d2)/Attributes(f01bef16-287c-e511-80d2-00155d2a68d2)
```

## AutoNumberFormat options

These examples show how you can configure the **AutoNumberFormat** property to get different results:

|**AutoNumberFormat value**|**Example value**|
|:----------|:----------|
|`CAR-{SEQNUM:3}-{RANDSTRING:6}`|`CAR-123-AB7LSF`|
|`CNR-{RANDSTRING:4}-{SEQNUM:4}`|`CNR-WXYZ-1000`|
|`{SEQNUM:6}-#-{RANDSTRING:3}`  |`123456-#-R3V`|
|`KA-{SEQNUM:4}`                |`KA-0001`|
|`{SEQNUM:10}`                  |`1234567890`|
|`QUO-{SEQNUM:3}#{RANDSTRING:3}#{RANDSTRING:5}`|      `QUO-123#ABC#PQ2ST`|
|`QUO-{SEQNUM:7}{RANDSTRING:5}` |`QUO-0001000P9G3R`|
|`CAS-{SEQNUM:6}-{RANDSTRING:6}-{DATETIMEUTC:yyyyMMddhhmmss}`|`CAS-002000-S1P0H0-20170913091544`|
|`CAS-{SEQNUM:6}-{DATETIMEUTC:yyyyMMddhh}-{RANDSTRING:6}`|`CAS-002002-2017091309-HTZOUR`|
|`CAS-{SEQNUM:6}-{DATETIMEUTC:yyyyMM}-{RANDSTRING:6}-{DATETIMEUTC:hhmmss}`|`CAS-002000-201709-Z8M2Z6-110901`|

The random string placeholders are optional.You can include more than one random string placeholder. Use any of the format value for datetime placeholders from [Standard Date and Time Format Strings](https://docs.microsoft.com/en-us/dotnet/standard/base-types/standard-date-and-time-format-strings).

### String length

The table shows the string length value for the random and sequential placeholders.

<table>
  <tr>
    <th>Placeholder</th>
    <th>String Length</th>
    <th>Output Scenario</th>
  </tr>
  <tr>
    <td><code>{RANDSTRING:MIN_LENGTH}</code></td>
    <td>The value of <b>MIN_LENGTH</b> is between 1 and 6.</td>
     <td>When you save the entity, the auto-number attribute generates the random string with the defined length if the value is between 1 and 6. If you use the <b>MIN_LENGTH</b> value as 7 or beyond 7, you get to see an Invalid Argument error.</td>
  </tr>
  <tr>
    <td><code>{SEQNUM:MIN_LENGTH}</code></td>
    <td>The minimum value of the <b>MIN_LENGTH</b> is 1. The number continues to increment beyond the minimum length.</td>
    <td>When you save the entity, the auto-number attribute works fine and continue to work fine for larger values of <b>MIN_LENGTH</b>.</td></table>

For sequential value placeholders, the **MIN_LENGTH** is a minimum length. If you set the value to be 2, the initial value will be 01, and the 100th entity value will be 100. The number will continue to increment beyond the minimum length. The value in setting the length for sequential values is to establish a consistent length for the auto-generated value by adding additional 0s to the initial value. It will not limit the absolute value. Random value placeholders will always be the same length.

Because the sequential values can get larger than the minimum length allotted for them, you should not adjust the **StringAttributeMetadata.MaxLength** property to match the length of your formatted value. There is little value in doing this and it could cause an error in the future if the value exceeds the **MaxLength** value. Make sure you leave enough room for a realistic range of sequential values.

> [!NOTE]
> There is no validation of the placeholder values when you create the attribute. The error appears only when you try to save an entity instance that uses an incorrectly configured **AutoNumberFormat** value.
> For example, if you specify the length of the random string more than 6, the first person creating a new entity instance gets an **Invalid Argument** error when they first try to save the entity containing the new auto-number attribute.

## Update auto-number attributes

If you create an auto-number attribute with an incorrect configuration or you want to modify an existing auto-number attribute, you can update the attribute the **AutoNumberFormat** value.

The following code snippet explains you to retrieve, modify, and update the auto-number attribute.

To modify an existing auto-number attribute, you must retrieve the attribute using the **RetrieveAttributeRequest** class.

```csharp
// Create the retrieve request
RetrieveAttributeRequest attributeRequest = new RetrieveAttributeRequest
            {
                EntityLogicalName = entityName.ToLower(),
                LogicalName = "new_serialnumber",
                RetrieveAsIfPublished = true
            };
// Retrieve attribute response
RetrieveAttributeResponse attributeResponse = (RetrieveAttributeResponse)_serviceProxy.Execute(attributeRequest);
```

After retrieving the auto-number attribute, you need to modify and update the attribute.

```csharp            
//Modify the retrieved auto-number attribute
AttributeMetadata retrievedAttributeMetadata = attributeResponse.AttributeMetadata;
retrievedAttributeMetadata.AutoNumberFormat = "CAR-{RANDSTRING:5}{SEQNUM:6}"; //Modify the existing attribute by writing the format as per your requirement 

// Update the auto-number attribute            
UpdateAttributeRequest updateRequest = new UpdateAttributeRequest
                        {
                            Attribute = retrievedAttributeMetadata,
                            EntityName = "newWidget",
                        };
// Execute the request
_serviceProxy.Execute(updateRequest);
```

## Set a seed value

By default, all auto-number sequential values start with 1000 and use 0 as the prefix depending on the length. In this way, the length of the value is always same. If you want to change the initial value, you need to change the initial seed value using the API below to set the next number that are used for the sequential segment.

For example, when the length of the sequential number is 5, you may want to start with an initial value of 10000 instead of the default value of 00001. 
If you want to choose a different starting value after creating the auto-numbering attribute, use the **SetAutoNumberSeed** message. Use the **SetAutoNumberSeedRequest** class when using the SDK assemblies and **SetAutoNumberSeed** action when using the Web API.
  
The **AutoNumberSeed** message has the following parameters:


|**Name**|**Type**|**Description**|
|:----------|:----------|:----------|
|EntityName|string|The logical name of the entity that contains the attribute you want to set the seed on.|
|AttributeName|string|The logical name of the attribute you want to set the seed on.|
|Value|int|Next auto-number value for the attribute.|

### Examples
The following samples set the seed to 10000 for an auto-number attribute named **new\_SerialNumber** for a custom entity named **new\_Widget**.

Using the Organization service with SDK assemblies **SetAutoNumberSeedRequest** class:
```csharp
//Define the seed 
SetAutoNumberSeedRequest req = new SetAutoNumberSeedRequest();
req.EntityName = "newWidget";
req.AttributeName = "newSerialnumber";
req.Value = 10000;
_serviceProxy.Execute(req);
```
Using the Web API **SetAutoNumberSeed** Action.

More information: [Use Web API actions > Unbound actions](https://msdn.microsoft.com/en-us/library/mt607600.aspx#bkmk_unboundActions)

#### Request

```http
POST [Organization URI]/api/data/v9.0/SetAutoNumberSeed HTTP/1.1
Accept: application/json
Content-Type: application/json; charset=utf-8
OData-MaxVersion: 4.0
OData-Version: 4.0

{
 "EntityName": "new_Widget",
 "AttributeName": "new_Serialnumber",
 "Value": 10000
} 
```

#### Response

```json
HTTP/1.1 204 No Content
OData-Version: 4.0
```
### See Also
[Metadata and data models](metadata-data-models.md)  
[Customize entity attribute metadata](customize-entity-attribute-metadata.md) 