+++
title = "A Tale of Two Technologies"
description = "Sid's thoughts on the whole X11-wayland debacle based on what they know."
[taxonomies]
  tags = ["tech", "software", "rambles"]
+++
Almost every linux user, whether someone using it as their only system, or someone dual booting, or even just playing around with it in a VM, might have at some point come across funny words like X11 or Wayland. These two have been a source of debate among developers and development-adjacent users for the past 15 years, and a source of confusion to end users. Having observed both closely, and having tried to understand their inner workings, I'm just using this space to dump my understanding, observations, and opinions. You're free to disagree, and if you do, feel free to catch me on one of my socials. Whether or not I will engage in further discussion, I can not guarantee.

For all of you confused: X11 and Wayland both provide the means to interact with your computer on a Unix[^1] system, handling app windows, and input.[^2]

### The state of things

X11 is old, *really* old. It's a product of MIT from 1984, and had its last major release nearly 12 years ago now. Being as old as it is, it's quite impressive how well it still holds up today. However, due to the age of the project, a lot of code in it is difficult to understand and change, and because developers wanted something more flexible than good ol' standardized X11, Wayland was born in 2008. 

Wayland takes a different approach to display server and client communication. I  t is a communication protocol and a specification, meaning those wanting to create a "window manager" with wayland are free to implement its features however they see fit, as long as it follows said specification.

### Not one, many Waylands

The push to wayland brought with it a mindset where a lot of developers are looking to "kill" X11. Which is a perfectly valid approach, since wayland, being modern, is meant to help developers speed up development. However, wayland being a set of protocols and a specification does not make sense to me. That approach to development, I feel, is better off for graphics APIs than a windowing system/display server. Yes, wayland offers much lower level control of the hardware than X11 did, and gives the developer freedom to implement things in better ways, but at the same time it presents a whole new problem: fragmentation.

Wayland is fragmented by design. The fact that it is a protocol and a spec and that devs are free to implement it how they wish to causes fragmentation at the root level. 12 years after the introduction of the spec, we have *at least* 6 different implementations of wayland: kwin, mutter, wlroots, smithay, mir[^3], libweston. Fragmentation is why the Android ecosystem has suffered for so long, with Google spending tons of resources into Project Treble, GKI, and other infrastructure to mitigate fragmentation on Android. However, wayland seems to encourage fragmentation, which just results in the ecosystem as a whole moving at varying levels depending on which implementation your "compositor" of choice chooses to be based on.

This *is* working out great for the kwin team. They have lower level control, and aren't held back by another project on implementing new things, such as the recently implemented HDR support. This also arguably works well for the mutter team for the same reasons, but mutter's new-feature-enablement is held back by other reasons I will not get into here. The lower level control and lack of a standard, however, is harmful to the ecosystem outside of KDE Plasma and Gnome (and all DEs that use their components), an ecosystem that flourished on X11.

### What X11 did better

X11 offered devs outside of KDE and Gnome a standard. It allowed people interested to create their own window managers, in a way that their window manager could easily support all the features that Plasma or Gnome had. This is missing on wayland. "But Sid," you might say, "libraries like wlroots and smithay exists to fill this role." And you are right, they do. But at that point the purpose of wayland is defeated. 

Wayland is meant to offer the developer lower level control, but with more and more non-DE compositors relying on libraries, the compositors fall back to the same development model an X11 window manager had: to build against a piece of software that they know is reliable. Except, due to aforementioned fragmentation, there are multiple libraries to chooose from, meaning two devs using two different libraries for their respective compositors could end up with compositors that vary in feature sets. And this is a real problem! Projects like [sway](https://github.com/swaywm/sway/issues/5601) and [hyprland](https://github.com/hyprwm/Hyprland/issues/4436) do not have working VRR since 2020, even on AMD GPUs, while kwin has it working across all vendors, and this stems from the fact that the base implementation between kwin and wlroots, the library sway and hyprland are built on, are different.

### Why Wayland will never* kill X11

Fragmentation aside, there's another major factor preventing Wayland from becoming the "X-killer:" Wayland is Linux only. Wayland's core set of protocols relies on an event loop, using `epoll` for it, *and* most compositors currently rely on Linux's `epoll` through libinput. `epoll` is a syscall that is Linux-only, which means the wayland server as well as compositors require ugly hacks to run on non-Linux UNIX systems, like how FreeBSD implements `epoll` on top of `kqueue`.

While most users, and sadly developers, don't care about BSD and the likes, breaking UNIX compatibility as well as not being POSIX compliant *does* mean wayland won't be able to fully replace X11 unless it goes through a major restructure, which, 12 years into development, while has been discussed, is ultimately difficult to achieve. X11, on the other hand, *is* both UNIX compatible and POSIX compliant, meaning it is possible to natively run the X server on Linux, BSD, Solaris, MacOS, and even Windows.

Linux and Unix have always been about choice, and trying things out, but wayland in its current state intentionally leaves out non-Linux systems.

### Everything bäd, nothing good

This post isn't meant to argue on which is better, X11 or Wayland. Both have their pros and cons, both fill a certain role in the ecosystem. One is ancient, almost unmaintained, while the other still sees active development. I haven't even gone into the other problems I have with the wayland ecosystem, but maybe that's a ramble for another post. In the end, use what works for you, and always provide feedback to your FOSS developers :>

---

[^1]: Linux based OSs are Unix-like, and only X11 is truly Unix compatible.

[^2]: They're both software responsible for holding a connection to any program wanting to display a window, that holds window state such as positions or maximized/minimized, that handles the routing of input to the right programs and that processes this information to produce the final input with all the windows composed in it that gets prevented to the screen.

[^3]: Mir is a special case. It started off as an alternative display server to X11 as well, but later implemented wayland.


### A correction

Shortly after publishing this post, it was pointed out to me that libwayland no longer has a hard dependency on epoll, details of which can be found for [FreeBSD](https://gitlab.freedesktop.org/wayland/wayland/-/merge_requests/147), [OpenBSD](https://gitlab.freedesktop.org/wayland/wayland/-/merge_requests/256), and [NetBSD](https://gitlab.freedesktop.org/wayland/wayland/-/merge_requests/258). I'm not going to edit the original post contents, for posterity.