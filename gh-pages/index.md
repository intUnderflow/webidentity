# WebIdentity
WebIdentity is a method for identifying users uniquely and globally across the internet and sharing information about users in an authorized manner.

Using WebIdentity you can discover information for users and roles they hold across the entire internet.

## Communication
ALL communication is done over HTTPS, HTTPS is generally considered secure for a wide range of sensitive uses over the internet and so we consider it reliable for this use.

## Identities

An identity is in the form `DOMAIN/PATH` where DOMAIN is a Fully Qualified Domain Name and `PATH` is any uniquely identifiable string. An example identity would be `example.com/intunderflow`

## Getting meta-information on an identity
To gain meta-information on a given identity, you perform this two-stage process:
1. You check if the path `DOMAIN/.well-known/webidentity.json` exists, is well formed and is a dictionary with a "server" attribute.
2. If 1 is true, you connect to that server (the attribute must be a fully qualified domain name) and you fetch the PATH, setting your Host header to the name of the original DOMAIN.
3. If 1 is not true, you connect to `DOMAIN/PATH`.
For all valid identities, the value returned from either 2 or 3 will be a JSON file that begins with a dictionary. The dictionary contains meta-information on the identity of the user. For example, it may contain a field marked `publicKeys`.


## Roles and meta-information from others
Certain identities can be assigned "roles" along with other meta-information by certain authorized servers. For example a user with the identity `example.com/intunderflow` could be assigned the role `foo.bar/administrator` or `cloud.google.com/project/webidentity/administrator`, whether the user has these roles is based on the consent of the operators of `foo.bar` and `cloud.google.com`. To get meta-information held on a domain about a user from another domain, you must:
1. Check if the path `DOMAIN/.well-known/webidentity.json` (with a standard, normal host header that matches the domain you are connecting to) exists, is well formed and is a dictionary with a "server" attribute.
2. If 1 is true, you connect to that server (the attribute must be a fully qualified domain name) and you fetch the PATH, setting  your Host header to the domain of the user who you are querying. For example if you were fetching meta-information from `foo.bar` about the user `example.com`, you would connect to `foo.bar` with a host header of `example.com`
3. If 1 is not true, you connect to the original server with the given path, and you set your host header as in 2
The server MAY return meta-information, which may include any observations or notes, and may also includes roles in the "roles" key.

Roles are made of the form `DOMAIN/PATH`, for example `example.com/administrator`. Roles are distinct from identities. For example it is legal for there to be a user with identity `example.com/administrator` who also has the role `example.com/administrator`

# Authentication and Authorization
Certain servers may require authentication in order to connect with them. The requirements to be authorized by that server are up to the server, but may depend on:
1. Being a certain identity
2. Possessing a certain role
3. Any other requirement from external sources

A server can either disclose the full set of information it holds on a user, or none at all. A server MUST NOT disclose partial information, or disclose different, inconsistent information to different users.

To authenticate with a server as a particular identity. A user must provide certain information that only the holder of the identity could disclose. While a user may authenticate with a server in any way (and any implementation may implement any method for any particular server) the default method for authenticating as an identity is through the use of JSON Web Token (https://jwt.io) 

When connecting to a server in the normal way, the user supplies a Basic Authorization header, the username is "webidentity", the password is a JSON web token signed using RSA or ECDSA.

The payload of the JWT holds the following fields:
`url` the full URL (except https://) of the connection being made
`host` the host header (if different from the domain being connected to)
`identity` the name of the identity being claimed
`key-id` the index in the `publicKeys` field of the identity where the public key that matches the key that signed this request is
`time` the time the request was initiated

The server verifies that the url and host match, that the time is not too far into the past. The server gets the meta-information for the identity and gets the key specified by key-id, the server verifies the JWT is signed by the key. If this is done, the server has proved that the user connecting is the holder of a specific identity and can now query for roles or do other authorization checks.

If a server decides a user is not authorized, it should return a 404 or 403 response. (404 being used if the server does not want to acknowledge existance of  certain information if it exists, akin to neither-confirm-nor-deny)
