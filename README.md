# Imagefap Favorites

Python code that downloads an entire Imagefap favorites image
(picture folder) collection, de-duplicates using multiple methods,
and helps organize them.

Started in January/2023, by Daniel Balparda.

## License

Copyright (C) 2023 Daniel Balparda (balparda@gmail.com).

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program. If not, see http://www.gnu.org/licenses/gpl-3.0.txt.

## Setup

This software was developed and tested in a linux-like environment,
but hopefully it can be run and used in other python-compatible
environments with little to no changes.
To install execute these commands inside the directory where you
want the GIT packages to be installed:

```
$ sudo apt-get install python3 python3-pip
$ sudo pip3 install -U click sanitize_filename Pillow imagededup Django

$ git clone https://github.com/balparda/baselib.git
$ git clone https://github.com/balparda/imagefap-favorites.git
```

The project has __more than 90%__ test coverage and its
code is pretty readable, but is still being developed for
more features and flexibility.

## User Journeys, A Simple Guide

### _"I just want to download a specific favorite album."_

For every album you want, do something like this:

```
./favorites.py get --user dirty999 --folder "my pics" --output "~/some-dir/"
```

That would read favorite album _"my pics"_ for user _"dirty999"_. If you
don't specify an explicit output it will go into `~/Downloads/imagefap/`.

### _"I want to read favorite albums from any number of users and browse them offline."_

Generate a database by reading the favorite albums for each user, like this:

```
./favorites.py read --user username1
./favorites.py read --user username2
[... etc ...]
```

Since we didn't give any specific location (`--output` flag), the database
will be placed in `~/Downloads/imagefap/`.
Those examples would read all favorites for users _"username1"_ and _"username2"_.
If you don't want all the albums for an user, just a specific one, add
the `--folder` flag. Example:

```
./favorites.py read --user dirty999 --folder "my pics"
```

Read as many users and folders as you want. They will be in the
database, thumbnails will be automatically generated, duplicates will
be found, many error conditions will be handled and corrected for, etc.
After the download, we search for duplicates using a hybrid intersection
of perceptual, average, differential and wavelet hashes, plus a
convolutional neural network (CNN). Duplicate images picked up by these
methods will be consolidated for the user, who can later decide which
images to keep. By default it will skip any album recently downloaded
in the previous 3 days (72h). This is usually very helpful to save
time when you have to restart the fetching, as it will gladly remember
the work done and save time. If you know something changed and want
to force it to download anyway, just add the `--force` flag.

Now you want to see the data and admire the downloaded albums and
images, so you do:

```
./process.py run
```

This will start the web app ___in you local computer only___, and
to see it you just leave `./process.py` running and
visit http://127.0.0.1:8000/viewer/ in your browser. If your
database is not in the default location (`~/Downloads/imagefap/`)
you should provide the `--dir` flag to the `./process.py` command.
The web app is not beautiful, for sure, but it is simple, fast, gets
the job done, does not waste your time, and 100% offline.
You can see all users, all favorite albums, images, duplicates,
and in the image list you can apply some filters.
To stop the web app go to `./process.py` and use `CTRL-C` to stop it.

The web app is ___not___ meant to be safe or to serve outside your
local computer, so please don't do that! Also, ___do NOT simultaneously
run the web app and other concurrent calls to `favorites.py` or `process.py`
or you might lose data!___

## Usage of `favorites.py`

Run `./favorites.py --help` for an options and flag summary.

### `favorites.py GET` command - _Simple Save of Favorite Images Gallery_

The basic download command is `get`. Use it if you know the user ID
(or user name), picture folder ID (or picture folder name),
and want those saved to a given (or default) output directory. In the absence
of an explicit directory, will default to `~/Downloads/imagefap/`.
This command defaults to preserving file names and just straight
saving them to disk instead of saving them as blobs. It will create
a simple database file that, if kept in the directory, will avoid
having to do repeated work for known images. (You can disable the
database file creation with the `--no-db` option, but there really
is no need to, as the file is typically tiny: _much less than 0.5%_
of the size of the downloaded images and usually less than 1Mb.)
If you use the database you will save a lot of time for repeated
uses or if your connection is broken. The system will remember
the images you have (and skip them) and will remember the pages that
were already seen (a huge difference for very big albums).
You do ___not___ have to worry about missing images because the
duplicate detection here uses `sha256` and will thus _only_ skip files
that are _exactly the same_. No possible mistake here.

This example will find the user _"dirty999"_ and the favorite
gallery _"my pics"_ and download all images to the default directory
(`~/Downloads/imagefap/`). The names are case insensitive and the
script will figure out the correct casing as it loads the IDs:

```
./favorites.py get --user dirty999 --folder "my pics"
```

This example will get folder 5678 of user 1234 and place them in
the given directory `~/some-dir/`:

```
./favorites.py get --id 1234 --folder 5678 --output "~/some-dir/"
```

If you use the `get` command with multiple favorite galleries and
leave all of them to be saved to the same output directory (either
default or some explicit other one), then exact duplicate images
(again by `sha256`) will ___not___ be saved twice, and the file
name will be the one give for the first time it was saved.

### `favorites.py READ` command - _Feed the Database!_

This `read` command is for when you want to do more than just download
images from one (or a few) galleries. This will store the images as blobs
and will feed the database with data. It is _not_ meant for immediate
consumption. The idea is to capture data for processing.

With this command duplicates will be found. We search for duplicates
using a hybrid intersection of perceptual, average, differential
and wavelet hashes, plus a convolutional neural network.
Duplicate images picked up by these methods will be consolidated
for the user.

This example will find the user _"dirty999"_ and the favorite
gallery _"my pics"_ and read all images into the database in
the default directory (`~/Downloads/imagefap/`). The flags behave
the same as for the `get` command:

```
./favorites.py read --user dirty999 --folder "my pics"
```

For the `read` command you may instruct it to find all favorite
image galleries in the user's favorite:

```
./favorites.py read --user dirty999
```

The `read` command will also generate thumbnails for the images,
that will be used to display them in the web app (see `run` command of
`process.py`).

## Usage of `process.py`

Run `./process.py --help` for an options and flag summary.

### `process.py STATS` command - _See Database Statistics_

The `stats` command prints interesting info on the database's metadata.
This is intended to be used on a database that has been already constructed.
This example will print stats for a database in `~/some-dir/`:

```
./process.py stats --dir "~/some-dir/"
```

The exact same stats are showed in the home page at
http://127.0.0.1:8000/viewer/ (see `process.py RUN` command).

### `process.py PRINT` command - _See All The Things!_

The `print` command will do a "pretty" print of useful database metadata
for a closer inspection. Can print a lot for a big database!
This is intended to be used on a database that has been already constructed.
This example will print all (relevant) data in a database located in
the default directory (`~/Downloads/imagefap/`):

```
./process.py print
```

### `process.py RUN` command - _Run Local Web App: Inspect and Change Data_

The `run` command will start a strictly local web app with the database
data that will allow you to navigate and view the data and do some tasks.
The web app will be in http://127.0.0.1:8000/viewer/ and is built for
function and not for looks.

___Do NOT simultaneously run the web app and other concurrent calls
to `favorites.py` or `process.py`! You might lose data!___

```
./process.py run
```

The app will list the users, the favorites that were downloaded for the
user, the tags you created, the duplicates, etc.

If you are developing this software, run it with the `--development` flag
so that the server (Django) will automatically reload the app when you
save your files, but be aware the startup will run twice. Definitively
ignore this if you are just using the app.

## Storage

___You don't need to read this section unless you are a developer
for this utility.___

### Default Storage (`get` command)

If no conflicting options are provided, the following storage of
files will be adopted for the `get` operation:

```
~/                                       ==> User root dir
~/Downloads/imagefap/                    ==> App root dir
~/Downloads/imagefap/imagefap.database   ==> serialized metadata file (see below)
~/Downloads/imagefap/original-sanitized-name-1.jpg   ==> image
~/Downloads/imagefap/original-sanitized-name-2.gif   ==> image
```

### Database Storage (`read` command)

If no conflicting options are provided, the following storage of
files will be adopted (`read` command), with the objective of
facilitating image tagging and re-organizing:

```
~/                                       ==> User root dir
~/Downloads/imagefap/                    ==> App root dir
~/Downloads/imagefap/imagefap.database   ==> serialized metadata file (see below)
~/Downloads/imagefap/blobs/              ==> raw images storage directory
~/Downloads/imagefap/blobs/ca978112ca1bbdcafac231b39a23dc4da786eff8147c4e72b9807785afee48bb.jpg  ==> blob
~/Downloads/imagefap/blobs/3e23e8160039594a33894f6564e1b1348bbd7a0088d42c4acb73eeaed59c009d.gif  ==> blob
[... etc ... each blob is:]
~/Downloads/imagefap/blobs/[file_sha_256_hex_digest].[jpg|gif|... original type]
~/Downloads/imagefap/thumbs/             ==> thumbnails storage directory
~/Downloads/imagefap/blobs/ca978112ca1bbdcafac231b39a23dc4da786eff8147c4e72b9807785afee48bb.jpg  ==> thumbnail
~/Downloads/imagefap/blobs/3e23e8160039594a33894f6564e1b1348bbd7a0088d42c4acb73eeaed59c009d.gif  ==> thumbnail
[... etc ... each thumbnail uses the same name as the blob it refers to]
```

### Database Schema

If allowed, will save a database of Imagefap files so we don't have to
hit the servers multiple times. The data will be serialized (Python pickle)
from a structure like:

```
{

  'configs' {
    'duplicates_sensitivity_regular': {
      # criteria for duplicate selection, per method, for regular images;
      # a value of -1 will deactivate the method
      'percept': int_max_hamming_distance,   # our default =4 (library default =10)
      'diff': int_max_hamming_distance,      # our default =4 (library default =10)
      'average': int_max_hamming_distance,   # our default =1 (library default =10)
      'wavelet': int_max_hamming_distance1,  # our default =1 (library default =10)
      'cnn': float_min_cosine_similarity,    # our default =0.95 (library default =0.90)
    },
    'duplicates_sensitivity_animated': {
      # criteria for duplicate selection, per method, for animated images
      # all values here *MUST* be stricter than in 'duplicates_sensitivity_regular'!
      # a value of -1 will deactivate the method
      'percept': int_max_hamming_distance,   # our default =3
      'diff': int_max_hamming_distance,      # our default =1
      'average': int_max_hamming_distance,   # our default =-1 (disabled)
      'wavelet': int_max_hamming_distance1,  # our default =-1 (disabled)
      'cnn': float_min_cosine_similarity,    # our default =0.97
    },
  },

  'users': {
    # stores the seen users
    user_id: {
      'name': user_name,
      'date_albums': int_time_last_success_list_all_albums,
      'date_finished': int_time_last_success_finish,
    },
  },

  'favorites': {
    # stores the downloaded picture folders metadata
    user_id: {
      folder_id: {
        'name': folder_name,
        'pages': max_pages_found,
        'date_straight': int_time_last_success_download_for_direct_saving,
        'date_blobs':    int_time_last_success_download_for_blobs,
            # these dates are int(time.time()) of last time all images finished; the fields will
            # start as 0, meaning never finished yet, and we have one for straight image saves
            # (coming from `get` operation) and one for blobs (coming from `read` operation)
        'images': [imagefap_image_id-1, imagefap_image_id-2, ...],  # in order
        'failed_images': {
          (failed_image_id-1, failure_int_timestamp-1, Optional[file_name_sanitized-1], Optional[full_res_url-1]),
          (failed_image_id-2, failure_int_timestamp-2, Optional[file_name_sanitized-2], Optional[full_res_url-2]),
          ...
        },
      },
    },
  },

  'tags': {
    # stores the user-defined tags for later image re-shuffle
    tag_id: {
      'name': tag_name,
      'tags': {},  # None, or a nested dict of sub-tags, just like the top one
    },
  },

  'blobs': {
    # stored blob metadata: where has each blob been seen and what tags have been attached
    file_sha_256_hex_digest: {
      'loc': {
        (imagefap_image_id-1, imagefap_full_res_url-1, imagefap_file_name_sanitized-1, user_id-1, folder_id-1),
        (imagefap_image_id-2, imagefap_full_res_url-2, imagefap_file_name_sanitized-2, user_id-2, folder_id-2),
        ... this is a set of every occurrence of the blob in the downloaded favorites ...
      },
      'tags': {tag_id-1, tag_id-2, ...},
      'sz': int_size_bytes,          # size of blob file
      'sz_thumb': int_size_bytes,    # size of saved thumbnail image, if any, else 0
      'ext': string_file_extension,  # the saved file extension ('jpg', 'gif', ...)
      'percept': perceptual_hash,    # 16 character hex (64 bit) string hash for the image
      'average': average_hash,       # 16 character hex (64 bit) string hash for the image
      'diff': difference_hash,       # 16 character hex (64 bit) string hash for the image
      'wavelet': wavelet_hash,       # 16 character hex (64 bit) string hash for the image
      'cnn': convolutional_neural_network_hash,  # np.ndarray of float32 and shape (576,) = 4.5kb
      'width': int,      # image width
      'height': int,     # image height
      'animated': bool,  # True if image is animated (gif), False otherwise
    },
  },

  'image_ids_index': {
    # this is a reverse index for `blobs`, so we can easily search by imagefap_image_id
    imagefap_image_id: file_sha_256_hex_digest,
  },

  'duplicates_registry': {
    tuple(sorted({sha1, sha2, ...})): {  # the key is the set of duplicates
      'sources': {  # for each method that contributed: the scores that turned up, pair by pair
        Literal['percept', 'average', 'diff', 'wavelet', 'cnn']: {
          tuple(sorted({sha1, sha2})): float_score_for_sha1_sha2_pair,
          tuple(sorted({sha1, sha2})): float_score_for_sha1_sha2_pair,
          ...
        },
        ...
      },
      'verdicts': {  # stores the decisions on each case, what to do with the files
        sha1: Literal['new', 'false', 'keep', 'skip'],  # what to do with sha1
        sha2: Literal['new', 'false', 'keep', 'skip'],  # what to do with sha2
            # 'new': this is a new entry to this set, so it needs a new revision
            # 'false': this is a false positive
            #      (if the set has only 2 hashes, then either both are 'false' or neither is!)
            # 'keep': this is "ignore", meaning image will not be affected, will be kept
            # 'skip': this image will be skipped, meaning disappear/delete
        ...
      },
    },
  },

  'duplicates_key_index': {
    # this is key index for `duplicates`, so we can easily find a `sha` in duplicates, and also
    # to guarantee that every `sha` will grouped in a *single* duplicate set
    file_sha_256_hex_digest: tuple(sorted({sha1, sha2, ...})),  # `blob` key -> `duplicates` key
  },

}
```

When compressed this structure takes less space than would seem at a first
glance, especially taking into account that it is stored compressed, and
has shown to be a minimal fraction compared to the downloaded images size.
Typically it will take much less than 0.5% of the size of the actual images
and usually the whole database file won't even reach 1Mb.
