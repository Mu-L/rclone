---
title: "rclone lsf"
description: "List directories and objects in remote:path formatted for parsing."
versionIntroduced: v1.40
# autogenerated - DO NOT EDIT, instead edit the source code in cmd/lsf/ and as part of making a release run "make commanddocs"
---
# rclone lsf

List directories and objects in remote:path formatted for parsing.

## Synopsis

List the contents of the source path (directories and objects) to
standard output in a form which is easy to parse by scripts.  By
default this will just be the names of the objects and directories,
one per line.  The directories will have a / suffix.

Eg

    $ rclone lsf swift:bucket
    bevajer5jef
    canole
    diwogej7
    ferejej3gux/
    fubuwic

Use the `--format` option to control what gets listed.  By default this
is just the path, but you can use these parameters to control the
output:

    p - path
    s - size
    t - modification time
    h - hash
    i - ID of object
    o - Original ID of underlying object
    m - MimeType of object if known
    e - encrypted name
    T - tier of storage if known, e.g. "Hot" or "Cool"
    M - Metadata of object in JSON blob format, eg {"key":"value"}

So if you wanted the path, size and modification time, you would use
`--format "pst"`, or maybe `--format "tsp"` to put the path last.

Eg

    $ rclone lsf  --format "tsp" swift:bucket
    2016-06-25 18:55:41;60295;bevajer5jef
    2016-06-25 18:55:43;90613;canole
    2016-06-25 18:55:43;94467;diwogej7
    2018-04-26 08:50:45;0;ferejej3gux/
    2016-06-25 18:55:40;37600;fubuwic

If you specify "h" in the format you will get the MD5 hash by default,
use the `--hash` flag to change which hash you want.  Note that this
can be returned as an empty string if it isn't available on the object
(and for directories), "ERROR" if there was an error reading it from
the object and "UNSUPPORTED" if that object does not support that hash
type.

For example, to emulate the md5sum command you can use

    rclone lsf -R --hash MD5 --format hp --separator "  " --files-only .

Eg

    $ rclone lsf -R --hash MD5 --format hp --separator "  " --files-only swift:bucket
    7908e352297f0f530b84a756f188baa3  bevajer5jef
    cd65ac234e6fea5925974a51cdd865cc  canole
    03b5341b4f234b9d984d03ad076bae91  diwogej7
    8fd37c3810dd660778137ac3a66cc06d  fubuwic
    99713e14a4c4ff553acaf1930fad985b  gixacuh7ku

(Though "rclone md5sum ." is an easier way of typing this.)

By default the separator is ";" this can be changed with the
`--separator` flag.  Note that separators aren't escaped in the path so
putting it last is a good strategy.

Eg

    $ rclone lsf  --separator "," --format "tshp" swift:bucket
    2016-06-25 18:55:41,60295,7908e352297f0f530b84a756f188baa3,bevajer5jef
    2016-06-25 18:55:43,90613,cd65ac234e6fea5925974a51cdd865cc,canole
    2016-06-25 18:55:43,94467,03b5341b4f234b9d984d03ad076bae91,diwogej7
    2018-04-26 08:52:53,0,,ferejej3gux/
    2016-06-25 18:55:40,37600,8fd37c3810dd660778137ac3a66cc06d,fubuwic

You can output in CSV standard format.  This will escape things in "
if they contain ,

Eg

    $ rclone lsf --csv --files-only --format ps remote:path
    test.log,22355
    test.sh,449
    "this file contains a comma, in the file name.txt",6

Note that the `--absolute` parameter is useful for making lists of files
to pass to an rclone copy with the `--files-from-raw` flag.

For example, to find all the files modified within one day and copy
those only (without traversing the whole directory structure):

    rclone lsf --absolute --files-only --max-age 1d /path/to/local > new_files
    rclone copy --files-from-raw new_files /path/to/local remote:path

The default time format is `'2006-01-02 15:04:05'`.
[Other formats](https://pkg.go.dev/time#pkg-constants) can be specified with the `--time-format` flag.
Examples:

	rclone lsf remote:path --format pt --time-format 'Jan 2, 2006 at 3:04pm (MST)'
	rclone lsf remote:path --format pt --time-format '2006-01-02 15:04:05.000000000'
	rclone lsf remote:path --format pt --time-format '2006-01-02T15:04:05.999999999Z07:00'
	rclone lsf remote:path --format pt --time-format RFC3339
	rclone lsf remote:path --format pt --time-format DateOnly
	rclone lsf remote:path --format pt --time-format max
`--time-format max` will automatically truncate '`2006-01-02 15:04:05.000000000`'
to the maximum precision supported by the remote.


Any of the filtering options can be applied to this command.

There are several related list commands

  * `ls` to list size and path of objects only
  * `lsl` to list modification time, size and path of objects only
  * `lsd` to list directories only
  * `lsf` to list objects and directories in easy to parse format
  * `lsjson` to list objects and directories in JSON format

`ls`,`lsl`,`lsd` are designed to be human-readable.
`lsf` is designed to be human and machine-readable.
`lsjson` is designed to be machine-readable.

Note that `ls` and `lsl` recurse by default - use `--max-depth 1` to stop the recursion.

The other list commands `lsd`,`lsf`,`lsjson` do not recurse by default - use `-R` to make them recurse.

Listing a nonexistent directory will produce an error except for
remotes which can't have empty directories (e.g. s3, swift, or gcs -
the bucket-based remotes).


```
rclone lsf remote:path [flags]
```

## Options

```
      --absolute             Put a leading / in front of path names
      --csv                  Output in CSV format
  -d, --dir-slash            Append a slash to directory names (default true)
      --dirs-only            Only list directories
      --files-only           Only list files
  -F, --format string        Output format - see  help for details (default "p")
      --hash h               Use this hash when h is used in the format MD5|SHA-1|DropboxHash (default "md5")
  -h, --help                 help for lsf
  -R, --recursive            Recurse into the listing
  -s, --separator string     Separator for the items in the format (default ";")
  -t, --time-format string   Specify a custom time format, or 'max' for max precision supported by remote (default: 2006-01-02 15:04:05)
```

Options shared with other commands are described next.
See the [global flags page](/flags/) for global options not listed here.

### Filter Options

Flags for filtering directory listings

```
      --delete-excluded                     Delete files on dest excluded from sync
      --exclude stringArray                 Exclude files matching pattern
      --exclude-from stringArray            Read file exclude patterns from file (use - to read from stdin)
      --exclude-if-present stringArray      Exclude directories if filename is present
      --files-from stringArray              Read list of source-file names from file (use - to read from stdin)
      --files-from-raw stringArray          Read list of source-file names from file without any processing of lines (use - to read from stdin)
  -f, --filter stringArray                  Add a file filtering rule
      --filter-from stringArray             Read file filtering patterns from a file (use - to read from stdin)
      --hash-filter string                  Partition filenames by hash k/n or randomly @/n
      --ignore-case                         Ignore case in filters (case insensitive)
      --include stringArray                 Include files matching pattern
      --include-from stringArray            Read file include patterns from file (use - to read from stdin)
      --max-age Duration                    Only transfer files younger than this in s or suffix ms|s|m|h|d|w|M|y (default off)
      --max-depth int                       If set limits the recursion depth to this (default -1)
      --max-size SizeSuffix                 Only transfer files smaller than this in KiB or suffix B|K|M|G|T|P (default off)
      --metadata-exclude stringArray        Exclude metadatas matching pattern
      --metadata-exclude-from stringArray   Read metadata exclude patterns from file (use - to read from stdin)
      --metadata-filter stringArray         Add a metadata filtering rule
      --metadata-filter-from stringArray    Read metadata filtering patterns from a file (use - to read from stdin)
      --metadata-include stringArray        Include metadatas matching pattern
      --metadata-include-from stringArray   Read metadata include patterns from file (use - to read from stdin)
      --min-age Duration                    Only transfer files older than this in s or suffix ms|s|m|h|d|w|M|y (default off)
      --min-size SizeSuffix                 Only transfer files bigger than this in KiB or suffix B|K|M|G|T|P (default off)
```

### Listing Options

Flags for listing directories

```
      --default-time Time   Time to show if modtime is unknown for files and directories (default 2000-01-01T00:00:00Z)
      --fast-list           Use recursive list if available; uses more memory but fewer transactions
```

## See Also

* [rclone](/commands/rclone/)	 - Show help for rclone commands, flags and backends.

