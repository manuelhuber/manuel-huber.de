---
tags: [Intellij, efficiency, programming]
titleFormatted: IntelliJ - <fat>Navigate</fat> the codebase like you <fat>own</fat> it
---

> War is peace. Freedom is slavery. Editor tabs are convenient.

Like most people I used the editor tabs in IntelliJ since they are on by default and are familiar from browsers. But after watching [this talk](https://www.youtube.com/watch?v=eq3KiAH4IBI) I decided to ditch this unhealthy habit. And you can too.

## What's wrong with editor tabs?
Best case: You regularly close unused tabs and have the few you need open  
Worst case: You have a thousand open tabs and spend time looking through them  
The issue: You do this with your mouse and it slows you down.
 
If you control your editor tabs with hotkeys than you will love what else IntelliJ let's you do! 

## But I need tabs to switch between the files I'm currently working on.

There's a better alternative! It's called "Recent Files" (``Ctrl + E``):
 
![Recent Files](/assets/2018/08/recent_files.png)

It's a little popup that shows you all recently visited files. You can navigate them with the arrow keys and you can also use **text search** simply by starting to type.

``Ctrl + Shift + E`` shows you all recently **edited** files - so if you go through a bunch of files to check documentation etc you can go right back to where you were actually working!

## But tabs are quicker than looking for files in the folder structure

Yes they are, but there's an even faster way: ``Ctr + N`` opens a search bar that let's you enter class names. If you need a non-class files use ``Ctrl + Shift + N`` to search for file names. And it's a typicall IntelliJ search, so if you're looking for ``AbstractCustomerInterceptorInterfaceFactory`` you can simply type the first (few) letters of each word, like "AbCuInInFa" or just "ACIIF" and you'll find it! 

![File Name](/assets/2018/08/file_name.png)

## But I use tabs to remember files I still need to work on!
Use bookmarks! ``F11`` marks a line as a bookmark and ``Shift + F11`` opens a list of all bookmarks. This is a way "safer" way to remember places you need to edit since you can actually chose a specific line. If you're done, just remove the bookmark.  
 

## But I keep some tabs pinned because I need them a lot!
Use bookmarks again - but this time **named** bookmarks! ``Ctrl + F11`` opens a popup where you can assign the bookmark a number or letter. If you choose a number you can use ``Ctrl + bookmarkNumber`` to instantly jump to the location.  

## But it's scary
I know! When I first disabled tabs I felt lost too. The tabs are like an anchor and without it feels like you're drifting. But you believed in Santa Clause for years, you can believe in yourself long enough to make the switch! 

## But I'm faster with the mouse
That's like saying you're faster by foot than by car because you don't have a drivers licence. If you plan on going from the east coast to the west coast you'll save quite a lot of time even if you have to invest in getting your license first. Invest in yourself - use hotkeys.

## How do I disable them?
Go to ``Settings -> Editor -> General -> EditorTabs`` (or use the search function) and set `Placement` to `none`
![Editor Tabs](/assets/2018/08/editor_tabs.png)

I'm proud of you.
