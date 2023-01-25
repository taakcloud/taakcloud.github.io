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

- [x] Select your app/workspace, see [Create app <i class="fa-solid fa-arrow-up-right-from-square"></i>]({% post_url 2023-01-22-create-app %}){:target="_blank"}{:rel="noopener noreferrer"}
- [x] Get your API Key, see [Generate API Key <i class="fa-solid fa-arrow-up-right-from-square"></i>]({% post_url 2023-01-24-generate-api-key %})

## Taak SDK JS

We use [Taak SDK JS <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://github.com/taakcloud/taak-sdk-js){:target="_blank"}{:rel="noopener noreferrer"} as client side library.
{% highlight shell %}
npm i taak-sdk
{% endhighlight %}
<br />

## Check permission

Following code is for React library, but basically must work in both TypeScript and JavaScript projects.
<details><summary>Click here for the code</summary>

{% highlight typescript mark_lines="80 100" %}
import { FC, useEffect } from 'react'
import { IonItem, IonLabel, IonToggle, useIonToast } from '@ionic/react'
import { t } from 'i18next'

interface WebPushToggleProps {}
const WebPushToggle: FC<WebPushToggleProps> = () => {
  const [webPush, setWebPush] = useState(false)

  const askForPermission = () => {
    return new Promise(function (resolve, reject) {
      const permissionResult = Promise.resolve(Notification.requestPermission())
      if (permissionResult) {
        permissionResult.then(resolve, reject)
      }
    }).then(function (permissionResult) {
      if (permissionResult !== 'granted') {
        if (permissionResult === 'denied') {
          console.log(`Permission denied, you need to unblock
            ${window.location.origin} in your browser to receive notifications.`)
        }
        setWebPush(false)
      }
    })
  }

  const checkIfPermissionGiven = async () => {
    if (navigator.permissions) {
      navigator.permissions
        .query({ name: 'notifications' })
        .then((res) => {
          console.log('push notification permission', res.state)
          if (res?.state !== 'granted') {
            askForPermission()
          }
        })
        .catch((err) => {
          console.log('push notification permission check failure', err)
          askForPermission()
        })
    }
  }

  useEffect(() => {
    console.log('webPush', webPush)
    if (webPush) checkIfPermissionGiven()
  }, [webPush]) // eslint-disable-line

  return (
    <IonItem lines='none'>
      <IonLabel>{t('Enable push notifications')}</IonLabel>
      <IonToggle checked={webPush}
        onIonChange={(e) => setWebPush(e.detail.checked)}></IonToggle>
    </IonItem>
  )
}
{% endhighlight %}

</details>
<br />

## Subscribe

<details><summary>Click here to see the code</summary>

{% highlight ts linenos %}
import TaakSDK from 'taak-sdk'
import { TaakResponse } from 'taak-sdk/dist/taak-response'
import { WebPushSubscribeCommand } from 'taak-sdk/dist/web-push/types'
import { FC, useState } from 'react'
import { osName, browserName, browserVersion } from 'react-device-detect'
import { IonButton, useIonToast } from '@ionic/react'
import { t } from 'i18next'
import { log } from '../../components/util/Log'
import { failure, success } from '../../components/util/Toast'
import { urlBase64ToUint8Array } from '../../components/util/WebPush'
import { connect } from '../../data/connect'
import { User } from '../../models/User'

interface OwnProps {
  onSuccess?: (data: any) => void
}
interface StateProps {
  webPush?: boolean
  user: User
}
interface WebPushSubscribeButtonProps extends OwnProps, StateProps {}

const WebPushSubscribeButton: FC<WebPushSubscribeButtonProps> = ({ webPush, onSuccess, user }) => {
  const [subscribing, setSubscribing] = useState(false)
  const [presentToast] = useIonToast()
  const taakClient = new TaakSDK({ apiKey: process?.env?.REACT_APP_WEB_PUSH_API_KEY || '' })

  const checkSubscribe = () => {
    setSubscribing(true)
    navigator.serviceWorker.ready.then(function (serviceWorkerRegistration) {
      // Get the push notification subscription object
      serviceWorkerRegistration.pushManager
        .getSubscription()
        .then(function (subscription: PushSubscription | null) {
          // If this is the user's first visit we need to set up
          // a subscription to push notifications
          if (!subscription) {
            subscribe()
            return
          }

          // Update the server state with the new subscription
          sendSubscriptionToServer(subscription)
        })
        .catch(function (err) {
          // Handle the error - show a notification in the GUI
          console.warn('Error during getSubscription()', err)
          setSubscribing(false)
        })
        .finally(function () {
          setSubscribing(false)
        })
    })
  }

  const subscribe = () => {
    setSubscribing(true)
    navigator.serviceWorker.ready.then(function (serviceWorkerRegistration) {
      serviceWorkerRegistration.pushManager
        .subscribe({
          userVisibleOnly: true,
          applicationServerKey: urlBase64ToUint8Array(TaakSDK.DEFAULT_WEB_PUSH_SERVER_PUBLIC_KEY),
        })
        .then(function (subscription: PushSubscription) {
          setSubscribing(false)
          // Update the server state with the new subscription
          return sendSubscriptionToServer(subscription)
        })
        .catch(function (e) {
          setSubscribing(false)
          if (Notification.permission === 'denied') {
            console.warn('Permission for Notifications was denied')
          } else {
            console.error('Unable to subscribe to push.', e)
          }
        })
    })
  }

  const sendSubscriptionToServer = async (subscription: PushSubscription) => {
    setSubscribing(true)
    log('Subscribing', JSON.stringify(subscription))
    const subscriptionObject = JSON.parse(JSON.stringify(subscription))

    const cmd: WebPushSubscribeCommand = {
      endpoint: subscriptionObject?.endpoint,
      key: subscriptionObject?.keys?.p256dh,
      auth: subscriptionObject?.keys?.auth,
      userId: user.sub,
      deviceId: `${osName} :: ${browserName} :: ${browserVersion}`,
    }
    const res: TaakResponse = await taakClient.subscribeWebPush(cmd)
    if (res.status === 201) {
      success('Success subscribing.', presentToast)
      if (!!onSuccess) onSuccess(res.data)
    } else {
      failure(`Failure ${res.status}`, presentToast)
    }
    setSubscribing(false)
  }

  return (
    <IonButton onClick={checkSubscribe} disabled={!webPush || subscribing}>
      {t('Subscribe')}
    </IonButton>
  )
}

export default connect<OwnProps, StateProps, {}>({
  mapStateToProps: (state) => ({
    webPush: state.data.webPush,
    user: state.user.user,
  }),
  component: WebPushSubscribeButton,
})

{% endhighlight %}
</details>