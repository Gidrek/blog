---
layout: post
title:  "Install Elixir in Debian/Ubuntu using precompiled packages"
date:   2019-10-25 15:00:00 -0600
description: Install Elixir in Debian/Ubuntu using precompiled packages
categories: Elixir
---

Today I had the necessity to update a production server using Debian to Elixir 1.9.2. 
The update from Elixir 1.6 to 1.9.2 was really easy but I will put my steps if anybody has the same requirement.

We will need three packages installed in our server:

* wget
* unzip
* Erlang 20 or up

## Installing Elixir

First, we need to go to our tmp folder to download the precompiled package, create a directory to unzip the package and then unzip it.

```
cd /tmp
wget https://github.com/elixir-lang/elixir/releases/download/v1.9.2/Precompiled.zip
mkdir elixir_precompiled
mv Precompiled.zip elixir_precompiled/
unzip Precompiled.zip
```

Now we have all the necessary stuff to get starting to install. We only need to move the bins and library from our temporal folder to **/usr/local**, 
or another directory. Just you need to remember to add the directory where you going to put Elixir to your system **PATH**.

```
cd bin
sudo cp elixir elixirc iex mix /usr/local/bin/
cd ..
cd lib
sudo cp -R * /usr/local/lib
cd ..
cd man
sudo cp * /usr/local/man
```

With those steps, we installed Elixir and the manuals for help.

As last check, we can see the elixir version in our system

```
Erlang/OTP 22 [erts-10.5.2] [source] [64-bit] [smp:4:4] [ds:4:4:10] 
[async-threads:1] [hipe] [dtrace]

Elixir 1.9.2 (compiled with Erlang/OTP 22)
```

And that is all. We have the last version installed in our system