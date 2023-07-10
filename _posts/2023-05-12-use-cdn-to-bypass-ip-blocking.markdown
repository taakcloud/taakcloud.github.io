---
title: "Use CDN to bypass IP blocking"
date: 2023-05-12 08:08:00 +0000
description: An unidentifiable mechanism to bypass IP blocking
categories:
 - blog
set: blog
order_number: 1
type: Document
layout: doc
toc: true
---

![In a big picture]({{ site.baseUrl }}/assets/img/cdn-by-pass-ip-blocking/cover.jpg "In a big picture")
{:.tofigure}

## Requirements

- [x] A remote server with a public IP(v4) address (e.g. 13.49.80.99)
- [x] A domain name (e.g. helper.example.com)

First make sure your remote server doesn't not have the limitation that your client machine have, so check your internet access from a remote server first. For example your remote server could be a micro instance from Amazon ec2 free tier. You also need a domain name or a sub domain for this method, and a trustworthy CDN provider, such as [cloudflare.com <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://cloudflare.com){:target="_blank"}{:rel="noopener noreferrer"}. please note some CDN providers are not suitable for this method, because they are under pressure of a government and can't help you to not report your server IP to blocker machines.

## Tested environment

- [x] Ubuntu 20.04 LTS
- [x] [Trojan-go <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://github.com/p4gefau1t/trojan-go){:target="_blank"}{:rel="noopener noreferrer"} v0.10.6

You could have a different environment as far as our requirements are supported, for example pay attention your Shadowsocks implementation does support CDN transfer.

## Enter A DNS Record

Following screenshot shows how you could create a type A DNS Record at cloudflare website, please note to keep _Proxy status_ on and use your server IP address and sub domain name for next steps.

![A DNS Record]({{ site.baseUrl }}/assets/img/cdn-by-pass-ip-blocking/cloudflare-a-dns-record.png "A DNS Record")
{:.tofigure}

## Install Trojan-go

{% highlight shell %}
cd /home/ubuntu/
wget https://github.com/p4gefau1t/trojan-go/releases/download/v0.10.6/trojan-go-linux-amd64.zip
unzip trojan-go-linux-amd64.zip -d trojan-go
{% endhighlight %}
<br />

Next create _server.json_ file and don't worry about not existed files yet, we will take care of them in next steps.
<details open><summary>server.json</summary>
{% highlight json %}
{
  "run_type": "server",
  "local_addr": "0.0.0.0",
  "local_port": 443,
  "remote_addr": "127.0.0.1",
  "remote_port": 80,
  "password": ["your-password"],
  "ssl": {
    "cert": "/home/ubuntu/.acme.sh/helper.example.com/fullchain.cer",
    "key": "/home/ubuntu/.acme.sh/helper.example.com/helper.example.com.key",
    "sni": "helper.example.com"
  },
  "websocket": {
    "enabled": true,
    "path": "/path",
    "host": "helper.example.com"
  }
}
{% endhighlight %}
</details>
_/home/ubuntu/trojan-go/server.json_
{: .right}

## SSL Certificate generation

For keeping it simple, we are going to use [acme.sh <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://github.com/acmesh-official/acme.sh){:target="_blank"}{:rel="noopener noreferrer"} to create 3 months valid Certificate manually, but keep in mind you could automate this part.

#### Install acme.sh

{% highlight shell %}
wget -O -  https://get.acme.sh | sh -s email=your-mail-address@gmail.com
{% endhighlight %}

<details><summary>output</summary>
{% highlight shell %}
[Fri May 12 12:44:09 UTC 2023] Installing from online archive.
[Fri May 12 12:44:09 UTC 2023] Downloading https://github.com/acmesh-official/acme.sh/archive/master.tar.gz
[Fri May 12 12:44:09 UTC 2023] Extracting master.tar.gz
[Fri May 12 12:44:09 UTC 2023] It is recommended to install socat first.
[Fri May 12 12:44:09 UTC 2023] We use socat for standalone server if you use standalone mode.
[Fri May 12 12:44:09 UTC 2023] If you don't use standalone mode, just ignore this warning.
[Fri May 12 12:44:09 UTC 2023] Installing to /home/ubuntu/.acme.sh
[Fri May 12 12:44:09 UTC 2023] Installed to /home/ubuntu/.acme.sh/acme.sh
[Fri May 12 12:44:09 UTC 2023] Installing alias to '/home/ubuntu/.bashrc'
[Fri May 12 12:44:09 UTC 2023] OK, Close and reopen your terminal to start using acme.sh
[Fri May 12 12:44:09 UTC 2023] Installing cron job
no crontab for ubuntu
no crontab for ubuntu
[Fri May 12 12:44:09 UTC 2023] Good, bash is found, so change the shebang to use bash as preferred.
[Fri May 12 12:44:10 UTC 2023] OK
[Fri May 12 12:44:10 UTC 2023] Install success!
{% endhighlight %}
</details>

{% highlight shell %}
~/.acme.sh/acme.sh --issue --dns -d helper.example.com --yes-I-know-dns-manual-mode-enough-go-ahead-please
{% endhighlight %}

<details><summary>output</summary>
{% highlight shell %}
[Fri May 12 12:53:34 UTC 2023] Using CA: https://acme.zerossl.com/v2/DV90
[Fri May 12 12:53:34 UTC 2023] Single domain='helper.example.com'
[Fri May 12 12:53:34 UTC 2023] Getting domain auth token for each domain
[Fri May 12 12:53:35 UTC 2023] Getting webroot for domain='helper.example.com'
[Fri May 12 12:53:36 UTC 2023] Add the following TXT record:
[Fri May 12 12:53:36 UTC 2023] Domain: '_acme-challenge.helper.example.com'
[Fri May 12 12:53:36 UTC 2023] TXT value: 'pvTeAEHR-ItbJPekq05btsa-EM1hn5atGVFcZxlzJIA'
[Fri May 12 12:53:36 UTC 2023] Please be aware that you prepend _acme-challenge. before your domain
[Fri May 12 12:53:36 UTC 2023] so the resulting subdomain will be: _acme-challenge.helper.example.com
[Fri May 12 12:53:36 UTC 2023] Please add the TXT records to the domains, and re-run with --renew.
[Fri May 12 12:53:36 UTC 2023] Please add '--debug' or '--log' to check more details.
[Fri May 12 12:53:36 UTC 2023] See: https://github.com/acmesh-official/acme.sh/wiki/How-to-debug-acme.sh
{% endhighlight %}
</details>

Next create a TXT DNS Record with values given by previous command.

![TXT DNS Record]({{ site.baseUrl }}/assets/img/cdn-by-pass-ip-blocking/cloudflare-txt-record.png "TXT DNS Record")
{:.tofigure}

After a minute or so, run following command to finish Certificate generation.

{% highlight shell %}
 ~/.acme.sh/acme.sh --renew -d helper.example.com  --yes-I-know-dns-manual-mode-enough-go-ahead-please
{% endhighlight %}

<details><summary>output</summary>
{% highlight shell %}
[Fri May 12 13:08:56 UTC 2023] The domain 'helper.example.com' seems to have a ECC cert already, lets use ecc cert.
[Fri May 12 13:08:56 UTC 2023] Renew: 'helper.example.com'
[Fri May 12 13:08:56 UTC 2023] Renew to Le_API=https://acme.zerossl.com/v2/DV90
[Fri May 12 13:08:56 UTC 2023] Using CA: https://acme.zerossl.com/v2/DV90
[Fri May 12 13:08:56 UTC 2023] Single domain='helper.example.com'
[Fri May 12 13:08:56 UTC 2023] Getting domain auth token for each domain
[Fri May 12 13:08:56 UTC 2023] Verifying: helper.example.com
[Fri May 12 13:08:58 UTC 2023] Processing, The CA is processing your order, please just wait. (1/30)
[Fri May 12 13:09:01 UTC 2023] Success
[Fri May 12 13:09:01 UTC 2023] Verify finished, start to sign.
[Fri May 12 13:09:01 UTC 2023] Lets finalize the order.
[Fri May 12 13:09:01 UTC 2023] Le_OrderFinalize='https://acme.zerossl.com/v2/DV90/order/kND...P8A/finalize'
[Fri May 12 13:09:02 UTC 2023] Order status is processing, lets sleep and retry.
[Fri May 12 13:09:02 UTC 2023] Retry after: 15
[Fri May 12 13:09:18 UTC 2023] Polling order status: https://acme.zerossl.com/v2/DV90/order/kND...P8A
[Fri May 12 13:09:18 UTC 2023] Downloading cert.
[Fri May 12 13:09:19 UTC 2023] Le_LinkCert='https://acme.zerossl.com/v2/DV90/cert/rZZ-...bbQ'
[Fri May 12 13:09:19 UTC 2023] Cert success.
-----BEGIN CERTIFICATE-----
MIIEDDCCA5KgAwIBAgIRAIU1Czr+VzBMW01Rp5hUP44wCgYIKoZIzj0EAwMwSzEL
MAkGA1UEBhMCQVQxEDAOBgNVBAoTB1plcm9TU0wxKjAoBgNVBAMTIVplcm9TU0wg
RUNDIERvbWFpbiBTZWN1cmUgU2l0ZSBDQTAeFw0yMzA1MTIwMDAwMDBaFw0yMzA4
...
w5oQT9b4Fe7CbKeYWliZECaCd0Y8nEBRBzAfBgNVHREEGDAWghRoZWxwZXIudGFh
a2Nsb3VkLmNvbTAKBggqhkjOPQQDAwNoADBlAjEA6+8yCrS3Fh2+dR0aKX0hKV+B
NaKg3k3etgp6mbftyehTgbs+GVLL++6uDwtMXSUgAjByPLMfWshlHqZVrGg/F+R3
1PLKmf91b1EZRRRGeMEQ+TBGqusOQf3d+cvizYSnq+I=
-----END CERTIFICATE-----
[Fri May 12 13:09:19 UTC 2023] Your cert is in: /home/ubuntu/.acme.sh/helper.example.com_ecc/helper.example.com.cer
[Fri May 12 13:09:19 UTC 2023] Your cert key is in: /home/ubuntu/.acme.sh/helper.example.com_ecc/helper.example.com.key
[Fri May 12 13:09:19 UTC 2023] The intermediate CA cert is in: /home/ubuntu/.acme.sh/helper.example.com_ecc/ca.cer
[Fri May 12 13:09:19 UTC 2023] And the full chain certs is there: /home/ubuntu/.acme.sh/helper.example.com_ecc/fullchain.cer
{% endhighlight %}
</details>

For Cloudflare CDN make sure SSL mode is set to full.

![SSL Mode Full]({{ site.baseUrl }}/assets/img/cdn-by-pass-ip-blocking/ssl-mode-full.png "SSL Mode Full")
{:.tofigure}


## Install apache2
{% highlight shell %}
sudo apt install apache2 -y
sudo systemctl enable apache2
{% endhighlight %}

## Test run server.json

{% highlight shell %}
cd ~/trojan-go
sudo ./trojan-go -config server.json
{% endhighlight %}

<details><summary>output</summary>
{% highlight shell %}
[INFO]  2023/05/12 13:36:40 trojan-go v0.10.6 initializing
[WARN]  2023/05/12 13:36:40 empty tls fallback port
[WARN]  2023/05/12 13:36:40 empty tls http response
{% endhighlight %}
</details>

Note: both port 80 and 443 must be open and accessible from outside.

## Test run client.json

<details open><summary>client.json</summary>
{% highlight json %}
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "helper.example.com",
    "remote_port": 443,
    "password": [
        "your-password"
    ],
    "ssl": {
        "sni": "helper.example.com"
    },
    "websocket": {
        "enabled": true,
        "path": "/path",
        "host": "helper.example.com"
    }
}
{% endhighlight %}
</details>
_/Users/frank/trojan-go-darwin-arm64/client.json_
{: .right}

{% highlight shell %}
sudo ./trojan-go -config client.json
{% endhighlight %}

<details><summary>output</summary>
{% highlight shell %}
[INFO]  2023/05/12 13:59:28 trojan-go v0.10.6 initializing
[INFO]  2023/05/12 13:59:28 adapter listening on tcp/udp: 127.0.0.1:1080
[INFO]  2023/05/12 13:59:28 cert is unspecified, using default ca list
{% endhighlight %}
</details>

Set socks proxy for your system, e.g. for mac:
![Mac socks proxy]({{ site.baseUrl }}/assets/img/cdn-by-pass-ip-blocking/mac-socks-proxy.png "Mac socks proxy")
{:.tofigure}

Then check your resolved IP, for example go to [ipleak.net <i class="fa-solid fa-arrow-up-right-from-square"/>](https://ipleak.net){:target="_blank"}{:rel="noopener noreferrer"}. If you see your proxy server IP address, then congratulations! you did bypass IP blocking with unidentifiable mechanism.

### Bonus #1 (run as service in server)

[Tmux <i class="fa-solid fa-arrow-up-right-from-square"/>](https://github.com/tmux/tmux/wiki){:target="_blank"}{:rel="noopener noreferrer"} is a convenient tool to run a program in background, but it worth the effort to run trojan-go as a service.

{% highlight shell %}
sudo vim /lib/systemd/system/trojan-go.service
{% endhighlight %}

<details open><summary>trojan-go.service</summary>
{% highlight bash %}
[Unit]
Description=Trojan-go service

[Service]
ExecStart=/home/ubuntu/trojan-go/trojan-go -config /home/ubuntu/trojan-go/server.json

[Install]
WantedBy=multi-user.target
{% endhighlight %}
</details>
_/lib/systemd/system/trojan-go.service_
{: .right}

{% highlight shell %}
sudo systemctl daemon-reload
sudo systemctl enable trojan-go
sudo systemctl start trojan-go
sudo systemctl status trojan-go
{% endhighlight %}

For debug
{% highlight shell %}
journalctl --follow -u trojan-go
{% endhighlight %}

### Bonus #2 (restart trojan-go service every hour)

You may notice your client may have difficulty to connect to the service sometimes, and in my case restarting the service was enough to fix them.

{% highlight shell %}
crontab -e
{% endhighlight %}
and add following line at the end:
{% highlight bash %}
*/60 * * * * systemctl restart trojan-go.service
{% endhighlight %}

### Bonus #3 (Android client)

Following Trojan URL format works for [Igniter app <i class="fa-solid fa-arrow-up-right-from-square"/>](https://play.google.com/store/apps/details?id=io.github.trojan_gfw.igniter){:target="_blank"}{:rel="noopener noreferrer"}

{% highlight bash %}
trojan://your-password@helper.example.com:443#hope
{% endhighlight %}

but with penalty of disabling _CDN Proxy shield_, so there is a chance your server's public IP gets blocked and you have to use another IP.

### Bonus #4 (iOS client)

[NapsternetV <i class="fa-solid fa-arrow-up-right-from-square"/>](https://apps.apple.com/us/app/napsternetv/id1629465476){:target="_blank"}{:rel="noopener noreferrer"} did work as client but same as Android doesn't work with _CDN Proxy status_ on. Maybe this issue is related to how cloudflare handles SSL between CDN servers and your server, so you may have luck with another CDN provider.

<details open><summary>NapsternetV config</summary>
{% highlight json %}

{"routing":{"domainStrategy":"Asls"},"inbounds":[{"sniffing":{"enabled":false},"listen":"127.0.0.1","protocol":"socks","settings":{"udp":true,"auth":"noauth","userLevel":8},"tag":"socks","port":10808}],"outbounds":[{"mux":{"enabled":false},"streamSettings":{"network":"tcp","tlsSettings":{"serverName":"helper.example.com","allowInsecure":true,"fingerprint":"chrome"},"security":"tls"},"protocol":"trojan","settings":{"servers":[{"password":"your-password","port":443,"method":"aes-128-cfb","ota":false,"level":8,"address":"helper.example.com"}]}}],"log":{"loglevel":"none"},"dns":{"servers":["8.8.8.8","8.8.4.4"]}}
{% endhighlight %}
</details>

### Bonus #5 ([Privoxy <i class="fa-solid fa-arrow-up-right-from-square"/>](https://www.privoxy.org/){:target="_blank"}{:rel="noopener noreferrer"} http proxy use this socks proxy)

Simply add following line to your _/etc/privoxy/config_
<details open><summary>/etc/privoxy/config</summary>
{% highlight bash %}
forward-socks5t   /               127.0.0.1:1080 .
{% endhighlight %}
</details>
_/etc/privoxy/config_
{: .right}

That's all, at the end I would like to thank all scientists who contribute to this solution and are fighting in the right side of history. Let's hope one day we live in a world that governments can't use Geo IP blocking to isolate people.