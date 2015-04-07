**Boblight is a collection of tools for driving lights connected to an external controller.**

Its main purpose is to create light effects from an external input, such as a video stream (desktop capture, video player, tv card), an audio stream (jack, alsa), or user input (lirc, http). Currently it only handles video input by desktop capture with xlib, video capture from v4l/v4l2 devices and user input from the commandline with boblight-constant.

Boblight uses a client/server model, where clients are responsible for translating an external input to light data, and boblightd is responsible for translating the light data into commands for external light controllers.

If you have any question about boblight, want to suggest a feature, or show your setup, you're welcome to post a message in the [boblight google group](https://groups.google.com/forum/#!forum/boblight) .