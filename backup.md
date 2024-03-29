---

copyright:
  years: 2019
lastupdated: "2019-06-22"

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

# Backup and restore
{: #backup}

To recover from potential disasters, you must back up and be prepared to restore {{site.data.keyword.texttospeechdatafull}} for {{site.data.keyword.icp4dfull}}. You are responsible for understanding your configuration, customization, and usage of the service. You are also responsible for being ready to re-create an instance of the service and to restore your data.
{: shortdesc}

Disaster recovery can become an issue if your cloud solution experiences a significant failure that includes the potential loss of data. In the event of data loss, you need to restore your data to return your service instance to its most recent state.

For the {{site.data.keyword.texttospeechshort}} service, only data for custom voice models is stored in the cloud. Your disaster recovery plan includes knowing, preserving, and being prepared to restore your custom voice models.

### Backing up custom voice models
{: #disaster-recovery-backup}

Preserve the following information about your custom voice models and their custom entries:

-   A list of all of your custom voice models and their definitions. To list information about your custom models:
    -   Use the `GET /v1/customizations` method to list information about all custom models. For more information, see [Querying all custom models](/docs/services/text-to-speech-data?topic=text-to-speech-data-customModels#cuModelsQueryAll).
    -   Use the `GET /v1/customizations/{customization_id}` method to list information about a specified custom model. For more information, see [Querying a custom model](/docs/services/text-to-speech-data?topic=text-to-speech-data-customModels#cuModelsQuery).
-   Information about all custom entries (word/translation pairs) in your custom voice models:
    -   Use the `GET /v1/customizations/{customization_id}/words` method to list information about all word/translation pairs from a custom model. For more information, see [Querying all words from a custom model](/docs/services/text-to-speech-data?topic=text-to-speech-data-customWords#cuWordsQueryModel).
    -   Use the `GET /v1/customizations/{customization_id}/words/{word}` method to list information about a specified word/translation pair from a custom model. For more information, see [Querying a single word from a custom model](/docs/services/text-to-speech-data?topic=text-to-speech-data-customWords#cuWordQueryModel).

It is a best practice to preserve this information in a format that you can use to re-create your custom voice models in the event of a failure. Actively maintaining the information about your custom models and custom entries, and preparing the calls listed in the following section ahead of time, can enable you to recover as quickly as possible.

### Restoring custom voice models
{: #disaster-recovery-restore}

If you need to recover from a disaster, you can use the backup information to re-create your custom voice models and their custom entries:

1.  To re-create your custom voice models, use the `POST /v1/customizations` method. For more information, see [Creating a custom model](/docs/services/text-to-speech-data?topic=text-to-speech-data-customModels#cuModelsCreate).
1.  To add multiple word/translation pairs to your custom voice models, use the `POST /v1/customizations/{customization_id}/words` method. For more information, see [Adding multiple words to a custom model](/docs/services/text-to-speech-data?topic=text-to-speech-data-customWords#cuWordsAdd).
1.  To add single word/translation pair to your custom voice models, use the `POST /v1/customizations/{customization_id}/words/{word}` method. For more information, see [Adding a single word to a custom model](/docs/services/text-to-speech-data?topic=text-to-speech-data-customWords#cuWordAdd).

You can add all of your custom entries at once, in groups, or one at a time.
