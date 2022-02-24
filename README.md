# Parcel Protocol

Parcel is a sockets communication protocol. It's intended to be very generic, allowing specific implementations without skeweing the base protocol.

## Packet Format

A packet is always defined with the following structure

| Field        | Length      | Description                                  |
|--------------|-------------|----------------------------------------------|
| Head (0xf0)  |     1 byte  | The head of the packet                       |
| Length       |     4 bytes | The total length of the messages             |
| Messages     |     N bytes | The messages                                 |
| Tail (0xff)  |     1 byte  | The tail of the packet                       |

**Head**

A single byte (0xf0) that identifies the head of the packet.

**Length**

The total length of the messages inside the packet.

**Tail**

A single byte (0xff) that identifies the tail of the packet.

## Message Format

Each message inside the packet will be defined as

| Field        | Length      | Description                          |
|--------------|-------------|--------------------------------------|
| Identifier   | 1 + N bytes | The message identifier               |
| Content Type | 1 + N bytes | The message content type             |
| Content      | 2 + N bytes | The message content                  |
| Signature    | 1 + N bytes | The packet hash                      |

All fields in the message are optional, however, their length still needs to be defined. For example, a *heartbeat* could be implemented without an *Identifer*, *Content* or *Content-Type*. Here's what the packet would look like

| Field        | Hex value |
|--------------|-----------|
| Head         | 0xf0      |
| Length       | 0x05 0x00 0x00 0x00 |
| Identifier   | 0x00      |
| Content Type | 0x00      |
| Content      | 0x00 0x00 |
| Signature    | 0x00      |
| Tail         | 0xff      |

**Identifier**

This is the message identifier. It is recommended that this value is unique, but it's not a rule and it depends entirely one the use case.
This field can be empty.

**Content Type**

This field should reflect the kind of data to be expected inside the message *Content*. If empty, binary content should be expected.

**Content**

The actual content of the message. The data type should be a reflection of the *Content Type*. The actual content format depends on the use case.

**Signature**

When used, this field should contain an hash value computed from the entire message (identifier, content type and content).
The way the hash is calculated depends on the use case and shouldn't be tied to the implementation.

## Encryption

There aren't any fixed rules around encryption, however, there are recommendations and best practices.

Unless we are talking about a TLS certificate, encryption means encrypting the message content before encoding and decrypting after decoding. This means an eavesdropper can acknowledge and recognize the format of the message, but it can't read its contents. Depending on the sensitivity of the data, there are at least two ways we could go over this.

- Encrypt content only
- Encrypt entire message

If the *Identifier* and the *Content-Type* aren't considered sensitive information, we can just encrypt the *Content* of the message. We can optionally *alter* the *Content-Type* to indicate the content is encrypted.

```json
{
    "id": "non-sensitive-identifier",
    "content-type": "application/json",
    "content": [] // json-content
}
// ==> encryption
{
    "id": "non-sensitive-identifier",
    "content-type": "application/json", // optional: "application/json+encrypted"
    "content": []                       // encrypted-content
}
```

If the entire message can or should be considered sensitive information, then we should encrypt the entire message, including *Identifier*, *Content-Type* and *Signature*. Upon doing so, we should encode a message where the *Content* is the encrypted content, with no *Identifier*. Optionally, we can set a *Content-Type* to let the decoder know it's an encrypted message (if not all). Also optional, we can use the *Signature* to hash the encrypted content.

```json
{
    "id": "identifier",
    "content-type": "application/json",
    "content": [] // json-content
}
// ==> encryption
{
    "id": null,
    "content-type": null, // optional: "application/octet-stream+encrypted"
    "content": [],        // encrypted-content
    "signature": null     // optional: "encrypted-content-hash"
}
```


**TLS**

Using a TLS certificate over the socket connection is a good way to ensure data encryption on transport. It has no link to the protocol and is applied at the socket transport level.
