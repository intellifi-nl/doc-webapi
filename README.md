Brain web API documentation
===========================

This document contains the official Intellifi Brain web API documentation. The Brain web API is a [RESTful API](https://en.wikipedia.org/wiki/Representational_state_transfer) that allows you to interact with our equipment in a powerful and simple way. Our end-to-end [solution](http://intellifi.nl/) allows you to localize your real world items based different RFID technologies. The results of these physical detections are immediately available on our cloud based API (on-prem is available).

By default the API is accessible on: https://`brain_host`/api/`resource`.`format`/`id`
* `brain_host` will be provided to you when you are evaluating or purchasing our product. Please contact us if you want to do some experiments. We can provide with information about our public sandbox.
* `resource` shall contain the resource that you want to query.
* **optional** `format` field allows you to request a different output format (json, csv or txt). The default (and most used) serialization is [JSON](https://en.wikipedia.org/wiki/JSON). For this reason you may completely leave out the extension.
* **optional** `id` indicates which specific resource you wish to access. Please refer to the individual resources for more information on the type of id that is used. If you omit `id` the server will return a list with all items in the resource.

You can authenticate your HTTP requests with an API key. You may add this key to query paramters (?key=`your_key`)) or as an extra header `X-Api-Key`=`your_key`. Your login session to the brain will also authenticate API requests. This makes exploring the API easier. More important details on security can be found on [this page](security.md).

As with every web API you can request new information by performing HTTP requests. We also offer [pushing technologies](https://github.com/intellifi-nl/doc-push). They will allow you to be directly informed when something changes, instead of polling for the changes. Most use cases that we had until now can be implemented by using the web API only.

Contents
--------
* [Getting started](#getting-started)
* [Terminology](#terminology)
  * [Item](#item)
  * [Location](#location)
* [Resources](#resources)
  * [Id](#id)    
  * [Time fields](#time-fields)
  * [Collections](#collections)
  * [Querying](#querying)
  * [References](#references)  

[Getting started](quick-start.md)
===============

We do advise you to take a quick look at our [quick start](quick-start.md) for some hints and tooling. In the rest of the documentation we assume that you understand HTTP, JSON and that you know which tools you can use to work with them.

Terminology
===========

The whole Intellifi concept is based upon [items](#item) and [locations](#location). Please take some time to familiarize yourself with these definitions. They will make it way easier to understand this API.

Item
----

An item is an object (or even a person) that you tagged with some kind of RFID emitter. At this moment our eco system can detect both RFID EPC Gen2 tags, also know as RAIN RFID tags, and modern Bluetooth LE beacons (including iBeacon and Eddystone). When an item is detected for the first time by one of our detectors then it's immediately created as an item resource. Our system keeps this resource as a reference to this item, it's never deleted. You can use it to see where the item is or where it has been seen for the last time.

Location
--------

A location is an area in which items can be detected.  The actual detections are made and transmitted to our server by devices ([Intellifi SmartSpot](https://intellifi.nl/home/products/)) that you install at the physical location. The events are transferred to the server by a default network connection. The detection area is naturally limited to the range of the used RFID technology. Passive EPC Gen2 tags have a range of approximately 10 meters, beacons can easily have a range of 100+ meters. If multiple SmartSpots are reporting to a one location then the events are merged at the server level. Please note that the location is a server-side abstraction that allows you to be flexible when you have multiple SmartSpots. I.e.: it's possible to connect external antennas to the SmartSpot. By default they are used to enlarge the reach of the overall SmartSpot. You may configure individual external antennas to report their detections to a seperate location. By doing so you are essentially creating a second virtual spot.

Location configuration is done automatically when you connect a SmartSpot to a brain for the first time. A location is created that contains the serial number of the spot. I.e 'spot203'. You can easily change this label by doing a `PUT` on the location resource itself. We encourage you to add a meaningful name to a location (e.g. 'kitchen') as it's being used in the user interface at several places.

Resources
=========

Resources are the core of the web API. We tried to limit the number of resource types to the core concepts. An elaborate description of each individual resource type is available on [this page](resources.md).

The top level resources are all collections, they contain the individual resources. Queries to these resources are wrapped inside a query object with information about the query and an array with the results.

Id
--

Individual resources always start with the `url` and `id` field. They are both unique identifiers of a resource (the `url` is actually built with the `id`). The `id` is always generated by the brain server and is chronological as well. This means that you can sort on the id if you wish. You may add the `id_only=true` condition to the query part of the url if you don't wish to receive the `url` field(s). This option will also apply to resource references. The `url` fields are shown by default to increase explorability of the API. If you are creating an automated import then you probably want to add `id_only=true` to all of your requests.

Time fields
-----------

Most time fields are appended with `_time`, the value is always an [ISO 8601 timestring](https://en.wikipedia.org/wiki/ISO_8601) in the UTC time zone, e.g. '2016-01-27T08:38:55.255Z'. The precision may vary, we add the number of milliseconds if avaialble.

Collections
-----------

| Name | Description | 
| ----- | ---- | ----------- |
| [/api/spots](resources.md#spots) | Status and config information for Intellifi SmartSpot devices. |
| [/api/locations](resources.md#locations) | Locations that SmartSpots may report to. |
| [/api/items](resources.md#items) | All detected items. |
| [/api/presences](resources.md#presences) | Live item detections on locations. Be aware that items can be detected at multiple places at the same time. |
| [/api/sets](resources.md#sets) | Create your Item sets; a collection of Items that should grouped. |
| [/api/subscriptions](resources.md#subscriptions) |  Subscriptions for events (see: /api/events) and Webhooks.
| [/api/events](resources.md#events) | Collection of events. |
| [/api/services](resources.md#services) | Background processes overview and configuration |

Querying
--------

You can query a collection by sending an HTTP request. This request may contain query parameters to further specify your query.

| Keyword | Description  | Default | Example |
| ------- | ------------ | ------- | ------- |
| `after`, `before` | Limits on `time_created`, exclusive. | | ?after=2016-01-27T08:38:55.255Z
| `after_id`, `before_id` | Limits directly on `id` excludes the given `id` value, please note that `id` is chronological. | |  ?after_id=56a88364e783152127d15340 |
| `from`, `until` | Limits on `time_created`, inclusive. | | ?until=2016-01-27T08:38:55.255Z
| `from_id`, `until_id` | Limits on `id`, includes the given `id` value. | | ?from_id=56a88364e783152127d15340 |
| `id_only` | Removes `url` fields from output and shows `_id` instead of `_url` in references. | false | ?id_only=true
| `results_only` | Removes wrapping JSON object with information about query, only sends back a JSON array with the applicable resources | false | ?results_only=true |
| `sort` | Allows you to sort on on or more fields in the resource. You may append a minus sign (`-`) to request reverse order (new to old). | -id | ?sort=-move_count,technology |
| `populate` | Expand a reference into the actual resource (lookup). You may add multiple fields by giving a comma seperated value. | | ?populate=location,item |
| `limit` | Sets the maximum number of returned resources. You may increase this number to large values, keep in mind that query times could become large. We advise you to use the [pagination](pagination.md) feature whenever you can. | 100 | ?limit=5

You can also query on one or more resource fields in your query parameters. Just add the field/value combinations that you want to query on. Multiple fields are combined with a logical AND (at this moment we don't support OR). I.e:

https://`brain_host`/api/items?location=5698f9c5e7831505e418755a&is_present=true

String fields can be queried with wildcards: label=*tag* would match mytag1, mytag2 etc. Pleae note that this is not supported for the `code_hex` field in the items resource, it's a binary field at this moment.

The default query behaviour (when no query paramters are given) is that the last 100 resources are shown (newest resources are shown first). The collections are [paginated](pagination.md) when they contain more than 100 resources. You can follow the next_url to retrieve the rest of the resources.

References
----------

Most resources contain one or more fields that reference another resource in the API. For example, an item resource will be pointing to some location. These references can be shown in three ways, depending on the arguments that you supply in the query part of your request url.

1. By default it's shown as a url. This helps you in navigating to it, if you have the right browser extension then you can just click it. The '_url' is postfixed to the field name. E.g. location becomes location_url.
![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/explore.png)

2. If you add `id_only=true` in the query then only the id of the resource is shown. The name of the field is postfixed with '_id'. E.g. location becomes location_id.
![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/explore_idonly.png)

3. If you add `populate=fieldname,fieldname2,etc` in the query string then the brain will try to add the documents as an extra level in the results. The name of the field is not appended with a postfix in this case. If the lookup fails then `null` is returned as value.
![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/explore_populate.png)
