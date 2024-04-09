---
title: "Capturing screen timelapses on Ubuntu (Gnome on Wayland)"
description: ""
date: 2024-04-09T08:58:51+02:00
draft: false
thumb: "images/blogpost.gif"
tags:
 - programming
 - gamedev
 - gamejam
 - linux
---

New [Ludum Dare](https://ldjam.com/) jam coming and I decided to record a screen timelapse, just like it was popular many years ago. I don't know if many people still do it, but it was something I really had fun watching before. I searched for tools to make this possible, but it seems that all of the options were for X11 server, and they do not work with Wayland. If you don't know what I'm talking about, it's just that Linux systems recently adopted another type of graphic display system and apparently most of the screen capture software was using the old X11.

One of the most recommended way to do it that works is using OBS with a low framerate capture. The problem with that approach is that it will record and produce a video with say 1FPS, then you'd still have to edit the video to change the final framerate to be 60, for example. Not only it is convenient, but it generates very huge video files even for a low framerate.

I wanted something to actually take screenshots and then assemble them together with ffmpeg, so I decided to write a custom solution. Gnome has a tool to grab screenshots, the [**gnome-screenshot**](https://github.com/GNOME/gnome-screenshot/). So I started writing a shellscript that would grab a screenshot on a fixed interval and then, whenever I decided to stop, it would generate a video with **ffmpeg** with all those screenshot files.

If you just want to use that for yourself, access the GitHub page of the project where all the instructions for use are: [Gnome Timelapse](https://github.com/CrociDB/gnome-timelapse). Stay here if you want to understand how I did it.

So far so good, but there was a problem: **gnome-screenshot** has this very annoying flash and shutter sound on every capture, and there's no config to remove it. I saw some people online removing the audio file so it would be silence, but the flash was still there. And, according to [this issue in the gnome project](https://gitlab.gnome.org/GNOME/gnome-shell/-/issues/3866), from years ago, nobody seems to be very active to add a configuration to this. But Internet is a good place (right?) and other people also had the same issue and shared their solutions: changing the source code to remove that flash and rebuilding the application, after all it's free software! I can't say that I didn't think of it, but I confess I was a bit lazy, but [this person shared the steps and it looked pretty simple to change](https://askubuntu.com/questions/854350/disable-gnome-screenshots-camera-flash-animation). So I cloned the repo and opened the file `src/screenshot-backend-shell.c`:


```c
// (...)
  if (screenshot_config->take_window_shot)
    {
      method_name = "ScreenshotWindow";
      method_params = g_variant_new ("(bbbs)",
                                     TRUE,
                                     screenshot_config->include_pointer,
                                     TRUE, /* flash */
                                     filename);
    }
  else if (rectangle != NULL)
    {
      method_name = "ScreenshotArea";
      method_params = g_variant_new ("(iiiibs)",
                                     rectangle->x, rectangle->y,
                                     rectangle->width, rectangle->height,
                                     TRUE, /* flash */
                                     filename);
    }
  else
    {
      method_name = "Screenshot";
      method_params = g_variant_new ("(bbs)",
                                     screenshot_config->include_pointer,
                                     TRUE, /* flash */
                                     filename);
    }
// (...)
```

It was just as simple as changing the lines `TRUE, /* flash */` to `FALSE` and then build, and it was the easiest project to build. Boom, now it can capture screenshots sneakily.

Back to the script, it basically works by having an infinite loop with a sleep time calculated by the interval of time set by the amount of captures per second. The first challenge was that the shell command `sleep` takes its argument in seconds, so anything lower than seconds is a floating-point number, however bash doesn't calculate floating-point divisions. At this point I was wondering why I didn't do it in Python or Perl, but I insisted. The way to solve it the shell way is using `awk` or any other script language:

```shell
cps=${2:-3} # gets the argument $2 and if it's not present, assign `3` to it
waittime=$(awk "BEGIN { printf(\"%.4f\", (1 / $cps)) }")
```

A whole `awk` script that outputs a four-digit floating-point number that is `1 / $cps`. Then it assigns that output to the shell variable `$waittime`. But not yet, it outputs with a `COMMA` as a decimal separator: `1 / 60 = 0,0167`, it uses the locale to determine that separator. So you need to specify before, with `LC_NUMERIC=C`:

```shell
cps=${2:-3}
LC_NUMERIC=C
waittime=$(awk "BEGIN { printf(\"%.4f\", (1 / $cps)) }")
```

Good. Now I wanted a way that the user could interact so they could stop the capture phase and generate the video with `ffmpeg`. The loop was basically sleeping for that `$waittime` interval and capturing the screen:

```shell
while [ true ]; do
    sleep $waittime

    filename="_timelapse_$(echo '('`date +"%s.%N"` ' * 1000000)/1' | bc).jpg"
    ./gnome-screenshot -f $filename
done

# now generate video
```

Then I found that there's a way to have _Ctrl+C_, the default way to cancel the current process, to exit the loop: `trap "break" INT`. It will break the loop rather than stopping the process. So now the loop can be:

```shell
while [ true ]; do
    trap "break" INT
    sleep $waittime

    filename="_timelapse_$(echo '('`date +"%s.%N"` ' * 1000000)/1' | bc).jpg"
    ./gnome-screenshot -f $filename
done

if [[ "$full" == "full" ]]; then
    generate_video $output

    if [ $? -eq 0 ]; then
        clean
    fi
fi

printf "\nDone\n\n"
```

I also implemented a simple command interface to interact with this script, so there's the help that shows the commands it supports: `capture`, `clean` and `video`:

```shell
$ /timelapse.sh

Gnome Timelapse

How to use:
         ./timelapse.sh COMMAND [parameters]

        This will start capturing. Press Ctrl+C to top the capturing and it will generate the final timelapse file.

Commands:
        - clean         Cleans all the capture files in this directory
        - capture       Captures the timelapse
                         - cps: Captures per second
                         - [full|novideo]: 'full' means it will take the captures and then produce a video in the end the 'novideo' will not produce the video and keep the captures
                         - filename:    Name of the output file
         - video        Makes the video of all the captures in the current directory

Example:
To capture a timelapse video of 1 frame per second, do:30
        ./timelapse capture 1 full out.mp4 
```

Took me a few hours to battle with shellscript, as it's something I usually avoid writing because I know how complicated that is. But now I have a completely functional timelapse tool that I will for sure use in this and the following [Ludum Dares](https://ldjam.com/). Let me know if this was useful to you! If you feel like there's an obvious improvement, feel free to create an issue or a pull request!

Ah, and of course I recorded a timelapse of the writing of this post:

![Screen timelapse of the writing of this post](images/blogpost.gif)

