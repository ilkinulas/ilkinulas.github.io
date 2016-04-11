---
layout: post
title: Free Software Licences
categories: development general
---
Global developer conference [Istanbul Tech Talks 2016](/development/technology/turkce/2016/04/02/istanbul-tech-talks.html) sponsored by [PeakGames](http://www.peakgames.net) has been held in Istanbul last week. For the ones who missed ITT2016, session videos will be available in [youtube channel](https://www.youtube.com/channel/UCzdCHjiDfrrol0QrnUY6DhQ) soon.

There were 8 great speakers talking about mobile technologies, software development best practices and (my favorite) **Free Software**. Among the 8 speakers there was a [legend](https://stallman.org/) too. Dr. Richard M. Stallman gave a talk about free software, digital freedom and privacy.

![Dr. Richard M. Stallman](/assets/freesoftware/RMS_ITT2016.jpg)

Richard Stallman's talk made me think about software licences again. As we build software we are using 3rd party libraries and we must know what each software license means and what restrictions they imply to the distribution of our own software. 

### GNU General Public License v3.0 [[description]](http://choosealicense.com/licenses/gpl-3.0/) 

The GNU GPL is the most widely used free software license and has a strong copyleft requirement. When distributing derived works, the source code of the work must be made available under the same license. 

If you are using a third party library that is licensed under GPL, you should also license your project in GPL. In 2010, a Turkish company called Argela violated this rule by mistake. They included [FFmpeg](http://ffmpeg.org/) in one of their non-GPL products and entered the **Hall of Shame** list. Fortunately they updated [their version](http://labs.argela.com.tr/wirofon/) that uses FFmpeg and licensed it under GPL.

### Apache License 2.0 [[description]](http://choosealicense.com/licenses/apache-2.0/)

Apache License is a popular and widely deployed license. All of the [Apache Software Foundation](http://www.apache.org/)'s projects including Apache HTTP server uses this license. Android operating system (except the linux kernel) is also licensed under Apache License 2.0.

If you have commercial project that you do not want to make its source open to everyone, you should look for this license in every 3rd party software you use.

Apache License has some restrictions:

* If you change any Apache licensed code, you must state so.
* You must include a copy of the license in any redistribution you may make that includes Apache licenced software.
* You can not use any marks owned by The Apache Software Foundation.

### MIT License [[description]](http://choosealicense.com/licenses/mit/)

This is the shortest and easiest to understand license. You can use MIT licensed source code in your commercial projects.

* The MIT License is a permissive license that is short and to the point. 
* It lets people do anything they want with your code as long as they provide attribution back to you
* The authors of MIT licensed software can not be hold legally responsible for any damages caused by using their software.

There are a lot of other Free and Open source software [licenses](https://en.wikipedia.org/wiki/Comparison_of_free_and_open-source_software_licenses). To respect the authors of free and open source software please know and pay attention to the software licenses.