---

copyright:
  years: 2019
lastupdated: "2019-06-16"

subcollection: text-to-speech-data

---

{:shortdesc: .shortdesc}
{:external: target="_blank" .external}
{:tip: .tip}
{:important: .important}
{:note: .note}
{:deprecated: .deprecated}
{:pre: .pre}
{:codeblock: .codeblock}
{:screen: .screen}
{:javascript: .ph data-hd-programlang='javascript'}
{:java: .ph data-hd-programlang='java'}
{:python: .ph data-hd-programlang='python'}
{:swift: .ph data-hd-programlang='swift'}

# Information security
{: #information-security}

{{site.data.keyword.IBM}} is committed to providing our clients and partners with innovative data privacy, security, and governance solutions.
{: shortdesc}

Clients are responsible for ensuring their own compliance with various laws and regulations, including the European Union General Data Protection Regulation. Clients are solely responsible for obtaining advice of competent legal counsel as to the identification and interpretation of any relevant laws and regulations that might affect the clients’ business and any actions the clients might need to take to comply with such laws and regulations.
{: important}

The products, services, and other capabilities described herein are not suitable for all client situations and might have restricted availability. {{site.data.keyword.IBM_notm}} does not provide legal, accounting or auditing advice or represent or warrant that its services or products will ensure that clients are in compliance with any law or regulation.

If you need to request GDPR support for {{site.data.keyword.cloud}} {{site.data.keyword.watson}} resources that are created

-   In the European Union (EU), see [Requesting support for {{site.data.keyword.cloud_notm}} Watson resources created in the European Union](/docs/services/watson?topic=watson-gdpr-sar#request-EU).
-   Outside of the EU, see [Requesting support for resources outside the European Union](/docs/services/watson?topic=watson-gdpr-sar#request-non-EU).

## European Union General Data Protection Regulation (GDPR)
{: #gdpr}

{{site.data.keyword.IBM_notm}} is committed to providing our clients and partners with innovative data privacy, security and governance solutions to assist them on their journey to GDPR compliance.

Learn more about {{site.data.keyword.IBM_notm}}'s own GDPR readiness journey and our GDPR capabilities and offerings to support your compliance journey [here](http://www.ibm.com/gdpr){: external}.

## Labeling and deleting data in the Text to Speech service
{: #gdpr-text-to-speech}

{{site.data.keyword.texttospeechdatafull}} for {{site.data.keyword.icp4dfull}} enables you to delete all data that is associated with speech synthesis requests and with custom voice models. To delete data, you must do the following:

1.  Use the `X-Watson-Metadata` header to associate a customer ID with data that is passed by a request to the service; see [Specifying a customer ID](#specify-customer-id).
1.  Use the `DELETE /v1/user_data` method to delete all data that is associated with a specified customer ID; see [Deleting data](#delete-pi).

By default, no customer ID is associated with data.

Experimental and beta features are not intended for use with a production environment and therefore are not guaranteed to function as expected when labeling and deleting data. Experimental and beta features should not be used when implementing a solution that requires the labeling and deletion of data.
{: note}

### Specifying a customer ID
{: #specify-customer-id}

To associate a customer ID with data, include the `X-Watson-Metadata` header with the request that passes the information. You pass the string `customer_id={id}` as the argument of the header. The following example associates the customer ID `my_ID` with the data passed with a `POST /v1/synthesize` request:

```bash
curl -X POST
--header "Authorization: Bearer {token}"
--header "X-Watson-Metadata: customer_id=my_ID"
--header "Content-Type: application/json"
--header "Accept: audio/wav"
--data "{\"text\":\"hello world\"}"
--output hello_world.wav
"{url}/v1/synthesize"
```

A customer ID can include any characters except for the `;` (semicolon) and `=` (equals sign). Specify a random or generic string for the customer ID; do not specify a personally identifiable string, such as an email address or Twitter ID. You can specify different customer IDs with different requests. A customer ID that you specify is associated with the instance of the service whose credentials are used with the request; only credentials for that instance of the service can delete data associated with the ID.

Use the `X-Watson-Metadata` header with the following methods:

-   With HTTP requests:
    -   `POST /v1/synthesize`
    -   `GET /v1/synthesize`

    The customer ID is associated with data that is sent with the request.

-   With WebSocket requests:
    -   `/v1/synthesize`

    You specify the customer ID with the `x-watson-metadata` query parameter to associate the ID with data that is sent with the request. You must URL-encode the argument to the query parameter, for example, `customer_id%3dmy_ID`.

-   With requests to add custom words to custom voice models:
    -   `POST /v1/customizations/{customization_id}`
    -   `POST /v1/customizations/{customization_id}/words`
    -   `PUT /v1/customizations/{customization_id}/words/{word}`

    The customer ID is associated with the custom words that are added or updated by the request.

### Deleting data
{: #delete-pi}

To delete all data that is associated with a customer ID, use the `DELETE /v1/user_data` method. You pass the string `customer_id={id}` as a query parameter with the request. The following example deletes all data for the customer ID `my_ID`:

```bash
curl -X DELETE
--header "Authorization: Bearer {token}"
"{url}/v1/user_data?customer_id=my_ID"
```

The `/v1/user_data` method deletes all data that is associated with the specified customer ID, regardless of the method by which the information was added. The method has no effect if no data is associated with the customer ID. You must issue the request with credentials for the same instance of the service that was used to associate the customer ID with the data.
