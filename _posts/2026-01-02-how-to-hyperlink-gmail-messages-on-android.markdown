---
layout: post
title:  "How to Create Universal Gmail Hyperlinks for the Desktop and Android"
date:   2026-01-02 14:34:00
categories: google gmail android
excerpt_separator: <!--more-->
---

My personal to-do list system often requires linking directly to specific Gmail threads for context. While copying a link from your browser’s address bar seems like the easiest method, it often fails in practice.

The Problem with Standard Links:

  * Fragility: Desktop links often break if the message status changes (e.g., moving from Inbox to an archive).
  * Incompatibility: Standard desktop Gmail links typically do not open correctly within the Android Gmail app, and vice versa.

Fortunately, there is a simple workaround to ensure your links remain functional across all devices.

<!--more-->

## Robust Desktop Gmail Hyperlinks

The secret to a "permanent" link is to ensure that it points to the "#all" folder rather than a less inclusive specific folder or search result. Use the following logic to clean your Gmail message URLs:

  * Folders: Change e.g. .../#inbox/ to .../#all/
  * Labels: Change e.g. .../#label/Next+Action/ to .../#all/
  * Categories: Change e.g. .../#category/promotions/ to .../#all/

The corrected URLs will be able to access the messages, even if they change state.

## Making the Link Work on Android

If you try one of these desktop links on Android, it probably will not work. Chrome will forward the link to the Gmail app, and the app won't understand it, and instead open the Inbox.

To keep Gmail handling within Chrome, turn off automatic forwarding to the Gmail app. On your phone, go to "Settings -> Apps -> Gmail -> Open by default", and sekect "In your browser".

Alternatively, use a different browser as your default. Netscape opens Gmail links in the browser by default.

Enjoy.
