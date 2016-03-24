---
layout: post
title:  "Creating signed tags in Git: 'from scratch' tutorial"
date:   2012-11-27
categories: git
---


## Creating signed tags in Git: a "from scratch" tutorial

Finally, I decided to sign my "version tags" in Git. I knew about it in theory (hey, how much simpler it can be - just
issue a `git tag -s ...`, right?), but never tried it in practice, and felt into a few pitfalls. So, here goes a
"newbie guide" to Git signed tags - from nothing to having a signed tag, which can be as well easily verified by
others. Stay tuned, there are a couple of tricks!


### Creating a signed tag

As easy as it is, create a signed tag with a command:

    $ git tag -s TAG_NAME [-m OPTIONAL_DETAILED_MESSAGE]

This will create a tag and will actually use gpg to sign it. So, first it means one should have gpg installed. Next,
it means that gpg needs to have a secret key to sign with, or you need to specify the key some other way.

### Install gpg

If you don't have it yet - you will need to install a gnupg (gpg). On Linux - use whichever package manager you have.
On Windows, if you use "msysGit" - it already comes with gpg; if you use "Cygwin" - get gnupg with the help of Cygwin
setup.

### A key to sign with

No matter how you will specify which key to use (there are several ways to do that), the key needs to be issued to the
**exact same user name and user email** as those you use in Git (that is, those defined by user.name and user.email in
Git config).

If you don't have a personal key pair yet, you will need to generate the one. You can use gpg for it as well:

    $ gpg --gen-key

This is an interactive command, where gpg will prompt user name, email and password for a private key. Use same user
name and user email as those you use in Git here.

Once the key is generated, it is already installed into gpg keyring - that is it is automatically available for you
whenever you (or git on behalf of you) will sign anything.

### Finally, creating a signed tag

So, now is the time to do the

    $ git tag -s TAG_NAME [-m OPTIONAL_DETAILED_MESSAGE]

which will actually create and sign a tag, if all goes well. Which looks like this:

    $ git tag -s v2.5.1 -m "Fresh user interface. First public beta."
    You need a passphrase to unlock the secret key for
    user: "User Name "
    2048-bit RSA key, ID FFFFFFFF, created 2001-01-01

What can go wrong? Most often gpg can miss the key or reject the one you have, which looks like this:

    $ git tag -s v2.5.1 -m "Fresh user interface. First public beta."
    gpg: skipped "User Name ": secret key not available
    gpg: signing failed: secret key not available
    error: gpg failed to sign the data
    error: unable to sign the tag

This most likely means that either there is no key for this user in the gpg keyring, or, if you are certain there is,
the user name or email is not the same as those configured in Git. To check both assumptions, use this:

    $ gpg --list-keys
    ~/.gnupg/pubring.gpg
    ------------------------------------------
    pub   2048R/FFFFFFFF 2001-01-01
    uid                  User Name
    sub   2048R/E24B9143 2001-01-01

In the list of keys displayed you should normally see the key with same user name and email as if you do:

    $ git config user.name
    User Name

    $ git config user.email
    user.name@example.com


### Other ways to define the key to use

What if you don't want to use the same user name in both Git configuration, and to sign the tag? Easy - there are two
other ways to define which key to use to sign a tag: through Git config, or command line parameter to you can define
the user name or email to look for a key in the keyring. For example:

    $ git tag -u "another.name@example.com" [-m OPTIONAL_DETAILED_MESSAGE]

or

    $ git config user.signingkey another.name@example.com

Both will instruct Git to use the defined key. Sure enough, the key needs to be in the gpg keyring (listed in
`gpg --list-keys`).

### Sharing the signed tag

Git tags are not pushed to remote when you push other changes, so, you need to push them explicitly, like this:

    $ git push origin v2.5.1

With this, you now have a signed tag, which is available to all users who has the access to the repository you pushed
to. Now, we'll see how one can verify the tag signature - this is the point of signing a tag, isn't it?

### Verifying tag signature

Now then, if you pulled a signed tag you may want to verify it. Again, as in `man git-tag`, you use the following to
verify the signature:

    $ git tag -v

If this you who have just created that tag as above, it runs fine.

    $ git tag -v v2.5.1
    object d123456789c52e50983e34cab917c0b625122d4a
    type commit
    tag v2.5.1
    tagger User Name 1353867440 +0100
    Fresh user interface. First public beta.
    gpg: Signature made Sun, Nov 25, 2012  7:17:21 PM CET using RSA key ID FFFFFFFF
    gpg: Good signature from "User Name"

What happens internally, is that Git actually uses gpg to verify the signature for the specified tag. So, this means
that whoever wants to verify the tag signature - they need to have signer's public key in your gpg keyring. Otherwise,
if the signer's public key is not available to gpg, you would see this message from Git:

    gpg: Signature made Sun, Nov 25, 2012  7:17:21 PM CET using RSA key ID FFFFFFFF
    gpg: Can't check signature: public key not found
    error: could not verify the tag 'v2.5.1'

So, how the one gets signer's key?

### Sharing your public key

Let's go one step back - we've missed one point. Once you've signed a tag, you may want to share your public key with
others in order for them to be able to verify your signature. There is a plenty of ways to share the public key, but
first, you need to extract the one as a sharable file:

    $ gpg --armor --export user.name@example.com > username.pub

The command above will export your public key and, as long you redirect the output, save it to `username.pub` file.

Next, you may share the file actually. You can do so by mail, share a file in a wiki or on your personal web site.
Then, it can be also stored in Git repository itself - hey, it is the closest place to where it will be used by others.
This is why, I prefer this one.

So, you could put your public key to the Git repository as a regular file. There are few drawbacks to this solution -
not so critical ones, but there are: having your public key in the source code may be not so nice - it isn't really a
source code; and then - you don't really need version control for it - even if you will change the key, both versions
need to be easily accessed to verify old and new signatures at the same time.

So, there is another smart solution here - you can store the key as a Git object, and share it on a remote. This object
will only hold a key data, and not the file name, or a commit information - that is pure Git object, nothing else:

    $ git hash-object -w username.pub
    f4a7478fb4543...
    $ git tag username-pub-rsa f4a7478fb4543...
    $ git push --tags

That's it! You've stored a public key in a Git repository, assigned a tag to it, and shared it. It is not a part of a
commit, or any tracked file - it only exists inside Git internal objects storage and can be easily accessed using a tag
assigned to it. So, once a user pulled the remote with a signer's public key already pushed to it, it can be easily
seen in it:

    $ git tag
    username-pub-rsa
    ... other tags if any

And one can now easily extract it and add to a keyring, in order to later use for tag verification:

    $ git cat-file blob username-pub-rsa | gpg --import

That's it! Back to verification, do `git tag -v v2.5.1` and this now should work as the signer's key is in the keyring
all right.

### References

All the information in this article can be found in `git man`, `gpg man` or Git Book all right, while this article
provides kind of a workflow, which I hope will be useful for those who never signed tags in Git before. Here are some
references where you can find much more detailed information about the steps and tricks described here:

[Git Tagging](http://git-scm.com/book/en/Git-Basics-Tagging) ,
[Git Objects](http://git-scm.com/book/en/Git-Internals-Git-Objects) ,
[GPG How-To](http://www.dewinter.com/gnupg_howto/english/GPGMiniHowto-3.html)
