---
layout: post
title:  "Two-Factor, How?"
date:   2018-12-16 09:26:00 +0000
categories: blog posts
---

# Enter your authentication code to continue...

We're probabally all familiar with Two-Factor Authentication (2FA), it's widely used on plenty of websites, for example [Heroku](https://www.heroku.com/), and greatly strengthens our account security. After entering the main password, the account holder is presented with an additional input field;


![Heroku 2FA prompt](/assets/images/2FA/heroku-2fa-request.png)

If at this point you've no idea what I'm talking about, then I'm sorry but this post isn't really going to be for you. This isn't going to be an advocation for 2FA (although you really should use it where possible) or a "How To" on using 2FA, but rather its going to be a "Tell me your secrets, little QR Code" kinda post where we dig into the mythical workings of the ~~tardis~~ funky square barcode in the context of 2FA.

Having said that, we do need to quickly highlight how 2FA works on the surface, as this is important to follow along the more detailed steps.
## 2FA Overview
1. Setup 
    1. The website presents us with a QR code
    2. We scan the QR code using our Authentication application
2. Usage
    1. We read the 6 digit code from our Authentication application and enter it into the website
    2. The website checks the code entered is the code which it expected to receive and grants, or denies, access

## Resources
I'm going to use [Daplie](https://daplie.github.io/browser-authenticator/) to generate the QR codes for the purpose of this example. It allows for entering custom information for the issuer, account, and even provide our own secret if we so wish. Finally it displays the corresponding 6 digit authentication code allowing us to validate what we are doing.

Additionally I created a sample application, [Pink Lemonade](https://github.com/david-mcqueen/Pink_Lemonade_2FA), which provides basic implementation of a 2FA Application - This allows for stepping through the process of scanning the token, through to it generating the 6 digit code so we can see at a code level what is happening.

## Tell me your secrets, little QR code

The QR code is often the face of 2FA, presented to the user when setting up 2FA it gives the authentication application, such as [Authy](https://authy.com/), the secret it needs in order to calculate the correct 6 digit code. With it you'll be able to unlock all the wonders of the world*, without it you'll feel just like a Balrog battling Gandalf in that 1st LOTR film where Sean Bean dies at the end (Spoilers! for those of you who've not yet seen the film in the 17 years since it came out).

\* _Well, at least the application you're trying to access_


### What's in a code?
The QA code itself is little more than a user-friendly way of transfering a _secret key_ (amongst other meta data) from the server to the users device. Without the use of a QA code (or indeed another medium), the user would need to manually enter the _secret key_ in order for 2FA to work.

Lets take a look at an example.


![Generated Code](/assets/images/2FA/daplie_generated_code.png)


Scanning this QR code (generated on [Daplie](https://daplie.github.io/browser-authenticator/)), we can see that it contains the following data payload;

![Code Data](/assets/images/2FA/code_data.png)

```
otpauth://totp/Dave%20McQueen:david@dmcqueen.co.uk?secret=AV4GZXBUXVVTQPNQAINJWWQ322QQGWG4&issuer=Dave%20McQueen&algorithm=SHA1&digits=6&period=30
```

And in a slightly more readable form;

```
1. otpauth://totp
2. Dave%20McQueen:david@dmcqueen.co.uk
3. secret=AV4GZXBUXVVTQPNQAINJWWQ322QQGWG4
4. issuer=Dave%20McQueen
5. algorithm=SHA1&digits=6&period=30
```

1. Indicates that the algorithm to use is _totp_ (Time-based One-time Password)
2. Dictates the issuer & account, separate by a colon
3. Is the main data we want, this is the secret key which is fed to the _totp_ algorithm. This is the focus of what we are looking at today
4. The issuer, or website, this token is for
5. Additional information which can be used as flags to use; a different hashing algorithm, generate a different number of digits in the 2FA code, and how long the code is valid for

When scanning a QR code, such as the one above, using a 2FA application, this data is stored on the device and is subsequently used to generate the 6 digit authentication token we are have come to ~~love~~ appreciate. 

That's step 1.1 & 1.2 from above, Done! A simple one step process of communicating the _secret key_ to the user.


### Keys go in, codes come out
Now that our application knows what the secret key is, it can use it as an input to the _totp_ algorithm (in Bytes), which in turn provides an output of a 6 digit number. 

The snippet below contains the abridged steps for this, taken from [Pink Lemonade](https://github.com/david-mcqueen/Pink_Lemonade_2FA); where _secret_ is part 3 of the QR code data payload, above.


{% highlight csharp %}

private byte[] SecretAsBytes
{
    get
    {
        if (string.IsNullOrWhiteSpace(Secret))
            return new byte[0];

        var bytes = Base32Encoding.ToBytes(Secret);
        return bytes;
    }
}
...
this._totp = new Totp(SecretAsBytes);
...
public string TokenCode
{
    get
    {
        return _totp.ComputeTotp();
    }
}

{% endhighlight %}

Voila!

![Pink Lemonade Code](/assets/images/2FA/pink_lemonade_code.png)


Step 2.1, Done!


However there is one incy wincy critical problem with what we have discussed so far - Using the same constants as inputs will yield exactly the same output. Every. Time.

What we need now is something which is different, but yet the same between the users drvice and the server, because the server needs to be able to validate the provided input code is allowed.


### What time is it Mr. ~~Wolf~~ Secret key?

We've got our secret key. Thats good. But by itself its pretty naff and opens us up to the same sort of attacks which have become prevalent with standard passwords. Such s MITM attacks, or that creepy guy standing over your shoulder in the coffee shop, exposing the password as we enter them into the browser.

Now for our super weapon! More than being something which allows wrist watches to have a purpose, time is actually useful. I know I was shocked as well. We can use time as a moving constant alongside our secret key as the input to the _totp_ algorithm. Because time is constantly progressing forward, and it is not trivial* to go back in time, we've always got a changing value and thus a changing 6 digit code which we use to validate ourselves. 

\* _dare I say it, impossible_

The constant _secret key_ communicated by the QR code when we first setup 2FA, and the changing constant of UTC time allows for an ever changing but predictable code, so long as you know the secret. 

MITM attacks, and the coffee shop guy, are both foiled through the use of 2FA as the secret key is not exposed when we enter the 6 digit code. So even if someone is watching the traffic or looking over your shoulder, they only see the current code which is typically only valid for upto 30 seconds. Even if an attacker did change the local time on the device which is being used to generate the code, the server would have a different (aka real) time stamp and the generated codes would not match. _Denied, Hacker person_.

### Server validation
When the server receives the 6 digit code which has been input by the user, it simply needs to generate its own version of the code and check that the 2 match - like it does for a standard hashed password already.

Step 2.2, Done!


### Closing Thoughts
Thats pretty much the fundamentals of what is happening under the hood of 2FA. A simple, easy to use solution which instantly adds an extra layer of security to users accounts withouth much complexity.

After we have setup 2FA the QR code (_secret key_) is not shown again meaning that another application can't be setup to generate codes. Although It is entirely possible to write down the _secret key_, or scan it into multiple 2FA applications and have them all generate the same 6 digit code. If they have the same secret key as their input, then they will generate the same output.

However sharing the QR code / secret key is a less than an ideal idea - we're using 2FA to strengthen our security, not pass around the key like pass the parcel at a kids birthday party.

Hopefully thats provided a little insight to how the QR code & 2FA codes work, and sorry about the Sean Bean spoiler. At least his death in LOTR wasn't as bad as his death in Goldeneye; Nobody wants to fall into a massive satelite dish and then get crushed by the cradle...