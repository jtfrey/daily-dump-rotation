# daily-dump-rotation

Each day at a fixed time database X is dumped to a file to prevent data loss.  In case an erroneous change is not caught within 24 hours, it pays to keep older dumps available, too.  Managing the resulting temporal collection of daily dump files can be challenging, but **must** be automated.

A number `N_daily` of recent daily dumps must be retained.  Once that limit is exceeded, the oldest files begin rolling into the set of `N_weekly` dumps with the caveat that subsequent weekly dumps must be `P_week` days apart.  As the weekly limit is exceeded, the oldest weekly files begin rolling into the set of `N_monthly` dumps with the caveat that subsequent monthly dumps must be `P_month` days apart.  Finally, as the monthly limit is exceeded, the oldest monthly files begin rolling into the set of `N_yearly` dumps with the caveat that subsequent yearly dumps must be `P_year` days apart.  These parameters are the *periods* and *retention* limits that constitute the *policy* for rotation.

Dump files are identified by a glob pattern search in a given directory (see [this python doc page](https://docs.python.org/3.9/library/fnmatch.html#fnmatch.fnmatch) for details):

- The asterisk (\*) wildcard matches zero or more characters
- The question mark (?) wildcard matches zero or one character
- A character sequence in square brackets ([a-z]) matches one character in the sequence
- A negated character sequence in square brackets ([!a-z]) matches one character **not** in the sequence

Dates are **not** taken from file metadata.  The file names must include a timestamp matching the regular expression

```
([0-9]{4})[_-]?([0-9]{2})[_-]?([0-9]{2})([_-]?([0-9]{2})[_:-]?([0-9]{2})([_:-]?([0-9]{2}))?)?
```

The four-digit year, two-digit month, and two-digit day are mandatory, with the components optionally separated by a dash (-) or underscore (\_).  The time may be omitted completely.  If included, an optional dash (-) or underscore (\_) can offset the time from the date.  The time must include the two-digit hour, followed optionally by the two-digit minute, followed optionally by the two-digit seconds.  The time components may optionally be separated by a dash (-), underscore (\_), or colon (:).  For example, all of the following represent the same timestamp:

- `19770325`
- `1977-03-25`
- `1977-03-2500`
- `19770325-0000`
- `1977-03-250000`
- `1977-03-25_00:00`
- `1977-03-25-00:00:00`

Any files in a directory that do not match the glob pattern or do not contain a matching timestamp are **not** considered for rotation.

## Usage

```
$ ./daily-dump-rotation --help
usage: daily-dump-rotation [-h] [-v] [-q] [-d] [--demo-mode <n-iters>] [-t <glob-pattern>]
                           [-p <days-in-week>:<days-in-month>:<days-in-year>] [-r <daily>:<weekly>:<monthly>:<yearly>]
                           <path> [<path> ...]

Manage temporal collections of daily dump files

positional arguments:
  <path>                Dump directory to process

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         Increase amount of information displayed as the program executes
  -q, --quiet           Decrease amount of information displayed as the program executes
  -d, --dry-run         Do not remove any files, just determine what would be removed
  --demo-mode <n-iters>
                        Generate and process this many files in the provided directory using this program
  -t <glob-pattern>, --filename-pattern <glob-pattern>
                        Consider filenames in the directories matching this pattern (default: *)
  -p <days-in-week>:<days-in-month>:<days-in-year>, --periods <days-in-week>:<days-in-month>:<days-in-year>
                        The number of days in a week, month, and year (default: 7:30:365)
  -r <daily>:<weekly>:<monthly>:<yearly>, --retention <daily>:<weekly>:<monthly>:<yearly>
                        Retention policy is colon-separated list of daily, weekly, monthly, and yearly counts
                        (default: 10:6:12:999)
```

### Demo mode

While developing the utility it was useful to be able to test the algorithm and different policy parameters' effects on it.  The *demo mode* was added to simulate from a fixed starting date a sequence of daily dump file creations with application of the rotation policy to them.  For example, using the default policy and running for 4000 days (with verbose output), the final iteration yields the following collection of dumps:

```
$ mkdir dumps; ./manage-db-dumps --demo-mode=4000 -v dumps
  :
INFO:root:    Will retain:
INFO:root:        [yearly ] 1977-03-25 04:30:00 => dumps/db-hpc-19770325-0430.txt 
INFO:root:        [yearly ] 1978-04-14 04:30:00 => dumps/db-hpc-19780414-0430.txt 
INFO:root:        [yearly ] 1979-05-04 04:30:00 => dumps/db-hpc-19790504-0430.txt 
INFO:root:        [yearly ] 1980-05-23 04:30:00 => dumps/db-hpc-19800523-0430.txt 
INFO:root:        [yearly ] 1981-06-12 04:30:00 => dumps/db-hpc-19810612-0430.txt 
INFO:root:        [yearly ] 1982-07-02 04:30:00 => dumps/db-hpc-19820702-0430.txt 
INFO:root:        [yearly ] 1983-07-22 04:30:00 => dumps/db-hpc-19830722-0430.txt 
INFO:root:        [yearly ] 1984-08-10 04:30:00 => dumps/db-hpc-19840810-0430.txt 
INFO:root:        [yearly ] 1985-08-30 04:30:00 => dumps/db-hpc-19850830-0430.txt 
INFO:root:        [yearly ] 1986-09-19 04:30:00 => dumps/db-hpc-19860919-0430.txt 
INFO:root:        [monthly] 1986-11-28 04:30:00 => dumps/db-hpc-19861128-0430.txt 
INFO:root:        [monthly] 1987-01-02 04:30:00 => dumps/db-hpc-19870102-0430.txt 
INFO:root:        [monthly] 1987-02-06 04:30:00 => dumps/db-hpc-19870206-0430.txt 
INFO:root:        [monthly] 1987-03-13 04:30:00 => dumps/db-hpc-19870313-0430.txt 
INFO:root:        [monthly] 1987-04-17 04:30:00 => dumps/db-hpc-19870417-0430.txt 
INFO:root:        [monthly] 1987-05-22 04:30:00 => dumps/db-hpc-19870522-0430.txt 
INFO:root:        [monthly] 1987-06-26 04:30:00 => dumps/db-hpc-19870626-0430.txt 
INFO:root:        [monthly] 1987-07-31 04:30:00 => dumps/db-hpc-19870731-0430.txt 
INFO:root:        [monthly] 1987-09-04 04:30:00 => dumps/db-hpc-19870904-0430.txt 
INFO:root:        [monthly] 1987-10-09 04:30:00 => dumps/db-hpc-19871009-0430.txt 
INFO:root:        [monthly] 1987-11-13 04:30:00 => dumps/db-hpc-19871113-0430.txt 
INFO:root:        [monthly] 1987-12-18 04:30:00 => dumps/db-hpc-19871218-0430.txt 
INFO:root:        [weekly ] 1988-01-15 04:30:00 => dumps/db-hpc-19880115-0430.txt 
INFO:root:        [weekly ] 1988-01-22 04:30:00 => dumps/db-hpc-19880122-0430.txt 
INFO:root:        [weekly ] 1988-01-29 04:30:00 => dumps/db-hpc-19880129-0430.txt 
INFO:root:        [weekly ] 1988-02-05 04:30:00 => dumps/db-hpc-19880205-0430.txt 
INFO:root:        [weekly ] 1988-02-12 04:30:00 => dumps/db-hpc-19880212-0430.txt 
INFO:root:        [weekly ] 1988-02-19 04:30:00 => dumps/db-hpc-19880219-0430.txt 
INFO:root:        [daily  ] 1988-02-26 04:30:00 => dumps/db-hpc-19880226-0430.txt 
INFO:root:        [daily  ] 1988-02-27 04:30:00 => dumps/db-hpc-19880227-0430.txt 
INFO:root:        [daily  ] 1988-02-28 04:30:00 => dumps/db-hpc-19880228-0430.txt 
INFO:root:        [daily  ] 1988-02-29 04:30:00 => dumps/db-hpc-19880229-0430.txt 
INFO:root:        [daily  ] 1988-03-01 04:30:00 => dumps/db-hpc-19880301-0430.txt 
INFO:root:        [daily  ] 1988-03-02 04:30:00 => dumps/db-hpc-19880302-0430.txt 
INFO:root:        [daily  ] 1988-03-03 04:30:00 => dumps/db-hpc-19880303-0430.txt 
INFO:root:        [daily  ] 1988-03-04 04:30:00 => dumps/db-hpc-19880304-0430.txt 
INFO:root:        [daily  ] 1988-03-05 04:30:00 => dumps/db-hpc-19880305-0430.txt 
INFO:root:        [daily  ] 1988-03-06 04:30:00 => dumps/db-hpc-19880306-0430.txt 
INFO:root:    Will purge:
INFO:root:                  1988-02-25 04:30:00 => dumps/db-hpc-19880225-0430.txt
```