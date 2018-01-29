# micronote
what i've learned today

## 2018-01-26

#### 1. TLSv1.1 and TLSv1.2 is disabled by default on Android < 4.4

> TL;DR: TLSv1.1/TLSv1.2 are only available in Android >= 4.4 (Kitkat)

We've have an issue with old android devices after distributing some traffic to our brand new 'API Edge', which is powered by Cloudfront with [Lambda@Edge](https://aws.amazon.com/lambda/edge/).

meaningful stack trace from fabric:

> Non-fatal Exception: javax.net.ssl.SSLHandshakeException
> javax.net.ssl.SSLProtocolException: SSL handshake aborted: ssl=0x77cdfb70: Failure in SSL library, usually a protocol error error:14077410:SSL routines:SSL23_GET_SERVER_HELLO:sslv3 alert handshake failure (external/openssl/ssl/s23_clnt.c:744 0x75bc9d74:0x00000000)

[fabric (crashlytics)](https://www.fabric.io) reports most of android versions are below 4.4 Kitkat.
it seems definitely SSL Handshake error also!

investigated handshake issue with awesome [testssl.sh](https://github.com/drwetter/testssl.sh) ([homepage](https://testssl.sh)) utility

I've found that old akamai edge supports TLSv1, but cloudfront didn't. cloudfront denies TLSv1 negotiation, because we configured our cloudfront to use TLSv1.1/TLSv1.2 only

After some googling, i found [interesting android document which describes about SSL Protocol Compabitlity between android releases](https://developer.android.com/reference/javax/net/ssl/SSLSocket.html) (see section named "Default configuration for different Android versions")

so issues are gone after re-configured our cloudfront distribution to support TLSv1.

also there are another workaround without enabling TLSv1 on server:
https://blog.dev-area.net/2015/08/13/android-4-1-enable-tls-1-1-and-tls-1-2/

and maybe it's off topic, Android Platform (Probably Google Play Services) has awesome feature which updates security dependencies like OpenSSL dynamically for Android Apps:
https://blog.dev-area.net/2015/08/17/protect-your-android-app-against-ssl-exploits/


#### 2. DCT based Audio Codecs (e.g. AAC, MP3) have priming frames (called "Encoder Delay")

https://developer.apple.com/library/content/documentation/QuickTime/QTFF/QTFFAppenG/QTFFAppenG.html

that's why i can't concatenate multiple aac audio streams without having "gap" between each aac audio streams without re-encoding.

and i found awesome article about implementing gapless playback with DCT-based codecs (AAC/MP3) using Media Source Extensions (MSE): http://dalecurtis.github.io/llama-demo/index.html

#### 3. How to Test Weighted Routing (on AWS Route 53) on Mobile Devices

[AWS Route 53](https://aws.amazon.com/route53/) has [Weighted Routing Policy](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html#routing-policy-weighted) feature to distribute traffics to multiple destinations

let example.com has two CNAME records.
- example.com.edgekey.net. (with weight 125)
- distribution-id.cloudfront.net. (with weight 255)

if i want to send traffics to "example.com.edgekey.net" only on Mobile Device (e.g. Android Phone), 
we have three choices:

1. Edit `/etc/hosts`
   - Easist way, but can'be used on non-rooted devices

2. Setup Local BIND Server, and change DNS configuration on Target Device
   - Configuring BIND Server may hard, but there's [Docker Image](https://hub.docker.com/r/sameersbn/bind/)!
   - See also: http://www.damagehead.com/blog/2015/04/28/deploying-a-dns-server-using-docker/

3. Setup HTTP Proxy with Charles Proxy, Configure Proxy on Target device, and Use DNS Spoofing feature.
   - [Charles Proxy](https://www.charlesproxy.com/) is definitely swiss-army-knife. Charles has builit-in DNS Spoofing feature.
   - See Guide: https://www.charlesproxy.com/documentation/tools/dns-spoofing/



