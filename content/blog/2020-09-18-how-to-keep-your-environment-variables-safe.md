---
title: How to keep your environment variables safe
date: 2020-09-18T00:31:21.672Z
description: With this post you will learn how to store your environment
  variables in the keychain, keep your secrets hidden from prying eyes.
---


![Security: https://unsplash.com/photos/8FxJi5wuwKc](/img/photo-1485230405346-71acb9518d9c.jpeg)

# Benefits of using the Keychain

macOS has this thing called the *Keychain* and *iCloud Keychain* (the only difference between them is that one syncs with your iCloud account and the other doesn't), the Keychain is secure by design, your computer has to be unlocked and it's encrypted using device-based encryption, it also asks you for administrator and Keychain password when reading from it.

We are going to leverage the security of the Keychain to store our sensitive environment variables to keep them safe at rest and from remote attacks (mostly).

Another benefit of using the Keychain is that you get a GUI to manage your secrets, sadly there's no way to sync or use the iCloud Keychain with the \`security\` binary you're going to learn about in this post.

Now, to my knowledge there's no easy way for you to use pin entry in the mac through an SSH session, this means that it should be impossible (or extremely hard) for an attacker to see your sensitive variables, even with SSH access since pin entry uses a GUI and it won't popup in a remote shell (duh!) so echoing them out won't expose them.

![](/img/security-ftw.jpg "Where's my precious secret? safe at my Keychain of course.")

Pin entry won't appear in a remote shell so the attacker won't be able to populate your environment variables making it hard for them to just dump them (well they can dump them but all they will find are going to be empty values)

![](/img/pinentry.jpg "No input for you Hackerman")

# Multiple Keychains

You can also leverage multiple local Keychains, for example I have one for my personal credentials and one for my work credentials, they have different passwords and will make it just a tad more difficult for an attacker to break it and of course gaining access to one of the keychains doesn't mean that they have access to the other one, plus I can use the multiple Keychains as "namespaces" of sorts for my accounts like my personal AWS credentials and my AWS work credentials. 

# How to actually use it

Right, so you've read all this and I haven't actually taught you how to use this, you only have to know three commands, that's it I swear.

## Creating a new Keychain (optional)

So, I recommend you create different Keychains but if you don't care about my advice then you can just use the default one but I'll have to teach the people that do care about my advice about how to create a new Keychain, so:

```shell
security create-keychain -P <keychain-name>
```

This will prompt you for a password and that's it! you now have a new keychain to store all your tasty secrets.

## Creating a new secret

Now, to add a secret to the keychain, you can either do it manually through the GUI (Keychain Access in the launchpad) or you can store it through your shell.

**Pro-tip:**

\    If you prepend your command with a space it won't be saved to your shell history.

```shell
 security add-generic-password -a "$USER" -s <my-secret-name> -w <my-actual-secret> <keychain-name>
```

So the command should look something like this in real-life (omit the keychain-name to use the default one)

`security add-generic-password -a "$USER" -s "AWS_ACCESS_KEY_ID" -w "AKIAXXXXXXXX" work-keychain`

Remember to prepend the command with a space or to add it through the GUI so that it's not stored in your shell history, this would defeat the whole purpose of using the Keychain if an attacker can just go through your plaintext shell history.

## Retrieving a secret

Pretty much the same as storing them but without the `-a "$USER"` portion of the command.

```shell
security find-generic-password -s <my-secret-name> -w <keychain-name>
```

Retrieving the example we created above would look something like this:

`security find-generic-password -s "AWS_ACCESS_KEY_ID" -w work-keychain`

And that command will return your secret from your keychain.

# All Together

So, how can this be leveraged to secure your local environment variables? simple, just retrieve the secrets in your shell rc file, the same thing would work for zsh and bash, just use the file that corresponds to your shell.

**\~/.bashrc OR \~/.zshrc**

```shell
export AWS_ACCESS_KEY_ID=$(security find-generic-password -s "AWS_ACCESS_KEY_ID" -w work-keychain)
```

Now, every time you open a new shell you'll get a prompt asking you for your keychain password (twice, you can press always allow so that it only popups once)

And when you input your password you will populate your environment variables, simple right?

# Final Notes

So, after reading this you should now be able to store your environment variables safely in your macOS keychain and prevent an attacker from reading them if they gain SSH access and even if they have full-access to your machine it should provide an additional layer of protection since they would have to crack the encryption on the Keychain files to read your secrets.

If you prefer [1Password](https://1password.com/) also has a [CLI](https://1password.com/downloads/command-line/) utility that kind of works like the keychain (but you can sync your passwords with 1pass!) you basically have the same benefits plus the data is synced between all your devices.

Thanks for taking the time to read this and I hope you find it useful.