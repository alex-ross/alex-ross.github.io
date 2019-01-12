---
layout:     post
title:      "Make all new commits verified at GitHub"
date:       2019-01-12
---

_A guide to add GPG signatures on all your git commits. Expects that you already have git and HomeBrew installed on your macOS machine._

When you create a git commit you are free to set any author. For example, let say you would like to create a git commit and  make it look like DHH, the creator of [Ruby on Rails](https://rubyonrails.org) and founder of [Basecamp](https://basecamp.com), did the commit. The only thing you need to do is to set the author like this.

```bash
$ git commit --author="dhh <david@basecamp.com>"
```

So you can't actually be sure who did commit the code. A solution is to GPG sign the commits to add a verified symbol on GitHub that the commit is authorized by you.

A signed commit will look like this on GitHub.

![Image of signed commit from DHH](https://dsc.cloud/aross/Screenshot-2019-01-12-at-15.05.01-ChEdIEJ3F8UXcYZiWs5LTRpKqFGp745VJFtm5fCgFnveSqOEBQusR2LWx6tfG7Z9ZoefoRwbzh3HfXlxYRqVyakEXkwncVHpgyge.png)

An unsigned commit will look like this on GitHub.

![Image of unsigned commit from DHH](https://dsc.cloud/aross/Screenshot-2019-01-12-at-15.05.07-lESI8EJ6Eu15AHR3TZk1sYjToFFyEInkUOJGo6UTgfSq5tZ7rD3xAwLzHKCi2pIsqopYBE3Qob8ySfJcLOo0jpY9zpdtaiZE8qJx.png)

## Generate GPG key

You need GnuPG to get started. Lets install it using HomeBrew.

```bash
$ brew install gnupg
```

Run the command below and enter your name when asked to. And when asked for your email address, make sure you set the same as the one you use with git.

Make sure you set a good pass phrase and remember it or store it in a password manager like 1Password and LastPass if you have to write it down anywhere.

```bash
$ gpg --default-new-key-algo rsa4096 --gen-key
```

## Export public key and add to GitHub

First you need to know your key-id. Run the following.

```bash
$ gpg --list-secret-keys --keyid-format LONG
/Users/foobar/.gnupg/pubring.kbx
------------------------------
sec   rsa4096/3AA5C34371567BD2 2019-01-12 [SC] [expires: 2021-01-11]
      XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
uid                 My Name <foobar@example.com>
```

In the example above the key-id is `3AA5C34371567BD2`. Copy your public key using command below, replace the `<key-id>` with your key-id.

```bash
$ gpg --armor --export <key-id> | pbcopy
```

The public key will know be in your clipboard and you can paste it using `⌘v`. So lets go to your SSH and GPG keys at GitHub.

Either you click at this link [https://github.com/settings/keys](https://github.com/settings/keys) or you navigate to GitHub, click at your profile image at the top right corner, click at settings and then SSH and GPG keys in the left menu.

Click at "New GPG key". Paste your public key and confirm.

## Use pinentry-mac to remember pass phrase in macOS Keychain

The pass phrase should be secure and hard to guess and it would be great if you don't have to remember it in your head. So let make sure the macOS Keychain will remember it for you.

Install pinentry-mac.

```bash
$ brew install pinentry-mac
```

Connect GPG agent to macOS Keychain via pinentry. Storing it in Keychain via pinentry will allow us to setup automatically key signing.

```bash
$ mkdir -p ~/.gnupg
$ echo "pinentry-program /usr/local/bin/pinentry-mac" > ~/.gnupg/gpg-agent.conf
```

## Configure git to automatically gpgsign your commits

Get your key id just as we did at "Export public key and add to GitHub". Then run the following in the terminal. Replace the `<key-id>` with your key-id.

```bash
$ git config --global user.signingkey <key-id>
$ git config --global commit.gpgsign true
$ git config --global tag.forceSignAnnotated true
```

## Try it out

If you had GPG agent installed before we recommend that you kill it to make sure it will restart with new settings.

```bash
$ killall gpg-agent
```

Create a test git directory

```bash
$ mkdir gittest
$ cd gittest
$ git init
$ echo "test" > test.txt
$ git add test.txt
$ git commit -m "test"
```

First time you commit a pine entry popup will appear and ask for the pass phrase. Make sure you also save it in Keychain so that you don't need to type in pass phrase every time.

![Image of popup window from Pinentry Mac](https://dsc.cloud/aross/Screenshot-2019-01-12-at-16.10.48-XO9aFzqMZIkLwr5SAeAByzQqSCzFInRQleiACDYv1wI7QGi5AYezLHFuwdhVFnZEUMFEsrX6qvnaIMmYpQUedLzQY23hDqtcE3PU.png)

If you create and push the repository to GitHub you will now see that it contains a verified commit.

![Image of verified test commit](https://dsc.cloud/aross/Screenshot-2019-01-12-at-16.19.24-5yjz5AknDg7daHt38lAc83QD7pyPQvI5v5lIJWXsherJS6oT2ugj7DLMq2GKoIjGVkKuPmbV6GexzAX8f4dzezftx1cTY99aEOBo.png)
