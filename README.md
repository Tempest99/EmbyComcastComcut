So with a lot of researching across everywhere I thought I'd post this up for any other Synology users that require live TV to have its commercials 
cut out and the final file transcoded to mkv.

I make no promises that this will work for you, and you might need to edit certain files in case they do not match your system.

### MySetup
Synology DS1513+ DSM 6.2.4-25556 Update 3

Emby Media Server

Mac's, iOS and AppleTV

All my Media is set at the route of my Synology in File Station

media/TVShows

media/TVRecordings

media/Movies etc

but the exact path to the media directory is /volume1/media/

If your not familer with Terminal/Commandline stuff this might not be that easy

Before you start you need to install the comskip and FFmpeg packages from SynoCommunity and any packages that they might require.

https://synocommunity.com
https://github.com/SynoCommunity/spksrc/wiki
https://github.com/SynoCommunity/spksrc/wiki/Comskip
https://github.com/SynoCommunity/spksrc/wiki/FAQ-FFmpeg


Create a directory called tmp in your media DIR in File Station to help keeps things a bit cleaner whilst testing this out
eg media/tmp

### Comskip .ini

my comskip.ini file I grabbed from somewhere and is (hopefully) setup for UK Freeview TV, 
obviously you will need one for your country

Comskip UK Freeview .ini forum
http://www.kaashoek.com/comskip/viewtopic.php?f=2&t=1066&p=6580&hilit=uk+freeview#p6580
Comskip .ini examples from other countries
http://www.kaashoek.com/comskip/viewforum.php?f=7

### Cutting Commercials

the comcut script is basically a modified version of what PhilWhite has created and the whole thread can be viewed here
https://emby.media/community/index.php?/topic/49900-automated-commercial-removal-from-tv-recordings/

Other useful bits from
https://github.com/BrettSheleski/comchap

### Emby Post Proccess

In Emby under Live TV on the Advanced tab under Recording Post Processing

Post-processing application: /volume1/@appstore/comskip/bin/comcut

Post-processor command line arguments: "{path}"

ssh into your Synology box (DMS6) with your username and password using Terminal or a similer app
```bash
ssh user@YourNasIPAddress
```
enter your password
then once logged in you need to become root
```bash
sudo -i
```
enter your root/admin password
```bash
sudo su
```
now you're root

### Finding your Comskip installation

Hopefully your packages are installed in volume1 on your Nas
so in Terminal just enter 
```bash
ls /volume1/@appstore/comskip
```
You should see a few directories bin, lib & var
we are only intereted in bin and var
If you don't see those direcotories or comskip your'll need to figure out which volume things are installed on and that's beyound this help

If you do see the var dir then we can move on.

```bash
cd /volume1/@appstore/comskip/var
```
So you need to edit your comskip.ini file, or create a new on, personally I find that Synology's implentation of vi or vim
is rubbish to use as an editor and it frustrates me,  
So what I tend to make a backup of the file I want to edit then create a new .ini file and paste in the contents of a file I've edited in a text editor
and any chnages I want to make I do so on my Mac in textastic or any other plain text editor.
I do this like this.
```bash
mv comskip.ini comskip.ini-bak
vi comskip.ini
```
If your not used to vi, I will give you a quic run through
To instert Text hit the letter i
If you copy the ini file from your text editor you can paste that in terminal
Once pasted, the pasted text may look a little disjointed, ignore that
To save the text, hit ESC (escape Key) followed by :w and then enter
That will save the file
ESC :q will quit vi

If you want to just check it pasted ok, you can run
```bash
cat comskip.ini
```
Once happy lets create the comcut file
```bash
cd ../bin
vi comcut
```
(same as above hit i to insert and paste in the contents of the comcut file)
ESC :w to write the file :q to quit vi
if you want to check the file
```bash
cat comcut
```

Okay, so now some permission changes
Then we need to give it the right permissions to execute (755)


I'm not sure if this is required but it seems that it works, so 
in your DSM, check your Users and Groups and see if emby is in the group users
If not add emby to your users group

Then run the following

```bash
cd ../../
chmod -R 755 comskip/bin
chown -R emby comskip
chgrp -R users comskip
```

chown (this changes everything in comskip to be owned by emby)

chgrp (this changes the group to users)

chmod (sets executable permissions on all comskip/bin files)
 


### To Test
In DSM File Explorer find a TV recording you did that should have commercials, right click and select properties

The dialog should give you a text box with the file location, select the whole location and copy it

Then back in terminal type
```bash
sh /volume1/@appstore/comskip/bin/comcut 
```
then paste in the file location in between single or double quotes in case the file location has spaces
e.g
```bash
sh /volume1/@appstore/comskip/bin/comcut '/volume1/media/RecordedTV/The Adventures of Sherlock Holmes (1979)/Season 2/The Adventures of Sherlock Holmes S02E06 The Final Problem.ts'
```
and hit enter, now go down the pub as this will take a while, it's not that bad depends upon your nas

If nothing moans when you start it and the tmp directory gets some files (viewd in your DSM File Explorer)

then you should be good to go, and when finished the tmp dir will be empty and you should have an .mkv file instead of a .ts file in the show directory.



