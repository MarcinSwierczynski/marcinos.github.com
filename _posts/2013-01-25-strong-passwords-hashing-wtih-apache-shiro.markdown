---
date: 2013-01-25 22:30:12 +0100
layout: post
slug: strong-passwords-hashing-wtih-apache-shiro
status: publish
title: Strong passwords hashing with Apache Shiro
categories:
- Programming
- Java
- Security
---

## What is it all about?
Recently, I wanted to provide a strong password hashing in a plain Java application. I decided to use [Apache Shiro](http://shiro.apache.org).

As Apache Shiro's website says:

> Apache Shiro is a powerful and easy-to-use Java security framework that performs authentication, authorization, cryptography, and session management.

Shiro is very powerful and provides all-in-one security solution. For instance, it can communicate with a persistence layer directly, so you don't need to take care of a boilerplate code. However, my intention was only to provide a strong password hashing, nothing more or less.

So, how to do that?

## Hashing salted passwords
My implementation uses a separate salt for each user. Although, there are [different opinions about that](http://stackoverflow.com/questions/2188507/help-with-salt-and-passwords), I've decided that salt will be stored in its own column in a database. Let's start coding then.

First of all we should generate a salt:

```java
protected ByteSource getSalt() {
	return new SecureRandomNumberGenerator().nextBytes();
}
```

Given salt, we can hash plain text password:

```java
	new Sha512Hash(password, salt, hashIterations).toHex();
```

Of course, we didn't want to store `ByteSource` directly in a database, so I converted it to a hex-encoded string:

```java
	salt.toHex();
```

Now we can store both hashed password and salt in a database. How about comparing a result with a plain text password?

## Comparing clear text and hashed passwords
Obviously, to compare a user-provided password with its hashed version, we need to hash it again using the same salt and compare the result with what is stored in a database.

```java
ByteSource salt = ???
String hashedPassword = new Sha512Hash(clearTextPassword, salt, hashIterations).toHex();
return hashedPassword.equals(dbStoredHashedPassword);
```

But how to get `ByteSource` object from a string we saved? It's easy as well.

```java
	ByteSource salt = ByteSource.Util.bytes(Hex.decode(salt));
```

## Putting it all together
The complete example could look like:

```java
public boolean passwordsMatch(String dbStoredHashedPassword, String salt, String clearTextPassword) {
	ByteSource salt = ByteSource.Util.bytes(Hex.decode(salt));
	String hashedPassword = hashAndSaltPassword(clearTextPassword, salt);
	return hashedPassword.equals(dbStoredHashedPassword);
}

public void hashPassword(String clearTextPassword) {
	ByteSource salt = getSalt();
	String hash = hashAndSaltPassword(clearTextPassword, salt);
	// persist salt and hash or return them to delegate this task to other component
}

private String hashAndSaltPassword(String clearTextPassword, String salt) {
	return new Sha512Hash(clearTextPassword, salt, hashIterations).toHex();
}

private ByteSource getSalt() {
	return new SecureRandomNumberGenerator().nextBytes();
}
```

## What is the hashIterations?
The hash iterations value indicates the number of times the clear password is hashed. The value larger, the hash algorithm applied more times, making the process slower and, as a result, making the password harder to break by a brute-force attack.

What's the perfect value? It depends on you security requirements and your hardware performance. Too many iterations, means slower log-in process from users perspective. Too few, means easier to compromise passwords.

I've decided to 200.000 iterations. More on that issue can be found on [Stormpath's blog](http://www.stormpath.com/blog/strong-password-hashing-apache-shiro).

## That's all, folks
Passwords hashing appeared to be quite easy with Apache Shiro. The code I provided lacks of some null-checks and exceptions handling. I omitted them for the sake of clarity.

And remember, never store plain text passwords in a database :-)
