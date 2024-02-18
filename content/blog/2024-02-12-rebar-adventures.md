+++
title = "Rebar is a steel bar used as a tension device in rein..."
description = "Sid's adventures with figuring out how to enable ReBAR on their laptop."
[taxonomies]
  tags = ["tech", "software"]
+++
Sorry, I thought that was funny. Over the past few months I've been upset about the hardware I have[^1], because if I could have waited a month more[^2]. I could've gotten the 2020 model with an RTX 2060 instead. But yes, I've been upset about missing out on all the cool kids features like hardware accelerated Ray Tracing, DLSS, all the AI shenanigans, and... Resizable BAR.

Now, there's nothing I could do about my GPU physically not having all the fancy RTX "cores" or the Tensor "cores" for RT and DLSS and AI 
shenanigans, but Resizable BAR... that was within reach, and I began my adventure in May, last year.

> `Wait but, what the heck IS Resizable BAR?`

Good question. Resizable BAR, or ReBAR[^3] for short, is a PCIe feature that lets devices in your computer adjust their memory space without rebooting, allowing for quicker access to device resources between, in the case I wished to achieve, the GPU and the CPU. By default, GPUs expose only 256 MB of memory to the CPU to write to, and this can be expanded to any 2^n size as long as the hardware and the firmware supports it, meaning it is possible to give the CPU access to your GPU's entire vieo memory.

Great then! all I had to do was head to my UEFI Firmware settings and enable it, right? Wrong.

### Nvidia strikes again, but do not give up hope...

While ReBAR has been a PCIe feature since PCIe 2.0 Extended Configuration Space, but it was standardized and widely implemented only with PCIe Base Specification 4.0. This means ReBAR has been around since 2007! However, since it only became a standard with PCIe 4.0, Nvidia decided to be mean and only enable it in their vBIOS starting with the RTX 3000 series GPUs, which was disappointing to find out, but I knew vBIOS modding was a thing, so I got searching.

While I found a lot of tools and resources on modding older NV GPUs, I learned that Nvidia had begun signing their vBIOSes with a special key, bringing an end to vBIOS modding. Okay, I thought, surely this is the end then. I can't do anything to enable vBIOS on my poor non-RTX Turing generation GPU.

### BIOS Modding, our savior?

Turns out, I wasn't the first one who wanted features big corpos told me I couldn't have. I stumbled across a project on GitHub called [ReBarUEFI](https://github.com/xCuri0/ReBarUEFI), which is a UEFI DXE driver to enable ReBAR on systems that don't have "official" support. I went through the repositorty and its wiki, trying to understand how I could use it to my advantage and finally have the big BAR of my dreams. 

But what I saw was daunting, and downright terrifying. Patching my BIOS? 4G Decoding? DSDT? The project and its wiki were speaking a language I wasn't very familiar with, but I was determined, and the wiki had a step by step on how to do it. So off I went, grabbing my BIOS file, and the tools I needed, and set off. In the process of patching the BIOS, I realized my laptop's BIOS was rather restrictive with the settings it exposed, which threw me on another tangent. 

After a short romp through sketchy sites and forums, I came across a [reputable-looking forum](https://winraid.level1techs.com/t/request-unlocked-acer-nitro-7-an715-51-bios/34678) that had exactly what I wanted, an "unlocked" BIOS that exposed more settings than the default one on my laptop. Cool, I thought, I'll flash this one to see if I can successfully flash another BIOS on it without bricking the laptop before I patch it myself. So I grabbed my favorite [Windows bootable environment](https://www.hirensbootcd.org/), since I only use linux, and successfully managed to flash the unlocked BIOS, following which I went through the process of patching that unlocked BIOS file with relative ease following xCuri0's guide, and flashed that onto my laptop as well. So far so good!

Except it wasn't. My laptop booted and worked just fine, but the tool to modify the BAR size that accompanied the ReBarUEFI project just did nothing on my laptop. It was able to set the required UEFI variables, but my GPU still reported a BAR size of 256MB. Frustrated, I started asking on the repo's issue tracker if I'd done something wrong, and after some more testing, I realized that I *did* have ReBAR working, I was able to shrink it down, but I was limited to a max size of 256 MB. Meaning that pesky vBIOS toggle that Nvidia was not enabling on everything-but-the-latest[^4] GPUs was still the blocker. Surely this was the end for me, a laptop with a small BAR, not a large one I could serve all my friends and family off of.

### The End..?

Out of sheer curiosity last month, to see if there was a change, I poked around my system regarding ReBAR again and noticed that my dmesg had a different output. Seeing this, I immediately head back to the issue tracker to ask for help, and was very kindly pointed to a new project called [NvStrapsReBar](https://github.com/terminatorul/NvStrapsReBar). It is a fork of ReBarUEFI with changes to the project that allow ReBAR to be enabled on *any* Nvidia gpu, vBIOS be damned. I won't get into detail here, you should read up on the project and how it was done, but the gist of it is that this new UEFI driver hardcoded[^5] the PCIe address of the GPU and overrides the BAR size that way. I happily re-modded my BIOS, flashed it, and ran into the same issue I had previously, my BAR size was limited to 256 MB. But it wasn't the driver's fault this time around, it turned out to simply be a udev rule I'd set that enabled PCIe power management on every device from the get go. One blacklisted PCI device later, my GPU reported a maximum BAR size of 8 GB, just like I'd configured it!

My device still refused to ran some applications on the GPU, but after some DSDT probing I realized I'd modded my DSDT wrong this time around, and as soon as I fixed that, I had a full, large, usable BAR, and could serve beers to all my friends.

{{ img(id="/media/rebar.png", alt="A screenshot of Alan Wake 2 running on my laptop, with a Konsole window on top of it, showing BAR size and usage.") }}

---

[^1]: an Acer Gaming Laptop from 2019 with a 1660Ti, the Nitro 7 AN715-51 specifically.

[^2]: I couldn't wait because of the CoVID lockdowns demanding me to have a laptop ASAP for school.

[^3]: SAM for you AMD users. Smart Access Memory.

[^4]: dunno if you paid attention, but I did say I started this adventure in May 2023.

[^5]: the project has since been updated and doesn't require you to hardcode the values before building the UEFI driver.
