# README for HAN Rooster parser

`han-rooster-parser` is a bash script which gathers the [rather flaky
html page](http://rooster.han.nl/SchoolplanFT_AS/rooster.asp) titled "Roosterinformatie FSK/IAS" containing roster information for a specific lesson group and translates that in to a valid iCalendar Specification
(ICS) file[^icalspec] for easy automated consumption by any [iCalendar client application](#icalclients).

   **DISCLAIMER**: *The script is tested only for the rosters for the "Faculteit Techniek en Life Sciences" (FSK) / "Institute for Applied Sciences" (IAS) department of the Hogeschool Arnhem Nijmegen (HAN), The Netherlands. The use of the script is not endorsed or supported by HAN.* 

## Quick start

Download and extract the source using git:

```bash
git clone https://github.com/ronalde/han-rooster-naar-ics.git
# updates can be fetched using
git pull
```

Alternatively, [the tarball of the current master](https://lacocina.nl/mpd-configure) can be downloaded and unpacked in the current directory using `wget` and `tar`:
````bash
wget https://github.com/ronalde/han-rooster-naar-ics/archive/master.tar.gz -O - | tar --strip-components=1 -zxf -
````

After this the script can be run from the command line like this:

```bash
## convert roster for BML group H1F
bash han-rooster-naar-ics -f bml-h1f -u http://rooster.han.nl/SchoolplanFT_AS/rooster.asp
```

## Configuration

The script uses translation tables in csv files in the `./data`
directory for teachers and courses. When a teacher is not found in the
teachers csv file, the script notifies the user.

To get a list of all possible command line arguments, run:

```bash
bash han-rooster-naar-ics -h
```

It is possible to run the script using a configuration file, using the
path to the file as the only argument, like this:

```bash
bash han-rooster-naar-ics data/settings
```

The settings file should at least specify the `group_name` and
`roster_url` variables, and may contain custom variables like
`webserver_root` in the following example:

```bash
## data/settings
## settings file for han-rooster-naar-ics, sourced by script
roster_url="http://rooster.han.nl/SchoolplanFT_AS/rooster.asp"
group_name="BML-2x"
webserver_root="/var/www"
output_path="${webserver_root}/${group_name}_$(date +'%Y%m%dT%H%M%S').ics"
symlink_path="${webserver_root}/han.ics"
```

## Rationale

The source webpage is created by HAN using the *Gruber&Petters/Grupet GP Untis*[^gpuntis] timetabling/roster
software.  
Although Untis natively supports ICS-exporting, the school has chosen
to *not* use that important feature for students. Although they did
develop/buy a custom non-native 'webapp' for iOS and Android (called [han4me](https://frankthuss.wordpress.com/2011/08/23/han4me-de-nieuwe-roosterapp-voor-han-docenten-en-studenten-yam-hanicto/) ), they
chose to *not* make it accessible using a normal webbrowser. This
means students are left with the rather unusable 90's web site
for viewing this important data. 

Furthermore, the script is aimed at users/admins who prefer to keep
their data theirs and not 'share' personal information with all kinds
of privacy mangling companies.

**NOTE**: *Former HAN-student [Stephan Heijl](http://stephanheijl.com/) has done [something comparable in python](https://github.com/StephanHeijl/RoosterLoader) a few years back and even offers a live website for end users to select their group.*   


## System requirments

The script needs the following external commands, for which it checks
existence in the $PATH and exits when they aren't found:
* `xmllint`
* `tidy`
* `date`
* `curl`

## Design

Because the source webserver
only support HTTP POST parameters, the lesson group is specified
through the `group_id` field, which is looked up by the script on the
website using the `group_name`. The `group_name` field is only used
internally for constructing a file name for the resulting ics file.

The script gathers information by week, starting with the week in
which the script is run, and stops `weeks_ahead` weeks further. Each
planned course for a week is stored in a temporary text file,
containing `ICAL VEVENT` items. After the week files are
(re)generated, the script gathers all week files, including older
ones, adds a ICS header and footer and stores the result in a valid
ICS file.

The timezone of the ICS file and the events within are currently fixed
to CET/CEST with summer/winter time (daylight savings).

## Usage scenario

The scripts is meant to be run each morning before school starts, so
the information is current. The resulting ics file should be made
accessible through a webserver, so that all (remote) clients can
synchronise with the (updated) ics file. 

 **NOTE**:
	When making the ICS file accessible locally, one should remember
	that any changes made to the file (through a local calendar
	application) are overwritten by the script when it runs again.


## Client support for ICS through a custom webserver

<a id="icalclients"></a>The following clients are known to support subscribing to ICS files
using non-Google/Microsoft/Apple webservers:

- Thunderbird (all platforms)
- Evolution, GNOME Online Accounts (GOA) and gnome-calendar (Linux only)
- Kontact (Linux)
- Windows Calendare App
- Apple Calendar 

See: https://en.wikipedia.org/wiki/List_of_applications_with_iCalendar_support

## Footnotes and references

[^gpuntis]:
	https://www.untis.at/
	
[^icalspec]:
	http://www.kanzaki.com/docs/ical/

[^sourceurl]:
	http://rooster.han.nl/SchoolplanFT_AS/rooster.asp
