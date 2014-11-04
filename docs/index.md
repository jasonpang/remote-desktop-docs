# Remote Desktop Documentation

Github Project Page: [https://github.com/jasonpang/RemoteDesktop](https://github.com/jasonpang/RemoteDesktop)

## System Requirements

Operating System: Windows 7 / Vista / Server 2008 R2

*Note:* The driver will likely not install on Windows 8, as [Windows 8 has done away with mirror drivers](http://msdn.microsoft.com/en-us/library/windows/hardware/ff568315(v=vs.85).aspx).

## How do I compile and run the project on my own computer?

As of November 4, 2014, the project does run successfully in its current state, though it requires some slight configuration.

1. [Download and install the DFMirage Mirror Driver](http://www.demoforge.com/dfmirage.htm). *Restart the computer after* (otherwise the driver will not be detected by the application).

2. Clone my project: `git clone https://github.com/jasonpang/RemoteDesktop.git`.

3. Build the project. I had no problem cloning and building the project on the first try; let me know if you have build issues so I can address them here. 

4. Open Windows Explorer and open `<project-root>/Introducer/bin/Debug/Introducer.exe` (assuming you built the project in Debug configuration). The key idea in this step is to run the `Introducer` separately from the `Client` and `Server` application, to prevent an error about two sockets fighting to listen on the same port.

5. Run the solution. The `Client` and `Server` project should be running simultaneously. If this is not the case, right click the Solution, click `Set StartUp Projects...` and make sure `Client` and `Server` are both set to `Start`. Make sure the `Introducer` project is *not* set to `Start`. We ran the `Introducer` separately in step 4.

6. You should see two windows pop up, one for the `Client` and one for the `Server`. On the `Client`, you should see two textboxes: one for an ID and one for a password. You'll see these fields populated on the `Server` window, so simply fill them in and hit connect.

7. You should see a black screen with a red button `Click to Begin`. Click it, and you should see the remote desktop stream (will likely be slow).

## Additional notes

1. If you have only one monitor, the screen capture you'll see will look a little odd, because it's capturing the screen within the screen within the screen ... ad infinitum. This demo works best on two remote computers (as it was intended) or on two monitors. The screen capture will occur on the primary monitor -- simply drag the window streaming the remote desktop into the secondary monitor, and you'll avoid the infinite cascading window effect.

2. If you're running this demo between two computers, the setup gets a little more complicated:

2a) You need to run the `Introducer` on a *globally reachable* server. This is because both the `Client` and `Server` connect to the `Introducer` to discover each other. The `Introducer` should be run on something like a VPS instead of a home computer behind tricky firewalls. I've hosted the Introducer previously on a public Amazon EC2 instance, but my free tier ran out :]

2b) Once the Introducer is running, you can run the `Client` on one computer, and the `Server` on another. If the `Server` successfully communicates with the Introducer, you'll see the `Server` report an ID and password. Similarly, you can view the `Log` tab on the `Introducer` to see it say something like:

```
11/4/2014 10:18:05 AM Registered X.X.X.X:60439 as h6w.
```

to see that the `Introducer` successfully communicated with the server.

2c) The sign that the `Client` can successfully communicate with the `Server` is a successful connection, where the initial `Client` connection window closes and a screen with a black background and a red `Click to Begin` button appear. If this doesn't happen, you'll likely see an error. You can view the `Introducer` logs to verify that the `Introducer` tried to exchange connection details between the `Client` and `Server` -- if the introduction and connection detail exchange was successful, there isn't anything more you can do, as the [UDP hole punching](http://en.wikipedia.org/wiki/UDP_hole_punching) between the clients has failed.

3. Screen transfer is slow.

I wrote this project many years ago without really knowing what an algorithm was or how to optimize one. The Server currently uses the mirror driver to receive small rectangular regions of the screen and then rapid-fire send them to the Client. I'm sure there are optimizations available, and it seems like an interesting problem, but I haven't gone back to this project in a few years.

The cool thing is that the screen transfer works using the mirror driver and sending rectangular regions.

4. There are bugs / random crashes.

There are a few reasons for this. Each reason is, unfortunately, rather hard to fix:

a) The server streams screen change image data as quickly as possible, without connection throttling. Recall the protocol used between these two applications is UDP (chosen for UDP's hole punching ability), and UDP doesn't take care of congestion in the way TCP does.

b) The screen capture code is buggy. Raising screen changes in the edges of the screen can raise an exception from time to time.

c) The network library used isn't entirely stable. The networking used between the Client and Server is a 4-year old (by today's count) code mix of [Lidgren's networking library](https://code.google.com/p/lidgren-network-gen3/) (as it was back in 2010) and [ermau/Tempest's](https://github.com/ermau/Tempest) networking library. I plastered Tempest's architecture on top of Lidgren's UDP functionality and it worked pretty well for most cases, although there would be occasional errors. They've each since added hundreds of commits in the last four years.

## Contact

Please feel free to email me at jasonpang2011@gmail.com for questions.