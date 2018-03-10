---
title: Publishing from the iCloud
date: 2012-08-02
comments: true
categories: [octopress, osx]
keywords: "octopress, osx, mountain lion, iCloud"

aliases: 
  - /blog/2012/08/02/publishing-from-the-icloud/
  - /2012/08/02/publishing-from-the-icloud/
---

I've recently been using IA Writer as my markdown editor.  I love the fact that I can use any of my iDevices and that it's all synced in the iCloud.  But how do I access the iCloud data so that I can include it in my Octopress git repository?

<!-- more -->

## Show me the data

On Mountain Lion (and Lion I believe) all the iCloud data is hidden away in your home directory.  Each application is given it's own area just like iOS apps.  Check out `~/Library/Mobile Documents`, you'll see folders for each iCloud application that you have launched.  Your files are stored in here.

Once you have found your editors data folder we can now create a link from here into your git repository.  Unfortunately, symbolic links are not good enough for either iCloud or git so we'll need to use hard links.  Mountain Lion does not ship with this tool but let's not worry, it's very easy to make our own.

> be careful, deleting from a hard link deletes from the source!

## Creating hard links

So, at the end of this post is the complete source code to create hard links.  Copy the code into a new file named `hlink.c` and compile it.

``` bash
$ gcc hlink.c -o hlink
```

Now we can create a hard link to link our blog posts into our iCloud documents.

``` bash
$ hlink ~/<octopress_dir>/source/_posts ~/Library/Mobile\ Documents/<app_dir>/Documents/pages
```

## What about DropBox?

I did a quick comparison between the iCloud integration and DropBox.  Unfortunately, DropBox seemed to be more manual regarding the syncing.  Also, if you lose your network connection (happens to me often whilst on a train) the DropBox integration moves the document to local storage and you have to manually copy back into DropBox - not a nice feature.  iCloud seems to handle this with ease and I don't have to install anything to get it to work.

* source code for hlink

``` c
#include <unistd.h>
#include <stdio.h>

int main(int argc, char* argv[]) {
  if (argc != 3) {
    fprintf(stderr, "Use: hlink <src_dir> <target_dir>\n");
    return 1;
  }

  int ret = link(argv[1], argv[2]);
  if (ret != 0) perror("link");
  return ret;
}
```
