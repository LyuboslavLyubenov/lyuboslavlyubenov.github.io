---
layout: post
title: >- I do love me some "TypeError: NetworkError" errors 
categories: [reactnative, react, ssl]
---


Hey everyone, I've got a story to share about a "quirky" bug I encountered while working on a new feature, how I tackled it, and the lessons I learned along the way.

So, I was assigned to develop a feature for our mobile app – adding a tab that could be customized to display a webpage upon opening. 

Sounds simple, right? I though so as well.

The implementation went smoothly, QA gave it a thumbs up, and we set a release date and configuration internally. Everything seemed to be going as planned, but, well, here I am writing about it, so clearly, things took a turn.

It's almost as if the god of mischief decided to ruin my day. How so, you ask? Well, during internal testing of our Android and iOS apps, we stumbled upon something ... interesting – our webpage wasn't loading on Android.

So, naturally, I switched into 'debugging mode':

- Checked the configuration – URL looked fine ✅
- Ran the app on my iOS emulator – all good there ✅
- Tested on Android – still working ✅

Feeling overwhelmed, I took a 30-minute break for a walk and a shower.

Unfortunately, that didn't magically spark a solution. So, I tried one more thing – installing the development version of the app on a physical device.

And... it failed.

Looking at the debug console i got the following cryptic erorr message: "TypeError: Network request failed."

"What do you mean, network request failed? It works on my emulator and other devices, and I have internet!"

In such desperate times, I turned to ChatGPT for help, and it provided a list:

1. Test connectivity
2. Verify URL and API endpoints
3. Check for proxy settings
4. Check SSL certificate

The suggestions seemed logical, so I started checking them off one by one:

- Tested connectivity ✅
- Verified URL and API endpoints ✅
- No proxy settings, so all good ✅
- Checked SSL certificate – it was valid ✅

This issue was mocking me. I could feel it. It wanted me to give up. But it doesn't know who it's dealing with!

![alt text](6ac30a77afe03112ff5518ec8000341e.jpg)

Since I was out of ideas, and ChatGPT seemed to be of no use, I resorted to my trusty Google skills.

After a couple of hours of searching, I stumbled upon this gem: https://github.com/facebook/react-native/issues/6827

So, I decided to be a bit goofy and loaded the page on my Android phone. Lo and behold, Chrome reported the website as insecure. Upon investigation, it turned out that the intermediate certificate provided by Let's Encrypt was invalid.

"What's a Certificate Authority?" you ask?

SSL certificates ensure secure encrypted connections between clients and servers. To ensure their authenticity, we have Certificate Authorities (CAs). They're arranged in a hierarchical structure, with the Root CA at the top, trusted by all browsers, followed by Intermediate CAs, hosting providers' CAs, and finally, the certificate for your website. For a website to be marked as trusted, the browser follows this chain, checking each certificate's validity until it reaches the bottom. If any certificate in this chain is invalid or if there's a missing trusted Root CA at the top, the certificate is deemed invalid.

So, the issue stemmed from the invalid OpenSSL CA. 
Beeing the lazy person I am, I opted for the classic 'turn it off and on again' approach – I deleted the website certificate and regenerated it. 

Voila! Problem solved. 

I felt a sense of pride as QA and the clients confirmed everything was working fine, and we proceeded with the public release.


At the end difficult situations like this teach us a lot. This particular issue helped me understand more about how SSL certificates are validated. As a developer, you should always embrace challenges, because not only it help with your mental state, but enables you to learn valueable lessons

