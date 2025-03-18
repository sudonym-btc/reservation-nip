NIP-XX
======

Reservations
--------------

`draft` `optional`

This NIP defines two new event types for recording reservations. `30437` is a `reservation request`, referencing another event kind ID that has been "reserved". `30438` is a `reservation`, representing the confirmation that a reservation has been accepted.
A `reservation request` is not meant to be published other than in a DM, but only commits to a buyer's intent.

To check that a reservation is non-maliciously published, clients and relays MUST. `reservation` can only be signed by the seller, otherwise they should be disregarded.

If the seller does not want to allow the buyer to publish `reservation` events on their behalf, they should not sign the `reservation` that they send to the buyer. Instead, upon receipt of a `reservation request` by the buyer, the buyer should sign it with their own keys and send it gift-wrapped back to the seller for them to publish.

The client MUST verify that any `reservation` received is correctly signed, and the buyer MUST verify that the included encrypted `reservation request` event id commits to the event id of the `reservation request` that they agreed to.
If the DM'd `reservation` is signed, it can then be broadcast.
The contained `reservation request` only needs to be signed by the buyer when the seller is publishing it.

## Protocol flow

1. User browses NIP-99 compatible classifieds. They want to place an order with the seller, but confirmation from the seller is required. User creates a new message thread to the seller after creating their signed `reservation request` attaching the `reservation request`'s `d` tag to the message.
The `reservation request` is then NIP-17 encrypted, and sent as the content in a new message to the seller.

### Buyer initiated request

`reservation request` signed by buyer
```json
{
  "id": "<usual hash>",
  "pubkey": senderPublicKey,
  "created_at": randomTimeUpTo2DaysInThePast(),
  "kind": 30437, // reservation request
  "tags": [
    ["e", "<d tag of classified>"],
    ["d", "<Thread for all subsequent reservation request updates>"],
    ["commit_preimage", "<Random preimage, encrypted for self>"]
  ], // no tags
  "content": "",
  "from": Date,
  "to": Date,
  "quantity": num,
  "cost": nip44Encrypt(cost, senderPublicKey, receiverPublicKey),
  "sig": "<signed by senderPrivateKey>"
}
```

The event is then NIP-17 encrypted and sent to the seller as an encrypted DM.

2. Seller sees incoming message and wants to accept `reservation request`, unwraps buyer-signed `reservation request` from giftwrap, confirms details, and attaches the SHA256(`reservation_request_preimage_hash`) tag to the `reservation` kind which is then broadcast.

### Seller initiated request

`reservation request`

```json
{
  "id": "<usual hash>",
  "pubkey": senderPublicKey,
  "created_at": randomTimeUpTo2DaysInThePast(),
  "kind":  30437, // reservation request
  "tags": [
    ["e", "<d tag of classified>"],
    ["d", "<Thread for all subsequent reservation request updates>"],
    ["commit_preimage", "<Random preimage, unencrypted>"]
  ], // no tags
  "content": "",
  "from": Date,
  "to": Date,
  "quantity": num,
  "cost": nip44Encrypt(cost, senderPublicKey, receiverPublicKey),
  "sig": "<signed by senderPrivateKey>"
}
```

`reservation`

```json
{
  "id": "<usual hash>",
  "pubkey": senderPublicKey,
  "created_at": randomTimeUpTo2DaysInThePast(),
  "kind":  30437, // reservation request
  "tags": [
    ["e", "<d tag of classified>"],
    ["d", "<Thread for all subsequent reservation request updates>"]
    ["commit_hash", "<SHA256(reservation_request.commit_preimage)>"]
  ], // no tags
  "content": nip44Encrypt(cost, senderPublicKey, receiverPublicKey),
  "from": Date,
  "to": Date,
  "quantity": num,
  "sig": "<signed by senderPrivateKey>"
}
```

In the case that the seller initiates the arrangement, they send the commit in the message to the buyer, such that the buyer can prove that a reservation belongs to them by revealing the preimage after publishing the filled-out hashed commit in the seller-signed `commit` tag.