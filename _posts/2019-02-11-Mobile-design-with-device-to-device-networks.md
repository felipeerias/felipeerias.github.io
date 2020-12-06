---
layout: post
title: Mobile design with device-to-device networks
date: 2019-02-11
---

_On February 2019, I gave a talk on [“Mobile design with device-to-device networks”](https://fosdem.org/2019/schedule/event/device_to_device_networks/) at the Open Source Design track in the FOSDEM conference. This post is adapted from my slides and notes._

![header](/assets/img/fosdem_talk.jpg "FOSDEM 2019, Open Source Design devroom")

As part of my work with Terranet, a Swedish R&D company, I designed and created prototypes for Wi-Fi Aware and other direct connectivity technologies. Exploring this novel space gave me the opportunity to reflect about how we may find out the possibilities of a new design material.

The apps shown here run on regular Pixel 2 devices with Android.

### Introduction: direct connectivity

Direct connectivity is the ability to create networks between two or more devices without needing any other infrastructure nor Internet access.

You might already know about technologies like Bluetooth, Hostpot or Wi-Fi Direct. There is a new one, called Wi-Fi Aware, which is what I will be using for these examples. In the near future, 5G will support device-to-device connections as well.

This field is gaining relevance because these technologies are progressively becoming **fast enough, convenient enough, and flexible enough** to enable new interactions and new solutions.

### “So what is this for?”

How can we start to find out what new things can be done with this new technology?

It is like exploring a new (design) space: you don't know what might be out there, so you have to feel your way around.

My main point here is that **in order to carry out this exploration, you need to be switching continuously between the perspective of the designer and the perspective of the engineer**. You need to be observing people and understanding them, and you need to know the technology and tinker with it. You need to design solutions and prototype them. And most important of all, after each step you need to reflect on what you have learned and how that moves you forward.

(I do realize that this is still a niche field; my hope is that by showing my own explorations, you might be able to extract from them some ideas that could be useful for your own work.)

### Wi-Fi Aware

Wi-Fi Aware is the implementation of a standard called "Neighbour Aware Networking" that allows devices to discover and connect to each other directly.

How does it work? A very simple explanation is this:

* First, there is a discovery stage where the devices announce their presence. These announcements include a service ID and optionally a small amount of data.
* Second, devices can exchange small messages for coordination without needing to establish a connection.
* Third, two devices can create a direct connection between them. All connections are one to one and only a small number of them are possible at the same time.

And that's it. That's our material.

Let's play with it.

### Approaching from the engineering p.o.v.

I built a small tool that uses Wi-Fi Aware to discover other devices and connect to them, which helped me understand and test the API.

{% include vimeoPlayer.html id=418946153 %}

Each announcement contains a user ID and a name. You can see how, after the devices have detected each other, we can tap on the peer's name to create a connection.

Tinkering with this, an idea came up: if I

* connected to another device,
* copied the remote IP,
* launched a different application,
* and pasted that IP in the new app

I should be able to use Wi-Fi Aware with applications that were not created for it, right?

Well… that actually almost never works, because Wi-Fi Aware uses IP6 addresses with a scope (the address includes the name of the network interface for which that address is valid) and many apps/libraries are not able to handle them correctly.

But there is one application that works out of the box, and it is… **OpenArena**.

{% include vimeoPlayer.html id=418946270 %}

OpenArena is a game based on the Quake 3 engine, ported from the desktop to mobile. Though exploring the technology and tinkering with it, we now have a cool demo of fast multiplayer gaming.

{% include vimeoPlayer.html id=418946396 %}

### Engineering: what have we learned?

First of all, that the technology works (although the implementation is sometimes still a bit unstable).

The Wi-Fi Aware API is not too easy to use, so there's some work to do in terms of libraries and utilities. Having done this exploration is a good starting point to know what is useful and needed.

Many apps and some protocols (VLC, WebRTC) don’t seem to work, usually because of the scoped IPv6 addresses. There is work left to do in adapting these to new connectivity modes.

**Tinkering** and playing with technology can lead to valuable insights and unexpected discoveries.

Finally, there are potential privacy issues with Wi-Fi Aware:

* service announcements are public, so everybody around you will receive whatever your phone advertises
* apps can use _any_ service name, which means that they can impersonate one another

### Approaching from the design p.o.v.

Now we change perspectives and look at this space from the designer's point of view.

A design process usually consist of research, design, prototyping, testing and evaluation. In this kind of explorations, this last step of critiquing your work and learning from it is the most important one. When exploring through iterative prototyping, it helps to think of those prototypes not as early versions of some future product, but as a tool to find valuable insights, like little gold nuggets. **Those lessons are what you want to take away so they can be guidelines for your future work.**

### Interaction Design Master project (2015)

I first got in touch with the field of direct connectivity while studying Interaction Design at the University of Malmö. I did my Masters thesis (directed by Jonas Löwgren) with Terranet AB on a project to design and prototype a way to carry out presentations using mesh networks.

We started our research looking at:

* more collaborative meetings and presentations
* improving collaboration in the work context
* exploring what other possibilities open up when devices may be conected in flexible ways

After the research phase, I got several important insights:

* Presentations usually have one person talking through one set of slides: allowing the audience to **share their own content** became a main goal of the design
* But when you allow people to share their own content, the presentation becomes something different: it becomes a **collaborative medium**.
* And when people can contribute easily, testing showed that we get **more participation** and exchanges among the audience.
* Introducing a small **social choreography** (sort of like shaking hands) at the beginning of the meeting was very valuable: people tapping their phones together to create the network. This also made the concept of direct connectivity easier to explain.

This is a video of the prototype that I created.

{% include vimeoPlayer.html id=130228635 %}

A lot of the functionality in this first prototype was simulated: each device already had all the images, and they only exchanged small messages to select which one to show. Simple, but it worked well enough that I was able to carry out two presentations in front of audiences at university, which was a good way to test and demonstrate the design in a realistic setting.

### MeshPresenter project

After the master, I joined Terranet as a R&D Engineer to bring this and other prototypes to life. This is a video of its latest status:

{% include vimeoPlayer.html id=418946453 %}

The devices use a NFC tap to exchange enough information to create the network. Participants can share their own photos and PDF documents. Media files are automatically distributed among all the participants. The camera is integrated in the app, so you can take a photo and have it show up on the other devices right away. There is Chromecast support, so the content may be shown on a nearby TV as well. Drawings are updated immediately, as the user drags their finger.

{% include vimeoPlayer.html id=418946498 %}

This prototype worked very well for demonstrating and communicating the usefulness and possibilities of this technology. Because of its wide range of features, it allowed us explore different use cases without having to build separate apps: load a book in PDF and now you have collaborative reader app, load a plain background image and you get a collaborative canvas for drawing, etc. It was also a very good opportunity to test and refine the underlaying framework and tools.

{% include vimeoPlayer.html id=418946615 %}

### Design: what have we learned?

Let's take a step back and look critically at this work, so we can learn some lessons for the future.

There is a tension between prototypes being very focused on specific aspects and them being open and flexible. This one started being very focused on the presentation use case, but later on we saw that there was value in flexibility: we could try out different scenarios easily, like collaborative drawing, annotating a PDF book together, or sharing the camera.

This prototype was very good for demos and communication, but only as long as somebody knowledgeable was available to set things up. But it is not easy to get people onboard on their own. There's of course the practical matter of needing two capable devices to test it. And the mental model is very different from the way people normally use their phones.

Using body gestures can help in communicating a **mental model** for direct connectivity that is easier for people to understand. Tapping the phones helps in grounding the interaction, it gives **a reason why it only works with people nearby**, it makes it almost intimate. You and me; and everybody else is outside. This is the kind of _**little interaction nugget**_ that I mentioned at the beginning.

### Next project: AwareBeam

Building on these ideas, I created a small tool that is much more focused: it lets you share large files with a friend just by tapping the phones together.

Share. Tap. Done.

{% include vimeoPlayer.html id=418946837 %}

It is fast and quite flexible: while one transfer is going on, the next one is already being prepared.

{% include vimeoPlayer.html id=418946910 %}

And you can of course send several files at the same time.

{% include vimeoPlayer.html id=418946952 %}

### Next areas to explore in Wi-Fi Aware

In closing, I would like to mention some areas around Wi-Fi Aware where I think that there is interesting work to do, and where Free Software can play a role.

🕵️‍♀️The first one is **privacy**. As I mentioned, service announcements are public and can be easily faked, both of which pose grave threats to pricavy and security. We a free and open system in place that lets you find your friends, but prevents other people from finding you.

📽The second area is **video**. There are some pretty cool scenarios that are possible when you can share your phone's camera with a friend nearby: take remote photos, record video from multiple points of view, stream HD content without a server, etc.

🚘And the third area is the **automotive** sector: if you are able to use these technologies to detect people and cars around you, you can make a car that can see behind corners and prevent accidents.

### Implications for design

The technology for direct connectivity is “getting there”, many scenarios and solutions are becoming now possible.

At the same time, it is also needed to find and define the **concrete scenarios** where this technology makes sense. It is not enough to have some cool technology, one needs to put it the work to uncover how it can be valuable and useful for people.

There is an opportunity in creating tools that are **aware of the people around us and support us when we are collaborating with them**, in a way that can be much more context-aware and private than an Internet-based solution.

Finally, solutions have to be build on top of a simple **mental model** that helps users understand how the technology works and what are its constraints and possibilities. A good starting point for that mental model is to explore _**embodied interactions**_, like “tap to connect”.

### Exploring a new design space

The process of exploring a new design space needs to combine different points of view.

From the design point of view, one has to find real use cases, craft solutions for them, and learn from that experience. This reflection should try to find insights about the whole design space, create guidelines to support future work, and point at further directions for exploration.

From the engineering point of view, one needs to study the technology and tinker and play with it. Understand its potential and limitations. Build prototypes that are focused and functional enough to study the desired scenario, but also flexible enough to mockup up unexpected ideas.

Solutions need to be built on top of a mental model that makes the technology easy to understand, and provide clear answers to questions of usefulness (“why should I use this?”) and required knowledge (“what do I need to understand to use this?”).

This design exploration through iterative prototyping is not necesarily aimed at the creation of a concrete future product. Rather, the outcomes of this process are valuable insights, little interaction nuggets, that will guide you in your future work.

**Don't be afraid to experiment and try things out, and always remember to reflect and learn from these experiences.**

### Watch the full talk

{% include vimeoPlayer.html id=418945947 %}

[(source: FOSDEM)](https://fosdem.org/2019/schedule/event/device_to_device_networks/)
