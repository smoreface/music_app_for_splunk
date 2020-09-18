# Music App for Splunk
The Music App for Splunk was written for a single instance Splunk deployment for personal use. It has not been tested on distributed instances.

## Download the Music App for Splunk
Get the app from Splunkbase here: https://splunkbase.splunk.com/app/4344/ 

## About the Music App for Splunk
I created this app for my own personal purposes, and am open sourcing it and sharing it on Splunkbase in the hope that others might also use Splunk to analyze their music data and share their dashboards, views, and add-ons with the community. 

## What's in the app
This app includes:

* Dashboards
* Saved searches
* Lookups

## Prerequisites
* Have a single-instance Splunk deployment. For details about how to install Splunk on your instance, see the Installation Manual. At the time of the writing, see [Install on Mac](https://docs.splunk.com/Documentation/Splunk/latest/Installation/InstallonMacOS) for the latest instructions to install on a Mac.
* Determine whether you will have a license for the Splunk instance. This app expects very low data ingest, so "Splunk Free" should be sufficient. See [About Splunk Free](https://docs.splunk.com/Documentation/Splunk/latest/Admin/MoreaboutSplunkFree) for more details.

## Install and configure the app
After you have a single-instance deployment of Splunk running, install the app using the in-product app browser. 
1. From Splunk Web, click the gear icon next to Apps.
2. Click **Install app from file**.
3. Locate the downloaded file and click Upload.

### Install recommended apps
The Music App for Splunk has no required app dependencies, but several recommended apps that will improve the app experience. 

* [Lookup File Editor](https://splunkbase.splunk.com/app/1724/) app, to easily maintain updates to concert and ticket purchase lookups. I would not use the Music App for Splunk unless you have this app installed. 
* [Splunk Common Information Model](https://splunkbase.splunk.com/app/1621/), to take advantage of future add-ons that contain a Songkick alert action, or modular inputs for music-relevant data sources like Last.fm or Spotify.
* [Splunk Dashboard Examples](https://splunkbase.splunk.com/app/1603/), to use the information and lookups included to enrich your visualizations. 

The app will still function without these apps installed.

## Add data to the Music App for Splunk
You can add data that will populate dashboards and views by updating the lookups (see next section) and/or uploading iTunes music library XML files to Splunk. No views will display data until you add and configure data.

### Index and search configuration
The app assumes an index called music. If you do not create that index, modify the `lastfm`, `lastfmearliest` and `itunes` macros to reflect the index you will use instead. Review those macros to see the sourcetypes that the macros contain and expect your data to conform to. If you use alternate sourcetypes, update the macros.

### Formatting the iTunes library XML
You must have iTunes and not Apple Music installed on your computer to access the Library.xml file. If you have already upgraded to Apple Music, you cannot use this configuration. 

The XML expected by the itunes_xml sourcetype included with the app must start with the <key></key> element.

Use the data contained in the Library.xml file in the iTunes directory of your computer.
The Library.xml file has two distinct sections: the tracks and the playlists. 
I have not tested if you can upload the entire Library.xml file to Splunk and index it that way. I don't think the playlist data would be extracted properly, and the data might not be supported in that format. Do that at your own risk! 
```xml
<key>166794</key>
		<dict>
			<key>Track ID</key><integer>166794</integer>
			<key>Size</key><integer>6253525</integer>
			<key>Total Time</key><integer>173654</integer>
			<key>Disc Number</key><integer>1</integer>
			<key>Disc Count</key><integer>1</integer>
			<key>Track Number</key><integer>1</integer>
			<key>Track Count</key><integer>1</integer>
			<key>Year</key><integer>2017</integer>
			<key>Date Modified</key><date>2018-01-26T04:45:47Z</date>
			<key>Date Added</key><date>2018-01-26T04:45:39Z</date>
			<key>Bit Rate</key><integer>256</integer>
			<key>Sample Rate</key><integer>44100</integer>
			<key>Release Date</key><date>2017-09-29T07:00:00Z</date>
			<key>Rating</key><integer>40</integer>
			<key>Artwork Count</key><integer>1</integer>
			<key>Persistent ID</key><string>1C5B351ACA3183D6</string>
			<key>Track Type</key><string>File</string>
			<key>Purchased</key><true/>
			<key>File Folder Count</key><integer>5</integer>
			<key>Library Folder Count</key><integer>1</integer>
			<key>Name</key><string>Never Too Late</string>
			<key>Artist</key><string>Alle Farben &#38; Sam Gray</string>
			<key>Album Artist</key><string>Alle Farben &#38; Sam Gray</string>
			<key>Composer</key><string>Sam Gray &#38; Frans Zimmer</string>
			<key>Album</key><string>Never Too Late - Single</string>
			<key>Genre</key><string>Electronica</string>
			<key>Kind</key><string>Purchased AAC audio file</string>
			<key>Sort Name</key><string>Never Too Late</string>
			<key>Sort Album</key><string>Never Too Late - Single</string>
			<key>Sort Artist</key><string>Alle Farben &#38; Sam Gray</string>
			<key>Location</key><string>file:///Users/<obfuscated>/Music/iTunes/iTunes%20Music/Music/Alle%20Farben%20&#38;%20Sam%20Gray/Never%20Too%20Late%20-%20Single/01%20Never%20Too%20Late.m4a</string>
		</dict>
```

Note: You can't rely on the <key> identifier that starts the song block to remain the same if you do intermediary file uploads, they appear to be regenerated.

### Add other music data
See below for information about adding information about concerts or ticket purchase data.

To add listen data or other library data, you must develop your own resources for that. 

* Write a Last.fm add-on for Splunk. I have one that I developed, but it needs improvements to be released.
* Write a Spotify add-on for Splunk. I have barely started work on this.
* Use online services to generate .csv files of events and upload those. This is relatively straightforward, and the best approach to get the data quickly, so long as you don't need it to be updated in real-time. Search "Spotify export" or "Last.fm export" to find services or scripts that do this for you. I don't endorse any particular options.

To get listening data, you must be using Last.fm to track your listens across services, or do basic metrics using the play_count in iTunes data. Spotify does not make listening data available via API.

## Identify whether the app is working
Different dashboards and panels depend on different types of information being available. Run the following search to identify the inline searches in the panels.

|rest /servicesNS/-/-/data/ui/views  splunk_server=local |table author eai:acl.app id eai:data title label|rex max_match=0 field=eai:data "\<query\>(?P<search_used>.*)\<\/query\>" |rename "eai:acl.app" as app_name |search search_used!="" app_name="music_app_for_splunk" |mvexpand search_used|fields - eai:data |table *

The searches that use an "itunes" macro rely only on iTunes library data. Those that rely on the concert or ticket lookup files will be apparent based on the searches used by the panels. 

Dashboards that have iTunes data reliance, and will display data after you have iTunes data uploaded:
* Dance Music Analysis
* iTunes Data Analysis


## Lookups included with and created by the Music App for Splunk
Several lookups are search-driven and will be created after the saved searches are run. All the saved searches and panels were created by an “admin” user and many expect the admin user to be present to run the search.

The following are created by scheduled searches:

* artist2mbid.csv
* artistthreshold.csv
* buythreshold.csv
* concert_threshold.csv
* earliestlistens.csv
* earliestlistensmbid.csv
* earliesttracklistens.csv
* headlinerthreshold.csv
* openerthreshold.csv
* track_release.csv
* trackbpm.csv
* trackratings.csv


The other lookups work as follows. I recommend that you use the Lookup File Editor app to manually maintain the details within. 

### concerthistoryparse.csv

Used to provide more semantic data than the other concert lookup. You are welcome to modify the app to combine this lookup with the other concert lookup.

| field         | description                                                                                                                                                                  | example value                  |
|---------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------|
| date          | Date of the concert in Month Day Year format. If you modify the format, modify the searches that use this lookup.                                                            | November 26 2018               |
| opener1       | Name of the band that opened the show. If there was only one opener, put their name here, otherwise put the first opener here. For festivals, put the earliest band you saw. | Alkaline Trio                  |
| opener2       | Name of the second opener, or another band that played the festival.                                                                                                         | Cursive                        |
| opener3       | Name of the third opener, or another band that played the festival.                                                                                                          | Rancid                         |
| headliner     | Name of the headlining band. If you only fill out one band name for the concert, use this one.                                                                               | The Aquabats                   |
| venue         | Name of the venue that the concert was at. For festivals, can also be a park name.                                                                                           | The Indepdendent               |
| city          | City where the venue is located.                                                                                                                                             | San Francisco                  |
| state         | Full name of the state, not the abbreviation. If you do use the abbreviation, modify the searches that use this lookup.                                                      | California                     |
| info          | Used to indicate additional detail about the concert. Currently used to specify "dj set", "festival", or "concert"                                                                       | dj set                         |
| festival_name | Name of the festival, if applicable.                                                                                                                                         | Treasure Island Music Festival |
| band5         | Additional field for a band name, usually for festivals.                                                                                                                     | Petit Biscuit                  |
| band6         | Additional field for a band name, usually for festivals.                                                                                                                     | Shallou                        |
| band7         | Additional field for a band name, usually for festivals.                                                                                                                     | Robyn                          |


### concerts.csv

Used to provide a multi-value artist field to searches.

| header | description                                                                                                             | example                   |
|--------|-------------------------------------------------------------------------------------------------------------------------|---------------------------|
| date   | Date of the concert, in Month Day Year format. If you modify the format, modify the searches that use this lookup.      | November 26 2018          |
| artist | Comma-separated list of artists at the concert. Multi-value field. No spaces. Use a trailing comma.                     | Fleece,Tokyo Police Club, |
| venue  | Name of the venue that the concert was at. For festivals, can also be a park name.                                      | Bottom of the Hill        |
| city   | City where the venue is located.                                                                                        | San Francisco             |
| state  | Full name of the state, not the abbreviation. If you do use the abbreviation, modify the searches that use this lookup. | California               |

### livestreams.csv

Used to provide a list of livestreamed DJ or concert performances watched online. 

| header | description                                                                                                             | example                   |
|--------|-------------------------------------------------------------------------------------------------------------------------|---------------------------|
| stream_date   | Date that the livestream first streamed online.      | September 18 2020          |
| watch_date | Date that you watched the livestream.                    | September 18 2020 |
| artist  | Artist name performing in the livestream. Does not support multiple artist names.                                     | Justin Jay        |
| length   | Length of time of the livestream, or amount of time that you watched the livestream, in HH:MM:SS format.                                                                                      | 00:45:00             |
| details  | Optional. Use for the title of the stream, livestream festival name, or whatever you choose. | Beatport ReConnect               |
| where_watched  | Site where you watched the livestream. | Twitch             |
| link  | Optional. Link to the archived version of the livestream. | https://www.youtube.com/watch?v=RvRhUHTV_8k             |


### ticketpurchase.csv

Used to keep track of various data relating to ticket purchases. 

| header        | description                                                                                                                              | example         |
|---------------|------------------------------------------------------------------------------------------------------------------------------------------|-----------------|
| purchase_date | Date that you purchased the ticket. Month Day Year format.                                                                               | November 9 2018 |
| onsale_date   | Date that the tickets went on sale, if known. Month Day Year format.                                                                     | November 9 2018 |
| concert_date  | Date that the concert is scheduled for. Month Day Year format.                                                                           | January 19 2018 |
| artist        | Artist headlining the concert. Should match with headliner and/or one of the multi-value artists listed for the concert in concerts.csv. | Mother Mother   |
| site          | Site where the ticket was purchased. Box office is also a valid option.                                                                  | Eventbrite      |
| promoter      | Promoter of the show, if known and relevant.                                                                                             | Noise Pop       |
| quantity      | Number of tickets purchased.                                                                                                             | 1               |
| cost          | Total cost of the ticket(s) purchased. Units omitted.                                                                                    | 22.06           |
| face_value    | Face value for the ticket(s) purchased. Units omitted.                                                                                   | 15              |
| discovery     | How you discovered the concert, if you remember.                                                                                         | songkick        |

### spotify*.csv

Used by the 2018_music and 2019_music dashboards. Specific lookup file names in use are: spotify2018.csv and spotify2019.csv, respectively.

These lookups are CSV representations of the artists and track names of the "top 100 songs" according to #spotifywrapped. You can use an online service to convert the playlist that Spotify gives you into a CSV representation and add it to Splunk using the Lookup Editor. 

| header        | description                                                                                                                              | example         |
|---------------|------------------------------------------------------------------------------------------------------------------------------------------|-----------------|
| artist | Artist of the track | Tourist |
| track_name | Song name of the track  | Elixir |
