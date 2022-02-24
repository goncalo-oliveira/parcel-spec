# Parcel Protocol Extensions

This document describes extensions to the base protocol. They may be experimental and are not necessarily required to be implemented.

## Mapping Requests and Responses

*Experimental*

The idea behind this is to allow to request data to the other end of the socket and retrieve a response mapped to that request.

To achieve this, we create an identifier with an unique value. After sending the message, the sender creates a handler to wait for a response with the same identifier.

The receiver, upon receiving the message with the request, will reply using exactly the same identifier.
Once the sender receives the response, it will cancel the waiting handler and handle the response.

```
Socket1     ------------------------> Socket2
                       message_id
WaitHandler1.wait()

Socket1     <------------------------ Socket2
                       message_id
WaitHandler1.release()
```

The implementation should support a generic way of doing this.

Depending on the use case, it might be a good idea to separate this from the basic usage; as such, it is recommended to include a prefix and a topic on the identifier, to define the request. An example could be

```
:/sensors/temperature1/readings@03be2cfe
```

The above could be a request to retrieve the readings on a temperature sensor.

| Element    | Value                          |
|------------|--------------------------------|
| Prefix     | :                              |
| Topic      | /sensors/temperature1/readings |
| Request Id | @03be2cfe                      |

The prefix helps identifying this is a request and that a response is expected. The topic identifies what is it that we are requesting. The request identifier ensures we are delivering the results to the right request.
