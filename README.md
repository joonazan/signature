Signature
=========

URL signing package for Go.

  * You can jump straight into the [API documentation](http://godoc.org/github.com/stretchrcom/signature).

## What does it do?

Signature secures web calls by generating a security hash on the client (using a private key shared with the server), to ensure that the request is geniune.  Only a client who knows the private key will be able to generate the same security hash.

Since the private key is only used to generate the security hash and not transmitted with the request (only some kind of public key is), the server and client must agree on the private key in order for the hash to be verified.

## Request signing

Request signing entails generating a hash based on the details of the request, PLUS a private key - and having the remote server try to generate the same hash, assuming you both agree on the private key.

### Encoding Process

Before signing the request, you must:

  * Generate the original request URL 
   * example: http://something.com?key=value&key2=value2
  * Add the public key value to the request URL
   * example: http://something.com?key=value&key2=value2?key=85Bad53987b3d851

SignatureKey parameter generation:

To generate a the signature for the request, the Signature package does the following:

  * Create a copy of the request URL
  * Add `PrivateKeyKey` key parameter
  * Add `BodyHashKey` value containing an SHA-1 hash of the body contents if there is a body - otherwise, skip this step (and do not add a `BodyHashKey` parameter at all)
  * Order parameters alphabetically.  Order first by keys, and if there are multiple values for one key (i.e. ?color=red&color=blue) then order the values alphabetically afterwards.
  * Prefix it with the HTTP method (in uppercase) followed by an ampersand (i.e. `GET&http://...`)
  * Hash it (using SHA-1)
  * Add the hash as `SignatureKey` to the _end_ of the original URL

### Decoding

To verify a signed request, the Signature package does the following:

  * Strip off the `SignatureKey` parameter (and keep it)
  * Lookup the account (using the public key) and get the `PrivateKeyKey` parameter, and add it to the URL
  * Hash it
  * Compare the generated hash with the `SignatureKey` value to decide if it the request is valid or not

## Response signing

Response signing refers to generating a hash based on the response, to validate that the remote server indeed was responsible for generating the response.  This prevents clients of the service from being tricked into accepting a response from an unreliable source.

### The `HashWithKeys` method

Signature provides the `HashWithKeys` method that allows you to hash a series of bytes (along with the public and private keys).  When this value is transmitted to the client, they can attempt to generate the same hash.  If the hashes match, then the response was geniune.

## Settings

The `signature` package provides some settings to allow you to use non-default field names in your code.  Remember that the client needs to use the same fields in order for the security hashes to match.

    // PrivateKeyKey is the key (URL field) for the private key.
    signature.PrivateKeyKey string = "~private"
    
    // BodyHashKey is the key (URL field) for the body hash used for signing requests.
    signature.BodyHashKey string = "~bodyhash"
    
    // SignatureKey is the key (URL field) for the signature of requests.
    signature.SignatureKey string = "~sign"

## Validating request signature

To validate your code is generating the correct hash, try these:

The URL

    http://test.stretchr.com/api/v1?~key=ABC123&:name=!Mat&:name=!Laurie&:age=>20
    
with HTTP method

    GET
    
and body

    body
    
and private key

    ABC123-private
    
should be hashed as

    df073ee4086eed5848d167871c7424937027728e

leaving the final URL as

    http://test.stretchr.com/api/v1?~key=ABC123&:name=!Mat&:name=!Laurie&:age=>20&~sign=df073ee4086eed5848d167871c7424937027728e


------

Contributing
============

Please feel free to submit issues, fork the repository and send pull requests!

When submitting an issue, we ask that you please include steps to reproduce the issue so we can see it on our end also!


Licence
=======
Copyright (c) 2012 Mat Ryer and Tyler Bunnell

Please consider promoting this project if you find it useful.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
