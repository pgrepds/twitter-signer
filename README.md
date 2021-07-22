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
Set the corresponding id in the tweet-sign script in the constant "binCollectionId".

**Example usage:** tweet-sign "hello world"

The generated signature is base64 encoded.

In order for the rest of the world to verify the authenticity of your tweets, you must provide them with your PGP public key, which is usually uploaded to a key server such as [keys.openpgp.org](https://keys.openpgp.org/), and your personal public jsonbin. But this is very straight forward.

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
