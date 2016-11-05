You may have already heard *Terminator*; it is a project that extends the [classic linux terminal](https://help.ubuntu.com/community/UsingTheTerminal), in order to help you fill your screen area with consoles, as effectively as you can. If you have not already installed it, you should really give it a try! Take a look on the [project's website](http://gnometerminator.blogspot.gr/p/introduction.html), where you will find detailed installation instructions.

If you use *Terminator* in your every day life, you may have noticed that the *Dash* and *Alt-Tab* icons are not exactly high quality images. This is quite a pity, because *Terminator* **is** a great application and when you switch windows  you see the humble 48x48 icon, between all these beautiful Firefox and Eclipse icons.

Thankfully, open source world is a great place to live in and in order to change the default icon to another one, you only have to:

1. Create or download an icon you like - 256x256 is a great resolution to try - and name it `terminator.png`. Let's say that your custom *png* exists in `${HOME}/Downloads` directory.
1. Copy the custom icon to:

        cp ${HOME}/Downloads/terminator.png /usr/share/icons/hicolor/48x48/apps/terminator.png
1. Open `/usr/share/applications/terminator.desktop` and replace the line:

        Icon=terminator
    with:
        Icon=/usr/share/icons/hicolor/48x48/apps/terminator.png

Restart *Terminator* to see the effect!

![](../img/cyborg-tux-icon.png)
