<frontmatter>
  title: Introduction to Password Storage
  footer: footer.md
  head: head.md
  siteNav: mainNav.md
  pageNav: 3
</frontmatter>

{{ navbar | safe }}

<div class="website-content">

# Introduction to Password Storage

Author: [Jeremy Choo](https://github.com/ChooJeremy) <br>
Reviewers: [Amrut Prabhu](https://github.com/amrut-prabhu), [Marvin Chin](https://github.com/marvinchin), [Tan Zhen Yong](https://github.com/Xenonym)

## Overview

While developing your own application, it is very common to implement login systems to be able to identify users. Whenever there is a need to store user-specific information, developers typically implement user accounts to identify these users separately. You might have seen this mechanism implemented on other websites - using registration forms and requring you to login with a username and password to identify you for the <trigger for="pop:session">session</trigger>.

<popover id="pop:session" title="A _session_  is used to broadly describe a user's visit" placement="top">
  <div slot="content">
Generally, a session starts when a user visits your page, and ends when the user closes the browser window.
  </div>
</popover>

As a developer, storing these usernames and passwords are essential to be able to refer to them later. While there are many ways to store this data - from text files to databases - this article will focus more on how to store passwords properly. For the purpose of this article, we will assume that you are storing the passwords in a database. Modern password storage mechanisms typically involve **salting** and **hashing** the password. This article will describe what these mean and why they are important.

### What's wrong with <trigger for="pop:plaintext">plaintext</trigger>?

<popover id="pop:plaintext" title="_Plaintext_ refers to unencrypted information" placement="top">
  <div slot="content">
It is a cryptography term generally referring to text before encyption or after decrypting it. Another term for it is cleartext.
  </div>
</popover>

Since usernames are typically public information, it is OK to store them in plaintext. However, passwords are typically secret information and hence, cannot be stored in plaintext.
	
</div>
</popover>

If passwords are stored in plaintext:
* Should your database be compromised, attackers can easily use the stored plaintext passwords to login as the user and potentially access other functionality. This grants the attacker more power than just reading values off the database. If the passwords aren't even in plaintext, the attacker might not be able to read the passwords.
* Your employees can see the plaintext passwords of your users. If your employees are ever compromised, they might be willing to simply leak the passwords.
* It is common for users to re-use the same password on multiple sites. If your passwords are stored as plaintext and accessed, attackers can easily re-use the same passwords on other websites with sensitive data (such as banking websites) and gain access to those.

To prevent these situations from occuring, it is good to perform hashing on passwords. Unlike <trigger for="pop:encrypt">encryption</trigger>, hashing is a <trigger for="pop:oneway">one-way function</trigger>. This means that an attacker cannot decrypt the encrypted password to retrieve the original password, thus ensuring that none of the aforementioned vulnerabilities are possible.

<popover id="pop:encrypt" title="Encryption refers to the process of changing a message such that it becomes essentially random text." placement="top">
  <div slot="content">
 The opposite process is called decryption, which converts the text back into the original message.
  </div>
</popover>

<popover id="pop:oneway" title="" placement="top">
<div slot="content">
	A one way function is a function that is easy to compute in one way, but is close to impossible to compute the other way.
</div>
</popover>

## What is hashing?

Hashing is the act of transforming a set of data into another set of data that is impossible to reverse. As an example, suppose you wanted to hash [The Complete Works of Shakespeare](https://www.gutenberg.org/files/100/100-0.txt). If you performed a SHA1 hash on the entire contents of that file, including the licensing information at the very end, you'll end up hashing the entire 157257 lines of that file into `C9B38187EA0CC82B72142B92034DA859A3550B57`. Obviously, it is impossible to condense all 157257 lines of text into just those 40 characters. 

This means that is is impossible to recreate the complete works of Shakespeare from just the hash result. Thus, we say that a hashing function is a one-way function, as it can only be performed in a one direction, and it would be impossible to perform it in the other direction.

Some examples of cryptographic hashing algorithms are [MD5](https://www.quora.com/How-does-the-MD5-algorithm-work) and [SHA1](https://deadhacker.com/2006/02/21/sha-1-illustrated/). However, when you perform hashing for passwords, you should use password hashing algorithms (such as [Argon2](https://github.com/P-H-C/phc-winner-argon2), [SCrypt](https://passlib.readthedocs.io/en/stable/lib/passlib.hash.scrypt.html) and [bcrypt](https://security.stackexchange.com/questions/4781/do-any-security-experts-recommend-bcrypt-for-password-storage)). These password hashing algorithms are designed to be slow to prevent brute force attacks, unlike cryptographic hashing algorithms which are built for speed. Despite that, if you plan on learning more about hashing algorithms, I recommend starting with learning about MD5 and SHA1, as they are easier algorithms to start learning with.

### Why isn't hashing enough?

Unfortunately, [rainbow tables](https://en.wikipedia.org/wiki/Rainbow_table) exist.

A rainbow table is a precomputed table of hashes for some set of passwords. Basically, since the hashing function is easy to compute one-way, people build huge tables of hashes wherein the plaintext is already known, so that attempting to crack hashes becomes a simple problem of looking up the hash in the table and it's corresponding value, instead of attempting to reverse the hash. Because of this, it is very easy to crack simple hashes by simply doing a lookup.

An example of a service that provides this is [Crackstation](https://crackstation.net/).

Since attacks like rainbow tables exist, passwords need another layer of security.

## Adding salt

A salt refers to appending a string of text, unique to each user, to their password before hashing them. Since each user has a unique salt, this makes rainbow tables ineffective, as the majority of the precomputed hashes won't even contain the salt, so they wouldn't even matter anyway! In this way, the salt forces the attacker to recompute the rainbow table for each password in order to be able to effectively use it. This effectively converts the attack to <trigger for="pop:brute">brute force</trigger>, as each hash must be recomputed for each possible password. Additionally, the computed rainbow table would only be useful for that specific user, as each user would have a different salt. It could take years before a password is cracked!

<popover id="pop:brute" title="A brute force attack is an attack where all possible combinations are tested to see if they work." placement="top">
  <div slot="content">
 A brute force attack usually takes very long to carry out.
  </div>
</popover>

Note that the salt should be randomly generated, as opposed to choosing a static value that is different for every user. For example, if an application used the username of the user as the salt, then attackers can pre-generate rainbow tables for common usernames, causing users with common users to be vulnerable to a rainbow table attack, even with salt applied to their passwords.

Thus, the way to store passwords properly is to use a salted hash - take the user's password, append some bytes to it, and hash the result. That hash is the user's password. When the user attempts to login again, perform the same procedure again. If the hash matches, then you know that the user is who he says he is, even if you don't actually know the original password that the user provided. 

One question that is commonly asked by developers is where to store the salt. The salt can be stored in plaintext, along with the user in the database. Since the goal of the salt is only to prevent precomputed rainbow tables from being used, it doesn't need to be encrypted in the database.

Many good password hashing algorithms today have built-in salts, such as [Argon2](https://github.com/P-H-C/phc-winner-argon2), [SCrypt](https://passlib.readthedocs.io/en/stable/lib/passlib.hash.scrypt.html) and [bcrypt](https://security.stackexchange.com/questions/4781/do-any-security-experts-recommend-bcrypt-for-password-storage). A good password hashing library and algorithm will salt automatically.

### If an attacker has gained access to the entire application, what does it matter if my passwords are stored in plaintext or not?

If an attacker has already gained access to the entire application, then he already has all the information that he could possibly want from your server. He would have access to all of your application's data, including data from your users or from any analytics software that you might be running. However, by adding salt and hashing your passwords, the attacker still doesn't know your customer's passwords and could take years to find out. Otherwise, since 59% of people use the same password across multiple sites <sup>[source](https://securityboulevard.com/2018/05/59-of-people-use-the-same-password-everywhere-poll-finds/)</sup>, the attacker could quickly try other websites such as banks to attempt to break into those accounts, which can yield great financial return.

Additionally, when a user signed up on your website and provided you a password, they implicitly trusted you to keep that information safe and secure for them. In a sense, you have a responsibility to keep their passwords secure as well. By doing proper password storage, if your servers ever get breached, you can assure your customers that their passwords are properly secured and maintain some of their trust in you.

## Getting started

<p style="color: #721c24; background-color: #f8d7da; border-color: #f5c6cb; padding: .75rem 1.25rem; border: 1px solid transparent; border-radius: .25rem;">
Reading this article doesn't make you an expert in writing your own password hashing code. It is far too easy to screw up and make a mistake. Instead, use one of the free libraries that provide a crypto function that has already been well-tested by the community. <b>Do not write your own crypto library.</b>
</p>

Here are some libraries you can use to implement password storage, depending on your language:
* If you're using PHP, the [password_hash](https://secure.php.net/manual/en/function.password-hash.php) function will do the job for you. 
* If you're using java, use [SecretKeyFactory](https://www.owasp.org/index.php/Hashing_Java) to apply hashes with SHA256.
* If you're using Python, use [PassLib](https://passlib.readthedocs.io/en/stable/narr/hash-tutorial.html).
* If you're using npm, use [bcrypt](https://www.npmjs.com/package/bcrypt).

## Other resources

* [Secure Salted Password Hashing - How to do it properly](https://crackstation.net/hashing-security.htm) is an excellent resource that explains how to perform salted password hashing correctly, including links to other good libraries and what else can be done.
* [Awesome Cryptography](https://github.com/sobolevn/awesome-cryptography) is a curated list of resources - articles, blogs, books, libraries and more.

</div>
