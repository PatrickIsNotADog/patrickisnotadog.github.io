# How to listen cool Youtube playlists using only CLI

Back in 2007 I came to Athens as a first year Computer Engineer student, so I bought a PC with 1G RAM and an Intel Core Celeron CPU for 400 euros. Ten years later this PC is still alive, running on Lubuntu and I use it mainly to listen music, while I am doing chores in home or relaxing. Its name is *dog*, that is used metaphorically in Greece for people or things that endure in difficulties and time.

![GOT - the Hound](../img/hound.jpg)

But browsers nowadays are hungry memory monsters that try to get all the available RAM from old PCs, so I was searching a way to hear plalists from my favourite site, preferably through CLI, I am going to share here.

## Find the Youtube list URI

First download a CLI browser to start browsing the web

    sudo apt install lynx

and visit [Kasetophono](http://www.kasetophono.com); it is a cool site with playlists organized on day/mood/time/season of the year. Find your way to navigate in the site and find a list that fits in your mood and copy paste the URI.

## Download the list

Download a tool that will enable you to download Youtube songs and playlists

    sudo apt install youtube-dl

and run

    youtube-dl <playlist URI>

> You can also checkout the options the tool provides for audio format etc.

## Play the list

You can play the list using a CLI audio player

    sudo apt install cmus

Run `cmus` and press 5 to get in *navigation* menu. Select the audio file(s) you want and start enjoying.

No GUI used this far.

![cool Tux](../img/penguin-headphones.jpg)
