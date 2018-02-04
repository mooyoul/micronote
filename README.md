# micronote
what i've learned today

## 2018-02-04

#### 1. Using Lambda with VPC is not recommended way (officially). OMG!

> According to: https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html#lambda-vpc
> Don't put your Lambda function in a VPC unless you have to.
>
> There is no benefit outside of using this to access resources you cannot expose publicly, like a private Amazon Relational Database instance. Services like Amazon Elasticsearch Service can be secured over IAM with access policies, so exposing the endpoint publicly is safe and wouldn't require you to run your function in the VPC to secure it.

We had some internet connectivity issue with our Lambdas which are configured to use VPC.
That VPC configured to use NAT Gateway to communicate with Internet.
Sometimes Outgoing http requests from cold-started lambdas are failing due to connection timeout.

I asked to aws support to know why i got annoying timeout issues, and got this answer:

> ... (truncated)  We also figured out the reason behind intermittent timeout issue. It is because Lambda containers are not ready properly sometimes before you are making http requests from your code. I understand this is something not in your control and caused due to service side issues.
> Since the issue is intermittent and is due to complex nature of tasks that is done on Lambda service side when it creates a container inside a VPC to access Internet, you have kindly agreed to work with this limitation for now. The lambda service team is aware of this issue and hopefully they should be able to improve on this in some future release of service.

and they said: Using Lambda with VPC is not recommended, [Lambda Best Practices Documentation describes that](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html#lambda-vpc) (!!)

OMG!


#### 2. You must handle timeouts if you access s3 without aws-sdk in Lambda, (even without VPC!)

We had some connectivitiy issues between Lambda and S3 also.
even we disabled VPC from our lambdas, sometimes we got connection timeout error.

recently we've added code that access s3 via curl, sometimes we was getting connection timeout error.
actually aws-sdk handles these errors such as "Connection Timeout", so we haven't see these errors before. but errors are coming now. so we had to implement retry logic manually.

#### 3. MP4/MOV conatiners cannot be created through pipes such as stdout

Yeah, You can't create video which has MOV/MP4 as container through stodut
By default, these container requires "seekable output" to write special metadatas like `moov` atom.

so that's why you can't create MOV/MP4 output through non-seekable destination.



## 2018-01-26

#### 1. TLSv1.1 and TLSv1.2 is disabled by default on Android < 4.4

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



