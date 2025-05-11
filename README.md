DIY NAS
=======

This page is basically a write-up of why putting together a simple small silent NAS isn't the no-brainer that I thought it would be.

At the moment, I'm inclined towards an 8GB Raspberry Pi 5 with two 4TB SSDs and a smaller system SSD and the 4-way NVMe hat (assuming the issue with booting from the hat, see below, really has been resolved) and using OMV and a RAID filesystem other than ZFS. This is very far from my ideal solution which would have involved an N100 or N150 based system with 32GB of RAM and running TrueNAS SCALE.

---

I've had a small 2-bay QNAP NAS for a long time. Two things have always disappointed me about it:

* It uses HDs rather than SSDs and these, along with its fan, make too much noise for it to be used in my living room alongside my TV.
* While it's Linux based, it has its own package management system and most of its config files are essentially read-only as they're maintained by a set custom QNAP tools - forcing you to do everything and diagnose any issues via its web UI.

These days commercial NASes and the open-source software for DIY NASes (see [TrueNAS SCALE](https://www.truenas.com/truenas-scale/) etc.) have morphed into general purpose servers, that run everything from the familiar adjuncts to a home NAS - like Plex Media Server - to docker and kubernetes. This perhaps reflects the power of modern devices and the feeling that it's got to be used for something.

I just wanted a classic NAS that had these features:

* Silent, i.e. SSD based, and small.
* RAID 1 and/or 5 support - i.e. everything shouldn't be lost if one drive fails.
* Serves files via a SAMBA share.
* Runs a standard Linux variant like Ubuntu.
* Hardware that can be bought off the shelf without specing up a mainboard etc.
* Low power consumption as it's going to be on 24x7.

My current NAS has two mirrored 2TB drives and it's almost full. During it's impressive 15 year life, it suffered two drive failures and a power supply failure.

For my new NAS, I wanted something with a useable capacity of 4TB. You need a system disk and the RAID drives:

* For RAID 1, i.e. simple mirroring, two 4TB drives would be required.
* For RAID 5, one could use:
  * Three 2TB drives (2 data drives and one parity drive).
  * Five 1TB drives (4 data drives and one parity drive).

SSD drives are connected using what are called M.2 slots and it turns out finding small ready-to-use hardware with multiple M.2 slots is the near impossible task in this effort.

So, mirroring requires the minimum of M.2 slots so ideally we want three M.2 slots (two for the mirrored drives and one for the sytem disk), though many people reduce this to two by running the system a small SSD drive connected via USB. Some of the hardware on a full-size SSD is managing the unusual characteristics of the SSD storage chips - this is missing on the small USB connected devices which are not expected to be in continuous use. This means a USB connected SSD that's used as a system disk will typically fail far faster than a full sized SSD. DIY NAS builders seem to be prepared to live with this on the basis that the system disk is small and it's easy to back up - so if it fails you just have to plug in a backup.

Now some terminology - M.2 slots are typically designed for one of two interfaces - the older SATA or the newer PCIe (you can also find _combo_ slots that support both). The communication protocol used by drives talking  over PCIe is called NVMe. So, for a modern setup, people talk about NVMe SSDs and M.2 PCIe slots.

As of early 2025, the majority of NVMe drives support PCIe gen 4. While can use such drives with older PCIe gen 3 slots, there are other reasons, in addition to just the reduced speed, not to do this.

Note: see the Jeff Geerling blog post and videos, linked to below, on the GMKtec G9 - there he covers the heat and reliability issues he experienced using gen 4 drives with a compute with gen 3 slots.

In older videos, people sometimes recommend choosing SATA SSDs rather than NVMe ones because they're cheaper. But this isn't the case anymore, NVMe gen 4 drives are essentially dominant and the older variants can actually be more expensive now as they're kept available for people who really need them.

Most NAS vendors produce locked-down hardware, i.e. you can't treat their boxes as standard x86 PCs and replace they're custom OSes with e.g. Ubuntu. An exception is Asustor (a subsidiary of ASUS), e.g. see Jeff Geerling's [blog post](https://www.jeffgeerling.com/blog/2023/how-i-installed-truenas-on-my-new-asustor-nas) on installing TrueNAS on his Asustor NAS. However, even then people still typically seem to have minor issues, e.g. the fan always running at full speed. And Asustor's cheap NASes are all Rockchip ARM based devices - it's only their larger and more expensive models that are x86 based.

To quote Jeff Geerling (when talking about the ASM1184e chip in a 4-way NVMe board for the Pi 5):

> Because the Pi 5 only exposes one lane (x1) of PCI Express Gen 2, you need a PCI Express switch chip (much like a
> network switch) to split that single lane into four connections.
>
> The ASM1184e is a solid, reliable PCIe Gen 2 switch, but it is limited to Gen 2 speeds (5 GT/sec), whereas the Pi's
> internal bus can be pushed to PCIe Gen 3 speeds (8 GB/sec).

Jeff Geerling blog posts and videos:

* [Raspberry Pi 5 with Geekworm's 4-way NVMe hat](https://www.jeffgeerling.com/blog/2024/4-way-nvme-raid-comes-raspberry-pi-5)
* [Raspberry Pi 5 with Radxa's 5-way SATA hat](https://www.jeffgeerling.com/blog/2024/radxas-sata-hat-makes-compact-pi-5-nas)

While SATA SSDs are already becoming uncommon, Jeff's review of the SATA hat brings up points that apply whatever the build - overheating seems to be a common problem with these kind of builds and he notes (after using a themal imaging camera on his build) that the SATA chip is getting hot enough that you'd probably want to add a fan even with his completely open setup. With the 4-way NVMe hat setup, he notes that it runs cool enough in an open setup but you'd need a fan if you put it in a case.

Jeff's [YouTube video](https://www.youtube.com/watch?v=yLZET7Jhza8) that accompanies his blog post on Geekworm's 4-way NVMe hat has the rather more downbeat title "impossible to recommend" - though if you watch the video, it's a little less absolute than it sounds.

Note: Jeff Geerling, in his follow-up video on the GMKtec G9 (see below), uses [ELUTENG M.2 SSD heatsinks](https://www.amazon.com/dp/B0CR69GG3W), despite the unknown name, they appear not to be the usual anonymous OEM product with random branding applied for sale on Amazon - there actually is a Chinese company that's clearly behind this name. But you can buy similar products from more well known brands, like this [SSD heatsink](https://www.bequiet.com/en/accessories/m-2-ssd-cooler/2249) from Be Quiet.

In the 4-way NVMe hat blog post and video, he also mentions FriendlyElec's [CM3588](https://www.friendlyelec.com/index.php?route=product/product&product_id=294) that features four proper PCIe gen 3 lanes (vs the Pi 5's single gen 2 lane). And he also links to Linus Tech Tips' [review of the CM3588](https://www.youtube.com/watch?v=QsM6b5yix0U). However, unlike Raspberry Pi, there's rarely any long term support for anything Rockchip based - Rockchip boards typically come out with one Linux kernel that's been hacked to get it working for that board and that's it and that's it, if there are any subsequent kernel builds they'll come from enthusiasts or projects like [Armbian](https://www.armbian.com/).

Note: a while ago, I wouldn't have linked to Linus Tech Tips but Linus seems to be less toxic these days and seems to have grown up a bit.

The Geekworm 4-way NVMe hat seems to fairly unique at the moment but Seeed, Pimoroni and Pineboards all produce 2-way NVMe hats (you can find them all [here](https://thepihut.com/collections/raspberry-pi-ssd-hats) on the Pi Hut site).

Note: Jeff comments that you can't currently boot directly from the 4-way hat, i.e. you have to boot from an SD card. However, he logged this [issue](https://github.com/raspberrypi/firmware/issues/1833) in Raspberry Pi firmware repo and this issue is now resolved (see [this comment](https://github.com/raspberrypi/firmware/issues/1833#issuecomment-2431608168)).

---

Jeff Geerling's [blog post](https://www.jeffgeerling.com/blog/2025/almost-perfect-mini-nas-my-mini-rack) on the Intel N150 based GMKtec G9 with four PCIe gen 3 M.2 slots and the accompanying [YouTube video](https://www.youtube.com/watch?v=M_Ft8OAPQ3g) and the [follow-up video](https://www.youtube.com/watch?v=TlsIuA8rBRg) cover the miseries of trying to build a NAS using one of these essentially no-name Chinese products that at first glance look like a near perfect combination of features.

Brian Moses became well known for his yearly builds of a full-far DIY NAS along with a much cheaper variant that he called the EconoNAS. E.g. here are the [2024 builds](https://blog.briancmoses.com/2024/09/diy-nas-2024-edition-and-econonas.html). However, since 2023 he's been building these device using Topton motherboards. Topton is a no-name Chinese brand that are one of the few manufacturers of low-powered boards for Intel N-series chips, like the N100, that feature multiple M.2 PCIe slots. Brian now also sells their boards via eBay - which _may_ lead him to be less critical of some of their shortcomings. If forums are anything to go by, Topton is not a top-tier manufacturer and many people have serious issues with their products.

A similar product to the GMKtec G9, mentioned above, is the Topton Pocket NAS. You can find a review of it [here](https://blog.briancmoses.com/2025/03/an-all-flash-diy-nas-that-fits-in-my-pocket.html) on Brian's site and it's available [here](https://www.aliexpress.com/item/1005007900081466.html) on the Topton AliExpress store. You can also find the Pocket NAS on [here](https://www.toptonpc.com/product/new-pocket-nas-4xm-2-nvme-board-12th-gen-firewall-mini-pc-intel-i3-n305-n100-2xi226-v-2-5g-soft-router-ddr5-4800-2xusb3-2-2xhd/) on the Topton site (the site seems to be aimed at resellers, you can only get quotes for bulk orders).

As always with these no-name Chinese brands, it's impossible to tell if Topton is the original manufacturer. On their AliExpress store, they also sell GMKtec branded products so it's unclear if they're just a reseller or the actual OEM.

A company called CWWK sells an identical product under the CWWK brand - it's available [here](https://cwwk.net/products/x86-p5-4-m2-nvme-12th-generation-intel-n100-i3-n305-ddr5-4800mhz-firewall-pc-2x-i226-v-2-5g-network-card) directly from their site (and is also available from various stores on AliExpress).

I found more reviews on YouTube for the CWWK branded version than the Topton one and some people _suspect_ CWWK may be the OEM.

Neither Topton nor CWWK provide anything like a normal support section where you can easily download updates for things like the BIOS. Topton just have a list of `.rar` files [here](https://www.toptonpc.com/driver-download/) with fairly random names that you somehow have to line up yourself with their generally unclearly named products (e.g. the Pocket NAS product page doesn't clearly specify model name but most of the `.rar` files just include a model name). The CWWK [support section](https://pan.changwang.com/) is just as bad - it's just Google Drive style directory on some unknown Chinese server.

To summarize the issues with these no-name brands:

* PCIe is not broken out (i.e. like the Pi, it's typically a single lane, with a splitter, no matter the number of M.2 slots).
* PCIe is gen 3 rather than gen 4.
* Prone to overheating.
* Unknown BIOS with no updates.

The BIOS issue is bigger than it may sound - modern BIOSes manage much more than they used to and a reliable system with low power consumption relies on the BIOS to have correctly initialized everything before the main OS boots up.

Note: the Pocket NAS looks like a giant heatsink and my impression is that such a design could work well but the no-name manufacturers apparently are just focusing on the look and there's no real thermal design, e.g. in some designs, the internals are practically insulated from the heatsink-like cases rather than being designed to transfer heat to the case.

**Update:** there are a number of Chinese companies in the same league as CWWK and Topton but _slightly_ better regarded (if for nothing more than they have websites that look a little more like what you'd expect from a consumer hardware companies) - they're [GMKtec](https://www.gmktec.com/) (as feature every now and then by Geef Geerling - often not in an entirely positive light), [Minisforum](https://www.minisforum.com/) and [Beelink](https://www.bee-link.com/products/beelink-me-mini-n150). Beelink _seem_ to be the best regarded (and slightly pricier than the others) and they're releasing a NAS model that should be available sometime in June 2025 (see [here](https://www.bee-link.com/products/beelink-me-mini-n150) for the product page). My first impressions are that while the site looks reassuring, clicking on the _Download Center_ under the _Support_ menu looks as underwhelming as the support sections typical for many Chinese ompanies - it takes you to a different `.cn` hosted site and a vanilla login page with no "register here" link or any other vaguely helpful information and a broken loading icon (and if you pull up Chrome's developer tools, you see lots of 503 errors). You can also get to this [driver downloads page](https://www.bee-link.com/pages/drivers-hardware-download) but it's clear they uploaded a collection of drivers between 3rd and 19th December 2023 and never touched it again.

---

Until recently, almost no ARM boards came with multiple M.2 slots so popular NAS management systems like [TrueNAS SCALE](https://www.truenas.com/truenas-scale/) and [Unraid](https://unraid.net/) haven't even bothered releasing ARM builds. An exception is the somewhat less popular OpenMediaVault which, for ARM, isn't bundled as a ready-to-go ISO but can be installed by first installing Debian and then installing the OpenMediaVault packages as described [here](https://docs.openmediavault.org/en/stable/installation/on_debian.html).

Note: OpenMediaVault has a [github repo](https://github.com/openmediavault/openmediavault) and a [website](https://www.openmediavault.org/). However, as of 27 April 2025, the website seems to have been hacked to deliver the kind of ads you typically see on Pirate Bay, etc., e.g. big banners with text like "your download is now ready", which take you to something that you almost certainly don't want to download, and pop-up modal ads that you have to click away to get to the underlying page (but which open another tab to some other site when you do so). I filed an [issue on GitHub](https://github.com/openmediavault/openmediavault/issues/1950) about this. I hope the server distributing their Debian packages hasn't been similarly compromised.

Note: Unraid is unusual in being a paid product in a space where everything is usually open source and free. But apparently, it's so convenient to use that it actually is successful with its paid model.

Raspberry Pi OS already has support for ZFS and other filesystems with RAID support. However, if you go this route then you have to come up with your own home-made solutions for things like alerting you when a drive fails.

Search for "TrueNAS on Arm" in this [blog post](https://www.jeffgeerling.com/blog/2024/4-way-nvme-raid-comes-raspberry-pi-5) from Jeff Geerling for links to ongoing discussions about TrueNAS and Unraid on ARM.

---

ZFS currently seems to be the most popular filesystem for RAID. For a filesystem, it has extraordinarily large memory requirements - 16GB seems to be recommended amount at the moment. This may seem ridiculous but the reasons are rather involved and if you're really interested just google it.

So, this requirement alone means that most cheap SBCs aren't suitable - most have at most 4GB. Raspberry Pi are relatively unusual in having 8GB and 16GB variants of the Pi 5.

You can of course choose a less memory hungry RAID filesystem.

---

Given the memory requirements of ZFS and the fact that systems like TrueNAS SCALE don't support ARM the ideal system would seem to be:

* N-series Intel based.
* Three or more PCIe gen 4 m.2 slots.
* From a known-name manufacturer.
* An Intel NIC.

Many cheaper boards come with Realtek NICs but these have poor Linux support hence the requirement for an Intel NIC.

The N-series is Intel's very low-power range that's meant to compete with ARM SBCs for things like set-top boxes. They typically a base [TDP](https://en.wikipedia.org/wiki/Thermal_design_power) of less than 10W - though watch out, the N350 has a base TDP of 7W while the very similarly named N355 has a base TDP of 15W. For TDP figures for these and other 12th generation Intel chips, see the Wikipedia [Alder Lake page](https://en.wikipedia.org/wiki/Alder_Lake).

Note: TDP is about heat and doesn't translate directly into electrical power consumption but it's a good indicator.

For something like a NAS that's just serving files, the N150 chip should be more than powerful enough.

As we'll see, it seems to be impossible to get name-brand systems or motherboards with multiple M.2 PCIe slots that support the N-series processors. The N-series really is targetting the same market as Pi and Rockchip ARM CPUs and so, with the exception of the CWWK/Topton system mentioned above, only seem to come on SBC-style boards similar to Raspberry Pis.

So, the best you seem to be able to do are the lower TDP variants of the the more classic Intel ranges which can be found with TDPs of around 30W, e.g. the 12th generation - Celeron 7300, Pentium 8500 and Core i3 1210U. Technically 12 generation chips came to the end of support in April 2025 but they're still commonly available and are the only ones where the prices are even in the same ballpark as the N-series chips like the N150.

---

It would be lovely to get something like a NUC with three M.2 slots. In 2023, Intel completed the sale of the NUC business to ASUS. Most NUC models come in a slim and a tall variant and some of the tall variants do have three M.2 slots, e.g. the 12 Pro and 14 Pro. However, this is as perfect as it first seems - only the first M.2 slots is for a full size 22x80 NVMe SSD, the other slot is for a smaller 22x42 NVMe SSD and the third is for a SATA drive. You could use the SATA drive for the system disk but large capacity NVMe drives generally only come in the 22x80 size so the second M.2 slot is useless for a mirrored setup.

Aside: ASUS produce a huge variety of NUCs - the only way to determine what you're looking at is to decode the model number using ASUS's naming convention [FAQ sheet](https://www.asus.com/support/faq/1052685/) - though I've found nothing that lists or explains the two-letter NUC family names.

The ASUS site isn't great for getting any insight into NUCs or any particular model. For a good review of the popular NUC 14 Pro (tall variant), see this [Reddit post](https://www.reddit.com/r/intelnuc/comments/1hivw04/review_ama_asus_nuc_14_pro_with_fedora/) that goes into details with lots of photos of what you get along with details of getting Linux up and running on it.

---

So, once you've determined there's no off-the-shelf solution, it's time to disapper down the DIY route that I wanted to avoid.

If you want to keep things small then you need a Micro-ATX or Mini-ITX mainboard. Confusing (to me, given the names "mini" and "micro"), Mini-ITX is the smaller of the two and these just don't come with multiple M.2 slots so you have to choose a Micro-ATX board.

If you go to the Gigabyte [mainboard section](https://www.gigabyte.com/Motherboard/All-Series) you can filter for Micro-ATX boards for Intel processors that have three M.2 slots (and they all support PCIe gen 4 as either pure PCIe slots or as combo slots that also support SATA). But watch out, the manufacturer of the NIC is usually hidden away in the details - as noted above, an Intel NIC is preferable.

You can also search [here](https://www.asus.com/ch-en/motherboards-components/motherboards/all-series/filter) for ASUS Micro-ATX boards for Intel processors with 3 or more M.2 slots. ASUS state the NIC manufacturer more clearly - usually it's Realtek for these kind of boards but they do have some with Intel NICs.

These boards start at around US$150 (if you're outside the US or if Trump has backed down on China tarrifs by the time you read this) and look horribly over-speced for a NAS. And you still have to buy a case, power supply, processor and RAM.

The cheapest board that I found that definitely had an Intel NIC and 3 suitable M.2 slots was the ASUS Prime Z790M-Plus at around US$200. ASUS is a well know motherboard manufacturer but their reputation seems to have suffered in recent years if forum comments are anything to go by.

**Update:** see next section on an alternative to choosing a board with three M.2 slots (as these are not very common in Micro-ATX format).

One thing that's common to the suitable lower-end Micro-ATX boards is that they have LGA 1700 sockets - this is the socket for the CPU and determines what CPU models you can use with them.

Note: LGA 1700 is the socket type, and a number of Intel chipsets uses this socket, the cheapest being the Intel 600 series and the cheapest within this series being the H610M series. Most low-end Micro-ATX boards are H610M boards and many feature H610M in the product name.

And again we hit a wall - while manufacturers of barebone systems and others seem to be able to get their hands on low-powered chips like the Core i3 1210U, the only chips readily available to retail customers for the LGA 1700 socket are chips like the Core i3-12100F with a much higher TDP of around 60W.

**Update:** actually this may not be such an issue after all, see this [post](https://forums.unraid.net/topic/147371-first-energy-efficient-build-intel-n100-or-i3-12100/) where an i3-12100 (but **not* the i3-12100F variant) is recommended for a low-power setup and might actual use less power than some N100 boards (the post also notes, as many others do, that Intel NICs are more stable than Realtek under Linux but that in fact either are probably fine). This post is worth reading for more general details on building a NAS.

Note: ASRock produce the [H610M-HDV/M.2](https://www.asrock.com/mb/Intel/H610M-HDVM.2/index.asp) that's a H610M board with an Intel NIC (most boards at this price point have Realtek NICs). ASRock seem to like confusing naming and there are various other H610M-HDV/M.2 boards, e.g. the H610M-HDV/M.2+ D5. Only the one without anything extra after the "M.2" part of the name features an Intel NIC. It's currently available for about Fr. 77 [here](https://alltron.ch/de/product/1335252) from ASRock's Swiss distributor (and for a bit more on sites like Digitec).

Warning: accurately listing the number and type of M.2 slots seems be beyond any retailer even Switzerland's usually excellent Digitec rarely get this right so if you find something listed as having three M.2 slots always check the manufacturer's site to see if this is the case and what kind of slots they are (SATA or PCIe gen 3/4).

Note: try [PcPartPicker](https://pcpartpicker.com/list/) if you're trying to put together a system - I came across it recommended on various forums.

As to the rest of the build, when it comes to a silent build, I've seen the CoolerMaster [Silencio S400 case](https://www.coolermaster.com/en-global/products/silencio-s400) recommended and I'm sure they have suitable power supplies to pair with it. Or you could buy parts from manufacturers like [be quiet!](https://www.bequiet.com/) who specialize in low-noise parts or from sites like [Quiet PC](https://www.quietpc.com/) that stock the quietest parts (like [Noctua fans](https://noctua.at/en/products/fan)).

Note: unfortunately, be quiet! don't seem to produce any sub-ATX size cases.

Random aside: for an interesting variant on the micro-ATX case (that isn't making any low-noise claims), see the white variant, with light-colored wood, of the [Lian Li A3-mATX](https://lian-li.com/product/a3-matx/) (see an in-depth review [here](https://gamersnexus.net/cases/excellent-budget-case-lian-li-dan-a3-matx-review-benchmarks) which rates it an excellent budget case).

---

An alternative to choosing a mainboard with three M.2 slots is to find one with enough PCIe slots such that fitting them with NVMe adapters means that you then have enough M.2 slots for your purposes when combined with however many M.2 slots the board already has.

On Amazon, it's normal to find the same product sold under multiple different brand names that you've never heard of. When it comes to PCIe NVMe adapters you'll notice that, even when it comes to the known-name brands, the cards all look remarkably similar. I think these adapters are a fairly generic product and most are just rebadged versions of cards from one main OEM manufacturer - [Glotrends](https://www.glotrends-store.com/).

So, if buying such cards, you might as well go directly to Glotrends, the only downside is that they produce a confusing number of very similarly priced cards and you'll have to work out which one you want.

They have:

* [Single drive cards](https://www.glotrends-store.com/collections/m2-pcie-adapter-372)
* [Two drive cards](https://www.glotrends-store.com/collections/2-port-m2-ssd-adapter)
* [Four drive cards](https://www.glotrends-store.com/collections/4-port-m2-ssd-adapter)

So, it might seem that the two or four drive options are the way to go - maximizing the drives you can install, given your fixed number of PCIe slots, and still relatively cheap.

This, though, is where the issue of lanes come into the picture - the pins on a full size PCIe slot are evenly split across 16 lanes. And an NVMe drive requires four lanes to run at full speed. So, a four drive adapter needs 4 times 4 lanes and as full width PCIe slot has 16 lanes, so this sounds fine. But this is where an issue called bifurication comes in - if the motherboard support bifurication then it can treat the 16 lane slot as four virutal four lane slots and this means the adapter card needs no real intelligence - it's essentially just four single drive cards merged with each feeding down to one quarter on the slots pins. However, not all motherboards support bifurication - in fact some manufacturers treat it as a high-end feature and only support on their boards targetting rack-mount servers while others, like ASRock support it on some of their most basic boards but not consistently. As bifurication isn't a feature manufacturers seem to bother including in their specs, the steps needed to determine if a board supports it (other than buying the board and checking the BIOS) border on insane - see this [guide](https://www.reddit.com/r/Amd/comments/14bnqh3/guide_about_how_to_check_pcie_bifurcation_support/).

So, if you e.g. look at the Glotrends four drive cards, you'll see there's a cheap variant (US$50) that supports the full 16 lanes and a much more expensive one (US$200) that surprisingly just supports 8 lanes. The differences is that the expensive one handles bifurication issues itself while the other expects the mother board to handle this. Full 16 lane card that handles bifurication themselves are available from other manufacturers but this get even more expensive. These cards are more expensive than many low-end motherboards that handle bifurication on their end.

Note: manufacturers label their cards fairly confusingly - a card that says it "does not support PCIe bifurication" means that it's a card where bifurication has to be handled by the motherboard and one that says "supports PCIe bifurication" means the card handles the bifurication issue (and is consequently far more expensive).

The whole PCIe slot story seems much more complicated than it should be:

* You can get full length slots with all lanes connected up.
* You can get full length slots with just eight or four lanes connected up (and you have to squint at the pins in the connector to see if they're empty or full or look at your motherboard documentation to work this out).
* You can get shorter four and eight lane slots.
* And on the card side, you can get four lane cards which are still full length, i.e. they'll only fit into a full length slot.
* You can get NVMe adapter cards with e.g. room four drives but which are quarter length, i.e. only require a four lane PCIe slot (and the a chip like the ASM1184e that essentially multiplexes the available bandwidth between the drives).

So, you want to look very carefully at the PCIe slot on your motherboard and the adapter card that you're looking at to make sure you're getting what you expected (and e.g. suffering a speed penalty due to your NVMe drives being multiplexed over fewer PCIe lanes than they could use).

See this video from _ExplainingComputers_ on [M.2 SSD adapters & enclosures](https://www.youtube.com/watch?v=Ktqa11VT1xI) - he makes clear the various issues about slot lengths, lanes and bandwidth touched on above. And he uses Glotrends cards as do various other YouTubers who seem to know what they're talking about.

PCIe generation is also a factor - as of May 2025, many motherboards support PCIe gen 5 and below while most NVMe drives still seem to be either gen 3 or gen 4. Gen 4 cards can e.g. be used with devices that only support gen 2 but there _may_ be issues (see earlier mention of this in reference to Jeff Geerling experiences with the GMKtec G9).

Notes:

* The cards that are full-length while e.g. only using four lanes are a mystery to me - why would a manufacturer want to restrict such a card from being used in shorter four and eight lane slots?
* You can get NVMe adapter cards which support just a single PCIe lane - this means that the attached NVMe still works but has a quarter of the bandwidth available to it that it can support. I don't know when or why one would use such a card as the smallest PCIe slots on conventional motherboards are four lanes. The Raspberry Pi 5 is probably the only situation in which you'll come across a setup that has just a single lane split out for use.
* In many YouTube videos, they mention that there's often only one full 16 lane PCIe slot and this is needed for the graphics card. However, for a NAS, you don't need anything beyond maybe the builtin graphics (during initial setup) so this isn't an issue.
* After saying to buy adapter cards from Glotrends, I'll mention some others. The ASUS 16 lane four drive [Hyper M.2 x16 Gen 4 Card](https://www.asus.com/ch-en/motherboards-components/motherboards/accessories/hyper-m-2-x16-gen-4-card/) - it seems to be a popular and cheap full speed four drive card that isn't just a rebranded Glotrends card. Ugreen also produce some adapter cards that they clearly make themselves.

---

What about a full-size PC from Lenovo or HP? There's basically nothing with multiple M.2 slots in the lower ends of these ranges - and once you enter the mid and high ends, you've completely given up on power consumption, size and silence.

---

The simplest ZFS setups for a 4TB NAS would seem to be:

* RAID-Z1 (single-parity) where you combine three 2TB drives (two for data and one for parity).
* Oddly, there's no name like RAID-Z1, for a two drive mirror setup, people just talk about using `mirror` mode. So, you'd need two 4TB drives with one mirroring the other.

The impression seems to have spread that a simple two drive mirror setup under ZFS isn't a great idea - but the people claiming this seem to noisy self-proclaimed experts who can't provide any reasoning for their opinion so, until I see a clear explanation otherwise, I'll assume there's no real issue with ZFS mirroring.

---

The WD Red range is intended for NASes (tho' as SSDs used as caches to NASes using hard drives seems to be far more common than the use of SSDs as the actual NAS data drives, I suspect the Red range is really targetting this usage).

As of May 2025, there are SN700 and SA500 Red drives available - the SA500 cards are an older and cheaper range (and the impression seems to be that the SN700s are a significant improval in reliability etc.).

So, you could get:

* A [500GB WD Red SN700](https://www.digitec.ch/de/s1/product/wd-red-sn700-500-gb-m2-2280-ssd-16770743)
* Three [2TB WD Red SN700](https://www.digitec.ch/de/s1/product/wd-red-sn700-2000-gb-m2-2280-ssd-17686802)

Note: there's also a 250GB SN700 but it's less than US$5 cheaper than the 500GB model.
