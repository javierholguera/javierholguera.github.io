---
layout: post
cover: false
title: Incompatible AVRO schema in Schema Registry
categories:
- Kafka
tags:
- kafka
- avro
- schema registry
---

My company uses [Apache Kafka](http://kafka.apache.org/) as the spine for its next-generation architecture. Kafka is a distributed append-only log that can be used as a pub-sub mechanism. We use Kafka to publish events once business processes have completed successfully, allowing a high degree of decoupling between producers and consumers.

These events are encoded using [Avro schemas](http://avro.apache.org). Avro is a binary serialization format that enables a compact representation of data, much more than, for instance, JSON. Given the high volume of events we publish to kafka, using a compact format is critical.

In combination with Avro we use [Confluent's Schema Registry](https://github.com/confluentinc/schema-registry) to manage our schemas. The registry provides a RESTful API to store and retrieve schemas.

### Compatibility modes

The Schema Registry can control what schemas get registered, ensuring a certain level of compatibility between existing and new schemas. This compatibility can be set to one of the next four modes:

- **BACKWARD**: a new schema is allowed if it can be used to read all data ever published into the corresponding topic.
- **FORWARD**: a new schema is allowed if it can be used to write data that all previous schemas would be able to read.
- **FULL**: a new schema that fullfils both registrations.
- **NONE**: a schema is allowed as long as it is valid Avro.

By default, Schema Registry sets BACKWARD compatibility, which is most likely your preferred option in PROD environment, unless you want to have a hard time with your consumers not quite understanding events published with a newer, incompatible version of the schema.

### Incompatible schemas

In development phase it is perfectly fine to replace schemas with others that are incompatible. Schema Registry will prevent updating the existing schema to an incompatible newer version unless we change its default setting.

Fortunately Schema Registry offers a [complete API](http://docs.confluent.io/1.0/schema-registry/docs/api.html) that allows to register and retrieve schemas, but also to change some of its configuration. More specifically, it offers a `/config` endpoint to `PUT` new values for its compatibility setting.

The following command would change the compatibility setting to NONE for all schemas in the Registry:

```
curl -X PUT http://your-schema-registry-address/config 
     -d '{"compatibility": "NONE"}'
     -H "Content-Type:application/json"
```

This way next registration would be allowed by the Registry as long as the newer schema were valid Avro. The configuration can be set for an specific schema too, simply appending the name (i.e., `/config/subject-name`).

Once the incompatible schema has been registered, the setting should be set back to a more cautious value.

### Summary

The combination of Kafka, Avro and Schema Registry is a great way to store your events in the most compact way possible, while still retains the ability to evolve the corresponding schemas.

However some of the limitations that the Schema Registry imposes make less sense on a development environment. On some occassions, making incompatible changes in a simple way is necessary and recommendable.

The Schema Registry API allows changing the compatibility setting to accept schemas that, otherwise, would be rejected.
