---
title: 'How to: Retrieve Information from a Service Broker Error Message (Transact SQL)'
description: "A message of type https://schemas.microsoft.com/SQL/ServiceBroker/Error is a Service Broker error message."
ms.prod: sql
ms.technology: configuration
ms.topic: conceptual
author: rwestMSFT
ms.author: randolphwest
ms.reviewer: mikeray, maghan
ms.date: "03/30/2022"
---

# How to: Retrieve Information from a Service Broker Error Message (Transact SQL)

[!INCLUDE [sql-asdbmi](../../includes/applies-to-version/sql-asdbmi.md)]

A message of type `https://schemas.microsoft.com/SQL/ServiceBroker/Error` is a Service Broker error message. Messages of this type are XML documents that contain a numeric code for the error and a description of the error.

## To retrieve the information from a Service Broker error message

1. Declare a variable of type int to hold the error code.

2. Declare a variable of type nvarchar(3000) to hold the error description.

3. Declare a variable of type XML to hold an XML representation of the message body.

4. CAST the message body from varbinary(max) to XML, and assign the results to the variable of type XML.

5. Use the value function of the XML data type to retrieve the error code.

6. Use the value function of the XML data type to retrieve the error description.

7. Handle the error as appropriate for your application. Errors with negative error codes are generated by Service Broker. Errors with positive error codes are generated by service programs that ran END CONVERSATION WITH ERROR.

## Example

```sql
    -- The variables to hold the error code and the description are
    -- provided by the caller.

    CREATE PROCEDURE [ExtractBrokerError]
      ( @message_body VARBINARY(MAX),
        @code int OUTPUT,
        @description NVARCHAR(3000) OUTPUT )
    AS
    BEGIN

    -- Declare a variable to hold an XML version of the message body.

    DECLARE @xmlMessage XML;

    -- CAST the provided message body to XML.

    SET @xmlMessage = CAST(@message_body AS XML);
    SET @code = @@ERROR

    IF @@ERROR<>0
      RETURN @code

    -- Retrieve the error code from the Code element.

    SET @code = (
          SELECT @xmlMessage.value(
            N'declare namespace
               brokerns="https://schemas.microsoft.com/SQL/ServiceBroker/Error";
                   (/brokerns:Error/brokerns:Code)[1]',
            'int')
            );

    -- Retrieve the description of the error from the Description element.

    SET @description = (
          SELECT @xmlMessage.value(
            'declare namespace
               brokerns="https://schemas.microsoft.com/SQL/ServiceBroker/Error";
               (/brokerns:Error/brokerns:Description)[1]',
            'nvarchar(3000)')
            );


    RETURN 0;

    END
    GO
```

## See also

- [Broker System Messages](broker-system-messages.md)
- [XQuery Basics](../../xquery/xquery-basics.md)
