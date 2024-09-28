+++
title = 'Compiling Heads on WSL'
date = 2024-09-28T18:10:17+01:00
draft = false
+++

One of my big projects has been getting an X220 to run with the heads firmware. It was tough, but I eventually pulled it off with some shot-in-the-dark reading, and more than a few swear words into it. Thankfully, I can distill the build instructions for heads, and make it happen, mostly due to the piss-poor seperation of instructions for developers and end users.

## Okay, what the hell is Heads?

Heads is a more secure firmware for certain laptops and boards. It does have GPG (I can see people hiding behind their keyboards in fear at this. I know, I don't like it either), but for the most part, it's intended to give some layer of security, in particular for Qubes and TAILS users.

## Right, so why should I build it?

Heads has made headway (gettit?) into reproducible builds. That way, no matter who builds it, it's going to be exactly the same every single time. It only changes when the repo changes, and that's pretty much it. It's intended to make sure that it hasn't been tampered with.

## So how do I do this, then?

You will need access to Linux in some way. In this guide, I'm using WSL. Not the best idea, I know, but it's easier than setting up a VM and wasting time trying to copy it out for flashing.

So first off, you'll need git to pull the repo. Just run:

	git clone https://github.com/linuxboot/heads.git

And you'll get the repo downloaded. If you don't have git installed, use your package manager to install it. You'll also need docker for this too. [All the instructions for various Linux Distros, even those on WSL, can be found here.](https://docs.docker.com/engine/install/)

Got it? Good. Now ignore the Nix instructions. You won't be needing them. Instead, all you need to run is:

	docker run -e DISPLAY=$DISPLAY --network host --rm -ti -v $(pwd):$(pwd) -w $(pwd) tlaurion/heads-dev-env:latest -- make BOARD=x230-hotp-maximized

Of course, change the board variable to suit what you have. There are boards listed in the boards directory, so have a look, and see what you need. This should compile it together and give you a working image. If there are more things that need to be done, just follow the onscreen instructions.

## Now what do I do with the rom file I have?

You flash it. To this, you will need an external flasher. You could use a CH341A-based flasher, but you need to get the right one, to prevent damage to your BIOS chip. You could also use a [Raspberry Pi Pico](https://codeberg.org/libreboot/pico-serprog) as one, or failing that, a regular Raspberry Pi. That being said, they do work with cops, and it's not out of the realms of possibility for backdoors to be in the sofware, so be careful.

