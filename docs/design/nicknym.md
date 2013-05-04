@title = 'Nicknym'
@toc = true
@summary = "Automatic discovery and validation of public keys."

Introduction
==========================================

Nicknym is a system to map user nicknames to public keys. With Nicknym, the user to be able to think solely in terms of nickname, while still being able to communicate with a high degree of security (confidentiality, integrity, and authenticity). Essentially, Nicknym is a system for binding human-memorable nicknames to a cryptographic key via automatic discovery and automatic validation.

Nicknym is a federated protocol: a Nicknym address is in the form `username@domain` just alike an email address and Nicknym includes both a client and a server component. Although the client can fall back to legacy methods of key discovery when needed, domains that run the Nicknym server component enjoy much stronger identity guarentees.

Nicknym is key agnostic, and supports whatever public key information is available for an address (OpenPGP, OTR, X.509, RSA, etc). However, Nicknym enforces a strict one-to-one mapping of address to public key.

Existing forms of secure identity are deeply flawed. These systems rely on either a single trusted entity (e.g. Skype), a vulnerable Certificate Authority system (e.g. S/MIME), or keys that cannot be made human memorable (e.g. OpenPGP, OTR). When an identity system is hard to use, it is effectively compromised because too few people take the time to use it properly.

The broken nature of existing identities systems (either in security or in usability) is especially troubling because identity remains a bedrock precondition for any message security: you cannot ensure confidentiality or integrity without confirming the authenticity of the other party. Nicknym is a protocol to solve this problem in a way that is backward compatible, easy for the user, and includes very strong authenticity when possible.

Goals
==========================================

**High level goals**

* Pseudo-anonymous and human friendly addresses in the form `username@domain`.
* Automatic discovery and validation of public keys associated with an address.
* The user should be able to use Nicknym without understanding anything about public/private keys or signatures.

**Technical goals**

* Wide utility: nicknym should be a general purpose protocol that can be used in wide variety of contexts.
* No revocation: instead of key revocation, support short lived keys that frequently and automatically refresh.
* Prevent dangerous actions: Nicknym should fail hard when there is a possibility of an attack.
* Minimize false positives: because Nicknym fails hard, we should minimize false positives where it fails incorrectly.
* Resistant to malicious actors: Nicknym should be externally auditable in order to assure service providers are not compromised or advertising bogus keys.
* Resistant to association analysis: Nicknym should not reveal to any actor or network observer a map of a user's associations.

**Non-goals**

* Nicknym does not try to create a decentralized or peer-to-peer identity system.

The binding problem
=============================================

Nicknym attempts to solve the problem of binding a human memorable identifer to a cryptographic key. If you have the identifier, you should be able to get the key with a high level of confidence, and vice versa. The goal is to have decentralized, human memorable, globally unique public keys. In other words, to violate [Zooko's triangle](https://en.wikipedia.org/wiki/Zooko's_triangle) by making a few consessions.

There are a number of established methods for binding identifier to key:

* [Web of Trust (WOT)](http://en.wikipedia.org/wiki/Web_of_trust)
* Trust on First Use (TOFU)
* [X.509 Certificate Authority System](https://en.wikipedia.org/wiki/X.509)
* [DNSSEC](https://en.wikipedia.org/wiki/Dnssec)
* [Shared Secret](https://en.wikipedia.org/wiki/Socialist_millionaire)
* Mail-back Verification
* [Network Perspective](http://convergence.io/)
* Global Append-only Log
* Nonverbal Feedback (a la ZRTP)

The methods differ widely, but they all try to solve the same general problem of proving that a person or organization is in control of a particular key.

Nicknym uses a combination of these methods, utilizing TOFU, X.509, Network Perspective, and additional methods we call "Provider Keys" and "Federated Web of Trust" (FWOT).

1. Nicknym starts with TOFU of user keys, because it is easy to do and backward compatible with legacy providers. In TOFU, your client naively accept the key of another user when it first encounters it. When you TOFU a user key, you are making a bet that possible attackers against you did not have the foresight to specifically target you with a false key during discovery.
2. Next, we add X.509. For those providers that publish the public keys of their users, we require that these keys be fetched over validated TLS. This makes third party attacks against TOFU more difficult, but also places a lot of trust in the providers (and the Certificate Authorities).
3. Next, we add a simple form of Network Perspective where the client can ask one provider what key another provider is distributing. This allows a user's client to be able to audit their provider and keep them honest in an automated manner. If a service provider distributes bogus keys, their users and other providers will be quickly alerted to the problem.
4. Next, we add Provider Keys. If a service provider has a provider key, the public keys of its users are additionally signed by the provider with the "provider key". If your client has the correct provider key, you no longer need to TOFU the keys of the provider's users. This has the benefit making it possible for a user to issue new keys, and to add support for very short-lived keys rather than trying to use key revocation. A service provider is much less likely to lose their private key or have it compromised, a significant problem with TOFU of user keys.
5. Finally, we add a Federated Web of Trust. The system works like this: each service provider is responsible for the due diligence of properly signing the keys of a few other providers, akin to the distributed web of trust model of OpenPGP, but with all the hard work of proper signature validation placed upon the service provider. When a user communicates with another party who happens to use a service provider that participates in the FWOT, the user’s software will automatically trace a chain of signature from the other party’s key, to their service provider, to the user’s own service provider (with some possible intermediary signatures). This allows for identity that is verified through an end-to-end trust path from any user to any other user in a way that can be automated and is human memorable. Support for a FWOT allows us to bypass entirely X.509 Certificate Authorities, to gracefully handle short lived provider keys, and to handle emergency re-key events if a provider's key is lost.

As we move down this list, each measure taken gets more complicated, requires more provider cooperation, and provides less additional benefit than the one before it. Nevertheless, each measure contributes some important benefit toward the goal of automatic binding of user identity to public key.

**Questions**

*Why not use WOT?* Most users are empirically unable to properly maintain a web of trust. The concepts are hard and it is easy to mess up the signing practice.

*Why not use DNSSEC?* Many reasons. DNS records are slow to update. RSA Public keys will soon be too big for UDP packets (though this is not true of ECC), so putting keys in DNS will mean putting a URL to a key in DNS, so you might as well just use TLS. DNSSEC could still be of added benefit if you put the fingerprint in the DNS record. Mostly, however, a simple HTTP get request is a lot easier to deal with than DNS, both for the client and the server.

*Why not use Shared Secret?* Shared secrets, like with the Socialist Millionaire protocol, are cool in theory but prone to user error and frustration in practice. Was the secret "Invisible Zebra" or "invisibleZebra"?

*Why not use Mail-back Verification?* If the provider distributes user keys, there is not any benefit to mail-back verification. However, it would be good to add support for mail-back verification for non-cooperating legacy providers.

*Why not use Global Append-only Log?* Maybe we should, they are neat. However, current implementations are resource intensive and experimental (e.g. namecoin).

*Why not use Nonverbal Feedback?* ZRTP can use non-verbal clues to establish secure identify because of the nature of a live phone call. This doesn't work for text only messaging.


Related work
===================================

**WebID and BrowserID**

What about WebID or BrowserID? These are both interesting cryptographic identity standards that are gaining support and implementations. So why do we need something new?

These protocols, and the poorly conceived OpenID Connect, are designed to address a fundamentally different problem: authenticating a user to a website. The problem of authenticating users to one another requires a different architecture entirely. There are some similarities, however, and in the long run Nicknym could be combined with something like BrowserID.

**STEED**

[STEED](http://g10code.com/steed.html) is a proposal with very similar goals to Nicknym. In a nutshell, Nicknym basically looks very similar to STEED when the domain owner does not support Nicknym. STEED includes four main ideas:

* trust upon first contact: Nicknym uses this as well, although this is the fallback mechanism when others fail.
* automatic key distribution and retrieval: Nicknym uses this as well, although we used HTTP for this instead of DNS.
* automatic key generation: Nicknym is designed specifically to support automatic key generation, but this is outside the scope of the Nicknym protocol and it is not required.
* opportunistic encryption: Again, Nicknym is designed to support opportunistic encryption, but does not require it.

Additional differences include:

* Nicknym is key agnostic: Nicknym does not make an assumption about what types of public keys a user wants to associate with their address.
* Nicknym is protocol agnostic: Nicknym can be used with SMTP, XMPP, SIP, etc.
* Nicknym relies on service provider adoption: With Nicknym, the strength of verification of public keys rests the degree to which a service provider adopts Nicknym. If a service provider does not support Nicknym, then effectively Nicknym opperates like STEED for that domain.


Nicknym protocol
==============================

Definitions
-------------------------

* **address**: A globally unique handle in the form user@domain (i.e. an email, SIP, or XMPP address).
* **provider**: A service provider that offers end-user services on a particular domain.
* **user key**: A public/private key pair associated with a user address. If not specified, "user key" refers to the public key.
* **provider key**: A public/private key pair owned by the provider. The address associated with this key is just the domain of the service provider.
* **validated key**: A key is "validated" if the nickagent has bound the user address to a public key.
* **nickagent**: Client side program that manages a user's contact list, the public keys they have encountered and validated, and the user's own key pairs.
* **nickserver**: Server side daemon run by providers who support Nicknym.

Nickserver requests
-----------------------

A nickagent will attempt to discover the public key for a particular user address by contacting a nickserver. The nickserver returns JSON encoded key information in response to a simple HTTP request with a user's address. For example:

    curl -X POST -d address=alice@domain.org https://nicknym.domain.org:6425

* The port is always 6425.
* The HTTP verb may be POST or GET.
* The request must use TLS (see [Query security](#Query.security)).
* The query data should have a single field 'address'.
* For POST requests to nicknym.domain.org, the query data may be encrypted to the the public OpenPGP key nicknym@domain.org (see [Query security](#Query.security)).

Requests may be local or foreign, and for user keys or for provider keys.

* **local** requests are for information that the nickserver is authoritative. In other words, when the requested address is for the same domain that the nickserver is running on.
* **foreign** request are for information about other domains.
* **user key** requests are for addresses in the form "username@domain".
* **provider key** requests are for addresses in the form "domain".

**Local, Provider Key request**

For example:

    https://nicknym.domain.org:6425/?address=domain.org

The response is the authoritative provider key for that domain.

**Local, User Key request**

For example:

    https://nicknym.domain.org:6425/?address=alice@domain.org

The nickserver returns authoritative key information from the provider's own user database. Every public key returned for local requests must be signed by the provider's key.

**Foreign, Provider Key request**

For example:

    https://nicknym.domain.org:6425/?address=otherdomain.org

1. First, check the nickserver's cache database of discovered keys. If the cache is not old, return this key.
2. Otherwise, fetch provider key from the provider's nickserver, cache the result, and return it.

**Foreign, User Key request**

For example:

    https://nicknym.domain.org:6425/?address=bob@otherdomain.org

* First, check the nickserver's database cache of nicknyms. If the cache is not old, return the key information found in the cache.
* Otherwise, attempt to contact a nickserver run by the provider of the requested address. If the nickserver exists, query that nickserver, cache the result, and return it in the response.
* Otherwise, fall back to querying existing SKS keyservers, cache the result and return it.
* Otherwise, return a 404 error.

If the key returned for a foreign request contains multiple user addresses, they are all ignored by nicknym except for the user address specified in the request.

Nickserver response
---------------------------------

A nickserver response is a JSON encoded map with a field "address" plus one or more of the following fields: "openpgp", "otr", "rsa", "ecc", "x509-client", "x509-server", "x509-ca".

A nickserver response is always signed with the OpenPGP public signing key associated with the address nicknym@domain.org. The signature is ASCII armored and appended to the JSON.

For example:

    {
      "address": "alice@example.org",
      "openpgp": "6VtcDgEKaHF64uk1c/crFhRHuFW9kTvgxAWAK01rXXjrxEa/aMOyXnVQuQINBEof...."
    }
    -----BEGIN PGP SIGNATURE-----
    iQIcBAEBCgAGBQJRhWO+AAoJEIaItIgARAAl2IwP/24z9CjKjD0fd27pQs+r+e3h
    p8KAYDbVac3+c3vm30DjHO/RKF4Zq6+sTAIkrFvXOwYJl9KgjMpQVV/voInjxATz
    -----END PGP SIGNATURE-----

If the data in the request was encrypted to the public key nicknym@domain.org, then the JSON response and signature are additionally encrypted to the symmetric key found in the request and returned base64 encoded.

Query balancing
------------------------

A nickagent must choose what IP address to query by selecting randomly from among hosts that resolve from `nicknym.domain.org` (where `domain.org` is the domain name of the provider).

If a host does not response, a nickagent must skip over it and attempt to contact another host in the pool.

Query security
--------------------------

TLS is required for all nickserver queries.

When querying https://nicknym.domain.org, nickagent must validate the TLS connection in one of three ways:

1. Using a commercial CA certificate distributed with the host operating system.
2. Using a seeded CA certificate (see [Discovering nickservers](#Discoverying.nickservers)).
3. Using a custom self-signed CA certificate discovered for the domain, so long as the CA certificate was discovered via #1 or #2. Custom CA certificates may be discovered for a domain by making a provider request of a nickserver (e.g. https://nicknym.known-domain.org/?address=new-domain.org).

Optionally, a nickagent may make an encrypted query like so:

0. Suppose the nickagent wants to make an encrypted query regarding the address alice@x.org.
1. Nickagent discovers the public key for nicknym@domain.org
2. Nickagent uses the OpenPGP key for nicknym@domain.org to encrypt the body of the request (using POST). The request body should consist of the address being queried and the second line a randomly generated 128 bit symmetric key. The request can be foreign or local.
3. The body of the nickserver' response is encrypted using AES128 using the symmetric key.

Comment: although it may seem excessive to encrypt both the request via TLS and the request body via OpenPGP, the reason for this is that many requests will not use OpenPGP.

Automatic key validation
----------------------------------

A key is "validated" if the nickagent has bound the user address to a public key.

Nicknym supports three different levels of key validation:

* Level 3 - path trusted: A path of cryptographic signatures can be traced from a trusted key to the key under evaluation. By default, only the provider key from the user's provider is a "trusted key".
* Level 2 - provider signed: The key has been signed by a provider key for the same domain, but the provider key is not validated using a trust path (i.e. it is only registered)
* Level 1 - registered: The key has been encountered and saved, it has no signatures (that are meaningful to the nickagent).

nickagent will try to validate using the highest level possible.

Automatic renewal
-----------------------------

A validated public key is replaced with a new key when:

* The new key is path trusted
* The new key is provider signed, but the old key is only registered.
* The new key has a later expiration, and the old key is only registered and will expire "soon" (exact time TBD).
* The agent discovers a new subkey, but the master signing key is unchanged.

In all other cases, the new key is rejected.

The nickagent will attempt to refresh a key by making request to a nickserver of its choice when a key is past 3/4 of its lifespan and again when it is about to expire.

Nicknym encourages, but does not require, the use of short lived public keys, in the range of X to Y days. It is recommended that short lived keys are not uploaded to OpenPGP keyservers.

Automatic invalidation
----------------------------

A key is invalidated if:

* The old key has expired, and no new key can be discovered with equal or greater validation level.

This means validation is a one way street: once a certain level of validation is established for a user address, no client should accept any future keys for that address with a lower level of validation.

Discovering nickservers
--------------------------------

It is entirely up to the nickagent to decide what nickservers to query. If it wanted to, a nickagent send all its requests to a single nickserver.

However, nickagents should discover new nickservers and balance their queries to these nickservers for the purposes of availability, load balancing, network perspective, and hiding the user's association map.

Whenever the nickagent is asked by a locally running application for a public key corresponding to an address on the domain `domain.org`, it may check to see if the host `nicknym.domain.org` exists. If the domain resolves, then the nickagent may add it to the pool of known nickservers.

Additionally, a nickagent may be distributed with an initial list of "seed" nickservers. In this case, the nickagent is distributed with a copy of the CA certificate used to validate the TLS connection with each respective seed nickserver.

Cross-provider signatures
----------------------------------

To be written.

Auditing
----------------------------

In order to keep the user's provider from handing out bogus public keys, a nickagent should occasionally make foreign queries of the user's own address against nickservers run by third parties.

In order to prevent a nickserver from handing out bogus provider keys, a nickagent should query multiple nickservers before a provider key is registered or path trusted.

Possible attacks:

**Attack 1 - Intercept Outgoing:**

* Attack: provider `A` signs an impostor key for provider `B` and distributes it to users of `A` (in order to intercept outgoing messages sent to `B`).
* Countermeasure: By querying multiple nickservers for the provider key of `B`, the nickagent can detect if provider `A` is attempting to distribute impostor keys.

**Attack 2 - Intercept Incoming:**

* Attack: provider `A` signs an impostor key for one of its own users, and distributes to users of provider `B` (in order to intercept incoming messages).
* Countermeasure: By querying for its own keys, a nickagent can detect if a provider is given out bogus keys for their addresses.

**Attack 3 - Association Mapping:**

* Attack: A provider tracks all the requests for key discovery in order to build a map of association.
* Countermeasure: By performing foreign key queries via third party nickservers, an agent can prevent any particular entity from tracking their queries.


Future enhancements
---------------------

Should we support additional discovery mechanisms:

* Webfinger includes a standard mechanism for distributing a user's public key via a simple HTTP request. This is very easy to implement on the server, and very easy to consume on the client side.
* There are multiple competing standards for key discovery via DNS. When and if one of these emerges predominate, Nicknym should attempt to use this method when available. DNS discovery, however, has some problems. DNS discovery of keys is much harder to implement, because the service provider must run their own customized authoritative nameserver. Also, since (RSA) keys can be too big for domain UDP packets, any future-proof DNS method relies on an HTTP request, thus undermining the potential benefit of decentralization you might get from using DNS rather than webfinger.



Reference nickagent implementation
====================================================

There is a reference nickagent implementation called "key manager" written in Python and integrated into the LEAP client. It uses Soledad to store its data.

Public API
----------------------------

**refresh_keys()**

updates the keys with fresh ones, as needed.

**get_key(address, type)**

returns a single public key for address. type is one of 'openpgp', 'otr', 'x509', or 'rsa'.

**send_key(address, public_key, type)**

authenticates with the appropriate provider and saves the public_key in the user database.

Storage
--------------------------

Key manager uses Soledad for storage. GPGME, however, requires keys to be stored in keyrings, which are read from disk.

For now, Key Manager deals with this by storing each key in its own keyring. In other words, every key is in a keyring with exactly 1 key, and this keyring is stored in a Soledad document. To keep from confusing this keyring from a normal keyring, I will call it a 'unitary keyring'.

Suppose Alice needs to communicate with Bob:

1. Alice's Key Manager copies to disk her private key and bob's public key. The key manager gets these from Soledad, in the form of unitary Keyrings.
2. Client code uses GPGME, feeding it these temporary keyring files.
3. The keyrings are destroyed.

TBD: how best to ensure destruction of the keyring files.

An example Soledad document for an address:

    {
      "address":"alice@example.org",
      "keys": [
        {
          "type": "opengpg"
          "key": "binary blob",
          "keyring": "binary blob",
          "expires_on": "2014-01-01",
          "validation": "provider_signed",
          "first_seen_at": "2013-04-01 00:11:00",
          "last_audited_at": "2013-04-02 12:00:00",
        },
        {
          "type": "otr"
          "key": "binary blob",
          "expires_on": "2014-01-01",
          "validation": "registered",
          "first_seen_at": "2013-04-01 00:11:00",
          "last_audited_at": "2013-04-02 12:00:00",
        }
      ]
    }

Pseudocode
---------------------------

get_key

    #
    # return a key for an address
    #
    function get_key(address, type)
      if key for address exists in soledad database?
        return key
      else
        fetch key from nickserver
        save it in soledad
        return key
      end
    end

send_key

    #
    # send the user's provider the user's key. this key will get signed by the provider, and replace any prior keys
    #
    function send_key(type)
      if not authenticated:
        error!
      end
      get (self.address, type)
      send (key_data, type) to the provider
    end

refresh_keys

    #
    # update the user's db of validated keys to see if there are changes.
    #
    function refresh_keys()
      for each key in the soledad database (that should be checked?):
          newkey = fetch_key_from_nickserver()
          if key is about to expire and newkey complies with the renewal paramters:
              replace key with newkey
          else if fingerprint(key) != fingerprint(newkey):
              freak out, something wrong is happening? :)
              may be handle revokation, or try to get some voting for a given key and save that one (retrieve it through tor/vpn/etc and see what's the most found key or something like that.
          else:
              everything's cool for this key, continue
          end
      end
    end

private fetch_key_from_nickserver

    function fetch_key_from_nickserver(key)
      randomly pick a subset of the available nickservers we know about
      send a tcp request to each in this subset in parallel
      first one that opens a successful socket is used, all the others are terminated immediately
      make http request
      parse json for the keys
      return keys
    end


Reference nickserver implementation
=====================================================

The reference nickserver is written in Ruby 1.9 and licensed GPLv3. It is lightweight and scalable (supporting high concurrency, and reasonable latency), and uses EventMachine for asynchronous network IO. Data stored in CouchDB.

For more information, see https://github.com/leapcode/nickserver
