# MSC3967: Do not require UIA when first uploading cross signing keys

When a user first sets up end-to-end encryption cross-signing, their client
uploads their cross-signing keys to the server.

This [upload operation](https://spec.matrix.org/v1.6/client-server-api/#post_matrixclientv3keysdevice_signingupload)
requires a higher level of security by applying User-Interactive Auth (UIA) to
the endpoint.

This creates a usability issue at the point of user registration where a client
will typically want to immediately set up cross-signing for a new user.

The issue is that the client will immediately need the user to re-authenticate
even though the user just authenticated.

This usability issue has given rise to workarounds such as a
[configurable grace period](https://matrix-org.github.io/synapse/latest/usage/configuration/config_documentation.html#ui_auth)
(`ui_auth`.`session_timeout`) in Synapse whereby UIA will not be required for
uploading cross-signing keys where authentication has taken place recently.

This proposal aims to provide for a standard way to address this UIA usability
issue with respect to setting up cross-signing.

## Proposal

For the `POST /_matrix/client/v3/keys/device_signing/upload` endpoint, the
Homeserver MUST require User-Interactive Authentication (UIA) _unless_:
 - there is no existing cross-signing master key uploaded to the Homeserver, OR
 - there is an existing cross-signing master key and it exactly matches the
   cross-signing master key provided in the request body. If there are any additional
   keys provided in the request (self signing key, user signing key) they MUST also
   match the existing keys stored on the server. In other words, the request contains
   no new keys. If there are new keys, UIA MUST be performed.

In these scenarios, this endpoint is not protected by UIA. This means the client does not
need to send an `auth` JSON object with their request.

This change allows clients to freely upload 1 set of keys, but not modify/overwrite keys if
they already exist. By allowing clients to upload the same set of keys more than once, this
makes this endpoint idempotent in the case where the response is lost over the network, which
would otherwise cause a UIA challenge upon retry.

## Potential issues

See security considerations below.


## Alternatives

There has been some discussion around how to improve the usability of
cross-signing more generally. It may be that an alternative solution is to
provide a way to set up cross-signing in a single request.

## Security considerations

This change could be viewed as a degradation of security at the point of setting
up cross-signing in that it requires less authentication to upload cross-signing
keys on first use.

However, this degradation needs to be weighed against the typical real world
situation where a Homeserver will be applying a grace period and so allow a
malicious actor to bypass UIA for a period of time after each authentication.

## Unstable prefix

Not applicable as client behaviour need not change.

## Dependencies

None.
