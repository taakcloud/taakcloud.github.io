---
title: "Web Push Notification"
date: 2023-01-24 08:08:00 +0000
description: Web push notification with ease on scale
categories:
 - products
set: products
order_number: 1
type: Document
layout: doc
toc: true
---

Send web push notification with ease and zero cost on scale.

## Prerequisites

- [x] Select your app/workspace, see [Create app]({% post_url 2023-01-22-create-app %})
- [x] Get your API Key, see [Generate API Key]({% post_url 2023-01-24-generate-api-key %})

## Taak SDK JS

We use [Taak SDK JS <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://github.com/taakcloud/taak-sdk-js){:target="_blank"}{:rel="noopener noreferrer"} as client side library. Full source code is available at [here <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://github.com/taakcloud/taak-web-push-example){:target="_blank"}{:rel="noopener noreferrer"}
{% highlight shell %}
npm i taak-sdk
{% endhighlight %}
<br />

### Check permission

Following code is for React library, but basically must work in both TypeScript and JavaScript projects.
<details open><summary>Check if permission given</summary>
{% highlight ts %}
{% include_code https://raw.githubusercontent.com/taakcloud/taak-web-push-example/main/src/components/WebPushToggle.tsx!19!33%}
{% endhighlight %}
</details>
[WebPushToggle.tsx <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://github.com/taakcloud/taak-web-push-example/blob/main/src/components/WebPushToggle.tsx#L19-L33){:target="_blank"}{:rel="noopener noreferrer"}
{: .right}

### Subscribe

<details open><summary>Check if subscribed</summary>
{% highlight ts %}
{% include_code https://raw.githubusercontent.com/taakcloud/taak-web-push-example/main/src/components/WebPushSubscribeButton.tsx!18!27 %}
{% endhighlight %}
</details>
[WebPushSubscribeButton.tsx <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://github.com/taakcloud/taak-web-push-example/blob/main/src/components/WebPushSubscribeButton.tsx#L18-L27){:target="_blank"}{:rel="noopener noreferrer"}
{: .right}

<details open><summary>Subscribe</summary>
{% highlight ts %}
{% include_code https://raw.githubusercontent.com/taakcloud/taak-web-push-example/main/src/components/WebPushSubscribeButton.tsx!29!45 %}
{% endhighlight %}
</details>
[WebPushSubscribeButton.tsx <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://github.com/taakcloud/taak-web-push-example/blob/main/src/components/WebPushSubscribeButton.tsx#L29-L45){:target="_blank"}{:rel="noopener noreferrer"}
{: .right}

### Send

<details open><summary>Send web push</summary>
{% highlight ts %}
{% include_code https://raw.githubusercontent.com/taakcloud/taak-web-push-example/main/src/components/WebPushSubscriptionList.tsx!28!37 %}
{% endhighlight %}
</details>
[WebPushSubscriptionList.tsx <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://github.com/taakcloud/taak-web-push-example/blob/main/src/components/WebPushSubscriptionList.tsx#L28-L37){:target="_blank"}{:rel="noopener noreferrer"}
{: .right}

### List

<details open><summary>Subscription list</summary>
{% highlight ts %}
{% include_code https://raw.githubusercontent.com/taakcloud/taak-web-push-example/main/src/components/WebPushSubscriptionList.tsx!20!26 %}
{% endhighlight %}
</details>
[WebPushSubscriptionList.tsx <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://github.com/taakcloud/taak-web-push-example/blob/main/src/components/WebPushSubscriptionList.tsx#L20-L26){:target="_blank"}{:rel="noopener noreferrer"}
{: .right}

### Delete

<details open><summary>Delete an subscription</summary>
{% highlight ts %}
{% include_code https://raw.githubusercontent.com/taakcloud/taak-web-push-example/main/src/components/WebPushSubscriptionList.tsx!39!47 %}
{% endhighlight %}
</details>
[WebPushSubscriptionList.tsx <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://github.com/taakcloud/taak-web-push-example/blob/main/src/components/WebPushSubscriptionList.tsx#L39-L47){:target="_blank"}{:rel="noopener noreferrer"}
{: .right}

### Test suite

[index.test.ts <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://github.com/taakcloud/taak-sdk-js/blob/main/src/web-push/index.test.ts){:target="_blank"}{:rel="noopener noreferrer"}
