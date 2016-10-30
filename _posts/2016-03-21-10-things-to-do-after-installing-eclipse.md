Well, I will not list exactly 10 things someone could do after installing Eclipse, but I always liked these cool titles, like *10 Things to do after installing Ubuntu* or *10 ways to boost your laptop* and decided to name my post similarly. Maybe by time it will actually be 10 things, or even keep adding and reach the even cooler *20 Things to do after installing Eclipse*!

So, let’s start working! You have already downloaded Eclipse Mars on Ubuntu 16.04 running Ambience, unzipped it in some directory, launched it and selected a proper workspace. Here are some things I usually do after I download a new version of Eclipse, in order to configure it the way I like.

1. Change *eclipse.ini*. This is the text file in Eclipse’s installation directory, which contains command-line options that are added to the command line used when Eclipse is started up. I present below an example of *eclipse.ini* file I usually use. There are some *eclipse-version-specific* values in it, that I present as variables, like *${equinox-hanlder-version}* or *${equinox-launcher-version}*, that are set by the specific eclipse version you have downloaded, and there is also the *${java-version}* variable, where you should put the path of your local Java. 

        -startup
        ${equinox-handler-version}
        --launcher.library
        ${equinox-launcher-version}
        -product
        org.eclipse.epp.package.jee.product
        --launcher.defaultAction
        openFile
        -showsplash
        org.eclipse.platform
        --launcher.XXMaxPermSize
        512m
        --launcher.defaultAction
        openFile
        -vm
        ${java-version}
        --launcher.GTK_version
        2
        --launcher.appendVmargs
        -vmargs
        -Dosgi.requiredJavaVersion=1.7
        -XX:MaxPermSize=512m
        -Xms512m
        -Xmx1024m
        -XX:+UseParallelGC

    The lines that configure the GTK version are there to handle an Ubuntu 16.04 problem that makes Eclipse extremely slow. 
1. If your team does not use the default Code Formatter, but has set its own conventions instead, you can insert your convention file by clicking: 

    **Window → Preferences → Java → Code Style → Formatter → Import → Choose file that contains your team’s conventions → Apply → OK**
1. I prefer to use Package Explorer View instead of the Project Explorer to navigate in my projects:

    **Window → Show View → Other → Package Explorer**

    You can then keep or close Project Explorer tab.
1. You can choose to use spaces instead of tabs while coding. For the Java editor this could be configured in the conventions file described above, but if it is not you can go to 

    **Window → Preferences → Java → Code Style → Formatter → Edit → Identation tab → Use tab character instead of spaces.**

    You can also edit the tab size here. For text editors:

    **Window → Preferences → Editors → Text Editors → Insert spaces for tabs**

    You can also configure Displayed tab width in this panel.
1. Insert the Terminal View: 

    **Window → Show View → Terminal**
1. Customize Javadoc author tag:

    **Window → Preferenes → Java → Code Style → Code Templates → Comments → Types → Edit → Configure the author tag as you wish → Apply → OK**
1. I prefer to configure the classic appearence theme:

    **Window → Preferences → General → Appearence → Theme**
1. Remove trailing whitespace:

    **Preferences → Java → Editor → Save actions → Perform selected actions on save → Additional actions → Configure → Code organizing tab → Remove trailing whitespace from all lines → Ok to all**
1. Change fonts. In order to change text fonts (Java, XML, HTML files etc):

    **Window → Preferences → General → Appearence → Colors and Fonts → Search for Text Font (most editor fonts link here) → Edit to choose your favourite.**

    I prefer DeJa Vu Sans Mono 9, or for low resolution monitors Andale Mono 8. You can also configure Part title font, Terminal Console Font, etc from here.
1. Configure Proxy:

    **Window → Preferences → General → Network Connect > Active Provider: Manual, HTTP/HTTPS edit to configure proxy; also make sure SOCKS is clear → Apply → OK**
1. Open pom.xml files as XML by default:

    **Window → Preferences → Maven → User Interface → Open XML page in the POM editor by default → Apply → OK**
1. Go to **Help → Eclipse Marketplace** and then find and install *Eclipse Moonrise UI Theme*. It will provide a nice dark theme for Eclipse. Then, just go to **Window → Preferences → General → Appearence** and choose *MoonRise (standalone)*. Follow the [fine tuning instructions](https://github.com/guari/eclipse-ui-theme#fine-tuning) provided in the Github project, to achive even better result.
1. Go to **Help → Eclipse Marketplace** and then find and install *Eclipse Color Theme*. It is a plugin that provides a variety of color themes for your editor panel, that can help you customize your IDE. Then go to **Window → Preferences → General → Appearence → Color Theme** and check the themes yourself. Make sure you give *Obsidian* a try!
1. There is a possibility, that when you open the *Javadoc* view, you see an ugly black background with high contrast white text font. [This is a common problem](http://askubuntu.com/questions/70599/how-to-change-tooltip-background-color-in-unity), that needs changes both on Eclipse and on Ubuntu configuration files. In order to change the *Javadoc* view's background you can go to **Window → Preferences → General → Appearence → Colors and Fonts → Java → Javadoc view background** and edit it. The default is `#f5f5b5`.

    This will change view's background, but you can not configure the Javadoc text color from inside Eclipse. In order to do so, you have to edit Ubuntu current appearence theme's configuration file - for me it is Ambience - and specifically the gtk version Eclipse uses - for me gtk2. The parameter you have to change is `tooltip_fg_color`. You can also change `tooltip_bg_color` to change the *hover* Javadoc message, that appears when you put your mouse over a Java class, in Eclipse's *Java Editor* view.

Wow! It’s actually more than 10! Hope I reach 20 then!
