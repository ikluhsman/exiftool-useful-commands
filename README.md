# ExifTool Useful Commands

I have always used exiftool to organize my digital camera photos, and I found myself writing down useful commands in my notes or documents. So I decided to put it on GitHub to share and keep it update for myself as well.

I have a couple DSLR cameras and a couple sport cameras, go pros, and drones. So I needed a good way to collect them all on my hard drives and store them. 

I don't take digital camera photos every day, but I do take them regularly during any given year, so that influences the folder structure  I use to store them on my disks.

## REFS
This document references the application [ExifTool](https://exiftool.org) written by [Phil Harvey](https://exiftool.org/#background). I have mostly picked up the commands below from forum posts from the [ExifTool Forum](https://exiftool.org/forum).

## WRITING TAGS

### Set the system file creation/modify date using the DateTimeOriginal tag.

```
exiftool "-FileModifyDate<DateTimeOriginal" "-FileCreateDate<DateTimeOriginal" DIR
```

## REGEX

### Extract only the parent directory name.
```
${directory;s(.*/)()}
```

### Last 4 characters of the filename in case you need to add a ImageNumber or Sequence number for a tag.
```
${filename;$_=substr($_,-8,-4)}
```

## MAKERNOTES
I have examined any special tags included from cameras that I own, here are some notes about these.

### Remove all makernotes:
```
exiftool -overwrite_original -makernotes:all= .
```
> [!warning]
> You might not want to remove the makernotes since it is the *extra* custom data written in the image about your specific camera and its setup at the time of the photograph. For example, Nikon DSLR's write a "SubSecTime" tag (see [Nikon DSLR Makernotes](#nikon-dslr-makernotes) below).

### Nikon DSLR Makernotes
In Nikon DSLR Cameras, there is a SubSecTime tag that records the time in fractions of a second in case the shutter is clicking at a rate of speed faster than 1 second. 

When the image gets recorded, the DateTimeOriginal tag will be exactly the same. If you attempt to rename images using a timestamp with exiftool, it would exclude the duplicate file names when it came across images with the same DateTimeOriginal tag. So you'll need to add some sequencing numbers or something so you don't wind up excluding files that were taken within the same 1-second timeframe.

### Olympus Sport Camera Makernotes
In Olympus sport cameras, the filename is the distinction between these kinds of images taken within the same second, so be cautious of altering the filenames especially if you KNOW you used a continuous shutter! You could still use the DateTimeOriginal tag but make sure to add a sequence using the below sequencing variables.

## WINDOWS COMMANDS

### Remove all empty subfolders from a path
This isn't really an exiftool command, but it is useful when you're processing photos using exiftool to clean up empty folders.
```
for /f "delims=" %d in ('dir C:\FullPathToTargetFolder /s /b /ad ^| sort /r') do rd "%d"
```
## COPY/MOVE

### Increment a file number 
If there are files that have the same name, add a number that is incremented for each duplicate file name. Each duplicate file starts at 1, so this is NOT to add a sequence number to EVERY file. Useful if datetimeoriginal is exactly the same between two files. A use case might be where you've used a continuous shutter of less than 1 second per photo, thus making renaming impossible using the datetimeoriginal as a unique file name.
```
%n.%c
```

### Testing the FileName command
Use the ```-TestName``` tag in place of ```-FileName``` to test renaming/moving files.

### Rename all files in place
Renames files to their original date/time stamp formatted like YYYYmmDD-HHMMSSpp.ext (20080412-042159PM.jpg)
```
exiftool "-FileName<${DateTimeOriginal}.%e" -r -d "%Y%m%d-%I%M%S%p" .
```

> [!WARNING]
> If you've used a continuous shutter of less than 1 second this might skip files because they would have duplicate names due to the same datetimeoriginal formatting!
>
> On some cameras there is no tag that shows a sequence number or SubSecTime to display the milliseconds the photo was taken at. Olympus sport cameras must rely on filenames to sequence photos with duplicate time stamps. Nikon DSLR cameras have a SubSecTime tag so you don't have to rely on file names for sequencing. You could use time since epoch or just name the files using the SubSecTime in the filename to avoid duplicates.

### Move files to a new directory structure.
The destination is the folder where you run this command. The directory structure will go by year, then by some event that happened in that year, then in that event the make and model of each camera used for that event will have a folder containing its own photos. The %f.%e preserves the original filenames and -r performs a recursive search on your source_directory.
```
exiftool "-FileName<${DateTimeOriginal}\${Comment}\${Make} ${Model}\%f.%e" -r -d "%Y" source_directory
```

### Move images into a Year\Month-Year - Comment\Camera Make & Model\Files
Move files using the comment as a subject or memorable event or tag for the photos in the folder. In this case the original parent directory name was preserved as a folder under the camera's folder. (See [REGEX](#REGEX's) above)
```
exiftool -o . "-FileName<${DateTimeOriginal}${Comment}\${Make} ${Model}\${directory;s(.*/)()}\%f.%e" -r -d "%Y\%m-%Y - " dir
```
This uses the directory name and strips everything behind the last forward slash in the directory's string, returning only the immediate parent directory's name without the rest of the heirarchical path, i.e. "/path/to/parent/file.jpg" would just return the string "parent".

### REPAIR images metadata reads all data, and writes it back - without makernotes. This fixes most common issues:
```
exiftool -overwrite_original -all= -tagsfromfile @ -all:all -unsafe -icc_profile -ExifVersion=0232 -makernotes= .
```
### Display info for a certain tag:
```
exiftool -r -Gl -a -s -Comment "dir"
```
### Write a comment to files in a folder:
```
exiftool -r "-comment=Testing comments!" dir
```
NOTE: For NIKON raw images use XPComment
```
exiftool -r -overwrite_original "-XPComment=MyCOmment" dir
```
### Send files in dir folder to a directory named from the camera make and model, separated by a space. Use the TestName flag to do a dry run and only show the changes that would be made. Use FileName when you're ready to do the actual move/rename operation.
```
exiftool '-TestName<${Make} ${Model}\%f.%e' dir
exiftool '-FileName<${Make} ${Model}\%f.%e' dir
```
### Send files in dir folder to directory based on DateTimeOriginal tag from camera:
```
exiftool '-Directory<DateTimeOriginal' -d %Y%m%d dir
```
OR recursively in place:
```
> exiftool '-TestName<%d\${Make} ${Model}\%f.%e' -r dir
> exiftool '-FileName<%d\${Make} ${Model}\%f.%e' -r dir
```
### Send files to folder with make and model of camera, followed by year with subfolders of dates and pics for that camera.
```
exiftool '-FileName<${Make} ${Model}\${DateTimeOriginal}\%f.%e' -d %Y\%Y%m%d -r ..\..\!raw\
```
### Add a comment to the metadata using the immediated parent directory name ".\dir" and recurse source files
```
exiftool '-comment<Directory' -r .\
```

## Python strftime()

> [!NOTE]
> Below is from the [man page for python's strftime() function](https://www.man7.org/linux/man-pages/man3/strftime.3.html).

<!-- Directive	Meaning	Example
%a	Abbreviated weekday name.	Sun, Mon, ...
%A	Full weekday name.	Sunday, Monday, ...
%w	Weekday as a decimal number.	0, 1, ..., 6
%d	Day of the month as a zero-padded decimal.	01, 02, ..., 31
%-d	Day of the month as a decimal number.	1, 2, ..., 30
%b	Abbreviated month name.	Jan, Feb, ..., Dec
%B	Full month name.	January, February, ...
%m	Month as a zero-padded decimal number.	01, 02, ..., 12
%-m	Month as a decimal number.	1, 2, ..., 12
%y	Year without century as a zero-padded decimal number.	00, 01, ..., 99
%-y	Year without century as a decimal number.	0, 1, ..., 99
%Y	Year with century as a decimal number.	2013, 2019 etc.
%H	Hour (24-hour clock) as a zero-padded decimal number.	00, 01, ..., 23
%-H	Hour (24-hour clock) as a decimal number.	0, 1, ..., 23
%I	Hour (12-hour clock) as a zero-padded decimal number.	01, 02, ..., 12
%-I	Hour (12-hour clock) as a decimal number.	1, 2, ... 12
%p	Locale’s AM or PM.	AM, PM
%M	Minute as a zero-padded decimal number.	00, 01, ..., 59
%-M	Minute as a decimal number.	0, 1, ..., 59
%S	Second as a zero-padded decimal number.	00, 01, ..., 59
%-S	Second as a decimal number.	0, 1, ..., 59
%f	Microsecond as a decimal number, zero-padded on the left.	000000 - 999999
%z	UTC offset in the form +HHMM or -HHMM.	 
%Z	Time zone name.	 
%j	Day of the year as a zero-padded decimal number.	001, 002, ..., 366
%-j	Day of the year as a decimal number.	1, 2, ..., 366
%U	Week number of the year (Sunday as the first day of the week). All days in a new year preceding the first Sunday are considered to be in week 0.	00, 01, ..., 53
%W	Week number of the year (Monday as the first day of the week). All days in a new year preceding the first Monday are considered to be in week 0.	00, 01, ..., 53
%c	Locale’s appropriate date and time representation.	Mon Sep 30 07:06:05 2013
%x	Locale’s appropriate date representation.	09/30/13
%X	Locale’s appropriate time representation.	07:06:05
%%	A literal '%' character.	% -->

## Add CreateDate Exif Property and Copy DateTimeOriginal Exif Property Value into It

Useful for if many pictures do not have the CreateDate exif-property, but do have the DateTimeOriginal exif-property. If you want the CreateDate exif-property to have the same value as the DateTimeOriginal exif property:

```
exiftool -overwrite_original '-createdate<datetimeoriginal' -r -if '(not $createdate and $datetimeoriginal)' <your directory>
```

## My process for renaming photos

### TLDR
1. Group photos into folders using comment/memorable-title.
    > [!NOTE]
    > This will give us an identifier for each image to use in our directory structure.
2. Run ```exiftool '-comment<Directory' -r .\``` to add a comment tag to each photo using the directory name you put them in.
    > [!NOTE]
    > Any file without a comment tag will not receive the tag, such as raw images.
3. Run ```exiftool '-XPComment<Directory' --ext nef -r .\``` to add the custom comment to the RAW images.
    > [!NOTE]
    > The 'nef' part is the Nikon raw file extension, use your camera's raw file extension here.
4. Run ```exiftool '-FileName<${DateTimeOriginal}\${Comment}\%f.%e' -r -d %Y dir``` to move files with the Comment tag to the current path.
    > [!NOTE]
    > My .MOV files include the Comment tag, so they are moved using this command.
5. Run ```exiftool '-FileName<${DateTimeOriginal}\${XPComment}\%f.%e' -r -d %Y dir``` to move files with the custom XPComment tag.
    > [!NOTE]
    > My RAW images use the XPComment, you might have to find the comment field for your type of raw image.

### My Process Explained

I want to store my photos in a directory structure that looks like this: ```YEAR\EVENT-OR-TITLE\OriginalFileName.ext```. This helps me when browsing my folder structure, since the comment reminds me of what the photos are taken for in a particular year.

To accomplish this, I add the "Comment" tag and "XPComment" raw image tag to each image, and then use that comment tag with exiftool to create the directory structure. Some directory names or "comments" I use are the name of an event like "Waterton Up and Down Kayaking Race", or maybe a Wedding "Rick & Julie's Wedding", or just "Practice Shots" for random junk. Raw pictures and video clips are stored in the same folder, they use a different file extension so I just include them together.

First I group my photos into folders named by an event or a memorable title. Then I use that parent folder name to add a comment tag to each file. Then I can move the files into the structure I wanted from above.

Also I leave the video clips mixed in with the photos to keep the collections together.

After placing photos in event/memorable-comment grouping within directories, I run exiftool to add a comment tag to for each image using the parent directory name:

```
exiftool '-comment<Directory' -r .\
```

We then run it again for files that have the XPComment tag or raw images with another custom tag:

```
exiftool '-XPComment<Directory' --ext nef -r .\
```

Now all files have a comment tag, and we can move them into their final destinations. First we move the files with the "Comment" tag:

```
exiftool '-FileName<${DateTimeOriginal}\${Comment}\%f.%e' -r -d %Y dir
```

Then we move files with the "XPComment" tag:

```
exiftool '-FileName<${DateTimeOriginal}\${XPComment}\%f.%e' -r -d %Y dir
```

> [!NOTE]
> Use the XPComment tag for RAW photos, or you may have to find the correct makernotes comment tag for the raw pictures for your camera.
