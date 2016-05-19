---
layout: post
title: Implement a speech-to-text bot using wit.ai API
tags:
- golang
- wit.ai
published: true
---

[wit.ai](https://wit.ai/) provides APIs for natural language processing.

It takes a human voice or text and convert it to a structured information for further use.

With this API and Telegram's bot API, I'll show you the whole way of building a speech-to-text bot which will receive your voice and response with the converted text message.

----

## 0. Prerequisite

All codes in this post will be written in golang, so you may have to installi it.

Also, due to the difference of voice file formats in Telegram bot API and wit.ai API, you'll need to install [ffmpeg](https://ffmpeg.org/) for converting .ogg files to .mp3.

## 1. Install needed libraries

I will use my go libraries for [Telegram bot API](https://github.com/meinside/telegram-bot-go) and [wit.ai API](https://github.com/meinside/wit.ai-go):

{% highlight bash %}
$ go get -u github.com/meinside/telegram-bot-go
$ go get -u github.com/meinside/wit.ai-go
{% endhighlight %}

## 2. Generate your Telegram and wit.ai API tokens

You can create your own Telegram bot with [this guide](/Building-a-Telegram-bot-on-Raspberry-Pi/), and a new wit.ai application on [this page](https://wit.ai/apps/new).

![New wit.ai application](https://cloud.githubusercontent.com/assets/185988/15387890/4df7d75c-1dea-11e6-833b-498b64f6ef89.png)

Generated tokens will be used later in this post.

## 3. Get a sample code

Here's the [gist](https://gist.github.com/meinside/56f2616c61f59768b4ce487bea4f9fd5) for the sample:

{% gist 56f2616c61f59768b4ce487bea4f9fd5 %}

Download it,

{% highlight bash %}
$ wget https://gist.githubusercontent.com/meinside/56f2616c61f59768b4ce487bea4f9fd5/raw/e9984ec1ac46a47d1b7b5ccf1a178147588c3552/stt_bot_sample.go
{% endhighlight %}

and edit variables that are commented as *// XXX - Edit this value to yours*.

It will not run as expected if you don't put your **TelegramApiToken** and **WitaiApiToken** values correctly.

## 4. Build and run

Run the edited code:

{% highlight bash %}
$ go run stt_bot_sample.go
{% endhighlight %}

If nothing goes wrong, you'll be able to start chat with your bot and send your voice:

![text-to-speech](https://cloud.githubusercontent.com/assets/185988/15387893/52d8987e-1dea-11e6-961f-9fcb21f5c2d1.png)

Converted text will be returned as you see.

## 5. Step by step explanation of the sample code

### a. Bot receives a new message from you

{% highlight go %}
b.StartMonitoringUpdates(0, MonitorIntervalSeconds, func(b *bot.Bot, u bot.Update, err error) {
	if err == nil && u.HasMessage() {
		b.SendChatAction(u.Message.Chat.Id, bot.ChatActionTyping) // typing...

		if u.Message.HasVoice() { // when voice is received,
			// codes omitted here
			// ...
		}
	}
}
{% endhighlight %}

We need to convert voice to text, so check if received message has a voice in it.

### b. Bot converts received voice file into monaural .mp3 format

If a voice exists in the message, convert it with ffmpeg.

{% highlight go %}
func oggToMp3(oggFilepath string) (mp3Filepath string, err error) {
	mp3Filepath = fmt.Sprintf("%s.mp3", oggFilepath)

	// $ ffmpeg -i input.ogg -ac 1 output.mp3
	params := []string{"-i", oggFilepath, "-ac", "1", mp3Filepath}
	cmd := exec.Command("ffmpeg", params...)

	if _, err = cmd.CombinedOutput(); err != nil {
		mp3Filepath = ""
	}

	return mp3Filepath, err
}
{% endhighlight %}

Telegram bot API transfers voices in .ogg format and wit.ai API doesn't receive .ogg,

so we need to convert it to monaural .mp3 format.

### c. Bot sends a request to wit.ai's speech API and reads the result

Send a request to speech API and return the result.

{% highlight go %}
func speechToText(w *witai.Client, fileUrl string) (text string, err error) {
	var oggFilepath, mp3Filepath string

	// download .ogg,
	if oggFilepath, err = downloadFile(fileUrl); err == nil {
		// .ogg => .mp3,
		if mp3Filepath, err = oggToMp3(oggFilepath); err == nil {
			// .mp3 => text
			if result, err := w.QuerySpeechMp3(mp3Filepath, nil, "", "", 1); err == nil {
				log.Printf("> analyzed speech result: %+v\n", result)

				if result.Text != nil {
					text = fmt.Sprintf("\"%s\"", *result.Text)
				}
			}

			// delete converted file
			if err = os.Remove(mp3Filepath); err != nil {
				log.Printf("*** failed to delete converted file: %s\n", mp3Filepath)
			}
		} else {
			log.Printf("*** failed to convert .ogg to .mp3: %s\n", err)
		}

		// delete downloaded file
		if err = os.Remove(oggFilepath); err != nil {
			log.Printf("*** failed to delete downloaded file: %s\n", oggFilepath)
		}
	}

	return text, err
}
{% endhighlight %}

### d. Bot sends the converted text back to you

When the result is successfully returned, send it back through Telegram bot API.

{% highlight go %}
if message, err := speechToText(w, b.GetFileUrl(*sent.Result)); err == nil {
	if len(message) <= 0 {
		message = "Failed to analyze your voice."
	}
	if sent := b.SendMessage(u.Message.Chat.Id, &message, map[string]interface{}{}); !sent.Ok {
		log.Printf("*** failed to send message: %s\n", *sent.Description)
	}
}
{% endhighlight %}

----

## Wrap-up

wit.ai is not only for speech-to-text but also for a lot more complicated tasks like building a personal assistant using its powerful natural language processing.

It is just a simple example, so I hope whoever reads this post would build more useful things from it :-)

