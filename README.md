#Public Radio Podcast Measurement Filter
In January 2016, the Public Radio Podcast Measurement Guidelines [Version 1.1](https://bit.ly/podcastguidelines) were issued. These Guidelines specify criteria for filtering web server log files in order to produce realistic counts of unique show downloads.

This repo contains a script to perform the specified filtering using standard Linux shell utilities, and another that acts as an example of how useful statistics can be simply produced from the output of the former.

##Why?
Standard web analysis tools such as AWStats, Webalizer and GoAccess are inappropriate for producing accurate download counts for media like podcasts, which may be streamed in chunks and where multiple differing types of access request can be made for a single file. These problems are well explained in a [video from Blubrry](http://www.powerpresspodcast.com/2016/01/23/deep-dive-on-blubrry-podcast-statistics/).

##What?
_prpm-filter.sh_ acts as a filter, so is most useful as part of a pipeline. It accepts log lines from stdin and writes data from those that meet the PRPM Guidelines to stdout. It takes a POSIX-style regular expression as its sole argument, with only URLs matching that regex being processed and included in the output.

Data output consists of four tab-separated fields: the show URL, download date (YYYY-MM-DD), IP address of the downloader, and the User Agent string of the downloader.

##Usage
Basic sample usage to produce a unique download count for each show matching the regex:

`# cat access.log | ./prpm-filter.sh "Episode[[:digit]]{1,3}\.(mp3|ogg)$" | cut -f1 | sort | uniq -c`

See _prpm-stats.sh_ for an example of how to process rotated logs, and to use the output from _prpm-filter.sh_ to produce unique counts for downloads over time, counts and geographic location of your unique downloaders and so on.

##Web Log Formats and Rotation
_prpm-filter.sh_ will happily process web logs in the standard _combined_ format, used both by Apache and Nginx. However, one of the PRPM Guidelines requires the filtering of lines on the basis of specific byte-range data, which is not recorded in the combined log format by default. If you want your generated statistics to be completely PRPM Guideline compliant, you will need to alter your existing log format (or write out an additional custom log) with the byte-range request data suffixed. That is, a [format](http://httpd.apache.org/docs/current/mod/mod_log_config.html) of:

`%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-agent}i\" \"%{Range}i\"`

The IAB is [working towards](http://www.iab.com/councils-committees-task-forces-and-working-groups/?key=a0Gj000000UVMNHEA5) producing download measurement guidelines that will likely become standard across the sector once released (with slow progress on these being cited by some as the reason that Public Radio released their own guidelines independently). Regardless, it is likely that any IAB measurement criteria will not differ significantly from those implemented here, and will also require the collection of byte-range request data. So the addition of such data to your log files is highly recommended for future compatibility.

Of also note is that in many Linux distros, web log rotation is configured to retain only a year's worth of data. If you'd like to generate meaningful historical statistics using this script, refer to your distro's documentation for details on how to change this behaviour.
