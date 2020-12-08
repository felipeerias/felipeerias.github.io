---
layout: post
title: P2P presentations
date: 2017-10-27
---


**MeshPresenter** is an Android app that explores the use of ad-hoc proximity networks to make presentations more open to collaboration. [The beta version is available here](https://play.google.com/apps/testing/se.terranet.multipresenter) [update: no longer available]. If you would like to know more about how the app came about and how it works, please keep reading.

## _"Death by PowerPoint"_

![header](/assets/img/madmenheader.jpg "header")

As Edward Tufte says, presentation software tends to be oriented towards helping presenters feel safe, rather than helping them craft valuable content that the audience can understand. This "PowerPoint" cognitive style shortens evidence and thought by forcing a single-path structure over every type of content.

Presentation software encourages the presenter to break up data and narrative into small sequential units, rather than laying them out in meaningful spatial configurations or allowing productive engagement with the audience.

Limitations in the current technology only make matters worse. Presentations often need to be preceded by a cumbersome set-up process where the presenter takes out her laptop and plugs it to a static projector (and prays that things works out). Everybody in the audience needs to be facing the surface where the presentation is projected, and they can not easily show their own content to others.

This becomes a theatrical monologue, where the roles of presenter and audience are separate and fixed. We can do better.

## IxD project

This application began as my project for the [Master in Interaction Design](https://edu.mah.se/en/Program/TAINE) at [Malmö University](https://edu.mah.se/en), thanks to a collaboration between the university and [Terranet](https://terranet.se/), a Swedish R&D company specialised in mesh networks and connectivity technology. My director was [Jonas Löwgren](http://jonas.lowgren.info/), who provided great insight and advice throughout the project

Terranet wanted to create a proof of concept for a presenter application using their technology, as well as explore opportunities for work and collaboration using ad-hoc networks without a central node.

Our goal was to turn presentations into more collaborative sessions by designing a fluid way to share and display content. We envisioned the presentation as the a result of a collaborative effort, with the presenter acting as a moderator.

I designed the main interactions and implemented an inital prototype. Most of the functionality was simulated (e.g. the images had been shared between the devices beforehand, so no file transfer was needed) but this rough prototype was enough to convey the concept.

Read the project report [here](https://dspace.mah.se/handle/2043/19439) ([alternative link](/assets/img/TP1_IDM_Felipe_Erias_FINAL.pdf)).


{% include vimeoPlayer.html id=130228635 %}

## Refinement & development

After my Master, I continued working with Terranet to develop this concept into a full mobile application.

At the same time, I took part on the development of the Terranet Connectivity framework, which enables phones to discover and connect to each other automatically without the need for Internet access. And that is how I ended up working on the whole stack, from interaction design and app development down to basic networking and communication.

### Overview

The main context of use is one where a group of people are attending a meeting, and want to share and discuss photos and documents.

MeshPresenter enhances collaboration by creating an ad-hoc network among the attendees, which enables them to share and participate in a much more flexible way. The presenter starts a new session by picking the initial image or PDF file. Then, the audience can join in with their phones in order to share their own content, and participate interactively through drawings, polls, and chat.

The app uses the Terranet Connectivity framework to provide discovery and networking, regardless of whether the phones are connected to the same WiFi network or not. We also use Google Cast to send content to a TV screen, which acts as a (smarter) projector.

{% include vimeoPlayer.html id=418958383 %}

### Joining

There are two main ways for the attendees to join the meeting. The first one to launch the application, wait for it to connect to the shared network and find the active sessions, and pick the desired one.

The second way is through NFC: tapping the presenter’s phone (or those of other members of the audience that have already joined) launchs the MeshPresenter app, which will automatically join the same session.

From an interaction design point of view, this tapping gesture is quite interesting: it is personal and physical, and helps put people in the mood of working as a group. It could become a sort of handshake, a small choreography of interaction that starts a collaborative meeting.

{% include vimeoPlayer.html id=488410833 %}

### Audience size

The session can be accommodated to the context where it is taking place. As our goal is that the presenter takes on the role of a moderator, we need to provide tools to carry out this task successfully.

By default, everybody is allowed to draw and share: this would fit well an informal meeting among a small group of colleagues. It might not fit, for example, a classroom or conference.

For those cases, the presenter can disable sharing and drawing. Every time that a person in the audience wants to share some content, she would need to send a request. The presenter can review all pending requests at her convenience (for instance, at the end of the talk) and pass control over to that person.

{% include vimeoPlayer.html id=488411176 %}

When "everybody can share" is disabled, the audience can still share their content by sending a request to the presenter. Here, the presenter receives and accepts a request.

### Sharing, drawing and more

MeshPresenter supports sharing PDF documents and images (including taking photos directly from the app).

Drawings are shared in real time, using a simple binary protocol for better performance. Updates are dispatched several times per second; for scalability, the exact rate depends on the number of online peers. The goal is to allow the speaker and the audience to call attention to features of the current content (think of a laser pointer in a traditional presentation).

{% include vimeoPlayer.html id=488411562 %}

The strokes are not permanent and will fade after a few seconds: this is both to keep the UI simple and to make the implementation easier. However, one can easily imagine a future version of the app where these strokes could be stored, enabling the participants to create a drawing together.

{% include vimeoPlayer.html id=https://vimeo.com/488411774 %}

Often, speakers carry out informal polls through a show of hands; we cover this use case by allowing the creation of polls right in the app. There is also a chat for the people attending the presentation. These are rather simple features, but they are interesting because they point the way towards more elaborate ways for the whole audience to interact with each other through the app.

The presenter may link her phone to a Google Cast screen so it acts as a projector. For this, I developed a HTML5 application that is loaded by the Cast device and which displays the current content (photo, PDF page, poll…). I have been doing some experiments with showing drawings on the TV as well, but unfortunately my old ChromeCast is not optimized for HTML5 Canvas operations and the performance was too poor.

{% include vimeoPlayer.html id=418947021 %}

### P2P networking

MeshPresenter uses the Terranet Connectivity framework, which implements automatic peer detection and network setup, automatically picking the most suitable technology available.

Although this framework has more features, I will discuss here only on those that are directly used by MeshPresenter.

The framework provides the app with the ability to detect other peers and establish a network with them. Devices discover one another via BLE and then negotiate the details of the connection. This can be an infrastructure network, an ad-hoc network created using Wi-Fi Direct or Hotspot, or Wi-Fi Aware. All of this happens transparently to the app.

The framework provides a simple API for fast message-based communications. In the case of MeshPresenter, the app uses JSON for its messages, plus a simple binary format for real time drawing updates.

An API to transfer files between peers is also provided. In MeshPresenter, each shared resource is assigned a unique identifier; whenever a new resource is shared, the peers exchange a series of messages to discover which ones already have it and can share it with the rest.

Finally, the framework includes a light HTTP server that lets the app publish files, local resources, and custom data streams. This is a simple solution which turns out to be very convenient.

MeshPresenter uses this HTTP server to send content to a Google Cast device: when the current image or PDF page changes, we create a new HTTP resource and share the URL with the Cast application, which simply updates the location of its background image.

## Future

### Security

Proximity networks may provide increased security, as neither a connection to the Internet nor a central server are required. Furthermore, the connection itself can be securely encrypted: at Terranet, we are adding support for a number of security protocols to the framework.

For increased security, we are also exploring ways to leverage the fact that peers need to be physically closer: for example, by using NFC to share a one-off key which will be used to secure the communications among attendees.

Besides its usefulness in setting up the security infrastructure, this gesture is also interesting from the interaction design point of view, as it creates a nice social choreography to start the meeting. Get together, tap phones, meeting is on.

But securing the communication channel is not enough, we also need reliable access control: we want to make it easy for friends and colleagues to join in while ensuring that others are not eavesdropping.

At the moment, MeshPresenter gives the user three options. The first one is to just let everyone join without hassle (this is the default in the beta version of the app). The second, to require that each new peer must get permission from the presenter before joining.

Finally, the third option leverages knowledge about the user's context to decide whether somebody is an expected guest or not: people who are in the user's contact list or attending the same event will join the presentation automatically, while everybody else will have to be explicitly accepted.

### Automatic sumaries

In MeshPresenter, the documents that have been shared are stored in a cache and may be accessed from the "Shared Documents" item in the app menu.

For now, this cache contains just a list of files, but I am considering ways to store as well the rich contextual data created during these collaborative sessions: who shared what and when, chat conversations, poll results, etc.

### Concept development

Besides being a fully functional way to carry out presentations, MeshPresenter is a very suggestive app that lets us easily test concepts and mock up possible applications.

For instance, it shows that this technology could be used to develop a much richer collaborative drawing app, or an app to let people annotate PDF documents together, or one where users could automatically share the photos that they take with their nearby friends, and so on.

I hope to write more about this soon 😉
