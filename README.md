# twitter-signer

This is a simple tool for tweeting via the command line and publishing the corresponding PGP signature to [jsonbin](https://jsonbin.io/).

## Prerequisites

You need to install ruby gems and twitter twurl. On Mac OS X this can be performed using brew.

```bash
brew update
brew install ruby
gem install twurl
```

Install GnuPG

```bash
brew install gnupg
```

(it is assumed that the user is familiar with PGP and GnuPG)

On Linux this can be performed as follows

```bash
apt-get update
apt-get install ruby-dev
gem install twurl
```

Install GnuPG

```bash
apt-get install gnupg
```

which was tested on Ubuntu version >= 18.04.

You will need a twitter developer account. The process is described [in this article](https://developer.twitter.com/en/docs/twitter-api/getting-started/getting-access-to-the-twitter-api). Create a new project in your twitter developer account.

Save the generated API key and API key secret for your project.

Next authorize twurl using the following command

```bash
twurl authorize --consumer-key key --consumer-secret secret
```

The response is going to be a link. Enter this link. Your account will be asked to authorize access to your project. Permit this access.

Your app is now authorized. Test via the following command

```bash
twurl accounts
```

This returns all API keys currently associated with your account. It might be necessary to edit the default account in order to generate the correct bearer token. The configuration file is usually located in "~/.twurlrc". The file is formatted in yaml. Make sure that the default profile uses the API key that matches your newly created application.

Generate a bearer token.

```bash
twurl authorize --bearer --consumer-key key --consumer-secret secret
```

We can now tweet using the command line. Here is an example

```bash
twurl -d 'status=Test tweet using the POST statuses/update endpoint' /1.1/statuses/update.json
```

The general process is described in [this article](https://developer.twitter.com/en/docs/tutorials/using-twurl).

Next, create an account on [jsonbin](https://jsonbin.io/) and save your API key. Set the api key in the tweet-sign script in the constant "jsonbinApiKey".

Create a collection called "twitter-signatures" and save the corresponding id (todo: we can easily automate this process as well).
Set the corresponding id in the tweet-sign script in the constant "collectionId".

**Example usage:**

```bash
tweet-sign "This is a tweet whose signature is stored in jsonbin"
```

The generated signature is base64 encoded.

The following json is published on jsonbin.

```json
{
  "tweet": {
    "uuid": "F0A2293D-3EDB-46AD-9DE1-C1568E97850B",
    "timestamp": "2021-07-23T00:13:33+0200",
    "content": "This is a tweet whose signature is stored in jsonbin",
    "signature": "LS0tLS1CRUdJTiBQR1AgU0lHTkFUVVJFLS0tLS0KCmlRSXpCQUFCQ0FBZEZpRUVwNEI3UitVY1pxQm5UMmk4WHRhdnR2RmtRTlVGQW1ENTdZd0FDZ2tRWHRhdnR2RmsKUU5WZWNBLy9UR2dlWXBzS0k3V3NKNHlhSjZjajBib1VmaVFDSFRhMFpGUWNuN1Z4TXJ2eU0rNmFzNVNwZCsxagpuUzlJZjBLek9ST0Z0OEFXYXQrUTZMNnROQk9DR0tsdGd3NU1xNjkrb3plVnU5akdSeGNFSFhTVmJ3WGx0enpaCnE4NkhrdDU3eDhIYnczNXhwRTBoU0xPbUlSd2VUOE13aCtNampJNUkxQkI0Wk9rcTBvd3VtbHMzREJhb2crZWMKUmRSN2tWZzdoUUZNWGs4SmcydXYwK2Y4SitZL0hzTXE1WW95bWh3S0RZUGRKMk1GbjlsY1J4S0lZaXhQTGlXMgpMOFVUUldzdi9iZFY2NDViN0ZpZEtpRVB6SjYxSkQyb1BWSUNJb1lhTmpZcEdUeGtmNGk0a2Y2RGdGSGRkU2R2CmhCTXZwQ0tiVDFpT0RiZG9JRENvYjloZlgrakN4NFdnZjJWMVNvbGdVRFpBQURJZFNMb2VTNnZieTFPc0FKTjIKMlVJTEdRR0t1eUhHU2c2VHVzTzFjTUNseXA3MGMvWU40NDZ5ZHN4RDVOeUE0K2c5Z0UxcXk4U1c4TUFGMDBLOQpoQmZ1SGduUm9KVWlkR3BSeFQ3clJ6cG9ETWdWMTUxZG13SzJlZTdKelJ2ZHZZcVowWnNscHBVTUhFWXFKMEh4ClFldW15VkIzeVY4M1U1OHkyOGZQYnB5aS9NL3ZjUk5FRzB3QzBIREpib1pMWE5sZ1VHYkYrTTBCRzFwNnZ4d0MKSFc5ZWlwU1l0RUlxVThPSEc0R0hIMGZtZnRNRG02aTRRWk1QcE1nSElEbk1MRngyMGVYN1NRdlJkeThVKzNvWQpidkVnVzE3U2lCUG9kNGs1VldxUzc4QWVDZXdKZlh6ODdPVlo5bFR4REh4My9LR24ydUU9Cj1XK1EvCi0tLS0tRU5EIFBHUCBTSUdOQVRVUkUtLS0tLQo="
  }
}
```

In order for the rest of the world to verify the authenticity of your tweets, you must provide them with your PGP public key, which is usually uploaded to a key server such as [keys.openpgp.org](https://keys.openpgp.org/), and your personal public jsonbin. But this is very straight forward.

We can verify as follows.

1. Decode the signature

```bash
echo "LS0tLS1CRUdJTiBQR1AgU0lHTkFUVVJFLS0tLS0KCmlRSXpCQUFCQ0FBZEZpRUVwNEI3UitVY1pxQm5UMmk4WHRhdnR2RmtRTlVGQW1ENTcvY0FDZ2tRWHRhdnR2RmsKUU5VeXJBLytQSzE0Zmdmc0JiYTRjYW9rR1c5Vzh3V2ZGZzlvYTNkWGhRWXdZWE9ZK2xCYzZObXNUYm9iSnlIMApWV0hnVW8vdG1OK09yMTlXYXhyWk5wREx2SmhvWk5DVHBWQ0RsSHN6Y0s2MlRmTVQyMU9kYWpnMmtXemFaY0c1ClJhazl4dlpUT3RpL2RGUjFFUUxTL1lHZ1JENSs0SXFYVDVMbE5HZ05PM3VTSCt2am1rUnhwU2RpdFl1Ujc1ZzcKQmdCTTBWdXNaTzFVd2JnajlRVkdmUGQzZjZtRTMrYU9ncm4wS0VtQnYweWtjQis3b0VPMEpJelJDOFhoSUlmdAppY0VTc1pyY2NQd0NjWUVQNTNndkFGTlY5Skg4NnNYaXNCeFZNSnlFbXIxYTdrRnJwNWRwV0s2VTc1ODcvZktVCnp0MXRzbFJoLzlhcW9mcDg1NTBSTFZVNmhPR3Vsb2EzTUZaa3FHZkZTWHhpdE9GMGlnKzdEeVd3TE5uTHBiNWMKdmIrQ1UzZjIvYWVReUdESndEWlY3QmtnbGdNZGwvS0FQNlI0TmRKemlLcWVIVE5ML2lZR3pjWGU1U0lkM1BBRgpKdVlJSTZCajkreXpPMDIxVXROMzFXYzJCK1pSaitkWnp4QlJvMW5kdTJtenZ1Y3NyWXdUT2ZDYUlwU1pKSmlpClB4UDlia2lhTWNwSnI1QWVXMHZkcmxqMUdqbzJoSVpUWXBFWnR6SXpkbXQrU2s5bmtYYXpvYUUwN0ZyWHJiUlUKQjZzNFVkRWdOSUo4ZXNDTjdCYVdvZGJ4OEtmY0owbmVsWi9iSTlDTWtITDdKVTNTNVhvRmZudjhwL0poNHFCZAp1K2RxQlE4WXFLZnJvVUtXeENMSVRCblpqK2RONXhBQmVDdHNjTzJ3SHV4bFh3dEk2S1k9Cj0wYmdHCi0tLS0tRU5EIFBHUCBTSUdOQVRVUkUtLS0tLQo=" | base64 --decode > signature.sig
```

The decoded signature looks like this.

```bash
-----BEGIN PGP SIGNATURE-----

iQIzBAABCAAdFiEEp4B7R+UcZqBnT2i8XtavtvFkQNUFAmD57YwACgkQXtavtvFk
QNVecA//TGgeYpsKI7WsJ4yaJ6cj0boUfiQCHTa0ZFQcn7VxMrvyM+6as5Spd+1j
nS9If0KzOROFt8AWat+Q6L6tNBOCGKltgw5Mq69+ozeVu9jGRxcEHXSVbwXltzzZ
q86Hkt57x8Hbw35xpE0hSLOmIRweT8Mwh+MjjI5I1BB4ZOkq0owumls3DBaog+ec
RdR7kVg7hQFMXk8Jg2uv0+f8J+Y/HsMq5YoymhwKDYPdJ2MFn9lcRxKIYixPLiW2
L8UTRWsv/bdV645b7FidKiEPzJ61JD2oPVICIoYaNjYpGTxkf4i4kf6DgFHddSdv
hBMvpCKbT1iODbdoIDCob9hfX+jCx4Wgf2V1SolgUDZAADIdSLoeS6vby1OsAJN2
2UILGQGKuyHGSg6TusO1cMClyp70c/YN446ydsxD5NyA4+g9gE1qy8SW8MAF00K9
hBfuHgnRoJUidGpRxT7rRzpoDMgV151dmwK2ee7JzRvdvYqZ0ZslppUMHEYqJ0Hx
QeumyVB3yV83U58y28fPbpyi/M/vcRNEG0wC0HDJboZLXNlgUGbF+M0BG1p6vxwC
HW9eipSYtEIqU8OHG4GHH0fmftMDm6i4QZMPpMgHIDnMLFx20eX7SQvRdy8U+3oY
bvEgW17SiBPod4k5VWqS78AeCewJfXz87OVZ9lTxDHx3/KGn2uE=
=W+Q/
-----END PGP SIGNATURE-----
```

2. Extract the payload, in this case "This is a tweet whose signature is stored in jsonbin"
3. Verify using

```bash
gpg --verify signature.sig content
```

## Implementation details

We are using gpg --detach-sig to create a detached signature for our tweet that we wish to sign.
This works as follows

1. The message is hashed using sha-256 (--digest-algo SHA256)
2. The hash is encrypted using our private key
3. The resulting signature is detached and base64 encoded (--detach-sig -armor)

In order to verify all commands, consider the following

```bash
echo "hello world" > plain
echo "hello world" | gpg --digest-algo SHA256 --detach-sig -armor > signature.sig
gpg --verify signature.sig plain
```

We can also check if the correct hashing algorithm is used. The id for the most common hashing algorithms is defined in [RFC4880](https://datatracker.ietf.org/doc/html/rfc4880). SHA-256 is 8.

```bash
cat signature.sig | gpg --list-packets | grep "digest algo"
```

which yields to "digest algo 8".

This script is using the new v3 jsonbin api and twitter [twurl](https://github.com/twitter/twurl).
We are creating the detached signature as described above and save the signature along with its corresponding plain text on jsonbin.

The json structure is defined as follows

```json
{
    tweet: {
        uuid: "30F20BE5-33E3-4F53-B5C9-1957C5DD9069",
        timestamp: "2021-07-22T20:02:26+0200",
        tweet: "hello world"
        signature: "base64 encoded signature"
    }
}
```

Timestamps are formatted as a fully ISO-8601 compliant date. The uuid is generated using uuidgen.
After the json is generated, the tweet is send using twitter twurl. Then we push the created json to jsonbin.

> NOTICE: We are not synchronizing these two operations! If the second operation fails, then there is no published signature of the tweet. If the tweet fails, there might be a signature published without its corresponding tweet. We throw an error in both cases but do not perform any rollback operatons in order to reduce complexity. Any rollback operations can be easily performed by hand using a custom script with twurl, for deleting any tweet, and by using the jsonbin v3 api to delete an invalid datapoint.

$1 is the tweet we wish to send.

Example usage:

```bash
tweet-sign "Hello World"
```

The signature is base64 encoded. In order to decode it, use [base64](http://manpages.ubuntu.com/manpages/bionic/man1/base64.1.html) and then verify the decoded PGP signature using gpg --verify signature.sig plain.

### Future plans

It is planned in the future that one can automatically verify tweets of a given user provided one has the corresponding PGP public key stored locally.

However, it is straight forward to do this by hand.

Also, the setup process could be easily automated.

## License

The tool is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
