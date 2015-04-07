# Introduction #

[boblightd](boblightd.md) needs a config file so it knows what to do at startup, the default file it tries to open is /etc/boblight.conf.

# Details #

boblight.conf has 4 different type of sections: [[global](boblightconf#global.md)], [[device](boblightconf#device.md)], [[color](boblightconf#color.md)] and [[light](boblightconf#light.md)].

## global ##
A `[global]` section has two settings: interface and port.<br>
You can specify as many global settings as you want, although there's no point in having more than one.<br>
<br>
Interface specifies where to bind the listening tcp socket, if interface is not specified boblightd will bind to all interfaces (INADDR_ANY) by default, for safety reasons you should bind to interface 127.0.0.1 if you don't plan to use boblight over a network.<br>
<br>
Port specifies to which port the tcp socket will bind (who would have guessed), if no port is specified boblightd will bind to port 19333 by default.<br>
<br>
Example <code>[global]</code> section:<br>
<pre><code>[global]<br>
interface 127.0.0.1<br>
port      19333<br>
</code></pre>

<h2>device</h2>

A <code>[device]</code> section specifies a device for boblightd, it has the following settings:<br>

<ul><li>name<br>Specifies the name of the device, this setting is mandatory and must be unique (you can't have two devices with the same name).</li></ul>

<ul><li>type<br>Specifies the type of the device, this setting is mandatory.<br>Possible values are ltbl, momo, atmo, sound, popen, dioder, karate, ibelight, sedu, lpd8806 and ws2801.</li></ul>

<ul><li>output<br>Specifies where the device has its output, this setting is mandatory except for ibelight devices.<br>For ltbl, momo, atmo, dioder and karate devices this is a serial port (/dev/ttyS0), for sound devices this is api:device (ALSA:default), for popen devices this is something that can be executed (dd of=/dev/null), for lpd8806 and ws2801 devices, this is the spi device node (/dev/spidev0.0).<br>This line is parsed from the first whitespace after the word output to the end of the line, including any comments and trailing whitespace!</li></ul>

<ul><li>channels<br>Specifies the number of channels the device has, this setting is mandatory.</li></ul>

<ul><li>interval<br>Specifies the update interval in microseconds for all device types except sound, this setting is mandatory for those devices.<br>A good value seems to be 20000 to update 50 times per second.<br>Clients can also request updates of the devices when they send light input so this value doesn't have to be very low to get good synchronization.</li></ul>

<ul><li>prefix<br>Specifies a prefix in hex sent before each burst of rgb information, this setting is optional and only applies to momo devices.</li></ul>

<ul><li>postfix<br>Specifies a postfix in hex sent after each burst of rgb information, this setting is optional and only applies to momo devices.</li></ul>

<ul><li>rate<br>Specifies the baudrate for momo, atmo, ltbl, dioder, karate, SEDU, lpd8806 and ws2801 devices, specifies the samplerate for sound devices. this setting is mandatory for those devices.</li></ul>

<ul><li>bits<br>Specifies the number of bits per channel, this only applies to momo, atmo and karate devices, this setting is optional, the default is 8.<br>Boblight will only send the number of bytes needed to hold the number of bits, in big endian format, so for 1 to 8 bits it will send one byte, for 9 to 16 it will send two etc.</li></ul>

<ul><li>max<br>This is the same as the bits value, except you specify the maximum output value, so that you're not limited to bits^2-1.<br>This setting is optional, the default is 255.</li></ul>

<ul><li>period<br>Specifies the period in samples for sound devices, in other words how many samples are written per iteration, this setting is mandatory.<br>A good value for a samplerate of 48000 is 1024, this updates the output 46.875 times per second.</li></ul>

<ul><li>latency<br>Specifies the latency for sound devices in microseconds, this setting is optional.<br>If this is not specified the default portaudio low output latency is used (whatever that is, it seems pretty low anyway).</li></ul>

<ul><li>allowsync<br>Allows sync mode when a client requests it, this setting is optional, the default is on.<br>When sync mode is on, a device gets woken up immediately when a client sends input to a light that that device is using, this works quite well with clients like boblight-X11 and boblight-v4l because the light updates are exactly synchronized to the client's input.<br>However when a client updates too frequently or too many clients are using one device, the device's output can become constipated and all hell will break loose.<br>Sync mode can also be disabled in boblight-X11 or boblight-v4l by using the "-y off "flags.<br>Sync mode doesn't work with sound devices, they run synchronized to the soundcard and can't be woken up early.<br></li></ul>

<ul><li>debug<br>Turns on debug mode for the device, this setting is optional, the default is off.<br>For momo, atmo, dioder, ltbl and karate devices any data that is written to the serial port or read from the serial port is printed as hex to stdout, for popen devices the float data is written to stdout. For ibelight devices it prints the data sent and received over usb, and it sets the libusb debug level to 3.<br></li></ul>

<ul><li>delayafteropen<br>Delays writing to the device for n microseconds after opening, this setting is optional and only applies to momo, atmo, karate, dioder, popen and LTBL devices, the default is 0 (no delay).<br>Useful for arduino devices who get reset after opening the serial port for the first time (because DTR is driven low), a value of 1000000 for a 1 second delay seems to work ok.<br></li></ul>

<ul><li>bus<br>Specifies the usb bus number of the device, this setting is optional and only applies to ibelight devices, when this setting is not present, boblightd will use the first ibelight device it finds, if the address setting it present it will only use a devices with that address.</li></ul>

<ul><li>address<br>Specifies the usb address of the device, this setting is optional and only applies to ibelight devices, when this setting is not present, boblightd will use the first ibelight device it finds, if the bus setting is present it will only use a device with that bus number.<br><br>The bus and address settings can be used together to select a specific ibelight device, this is useful if you have multiple ibelight devices, if you don't know the bus and address of the device, just leave them out, boblightd will print any ibelight devices it finds to its logfile, after that you can enter them in boblight.conf and restart boblightd.</li></ul>

<ul><li>threadpriority<br>When this is set, it sets the scheduler of the device thread to SCHED_FIFO, and the number after it specifies the priority of the thread. The minimum and maximum values are determined by sched_get_priority_min() and sched_get_priority_max(), usual values for linux are 1 and 99. Setting this value will improve timing of the device thread.<br>It is highly recommended that you set this to 99 when using a ws2801 device, because the ws2801 latches in data when the clock pin has been low for more than 500 microseconds, which might cause flicker when the boblightd device thread gets preempted during an spi write.<br><br>In order to use this, the user you run boblightd as must have permissions to use SCHED_FIFO priority, the easiest way to accomplish this is to run boblightd as root, but this is not recommended (however it's convenient for testing).<br><br>If you want to give just your user SCHED_FIFO permissions, you can add this line to /etc/security/limits.conf:<br>user   -  rtprio     99<br>However this only works for PAM login shells, it will not work in a boot script.<br><br>If you want to use SCHED_FIFO in boblightd, and run boblightd as a regular user, you can use this in your boot script:<br>ulimit -r 99<br>sudo -u user /usr/local/bin/boblightd<br>For this to work the boot script has to run as root.</li></ul>

Example <code>[device]</code> sections:<br>
<br>
<pre><code>#ltbl device with 3 channels on /dev/ttyUSB0, baudrate of 115k2 and updated 50 times per second (20000 microseconds, 1/50 seconds).<br>
<br>
[device]<br>
name 		device1<br>
output		/dev/ttyUSB0<br>
channels	3<br>
type		ltbl<br>
interval	20000<br>
rate            115200<br>
allowsync       on<br>
debug           off #turn this on to see what it's doing with the serial port<br>
</code></pre>

<pre><code>#momo device with 12 channels, FF prefix and 19k2 baudrate<br>
<br>
[device]<br>
name 		device1<br>
output		/dev/ttyACM0<br>
channels	12<br>
type		momo<br>
interval	20000<br>
prefix          FF<br>
rate            19200<br>
</code></pre>

<pre><code>#popen device dumping everything to /dev/null<br>
<br>
[device]<br>
name 		device1<br>
output		dd bs=1 &gt; /dev/null 2&gt;&amp;1<br>
channels	6<br>
type		popen<br>
interval	20000<br>
</code></pre>

<pre><code>#sound device using the default alsa output<br>
<br>
[device]<br>
name		device1<br>
output		ALSA:default<br>
channels	6<br>
type		sound<br>
latency		20000<br>
period		1024<br>
rate		48000<br>
</code></pre>

<pre><code>#ltbl device with 3 channels and 115k2 baudrate<br>
<br>
[device]<br>
name 		device1<br>
output		/dev/ttyUSB0<br>
channels	3<br>
type		ltbl<br>
interval	20000<br>
rate            115200<br>
</code></pre>

<pre><code>#dioder device from http://cauldrondevelopment.com/blog/2009/12/29/a-real-ikea-dioder-hack/<br>
<br>
[device]<br>
name		device1<br>
output		/dev/ttyUSB0<br>
channels	3<br>
type		dioder<br>
interval	20000<br>
rate		38400<br>
</code></pre>

<h2>color</h2>
A <code>[color]</code> section specifies a color for boblightd, it has 5 settings:<br>
<br>
<ul><li>name<br>Specifies the name of the color, this setting is mandatory and must be unique (you can't have two colors with the same name.</li></ul>

<ul><li>rgb<br>Specifies the rgb value of this color, this setting is mandatory, it's in RRGGBB hex format.</li></ul>

<ul><li>gamma<br>Specifies the gamma for this color, this setting is optional, the default is 1.0.</li></ul>

<ul><li>adjust<br>Specifies the adjust for this color, this setting is optional, the default is 1.0.<br>This is a multiplier for the channel this color is on, the ranges goes from 0.0 to 1.0.</li></ul>

<ul><li>blacklevel<br>Specifies the blacklevel for this color, this setting is optional, the default is 0.0.<br>This setting is a minimum value for the channel this color is on, the range goes from 0.0 to 1.0.</li></ul>

Example color sections:<br>
<br>
<pre><code>[color]<br>
name		red<br>
rgb		FF0000<br>
<br>
[color]<br>
name		green<br>
rgb		00FF00<br>
<br>
[color]<br>
name		blue<br>
rgb		0000FF<br>
<br>
[color]<br>
name		yellow<br>
rgb		FFFF00<br>
adjust		0.5<br>
blacklevel	0.1<br>
gamma		2.3<br>
<br>
[color]<br>
name		white<br>
rgb		FFFFFF<br>
adjust		0.3<br>
blacklevel	0.7<br>
gamma		1.6<br>
</code></pre>

<h2>light</h2>

A <code>[light]</code> section specifies a light for boblightd, it has 4 settings:<br>
<br>
<ul><li>name<br>Specifies the name of this light, this setting is mandatory and must be unique (you can't have two lights with the same name).</li></ul>

<ul><li>color<br>Specifies a color on this light, this setting is optional, you can specify as many colors as this light has.<br>The synax is color colorname devicename devicechannel.<br>Example: color red device1 1</li></ul>

<ul><li>hscan<br>Specifies the horizontal scanrange for this light, this setting is optional, the default is 0.0 100.0<br>This setting only has an effect for clients that react to an image like boblight-X11 and boblight-v4l. The syntax is hscan begin end.<br>Begin and end specify the percentage of the horizontal range the light should react to. Example: hscan 0.0 50.0</li></ul>

<ul><li>vscan<br>Specifies the vertical scanrange for this light, same as hscan.</li></ul>

Example light sections:<br>
<br>
<pre><code>[light]<br>
name		right<br>
color		red 	device1 1<br>
color		green 	device1 2<br>
color		blue 	device1 3<br>
hscan		50 100<br>
vscan		0 100<br>
<br>
[light]<br>
name		left<br>
color		red     device1 4<br>
color		green   device1 5<br>
color		blue    device1 6<br>
hscan		0 50<br>
vscan		0 100<br>
<br>
[light]<br>
name		center<br>
color		red     device1 7<br>
color		green   device1 8<br>
color		blue    device1 9<br>
hscan		33.3 66.6<br>
vscan		33.3 66.6<br>
<br>
[light]<br>
name		top<br>
color		red     device1 7<br>
color		green   device1 8<br>
color		blue    device1 9<br>
hscan		0.0 100.0<br>
vscan		0.0 50.0<br>
</code></pre>

<h2>example configuration files</h2>

<pre><code>#single light on a 3 channel ltbl device<br>
<br>
[global]<br>
interface	127.0.0.1<br>
<br>
[device]<br>
name 		device1<br>
output		/dev/ttyUSB0<br>
channels	3<br>
type		ltbl<br>
interval	20000<br>
rate            115200<br>
<br>
[color]<br>
name		red<br>
rgb		FF0000<br>
<br>
[color]<br>
name		green<br>
rgb		00FF00<br>
<br>
[color]<br>
name		blue<br>
rgb		0000FF<br>
<br>
[light]<br>
name		main<br>
color		red     device1 1<br>
color		green   device1 2<br>
color		blue    device1 3<br>
hscan		0 100<br>
vscan		0 100<br>
</code></pre>

<pre><code>#two lights on a 12 channel momo device<br>
<br>
[global]<br>
interface	127.0.0.1<br>
<br>
[device]<br>
name 		device1<br>
output		/dev/ttyACM0<br>
channels	12<br>
type		momo<br>
interval	20000<br>
prefix          FF<br>
rate            19200<br>
<br>
[color]<br>
name		red<br>
rgb		FF0000<br>
<br>
[color]<br>
name		green<br>
rgb		00FF00<br>
<br>
[color]<br>
name		blue<br>
rgb		0000FF<br>
<br>
[light]<br>
name		right<br>
color		red 	device1 2<br>
color		green 	device1 4<br>
color		blue 	device1 6<br>
hscan		50 100<br>
vscan		0 100<br>
<br>
[light]<br>
name		left<br>
color		red     device1 1<br>
color		green   device1 3<br>
color		blue    device1 5<br>
hscan		0 50<br>
vscan		0 100<br>
</code></pre>