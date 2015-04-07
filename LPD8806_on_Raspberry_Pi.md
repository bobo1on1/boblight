# Compiling boblight on Raspberry Pi and LPD8806 LEDs. #

# Update #
Since http://code.google.com/p/boblight/source/detail?r=459 this should also work for WS2801 led strips.

# Introduction #

Objecive is to have a stand-alone boblight configuration without using arduino attached to a computer. Raspberry Pi is setup to receive the rendered information from [XBMC Boblight Addon](http://forum.xbmc.org/showthread.php?tid=116331) and feed the info to LPD8806 LED strand.

# Whats Needed #
> [Raspberry Pi](http://www.raspberrypi.org/)

> [LPD8806](http://adafruit.com/products/306) LEDs

> [Occidentalis Raspberry PI image](http://learn.adafruit.com/adafruit-raspberry-pi-educational-linux-distro/occidentalis-v0-dot-2) (Others might also work as long as there is SPI support)



# Wiring #
Connect as explained on adafruit.com: http://learn.adafruit.com/light-painting-with-raspberry-pi/hardware
<br>
<br>
<img src='http://boblight.googlecode.com/svn/wiki/diagram.png' />
<br>
<h1>Installation</h1>

1.<a href='http://learn.adafruit.com/adafruit-raspberry-pi-educational-linux-distro/occidentalis-v0-dot-2'>install Occidentalis v0.2</a>

2.Log into your rpi<br>
<pre><code>  $ ssh pi@raspberrypi.local<br>
</code></pre>
3. Install subversion<br>
<pre><code>  $ sudo apt-get install subversion  <br>
</code></pre>
4. Clone boblight source<br>
<br>
<pre><code>  $ cd ~<br>
  $ svn checkout http://boblight.googlecode.com/svn/trunk/ boblight-read-only<br>
</code></pre>
5. Compile:<br>
<pre><code>  $ cd boblight-read-only/<br>
  $ ./configure --without-portaudio --without-x11 --without-libusb <br>
</code></pre>
<blockquote>If everything goes well.....<br>
<pre><code>  $ make<br>
  $ sudo make install<br>
</code></pre></blockquote>

6. now configure your setup, see conf/LPD8806.conf for examples and move .conf file to /etc/boblight.conf<br>
<pre><code>  $ sudo cp conf/LPD8806.conf /etc/boblight.conf<br>
</code></pre>

7. you can test now by starting boblightd<br>
<pre><code>  $ sudo boblightd<br>
</code></pre>
<blockquote>we have to run boblightd using sudo for SPI to work</blockquote>

8. if you like, you can add boblightd to /etc/rc.local for it to start on boot<br>
<pre><code>  $ sudo nano /etc/rc.local<br>
</code></pre>
<blockquote>and add<br>
<pre><code>  /usr/local/bin/boblightd -f<br>
</code></pre>
before exit</blockquote>

9. Setup <a href='http://forum.xbmc.org/showthread.php?tid=116331'>XBMC Boblight Addon</a> by pointing it to Raspberry Pi IP address.<br>
<br>
<br>
Written by:<br>
amet