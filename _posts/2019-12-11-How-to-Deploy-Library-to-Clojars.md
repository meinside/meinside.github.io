---
layout: post
title: How to deploy your library to Clojars (on macOS)
tags:
- clojure
- macOS
- clojars
published: true
---

I wanted to publish my clojure library to [Clojars](https://clojars.org/),

but documents or how-tos about it were too complicated or outdated,

so I'm writing ths post for the record.

It was tested on my macOS machine.

----

## 1. Generate your gpg key,

Generate a gpg key with a passphrase,

{% highlight bash %}
$ gpg --gen-key
{% endhighlight %}

## 2. Deploy to Clojars

{% highlight bash %}
$ lein deploy clojars
{% endhighlight %}

Then you'll be asked to enter your Clojars username, password, and gpg key's passphrase.

If nothing goes wrong, you library will be uploaded to Clojars.

----

## Trouble Shooting

### 1. GPG signing fails while deploying to Clojars

If you suffer from this error: `gpg: signing failed: Inappropriate ioctl for device`,

{% highlight bash %}
$ export GPG_TTY=$(tty)
{% endhighlight %}

then run `lein deploy clojars` again.

### 2. Don't want to type username and password again and again

If you don't want to type your username and password everytime,

you can avoid it by creating `.lein/credentials.clj.gpg` file by:

{% highlight bash %}
$ vi ~/.lein/credentials.clj
{% endhighlight %}

and fill it with:

{% highlight clojure %}
{#"https://clojars.org/repo"
  {:username "your-clojars-username"
   :password "your-clojars-password"}}
{% endhighlight %}

then convert it with:

{% highlight bash %}
$ gpg --default-recipient-self -e \
      ~/.lein/credentials.clj > ~/.lein/credentials.clj.gpg

# for security, delete the original .clj file:
$ rm ~/.lein/credentials.clj 
{% endhighlight %}

Now `lein deploy clojars` will not ask your username and password.

### 3. GPG decryption fails while deploying to Clojars

If you see following kind of error:

```
gpg: decryption failed: No secret key
Could not decrypt credentials from /Users/SomeUsername/.lein/credentials.clj.gpg
gpg: decryption failed: No secret key
```

decrypt your gpg key once,

```bash
# You need to type your gpg key's passphrase
$ gpg --decrypt ~/.lein/credentials.clj.gpg
```

then retry `lein deploy clojars`.

