---
title: "Effective Notes for OSCP, CTFs and Pentests with Obsidian (2025)"
date: 2025-03-15
categories: [Articles & Writeups, InfoSec Education]
tags: [OSCP, obsidian, note-taking]
description: Having well-organized notes is crucial for penetration testing, OSCP preparation and exams, CTFs, etc. They help you quickly identify previously exploited vulnerabilities and map the interconnections between machines within a network. In this video, I'll show you how I take effective notes using Obsidian's Canvas and Excalidraw, and how I structure them.
toc: true
media_subpath: /assets/img/EffectiveNotesForOSCP/
image: ObsidianNotes.png
---

## Introduction

In 2022, I made a [video]({{site.url}}/writeups/infosec-education/taking-effective-notes-for-oscp/) explaining my note-taking methodology useful for CTFs, security certifications like the OSCP, and more generally any penetration test.

Many things have changed since the last time, including the way I think and take notes, and considering, also, the number of views the old video is getting, I decided to make a newer one.

The note-taking application I'm using is again Obsidian, this time featuring some third-party plugins that really empower most of its features, but I moved from a traditional "vertical text-based notes" to "whiteboard" ones.

Why did I decide to use whiteboards? Are you familiar with those murder boards popular in thriller films and TV series, where the investigators have everything in front of them?

The reason is the same: I wanted something that would allow me to pin down everything I need on the fly and that would allow me to keep ALL findings in view at the same time.

Having a whiteboard on which you can draw and move things around will dramatically increase the potential of your notes, allowing you to create graphs and diagrams that would otherwise not be possible with a traditional approach.

I won't go into detail about how Obsidian works as I've already talked about it in the past and the basic operation hasn't changed. Take a look at that video first in case you missed the basics, otherwise let's download the project, open it with Obsidian, and take a look to the required community plugins first.

There are several community plugins that could be useful for you and that you could perhaps integrate into your workflows. For example, I use 19 different plugins in my personal project, most of which are quality of life enhancements, but the key ones in our case are a few:

Advanced Canvas and alternatively Excalidraw, are the core of the whole mechanism and it is they who enable the implementation of whiteboards within Obsidian. Depending on your tastes and use cases, using one may be more convenient than using the other, but we will go into the differences between the two later in the video.

Then, the Kanban plugin allows us to create a board for managing tasks and ToDos using a Kanban approach.

"Paste image rename" and "Paste URL into selection" are two utilities that enhance Obsidian's pasting capabilities saving you times when pasting links or images.

Finally, "Quiet outline" enhances Obsidian's outline and table of contents with better GUI and more features. Again, this is a quality of life improvement.

OK, almost there. Before working live with the template, I wanted to digress a little more on some settings that I consider fundamental. This section of the video is not obligatory, but I still recommend you follow it if you are thinking of integrating your vault with GitHub. If not, feel free to skip to the next chapter.

The first settings we must change are "Use Wikilinks" and "New link format". Here we must disable the use of Wikilinks and set "Relative path to file" as the default behaviour. Then we have to change the setting "Default location for new attachments" to "In subfolder under current folder". The reason for these changes is to facilitate the integration and use of our notes also on other platforms, especially if we plan to use Obsidian as a cheatsheet.
Let’s take GitHub as an example. GitHub does not support the use of Wikilinks, and all files uploaded to GitHub will be hosted in the root directory of your repository.
This means that if we want to use GitHub as a backup and properly navigate our notes on the platform, we won’t be able to use the other settings in Obsidian because, in either case, the links would be broken. However, by using the combination mentioned above, we will ALWAYS be able to maintain working links suitable for the GitHub format.

I also suggest enabling all the plugins you see active right now while scrolling the list, especially the "Templates" plugin. It will allow you to create template notes inside a specific folder that you can quickly import into other Obsidian pages. In the project I uploaded for you, I’ve already created some useful templates, so by enabling this plugin, you can easily embed them into your notes using Obsidian's quick commands.

Great! Now that everything is set up correctly, let's drop a like to the video and subscribe to my channel. It would be appreciated so much, so thanks in advance, but now let's take a hands-on look at the template!

## Video
<iframe width="736" height="491" src="https://www.youtube.com/embed/4t1MvfNK8Wc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## References

- [Obsidian template - GitHub](https://github.com/0xb120/obsidian-template)
- [Obsidian template - Google Drive](https://drive.google.com/file/d/15QkvsD4GCf8KH85yGX3gXrfX-5OCK59O/view) (password is notes-4-oscp!)
