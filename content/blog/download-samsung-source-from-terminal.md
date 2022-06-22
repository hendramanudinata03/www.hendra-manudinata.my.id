---
title: "Downloading Samsung Source (OSRC) From Terminal"
date: 2022-06-22T10:59:22+07:00
cover:
    image: "/assets/img/download-samsung-source-from-terminal/1.png"
    alt: "Example"
    caption: "Downloading `J400FXXU9CUJ4` source (SM-J400F)"
---

# Introduction
I have a server that is intended for compilation stuff. Some of them are kernel for Samsung phones, released by Samsung itself in their [Samsung Open Source site](https://opensource.samsung.com).

As we all know, kernel sources size are mostly big. And the worse part is, Samsung doesn't use Git! Unlike Xiaomi that releases their kernel source from [their GitHub repository](https://github.com/MiCode/Xiaomi_Kernel_OpenSource), Samsung packs the source to a compressed _tarball_, and then to a ZIP file along with compilation instruction. Samsung doesn't provide direct download link either. They use POST request to retrieve the correct source.

So, what should I do before compiling? Sure thing, downloading it first **from my computer** and then uploading it into the server. Whoa, what a time-wasting! And no, Internet connection here isn't that fast. I would drink a tea first, waiting for the upload process to be completed. I won't install a GUI + web browser just to download a file as well.

Because of that, I came up with an idea to create a simple script for downloading a source from Samsung Open Source. And I succeed!

# Hello, `osrc_download`!
It's called `osrc_download`. Quite simple naming, _yeah how should I name it though?_

`osrc_download` is a simple script to download a soruce from Samsung Open Source (OSRC) site. When ran, the script will ask you to enter the search query (could be device model, source version, etc.). It will list all available sources **from the first page (10)**. Choose one, then `osrc_download` will automatically sends the required POST request to OSRC site. The site will return the choosed source file, thus allowing you to download it. Basically, just input the required questions, and then sit back nicely. `osrc_download` handles it!

# Usage
I've put installation and usage in the README. [Check it out](https://github.com/hendramanudinata03/osrc_download)!

In a nutshell, after installing the required dependencies. simply run the script:

```shell
$ python3 osrc_download.py
```

For example, downloading `J400FXXU9CUJ4` source (SM-J400F):
![Example](/assets/img/download-samsung-source-from-terminal/1.png)

# Credits
I want to thank to @fourkbomb for his [Gist](https://gist.github.com/fourkbomb/9f0aeadb5b300a4fdd23559c368d75dd) and @Linux4 for his [code component](https://github.com/Linux4/SamsungFirmwareBot/blob/master/src/main/java/de/linux4/samsungfwbot/SamsungKernelInfo.java) of [SamsungFirmwareBot](https://github.com/Linux4/SamsungFirmwareBot). I use both of those as references. Thank you!

# Conclusion
There's not much to type here. Simply check the [repository](https://github.com/hendramanudinata03/osrc_download) to find out more.

Thanks for reading!
