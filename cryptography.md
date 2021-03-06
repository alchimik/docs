# Cryptography

## Table of Contents
1. [Introduction](#introduction)
2. [Hashing](#hashing)
  1. [bcrypt](#bcrypt)
3. [Encryption](#encryption)
  1. [Encryption Keys](#encryption-keys)
  2. [Encrypting Data](#encrypting-data)
  3. [Decrypting Data](#decrypting-data)

<h2 id="introduction">Introduction</h2>
Keeping user data secure is of the utmost importance.  Unfortunately, PHP's built-in cryptographic support is somewhat fragmented and not easy to use.  Lucky for you, Opulence has a `Cryptography` library to simplify all this.

<h2 id="hashing">Hashing</h2>
Hashers take input and perform a one-way mapping to a hashed value.  It is impossible to decrypt a hashed value because of the way this mapping is generated.  This hashes suitable for storing sensitive data, like user passwords.

<h4 id="bcrypt">bcrypt</h4>
`bcrypt` is a popular password hashing function.  It accepts a "cost" parameter, which adjusts the CPU cost to hash a password.  This slows down attacks against compromised data.  Increasing the cost parameter by one causes the hashing to take twice as long, which future-proofs it as CPUs get faster.  Let's take a look at how to use it:

```php
use Opulence\Cryptography\Hashing\BcryptHasher;

$bcryptHasher = new BcryptHasher();

// Let's create a hash with a pepper of "bar"
// $hash is automatically salted and suitable for database storage
$hashedValue = $bcryptHasher->hash("foo", ["cost" => 10], "bar");
```

To verify that an unhashed value hashes to a particular value, use `verify()`:

```php
$unhashedValue = "foo";
echo $bcryptHasher->verify($hashedValue, $unhashedValue, "bar"); // 1
```

<h2 id="encryption">Encryption</h2>
Sometimes, your application needs to encrypt data, send it to another component, and then decrypt it.  This is different from hashing in that encrypted values can be decrypted.  To make this process as secure and simple as possible, Opulence has an easy-to-use wrapper around `OpenSSL` in its `Encrypter` class:

```php
use Opulence\Cryptography\Encryption\Ciphers;
use Opulence\Cryptography\Encryption\Encrypter;
use Opulence\Cryptography\Encryption\EncryptionException;
use Opulence\Cryptography\Encryption\Keys\Password;

$encrypter = new Encrypter(new Password("mySecretApplicationPassword"));
```

You can change the underlying `OpenSSL` cipher by passing it in via the second parameter:

```php
$encrypter = new Encrypter(new Password("mySecretApplicationPassword"), Ciphers::AES_128_CBC);
```

> **Note:** The default cipher is AES-256-CTR.  It is strongly recommended you use an AES CTR or CBC cipher such as AES-256-CTR or AES-256-CBC. 

<h4 id="encryption-keys">Encryption Keys</h4>
`Encrypter` takes in a secret.  This secret can either be:

* a key
  * Must be at least 32 bytes long and generated by a <a href="https://en.wikipedia.org/wiki/Cryptographically_secure_pseudorandom_number_generator" target="_blank">CSPRNG</a> such as `random_bytes()`
* or a password
  * Any lower-entropy value that is not a suitable cryptographic key
  
Opulence uses a key derivation function (PBKDF2 by default) to turn your key or password into cryptographically-strong encryption and authentication keys.

> **Note:** The encryption key is used to encrypt/decrypt data.  The authentication key is used to create and verify the HMAC contained in the encrypted value.

To use a cryptographic key in your encrypter, simply pass in a `Key` object:

```php
use Opulence\Cryptography\Encryption\Keys\Key;

$encrypter = new Encrypter(new Key(random_bytes(32)));
```

<h4 id="encrypting-data">Encrypting Data</h4>
```php
try {
    $encryptedData = $encrypter->encrypt("foobar");
} catch (EncryptionException $ex) {
    // Handle the exception
}
```

If there was any issue encrypting the data, an `Opulence\Cryptography\Encryption\EncryptionException` will be thrown.

<h4 id="decrypting-data">Decrypting Data</h4>
```php
try {
    $encryptedData = $encrypter->encrypt("foobar");
    $decryptedData = $encrypter->decrypt($encryptedData);
} catch (EncryptionException $ex) {
    // Handle the exception
}

// Verify the decrypted data matches our original value
echo $decryptedData === "foobar"; // 1 
```

If there was any issue decrypting the data, an `Opulence\Cryptography\Encryption\EncryptionException` will be thrown.