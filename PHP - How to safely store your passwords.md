# PHP - How to safely store your passwords

_Basic background, understanding, misconceptions & solutions to storing sensitive data in PHP_

**Published:**  2012/05/22 (May 22 2012)

If you have ever asked yourself, "what's the best way to store my user's passwords and sensitive data" Then this article is for you. Would you use sha512, sha256, md5, PBKDF2, or Bcrypt?

Throughout this article I will explain as much as I can about the different types of hashing algorithm and why some are better than others. I will also dive into methods to prevent rainbow table attacks and protecting your users from stolen password or other sensitive data.

## The Basics
Let's start off with a few definitions so we are all on the same page.

**Hashing Algorithm**
> The method to hash your data, example; sha512, sha256, md5, PBKDF2, Bcrypt.

**One-way Hashing Algorithm**
> Once the text has been hashed it can not be un-hashed/ Decrypted (Well sorta, Rainbow Attack).

**Clear Text**
> Text that is not hashed and can be read.

**Salt**
> A random string produced to make stronger hashes.

**Pepper**
> A string similar to a Salt that is added to your hash to increase the security, however this one is not random and hard coded into your code. (Not required but adds another layer of security)


**Deploy Massive Funds and Hardware**
> A term used when a hacker has the funds and hardware to attack your database either by rainbow tables/brute force (We will explain more later with the Amazon EC2 Service).

**Rainbow Table**
> A huge database of commonly used passwords, sometimes they come pre-hashed for md5, sha1, etc... if of course you are not using a Salt, they can however try to guess your Salt.

**Brute Force Attack**
> When someone uses a rainbow table to consistently enter passwords in the attempt of getting one right.

**Re-Hashing/Rounds/Iterations**
> Terms used to specify how many times the data has been reentered into the hashing algorithm.

**Double Hashing**
> The act of chaining together different hashing algorithms to form the so called "Super Hash" (Read the misconceptions)

**Hash Collision**
> When two different data inputs generate the same hash.

**CUDA**
> "CUDAT is a parallel computing platform and programming model invented by NVIDIA. It enables dramatic increases in computing performance by harnessing the power of the graphics' processing unit (GPU)" (Nvidia Site). In layman's terms; Turn your GPU into a processor for arbitrary processing.

## Common Misconceptions

Got the definitions down? Great! Now lets move to some misconceptions.

**You should Double Hash**
> This is one of the worst things you can do, it can actually create redundant hashes. You can also lower the security by double hashing, by creating redundant data, like so: sha1(md5($password)) Pointless...

**Salts are just a second layer of security**
> False; Salts should always be used no matter what.

**I only need Hashing Algorithm to protect data**
> False; Thought Hashing Algorithms greatly increase security, it is also the implementation that counts.

**Users are helpless, and should not be trusted**
> Thought I would never trust a user to come-up with a safe password, which is why we set minimum password lengths with characters consisting of A-Za-z0-9, users can be one of the best assets to keeping data safe. Teach the user about security and warn them about the dangers of insecure passwords. Don't scare them, but make them feel needed in keeping data safe.

Now that misconceptions are behind us, it only gets better! Lets dive deeper into why double hashing is bad.

## Why is double hashing so bad?

Ill start of with saying that double hashing will not make your passwords completely open to attacks (like walking in thought a door), hackers will still need break the hashes, however you run into the chance that you will get a hash collision and generate same hash two different times, which is very, very, very bad.

Basically... when you hash one time you have a 1 in n chance of clashing with another hash. Hash it a second time and now you have a 1 in (n/2), a third time 1 in (n/3), and the forth time, 1 in (n/4), and so on.

Eventually you will get the same matching hashes, however it will take a while. Even thought the chances are low it's still bad practice and should not happen.

If this is still confusing then read this comment from php.net posted by "nathan"

> "The suggestion below to double-hash your password is not a good idea. You are much much better off adding a variable salt to passwords before hashing (such as the username or other field that is dissimilar for every account). Double hashing is *worse* security than a regular hash. What you're actually doing is taking some input $passwd, converting it to a string of exactly 32 characters containing only the characters [0-9][A-F], and then hashing *that*. You have just *greatly* increased the odds of a hash collision (ie. the odds that I can guess a phrase that will hash to the same value as your password). sha1(md5($pass)) makes even less sense, since you're feeding in 128-bits of information to generate a 256-bit hash, so 50% of the resulting data is redundant. You have not increased security at all."

Hopefully that is a good explanation as to why double-hashing is bad, now lets take a look at some of the most popular hashing algorithms and a brief description of each one.

## Different Hashing Algorithms

**MD5 (Out dated)**
> NO, NO, NO, NO, NO, MD5 is the worst thing for hashing sensitive data. This is almost as bad as using clear text, even with a Salt it's still horrible.I recommend you to stop using this. Actually MD5 is really a check-sum, I think of it as a alternative to the mt_rand() function.

**SHA-1 (Out dated)**
> 160-bit hash function which resembles the earlier MD5 algorithm, don't use it.
	
**SHA-2**
> SHA-2 has two child forms; They differ in the word size:
	
**SHA256**
> 32-byte (256 bits) per word; Good but not the best
	
**SHA512**
> 64-byte (512 bits) per word; This is the current recommend SHA-2 hashing type
	
**SHA-3**
> Still in development, so you can't use it yet ...
	
**PBKDF2 (Password-Based Key Derivation Function v2)**
> Works off MD5, however this takes the current text and re-hashes thousands of times using md5. So it's actually really good, but does something better exist?
	
**Bcrypt (Support using Crypt using the code $2y$)**
> Currently considered the best hashing practice (2012); It creates it's own random Salts while hashing your text, while requiring an addition input Salt. You can also scale its hashing depending on the server's hardware and the best part of all, it's slow! Wait, what!?! It's Slow? Why use that? Let's find out.

## What's so special about Bcrypt & why is slow better?

The only downside to Bcrypt is that you can only use it on strings that are less than 55 characters long. However very few people create 55 character long passwords and this problem is being addressed in the next release of Bcrypt.

So far all I have told you is some definitions, misconceptions and that Bcrypt is awesome, not really helpful right? Well lets fix that, let my explain why Bcrypt is better. Bycrpt has three major advantages.

1. It has a better hashing algorithm.
2. It can scale with hardware
3. It's slow!

Ok, so reason one is easy to understand, a more advance way to crypt passwords. What about number two and three?

Well number two means that you can bump-up the hashing rounds as your servers hardware gets better. More iterations = stronger password, I currently recommend a hashing round of 12.

Number three may surprise you. Why is slower better??? Well if a hacker is trying to attack your database without a rainbow table they can blow threw the hashes in minutes. With a rainbow table we are talking seconds! However if we slow the encryption rate we also slow his attack rate. Mind you this can be done on any pc, even a slower one.

But wait! What if we had lots of money? Their are services that you can let you temporarily buy servers which are much stronger than your pc and can do the work in half the time. Like Amazons EC2 service. They can do the work a regular home pc can do incredibly fast.

Want an example? How about some Math! (YAH MATH!)

By the way I found this example off the StackExchange/Security site.

Say your users passwords consisted of only theses characters:
ABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz 1234567890

That's 62 characters, and each password was only 8 characters long, the average password.

62^8 gives use the total number of possibilities; 218,340,105,584,896!

However if you took a stock computer with the average stuff on it, example a Quad Core 2.5Ghz, 4 GB Ram, Average GPU (Using CUDA, you can run arbitrary code using this method) one could create half a billion possible matches, again only on a stock machine!

But how!?!?! That's a lot of guessed passwords, well... Thank the developers of simple algorithms like Sha1, Sha2, etc... They are meant to be fast. See the problem? The faster they can try to crack it, the faster they get your data. Having a slower algorithm hashing starts to take a toll. Then adding salts makes the process even worse for hackers, having to try and crack what your salt is. Now this is getting interesting!

Ok, time for a little recap. We want a hashing algorithm that is complex and slow, like Bcrypt or PBKDF2. But does that mean everything else is bad? Well sorta, you can still use it but under special circumstances. For example, what about PBKDF2? Why does it use md5, and why is it considered better than SHA-2? Well the re-hashing. This makes PBKDF2 strong but not as good as Bcrypt.

Quick note: Bcrypt was made by 2 guys while PBKDF2 was made by 1 guy, so Bcrypt has double the ingenuity put into it!

Ok, so Sha-2 sucks now? Well... No. In-fact Sha512 is amazing if you re-hash it. You may be asking "How many times do I need to re-hash"? Well about 5,000. Yes 5,000 times. Only about a year ago you could be 1,000 times re-hashed and you would be fine. How things have changed over the years.

So final recap before we start coding some solutions, old hashing algorithms are not horrible, but it's recommended you re-hash them, but never use different hashing algorithms together! If you have Bcrypt support, use it (Supported in PHP 5.3.0 >). I mean why not use the latest and greatest right! e tired of all this right? So much information and yet no code. Lets create something.

Alright, enough explanation and theory, so much information and yet no code. Lets create something.

## Examples of Hashing Passwords

This is an example of the worst password hashing:
```php
<?php
$password = md5('password');
echo $password;
```

How about Sha256?

```php
<?php
/* Gen Salt */
function genSalt() {
    $salt = uniqid(rand(), true) . md5(uniqid(rand(), true));
    $salt = hash('sha256', $salt);
    return $salt;
}

/* Gen Password */
function genHash($salt, $password) {
    /* Hash Password with sha256 */
    $hash   = $salt . $password;
    /* ReHash the password */
    for($i = 0; $i < 100000; $i ++) {
        $hash = hash('sha256', $hash);
    }
    /* Salt + hash = smart */
    $hash   = $salt . $hash;
    return $hash;
}

$password = genHash(genSalt(), 'password');
echo $password;
```

What about PBKDF2?

I found this really well created function from a site called itnewb.com, they have great documentation of it (check the cited for more info)

```php
<?php
/** PBKDF2 Implementation (described in RFC 2898)
 *
 *  @param string p password
 *  @param string s salt
 *  @param int c iteration count (use 1000 or higher)
 *  @param int kl derived key length
 *  @param string a hash algorithm
 *
 *  @return string derived key
*/
function pbkdf2( $p, $s, $c, $kl, $a = 'sha256' ) {
 
    $hl = strlen(hash($a, null, true)); # Hash length
    $kb = ceil($kl / $hl);              # Key blocks to compute
    $dk = '';                           # Derived key
 
    # Create key
    for ( $block = 1; $block <= $kb; $block ++ ) {
 
        # Initial hash for this block
        $ib = $b = hash_hmac($a, $s . pack('N', $block), $p, true);
 
        # Perform block iterations
        for ( $i = 1; $i < $c; $i ++ )
 
            # XOR each iterate
            $ib ^= ($b = hash_hmac($a, $b, $p, true));
 
        $dk .= $ib; # Append iterated block
    }
 
    # Return derived key of correct length
    return substr($dk, 0, $kl);
}
$pass = 'users password';
$salt = 'the salt';
 
$hash = pbkdf2($pass, $salt, 1000, 32);
```

How about Bcrypt, well first we need to check if it's supported. Since it uses BLOWFISH we can check this way:

```php
<?php
if (defined("CRYPT_BLOWFISH") && CRYPT_BLOWFISH) {
    echo "CRYPT_BLOWFISH is enabled!";
} else {
    echo "CRYPT_BLOWFISH is not available";
}
```

Ok, now this is an example of Bcrypt:

```php
<?php
/* Bcrypt Example */
class bcrypt {
    private $rounds;
    public function __construct($rounds = 12) {
        if(CRYPT_BLOWFISH != 1) {
            throw new Exception("Bcrypt is not supported on this server, please see the following to learn more: http://php.net/crypt");
        }
        $this->rounds = $rounds;
    }

    /* Gen Salt */
    public function genSalt() {
        /* openssl_random_pseudo_bytes(16) Fallback */
        $seed = '';
        for($i = 0; $i < 16; $i++) {
            $seed .= chr(mt_rand(0, 255));
        }
        /* GenSalt */
        $salt = substr(strtr(base64_encode($seed), '+', '.'), 0, 22);
        /* Return */
        return $salt;
    }

    /* Gen Hash */
    public function genHash($password) {
        /* Explain '$2y$' . $this->rounds . '$' */
            /* 2a selects bcrypt algorithm */
            /* $this->rounds is the workload factor */
        /* GenHash */
        $hash = crypt($password, '$2y$' . $this->rounds . '$' . $this->genSalt());
        /* Return */
        return $hash;
    }
    
    /* Verify Password */
    public function verify($password, $existingHash) {
        /* Hash new password with old hash */
        $hash = crypt($password, $existingHash);
        
        /* Do Hashs match? */
        if($hash === $existingHash) {
            return true;
        } else {
            return false;
        }
    }
}
/* Next the Usage */
/* Start Instance */
$bcrypt = new bcrypt(12);

/* Two create a Hash you do */
echo 'Bcrypt Password: ' . $bcrypt->genHash('password');

/* Two verify a hash you do */
$HashFromDB = $bcrypt->genHash('password'); /* This is an example you would draw the hash from your db */
echo 'Verify Password: ' . $bcrypt->verify('password', $HashFromDB);
```

See, a little confusing but not to bad. The only thing I want to point out is this line:

```php
$hash = crypt($password, '$2y$' . $this->rounds . '$' . $this->genSalt());
```

Remember early with the list of hashing algorithms I said you needed to use Crypt in PHP using the `$2y$` selector? Well this is an example. First in the `Crypt()` function we put the data to be hashed, then in the second part we put the salt created earlier, however we need to use the `$2y$` select before `' . $this->rounds . '$' . $this->genSalt()` to enable Bcrypt. The second part `' . $this->rounds . '$'` is telling how many times to re-hash the password. By default it is 12. Which is very nice but you can go higher depending on your server. On my PC (AMD Phenom x6 3.3GHz, 16GB RAM, ATI Radeon 6950 2GB) I started to have some issues around 20 rounds where it takes forever to hash it.
Quick note: You could actually make this hashing process run for days but that's your choice!

So is this the solution to insecure database? As long as we create better hashing algorithms we will always have to keep-up with security. I recommend staying up-to-date about security news and patch/exploits to current technologies. Also read-up on sites like "The Register" (theregister.co.uk) for tech news.

Don't worry! This security journey of ours is almost over, we just finished the first part. Next is ways to store salts + passwords and then things you can do to protect from Rainbow attacks.

## Storing salts and passwords (In your database)

Storing salts and passwords can be done in many ways but we will cover only a few examples.

First, lets start with a very basic example.

**Hard-Coded-Salt + Hash**

At a bare minimal we would store our password using a hashing algorithm and with a hard coded salt. This would help against rainbow table, but has a minor flaw. If our hacker/attack gets into our code and sees the Hard-Coded-Salt (a.k.a Pepper) then we have lost the single barrier that helps increase out hash strength. Now the attacker can simply add the salt he has discovered and start matching passwords.

**nique Salt + Hash**

The basic idea here is we have a salt to hash our passwords but its different for each user. We would store this salt in the database with the users information and when the username is called, in a login field for example, we retrieve unique salt and hash it with the supplied password. We then see if the hashes match This however also has a minor flaw. If a hacker manages to gain access to your database he now knows the salt for each user, this will definitely slow him down but we could do better.

**Hard-Coded-Salt + Unique Salt + Hash**

This setup really starts to increase are security. We have a hard-coded-salt in the hashing script, that then is hashed with a unique salt that then is hashed all together into the database. This now requires the user to breach the code and the database to retrieve the necessary information to start discovering the hashed passwords.

BUT! We can do better! In a perfect world you have something fairly similar to this:

**Hard-Coded-Salt + Unique Salt + Hash + Separate database storage**

First we have the hard-coded-salt in our code, then we have a function that generates a unique salt for each user. We then have a function that generates a hash with both the unique salt and hard-coded-salt. However this time you store the unique salt in one database and the password into another database each with a different connection (and passwords). This now requires the hacker to breach the code, the salt database, and the hash database. It's a triforce of protection.

Now it's up to you if you can acquire this type of setup but if your doing something that requires the up most security I really recommend the last option.

One finial note about storing salts and passwords. You might also want to consider a pin system the requires a user to enter a 4 - 16 digit pin number either on login or accessing a sensitive part of the sight, just a suggestion.

## Protect from Rainbow attacks

So like we said Rainbow tables are a huge database of commonly used passwords that are then used to expose the users sensitive data that has been hashed. You can even find pre-made Rainbow tables online, or you can create your own, like we discussed above. How can I prevent from a Brute force attack using a Rainbow table?

1. Uses the hashing methods above with unique randomly generated Salts for each user
2. Create a maximum amount of possible logins per minute/hour/day/week/year etc...
3. Log the login attempt traffic and try to match patterns
4. Lock user accounts after multiple logins (Harsh, but can be a good on security heavy sites)
5. Delete user account after number of failed logins (Super Harsh, let's not do this)

From this point on you must learn by your self. Take the tools I gave you and try implanting them into your work. Below is a list of resources that can provide more information if you do desire to learn more.

## Resources, Discussions & Papers

Below is a list of resources I used to create this tutorial, please visit them to learn more information

**Good Resources/Articles**
* http://php.net/crypt
* http://php.net/manual/en/faq.passwords.php
* http://codahale.com/how-to-safely-store-a-password/ (A Must Read)
* http://yorickpeterse.com/articles/use-bcrypt-fool (A Must Read)
* http://hungred.com/useful-information/php-better-hashing-password/
* http://www.ibm.com/developerworks/opensource/library/os-php-encrypt/
* http://www.itnewb.com/tutorial/Encrypting-Passwords-with-PHP-for-Storage-Using-the-RSA-PBKDF2-Standard

**Good Code Snippets**
* https://gist.github.com/1053158
* http://www.itnewb.com/tutorial/Encrypting-Passwords-with-PHP-for-Storage-Using-the-RSA-PBKDF2-Standard

**Good Discussions on StackOverflow & Other Sites**
* http://stackoverflow.com/questions/2235158/sha1-vs-md5-vs-sha256-which-to-use-for-a-php-login
* http://stackoverflow.com/questions/4795385/how-do-you-use-bcrypt-for-hashing-passwords-in-php
* http://stackoverflow.com/questions/7072478/whats-the-difference-between-bcrypt-and-hashing-multiple-times
* http://stackoverflow.com/questions/4443476/optimal-bcrypt-work-factor/4766811
* http://phpmaster.com/why-you-should-use-bcrypt-to-hash-stored-passwords/

**Rainbow Tables**
* http://security.stackexchange.com/questions/3448/how-long-does-it-take-to-actually-generate-rainbow-tables
* http://www.codinghorror.com/blog/2007/09/rainbow-hash-cracking.html

**Benchmarks for Hardware usage for MD5, SHA-1, SHA-256, SHA-512, etc...**
* http://www.cryptopp.com/benchmarks-amd64.html

**More About Crypt**
* http://nl3.php.net/manual/en/function.crypt.php

**Interesting Papers about Protecting Sensitive Data**
* http://static.usenix.org/events/usenix99/provos/provos.pdf
* http://www.tarsnap.com/scrypt/scrypt.pdf

**Propitiatory Software Talk about in this Tutorial**
* http://aws.amazon.com/ec2/
* http://www.nvidia.com/object/cuda_home_new.html

**Double-Hashing Arguments/Discussions**
* http://www.sitepoint.com/forums/showthread.php?552072-Double-Hashing-Passwords-for-Extra-Security
* http://www.php.net/manual/en/function.sha1.php#81388 (This is a comment from a user)

**Cracking Conman Hashes**
* http://crackstation.net/
