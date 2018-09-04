>Download latest version from here: [![NuGet](https://img.shields.io/nuget/v/QueueIT.QueueToken.svg)](https://www.nuget.org/packages/QueueIT.QueueToken/)
# Queue-it Queue Token SDK for .NET
The Queue-it Queue Token SDK is used to ensure that end users cannot enter the queue without a valid token and to be a container which can car-ry sensitive user information from integrating system into the queue. The token can be issued by  It supports .NET Framework 4.0+.
## The Token
The token consists of two parts. Firstly, a header containing non-sensitive metadata. Secondly the payload of the token. Both header and payload are in JSON format.
### Token Header
```
{ 
  "typ": "QT1",
  "enc": "AES256",
  "iss": 1526464517,
  "exp": 1526464817,
  "ti": "159aba3e-55e1-4f54-b6ee-e5b943d7e885�,
  "c": "ticketania", 
  "e": "someeventid�
}
```
- `typ`: The type of the token. Value must be �QFT1�. Required.
- `enc`: Payload encryption algorithm. Value must be �AES256�. Required.
- `ts`: NumericDate of when token was issued. Required.
- `exp`: NumericDate of when token expires. Optional.
- `ti`: Unique Token ID (e.g. uuid). Used to uniquely identify tokens and restrict replay attacks. Required.
- `c`: The Customer ID of the issuer. Token will only be valid on events on this account. Required.
- `e`: The Event ID of the issuer. If provided, token will only be valid on this event. For example to give a user a better rank at a specific event. Optional.

### Token Payload
```
{ 
  "r": 45,
  "k": "XKDI42W",
  "cd": { "somekey": "somevalue" }
}
```
- `r`: The rank of the user. Must be a decimal value between 0 to <1. Used for determining the priority of the user. If no rank is provided the queue will assign a default rank (e.g. 0). Optional
- `k`: A unique key that holds value to the integrating system (e.g. email or user id). Used to restrict users from issuing multiple queue ids. Op-tional.
- `cd`: Any custom data of the user. This is a set of key-value pairs. Op-tional

## Usage
```
var secretKey = ...;
var token = Token
    .Enqueue("ticketania")
    .WithPayload(Payload
		.Enqueue()
		.WithKey("user@email.com")
		.WithRank(0.4578)
		.WithCustomData("size", "medium")
		.Generate())
    .WithEventId("demoevent")
    .WithValidity(60000)
    .Generate(secretKey);
```

## Serialized Token
> eyJ0eXAiOiJRVDEiLCJlbmMiOiJBRVMyNTYiLCJpc3MiOjE1MzQ3MjMyMDAwMDAsImV4cCI6MTUzOTEyOTYwMDAwMCwidGkiOiJhMjFkNDIzYS00M2ZkLTQ4MjEtODRmYS00MzkwZjZhMmZkM2UiLCJjIjoidGlja2V0YW5pYSIsImUiOiJteWV2ZW50In0.0rDlI69F1Dx4Twps5qD4cQrbXbCRiezBd6fH1PVm6CnVY456FALkAhN3rgVrh_PGCJHcEXN5zoqFg65MH8WZc_CQdD63hJre3Sedu0-9zIs.aZgzkJm57etFaXjjME_-9LjOgPNTTqkp1aJ057HuEiU

The format of the token is [header].[payload].[signature] where each part is Base64Url encoded. The payload is AES 256 encrypted with the secret key supplied in the `.Generate(secretKey)` method. If the �e� key is provided in the header, the secret key on the event must be used. If no �e� key is provided the default key on the customer account must be used.
The token is signed with SHA 256 using the same secret key.
