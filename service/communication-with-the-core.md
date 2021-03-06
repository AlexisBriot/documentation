---
description: Sending commands
---

# Emit an Event

## Why emit an Event? 

Events are emitted from a Service \(e.g.: a web server receiving a request, or a blockchain technology receiving a new transaction\). These events are emitted to achieve a desired effect or to be used as a trigger to make another task happen. Each Service has different kinds of events that you can send to Core. 

## Steps to follow

To emit events from your Service, you'll need to:

* [ ] [Add the definition of the event](communication-with-the-core.md#create-your-event) in the Service's [`mesg.yml`](service-file.md) file
* [ ] [Emit the event](communication-with-the-core.md#publish-your-event) when it happens on the Service

## Event definitions

{% tabs %}
{% tab title="Definition" %}
To create an event, the first step is to update the Service's [`mesg.yml`](service-file.md) file and add an event indexed by its key with the following attributes:

| **Attribute** | **Default value** | **Type** | **Description** |
| --- | --- | --- | --- |
| **name** | `id` | `String` | Name of the event, if not set the name will be the same as the ID you choose for the event. |
| **description** | `""` | `String` | Describe the event, what's its purpose and why users might want to use it. |
| **data** | `{}` | `map<id,`[`Data`](communication-with-the-core.md#data-of-your-event)`>` | The structure of the event's data. |

### Event's data

| **Attribute** | **Default value** | **Type** | **Description** |
| --- | --- | --- | --- | --- |
| **name** | `id` | `String` | Name of the data |
| **description** | `""` | `String` | Description of the data |
| **type** | `String` | [`Type`](communication-with-the-core.md#type-of-your-data) | Type of data |
| **optional** | `false` | `boolean` | Mark the data as optional |

### Data's type

The data type can be one of the following:

* `String`
* `Boolean`
* `Number`
* `Object`
{% endtab %}

{% tab title="Example" %}
Example of an event definition in a [`mesg.yml`](service-file.md) file:

{% code-tabs %}
{% code-tabs-item title="mesg.yml" %}
```yaml
...
events:
    eventX:
        name: "Event X"
        description: "This is event X"
        data:
            foo:
                name: "Foo"
                description: "Foo is the first data"
                type: String
                optional: false
            bar:
                name: "Bar"
                description: "Bar is the second data"
                type: Boolean
                optional: true
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endtab %}
{% endtabs %}

## Emit an Event

To emit events from the Service to the Core, the Service has to follow the [Protobuffer definition](https://github.com/mesg-foundation/core/blob/dev/api/service/api.proto) and use [gRPC](https://grpc.io/).

{% hint style="info" %}
Consider emitting event when the service is ready. If the service needs to synchronize data first, it should wait for the synchronization to complete before emitting events.
{% endhint %}

{% tabs %}
{% tab title="Request" %}
### `Service.EmitEvent`

| **Name** | **Type** | **Required** | **Description** |
| --- | --- | --- | --- |
| **service** | [Service](service-file.md) | Required | Object containing the service definition loaded from the yml service file. |
| **eventKey** | `String` | Required | The event's key defined in the [service file](https://github.com/mesg-foundation/documentation/tree/c1028b6f9d709adf2ad46364ce7baaa37e27ff8e/service/service/service-file.md) |
| **eventData** | `String` | Required | The event's data in JSON format |

```javascript
{
    "service": {
      ...
      "events": {
        "eventX": {
          "data": {
            "foo": { "type": "String" }
          }
        }
      },
      ...
    },
    "eventKey": "eventX",
    "eventData": "{\"foo\":\"hello\",\"bar\":false}"
}
```
{% endtab %}

{% tab title="Reply" %}
| **Name** | **Type** | **Description** |
| --- | --- | --- |
| **error** | `String` | A string that contains the error if any |

```javascript
{
    "error": ""
}
```
{% endtab %}
{% endtabs %}

### Examples

{% tabs %}
{% tab title="Node" %}
{% code-tabs %}
{% code-tabs-item title="index.js" %}
```javascript
const grpc = require('grpc')
const yaml = require('js-yaml')
const fs = require('fs')
const api = grpc.load(__dirname + '/api/service/api.proto').api
const service = new api.Service(
  process.env.MESG_ENDPOINT,
  grpc.credentials.createInsecure()
)

service.EmitEvent({
  service: yaml.safeLoad(fs.readFileSync("./mesg.yml")),
  eventKey: "eventX",
  eventData: JSON.stringify({
    foo: "hello",
    bar: false
  })
}, (err, reply) => {
  // handle response if needed
})
```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endtab %}

{% tab title="Go" %}
{% code-tabs %}
{% code-tabs-item title="main.go" %}
```go
package main

import (
	"context"
	"encoding/json"
	"io/ioutil"
	"log"
	"os"


	api "github.com/mesg-foundation/core/api/service"
	"github.com/mesg-foundation/core/service"
	"google.golang.org/grpc"
	yaml "gopkg.in/yaml.v2"
)

type EventX struct {
	Foo string
	Bar bool
}

func main() {
	content, _ := ioutil.ReadFile("./mesg.yml")
	var service service.Service
	yaml.UnmarshalStrict(content, &service)

	connection, _ := grpc.Dial(os.Getenv("MESG_ENDPOINT"), grpc.WithInsecure())
	cli := api.NewServiceClient(connection)

	eventX, _ := json.Marshal(EventX{
		Foo: "hello",
		Bar: false,
	})

	reply, _ := cli.EmitEvent(context.Background(), &api.EmitEventRequest{
		Service:   &service,
		EventKey:  "eventX",
		EventData: string(eventX),
	})
	log.Println(reply)
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endtab %}
{% endtabs %}



