---
title: "setup self hosted CI server"
date: 2023-05-10
mathjax: false
draft: false
series: infrastructure
tags: [github, infrastructure]
---

In this post, I will set up a GitHub runner for automatically testing Julia + KSP application.

To do this, I need to set up:

1. A working GitHub runner.
1. Kerbal Space Program with relevant mods.
1. Automatic shutdown and startup of the system. (will be covered in later post)

## GitHub runner

[Documentation](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners) has nice instruction on how to set up a runner. First, I install prerequisites listed in [readme](https://github.com/actions/runner), do a distribution update, and reboot.

```shell
rhahi@builder ~> sudo zypper in lttng-ust libopenssl1_1 krb5 zlib libicu60_2
...
'krb5' is already installed.
No update candidate for 'krb5-1.19.2-150400.3.3.1.x86_64'. The highest available version is already installed.
'zlib' not found in package names. Trying capabilities.
'libz1' providing 'zlib' is already installed.
'lttng-ust' not found in package names. Trying capabilities.
Resolving package dependencies...

The following 2 packages are going to be upgraded:
  libopenssl1_1 openssl-1_1

The following 7 NEW packages are going to be installed:
  libicu60_2 libicu60_2-ledata liblttng-ust0 liblttng-ust-ctl4 liblttng-ust-python-agent0 liburcu-devel
  lttng-ust-devel
```

The exact match for the requirements are not found, but all capabilities are met.

To install runner, I only need to copy and paste commands given in GitHub.

![config](/images/runner/configure.png)

To configure it to start on boot, there is a convenient script that sets everything up. [Guide](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/configuring-the-self-hosted-runner-application-as-a-service)

Finally, I also need to configure the runner to only run on private repositories. I can do this by going into runner groups, and then limit access to selected repositories and workflows.

Now it's ready to take jobs.

## KSP

### Steam

OpenSUSE wiki has some information about setting up [Steam](https://en.opensuse.org/Steam). It seems like I can use [SteamCMD](https://software.opensuse.org/package/steamcmd).

According to [Valve's guide](https://developer.valvesoftware.com/wiki/SteamCMD), I create a steam user. Since installation step is already done, I should be able to run `steamcmd` immediately. However, I get an error on startup.

```shell
steam@builder ~ [254]> steamcmd
Redirecting stderr to '/home/steam/Steam/logs/stderr.txt'
ILocalize::AddFile() failed to load file "public/steambootstrapper_english.txt".
[  0%] Checking for available update...
[----] Download Complete.
```

So instead of using ones in OpenSUSE, I used the Valve guide directly.

```shell
steam@builder ~> cd ~/Steam
steam@builder ~/Steam> curl -sqL "https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz" | tar zxvf -
steam@builder ~/Steam> ./steamcmd.sh
Steam>
```

And this works.

### Kerbal Space Program

After setting up installation directory and login, and I can proceed to KSP installation. I looked up the app ID from [steamdb](https://steamdb.info/app/220200/).

```shell
-- type 'quit' to exit --
Loading Steam API...OK

Steam>force_install_dir ./kerbal
Steam>login rhahi
Logging in user 'rhahi' to Steam Public...OK
Waiting for client config...OK
Waiting for user info...OK

Steam>app_update 220200 -beta 1.12.3
```

I chose version 1.12.3 because that is the version supported by [Realistic Progression 1](https://github.com/KSP-RO/RP-0)

```shell
steam@builder Xvfb :99 -screen 0 1024x768x24 &
steam@builder ~/a/kerbal_vanilla> echo $last_pid
12423
steam@builder ~/a/kerbal_vanilla> DISPLAY=:99 ./KSP.x86_64 -force-wayland &

```

The game is installed, but it does not properly run. I suspect that the culprit is lack of some graphics drivers. Not knowing which one I should have exactly, I decided to just install a desktop environment, Steam, and then downloaded and installed KSP.

```shell
sudo zypper in patterns-lxde-lxde
sudo shutdown -r now
startx  # start desktop environment
sudo zypper in steam
steam
```

After logging in, installing KSP, changing beta, (I couldn't click beta drop down menu, but I learned that I can [ctrl+click as a workaround](https://github.com/ValveSoftware/steam-for-linux/issues/9273).) I finally launched the game. Steam downloaded and installed extra Vulcan drivers. I guess that is what I was missing.

![the game](/images/runner/ksp.png)

Yes!

I plugged in mouse and fiddled with some settings to make sure that the game is on the lowest possible graphics settings. I also added mods, kOS and kRPC, for programming. There might be a few more that would be useful, but this is the bare essentials that I need.

## More to come...

Next step would be to

- Integrate KSP into CI workflow
- Test high-power Julia programs using builder
- Power management

However, these are future problems that should be solved as they arise. For now, let's just celebrate.
