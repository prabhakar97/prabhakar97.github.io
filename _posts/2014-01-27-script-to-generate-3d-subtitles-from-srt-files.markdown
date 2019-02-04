---
layout: post
title: "Ruby script to generate 3D subtitles from srt files"
date: 2014-01-27 00:09
comments: true
tags: [ruby]
---
So, you've got a bunch of 3D movies with SRT subtitles which your 3D TV can play from a USB storage and show the subtitles correctly but can not if you are running the movie on your laptop and sending the display to TV over HDMI; for obvious reasons.
The solution is to use *.ass* format subtitles with your media player(I recommed mplayer/smplayer). Just run this script from a directory that contains all your movies in separate directories under that directory. Make sure, `srt` file is present in each 3D movie directory and has the same name as the movie file. Also make sure, the directories for 3D movies have 3D in their names somewhere. Moreover, you must have the program `sub3dtool` installed. Get it from [here](https://code.google.com/p/sub3dtool/). After you run the script, all your 3D movie directories will be populated with .ass files automatically. Remember, VLC media player does not play well with `.ass` subtitles. It skips many dialogues.

{% highlight ruby %}
Dir.entries(".").each do |file|
  if file.include? "3D"
    puts "3D movie found - #{file}. Going ahead to generate .ass subtitle for it"
    Dir.entries(file).each do |file1|
      if file1.include? ".srt"
        puts "Found srt file: #{file1}"
        outfile_name = "#{file1[0,file1.length-3]}ass"
        puts "Out file name : #{outfile_name}"
        `sub3dtool "#{file}/#{file1}" --3dsbs -o "#{file}/#{outfile_name}"`
        puts "Generated successfully"
      end
    end
  end
end
{% endhighlight %}

All the movie directories will be populated with corresponding *.ass* files. Now just run it by `mplayer -ass movie_file_name`. Use `j` to cycle through subtitles, `f` for fullscreen, `q` to quit mplayer.
