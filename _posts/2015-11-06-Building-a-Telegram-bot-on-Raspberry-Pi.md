---
layout: post
title: Building a Telegram bot on Raspberry Pi
tags:
- raspberry pi
- golang
- telegram
published: true
---

[Telegram](https://telegram.org/) provides free but extremely useful [bot API](https://core.telegram.org/bots/api).

Here are my notes about building one on my [Raspberry Pi](https://www.raspberrypi.org/) server.

----

## 0. Prerequisite

I'm quite into [Golang](https://golang.org/dl/) these days, so I made up my mind to build things with it.

So if you want to follow my notes, you need a running Raspberry Pi and Golang installed on it.

## 1. Create a bot library

I could use other developers' so many Telegram bot libraries, but I wanted to learn about the API, so I made one.

You can find my library [here](https://github.com/meinside/telegram-bot-go).

I'll use this library all through this post, so let's install it anyway:

{% highlight bash %}
$ go get -u github.com/meinside/telegram-bot-go
{% endhighlight %}

## 2. Create a bot on Telegram

It should be done in *(maybe official?)* Telegram applications.

Search for **botfather** and start conversation with it.

![search_botfather](https://cloud.githubusercontent.com/assets/185988/10966167/2062106c-83f5-11e5-8f91-8af0bcdea096.png)

Input **/newbot** or select it from the command menu and follow the instruction from the bot,

![newbot_botfather](https://cloud.githubusercontent.com/assets/185988/10962918/4c90ed86-83df-11e5-8b88-1d84ee63e135.png)

then you'll get a token like this:

![token_botfather](https://cloud.githubusercontent.com/assets/185988/10962922/4fbd9bb2-83df-11e5-9bd1-639d7cbf7468.png)

In my case, token is *143698981:AAEqRwAitUr9PYbHyCLk6w32MNHizRZ9yvk*.

You can run a bot with this token from now on.

If you search for your bot, it will be found like this:

![search_my_bot](https://cloud.githubusercontent.com/assets/185988/10966169/2338707e-83f5-11e5-83aa-445b450a4b17.png)

OK, good to go on!

## 3. Create a self-signed certificate for callbacks from Telegram

Telegram demands SSL connection on bots' webhook servers.

Fortunately, it supports self-signed certificates, so I didn't have to waste my money on it.

I had some difficulties in generating proper certificate and key file.

If I read [this page](https://core.telegram.org/bots/self-signed) carefully, it wouldn't have been so hard.

I made a simple shell script that will generate self-signed certificate and key so no one else would suffer from it:

{% highlight bash %}
#!/bin/bash

DOMAIN="my.raspberrypi.com"    # XXX - edit
EXPIRE_IN=3650  # XXX - edit

NUM_BITS=2048   # 2048 bits
C="US"
ST="New York"
L="Brooklyn"
O="Example Company"

PRIVATE_KEY="cert.key"
CERT_PEM="cert.pem"

openssl req -newkey rsa:$NUM_BITS -sha256 -nodes -keyout $PRIVATE_KEY -x509 -days $EXPIRE_IN -out $CERT_PEM -subj "/C=$C/ST=$ST/L=$L/O=$O/CN=$DOMAIN"
{% endhighlight %}

Change **DOMAIN** to your server's and run it, then you'll get your *cert.pem* and *cert.key* file.

## 4. Testing with a simple echo bot

Now let's start with a simple echo bot.

This bot will echo back received messages:

{% highlight go %}
// echo_bot.go
package main

import (
	"fmt"
	bot "github.com/meinside/telegram-bot-go"
)

const (
	ApiToken     = "143698981:AAEqRwAitUr9PYbHyCLk6w32MNHizRZ9yvk"
	WebhookHost  = "my.raspberrypi.com"
	WebhookPort  = 8443
	CertFilename = "cert.pem"
	KeyFilename  = "cert.key"
)

func main() {
	client := bot.NewClient(ApiToken)
	client.Verbose = true

	// get info about this bot
	if me := client.GetMe(); me.Ok {
		fmt.Printf("Bot information: @%s (%s)\n", me.Result.Username, me.Result.FirstName)

		// set webhook url
		if hooked := client.SetWebhook(WebhookHost, WebhookPort, CertFilename); hooked.Ok {
			fmt.Printf("SetWebhook was successful: %s\n", hooked.Description)

			// on success, start webhook server
			client.StartWebhookServerAndWait(CertFilename, KeyFilename, func(webhook bot.Webhook, success bool, err error) {
				if success {
					botMessage := webhook.Message.Text
					options := map[string]interface{}{
						"reply_to_message_id":      webhook.Message.MessageId,
					}
					if sent := client.SendMessage(webhook.Message.Chat.Id, &botMessage, &options); !sent.Ok {
						fmt.Printf("*** failed to send message: %s\n", sent.Description)
					}
				} else {
					fmt.Printf("*** error while receiving webhook (%s)\n", err.Error)
				}
			})
		} else {
			panic("failed to set webhook")
		}
	} else {
		panic("failed to get info of the bot")
	}
}
{% endhighlight %}

Save this code to a file named *echo_bot.go*.

Edit **ApiToken**, **WebhookHost**, and **WebhookPort** to yours.

Make sure that your WebhookPort is one of **80**, **88**, **443**, or **8443**.

Now try running it:

{% highlight bash %}
$ go run echo_bot.go
{% endhighlight %}

Whenever you send messages to this bot, it will send them back to you:

![echo_bot](https://cloud.githubusercontent.com/assets/185988/10966172/257b1a08-83f5-11e5-946d-e4ec1714a473.png)

Echo bot is good for testing, but not enough for Raspberry Pi, so let's add some more features on it.

## 5. Creating a bot for Raspberry Pi

I want the bot to function as a server-status checker.

For example, it should return the uptime of Raspberry Pi when I input **/uptime** command and so on.

Let's add following code to the previous bot code:

{% highlight go %}
import "os/exec"

const (
	DefaultMessage = "Your command, master?"
	CommandUptime  = "/uptime"
	CommandDate    = "/date"
)

func getUptime() string {
	if output, err := exec.Command("uptime", "-p").CombinedOutput(); err == nil {
		return string(output)
	}
	return "failed to get uptime"
}

func getDate() string {
	if output, err := exec.Command("date").CombinedOutput(); err == nil {
		return string(output)
	}
	return "failed to get date"
}
{% endhighlight %}

and alter several lines in main():

{% highlight go %}
botMessage := DefaultMessage

switch webhook.Message.Text {
case CommandUptime:
	botMessage = getUptime()
case CommandDate:
	botMessage = getDate()
}
options := map[string]interface{}{
	"reply_markup": bot.ReplyKeyboardMarkup{
		Keyboard: [][]string{
			[]string{CommandUptime, CommandDate},
		},
	},
}
{% endhighlight %}

Here is the full code for your convenience:

{% highlight go %}
// cmd_bot.go
package main

import (
	"fmt"
	"os/exec"

	bot "github.com/meinside/telegram-bot-go"
)

const (
	ApiToken     = "143698981:AAEqRwAitUr9PYbHyCLk6w32MNHizRZ9yvk"
	WebhookHost  = "my.raspberrypi.com"
	WebhookPort  = 8443
	CertFilename = "cert.pem"
	KeyFilename  = "cert.key"
)

const (
	DefaultMessage = "Your command, master?"
	CommandUptime  = "/uptime"
	CommandDate    = "/date"
)

func getUptime() string {
	if output, err := exec.Command("uptime", "-p").CombinedOutput(); err == nil {
		return string(output)
	}
	return "failed to get uptime"
}

func getDate() string {
	if output, err := exec.Command("date").CombinedOutput(); err == nil {
		return string(output)
	}
	return "failed to get date"
}

func main() {
	client := bot.NewClient(ApiToken)
	client.Verbose = true

	// get info about this bot
	if me := client.GetMe(); me.Ok {
		fmt.Printf("Bot information: @%s (%s)\n", me.Result.Username, me.Result.FirstName)

		// set webhook url
		if hooked := client.SetWebhook(WebhookHost, WebhookPort, CertFilename); hooked.Ok {
			fmt.Printf("SetWebhook was successful: %s\n", hooked.Description)

			// on success, start webhook server
			client.StartWebhookServerAndWait(CertFilename, KeyFilename, func(webhook bot.Webhook, success bool, err error) {
				if success {
					botMessage := DefaultMessage

					switch webhook.Message.Text {
					case CommandUptime:
						botMessage = getUptime()
					case CommandDate:
						botMessage = getDate()
					}
					options := map[string]interface{}{
						"reply_markup": bot.ReplyKeyboardMarkup{
							Keyboard: [][]string{
								[]string{CommandUptime, CommandDate},
							},
						},
					}
					if sent := client.SendMessage(webhook.Message.Chat.Id, &botMessage, &options); !sent.Ok {
						fmt.Printf("*** failed to send message: %s\n", sent.Description)
					}
				} else {
					fmt.Printf("*** error while receiving webhook (%s)\n", err.Error)
				}
			})
		} else {
			panic("failed to set webhook")
		}
	} else {
		panic("failed to get info of the bot")
	}
}
{% endhighlight %}

When starting chat with this bot, you will be asked to input commands:

![cmd_bot](https://cloud.githubusercontent.com/assets/185988/10966176/2861915c-83f5-11e5-91a6-ed2aa409db51.png)

then it will respond to each of them as you select.

## 6. Trouble shooting

### Is your bot not working as expected?

Check your Raspberry Pi's network connection.

Is the port(80, 88, 443, or 8443) open to public? If not, open it.

### Do you see this kind of message repeated again and again?

**2015/11/05 18:59:24 http: TLS handshake error from 149.154.167.200:38869: remote error: unknown certificate**

Check your certificate. It could happen if your domain does not match.

## 7. Wrap-up

With Telegram's bot API, so many things can be achieved easily.

If you add some more code, you could control not only your Raspberry Pi but all of your home appliances remotely.

It can be a nice entry point to the so-called 'IoT'.


I love Telegram. It is the best IM platform in the world :-)

