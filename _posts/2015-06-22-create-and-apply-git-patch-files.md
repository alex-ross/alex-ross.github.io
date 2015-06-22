---
layout:     post
title:      "Create and apply Git patch files"
date:       2015-06-22
tags:
- git
- git patch
- git apply
---

When working with Git most of us are used to have something like GitHub or
Bitbucket where we can create pull requests and assign to people. This may not
always be the case and then it is good to have another option. With Git you can
also create a patch file which later on can be sent by email or transfered over
an USB-stick to someone that will apply it in the main repository.

Let's assume that you and me are two awesome people in a train without internet
and we have a really good project idea. We are going to write a story together using
Git.

## Let me start by creating the first chapter

I will create our project in the terminal and name it "The Pink Octopus" just as
we agreed on, right?

```bash
mkdir the-pink-ocptopus
cd the-pink-ocptopus
git init
```

I will also create our first chapter with some todos in it for you and make the
first commit.

(I'm using `cat` and not vim or any other editor just to make it as copy-and-paste
friendly as possible)

```bash
cat > chapter-1.md << THE_END
# Chapter 1

Once upon a time a pink octopus was borned.
It's favorite color was FILL_IN, but the octopus didn't know how to change to
that color.
THE_END

git add chapter-1.md
git commit -m "Initial commit"
```

Let's now pretend that I'm transferring this project to you with and USB-stick or
similar.

## You can now fill in the missing content

Create and navigate to the branch `choose-color`.

```bash
git checkout -b choose-color
```

Open the file `chapter-1.md` in your favorite editor and replace `FILL_IN` with
any color of your choice. In the examples below I guessed that you choosed
"yellow as a banana".

Commit your changes.

```bash
git commit -am "Pickes a favorite color for the octopus"
```

Now when you run `git diff master` you will get something similar to this

```
diff --git a/chapter-1.md b/chapter-1.md
index c4b6233..d85aa9d 100644
--- a/chapter-1.md
+++ b/chapter-1.md
@@ -1,5 +1,5 @@
 # Chapter 1

 Once upon a time a pink octopus was borned.
-It's favorite color was FILL_IN, but the octopus didn't know how to change to
+It's favorite color was yellow as a banana, but the octopus didn't know how to change to
 that color.
```

## Now you are ready to create your patch file

To create the patch file use this

```bash
git format-patch master --stdout > choose-color.patch
```

This will now create a file in the same directory called `choose-color.patch`.
If you skip the `--stdout > choose-color.patch`-part you will get one patch file
per commit. That may work fine in this example because we have only one commit.
Feel free to add another commit just to see what the difference will be if you
skip that part.

Let's now again pretend that you are transferring this file to me over an
USB-stick or another awesome method.

## I will now test the patch file and apply it

(Checkout master using `git checkout master` just because I guess you are doing
my part to.)

Before we apply the patch I like to check the diffstat which is kind of a
summary of what the patch will do.

```bash
git apply --stat choose-color.patch
```

This command will not apply anything. The output will be something similar to
this.

```
 chapter-1.md |    2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

Next step is to see if we will get any errors when we applying it.

```bash
git apply --check choose-color.patch
```

If I get no output I'm good to go. Lets apply this patch!

When applying the patch I will actually __not__ use `git apply choose-color.patch`.
If I do, I will only apply the changes to my git index but I will not obtain any
commit messages. Instead I will use `git am` and I will also use the `--signoff`
option so that other people can see in the history (using `git log`) that I was
the one approving this changes.

```bash
git am --signoff < choose-color.patch
```

The output will be.

```
Applying: Pickes a favorite color for the octopus
```

Now your patch file is applied and we can continue writing our story on the
train.















