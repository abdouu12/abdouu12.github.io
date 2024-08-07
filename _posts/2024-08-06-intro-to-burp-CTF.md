---
title: "intro to burp CTF"
date: 2024-08-06 17:07:00 +0000
categories: [CTF]
tags: [burp]
---

what did I learn from solving this CTF?
- how to read and understand the requests sent by the client to the server
- how to read and understand the responses sent from the server to the client
- what HTTP does
- how to use burp and specifically the repeater tool to modify and manipulate requests 

let's look at a live example, this is a simple 50 point CTF.

this is what we're presented with once we accept the challenge:
![Image 1](/assets/images/image1.jpg)

We will try to use burp here, burp is a software that allows us to see the requests that the client sends via HTTP to the server and it also lets us analyse the response the server gives.

the second hint the picoCTF challenge gives us is "Try mangling the request, maybe their server-side code doesn't handle malformed requests very well."

if we fill the registration form it will take us to a 2fa authentication, if we fill this further we will get the message invalid OTP, this is our hint.

I launched burp and turned the interception on, here is what we got:
![2FA Intercepted](/assets/images/2FA.jpg)

I sent it to the repeater:
![Repeater Request](/assets/images/repeater1.jpg)

and I did what the hint told me to do, I mangled with the request by deleting my OTP code that I entered.

Boom. Flag captured.
![Flag Captured](/assets/images/repeater2.jpg)