# keen-cli

[![Build Status](https://travis-ci.org/keenlabs/keen-cli.svg?branch=master)](https://travis-ci.org/keenlabs/keen-cli)

A command line interface for the Keen IO analytics API.

### Installation

keen-cli is built with Ruby, so you'll need a working Ruby 1.9+ environment to use it. You can find Ruby installation instructions [here](https://www.ruby-lang.org/en/installation/).

Install the gem:

``` shell
$ gem install keen-cli
```

Verify the `keen` command is in your path by running it:

``` shell
Commands:
  keen average              # Alias for queries:run -c average
  keen count                # Alias for queries:run -c count
  keen count-unique         # Alias for queries:run -c count_unique
  keen events:add           # Add one or more events and print the result
  keen extraction           # Alias for queries:run -c extraction
  keen help [COMMAND]       # Describe available commands or one specific command
  keen maximum              # Alias for queries:run -c maximum
  keen median               # Alias for queries:run -c median
  keen minimum              # Alias for queries:run -c minimum
  keen percentile           # Alias for queries:run -c percentile
  keen project:collections  # Print information about a project's collections
  keen project:describe     # Print information about a project
  keen project:open         # Open a project's overview page in a browser
  keen project:workbench    # Open a project's workbench page in a browser
  keen queries:run          # Run a query and print the result
  keen select-unique        # Alias for queries:run -c select_unique
  keen sum                  # Alias for queries:run -c sum
  keen version              # Print the keen-cli version
```

You should see information about available commands.

If `keen` can't be found there might be an issue with your Ruby installation. If you're using [rbenv](https://github.com/sstephenson/rbenv) try running `rbenv rehash` after installation.

### Environment configuration

Most keen-cli commands require the presence of a project and one or more API keys to do meaningful actions. By default, keen-cli attempts to find these in the process environment or a `.env` file in the current directory. This is the same heuristic that [keen-gem](https://github.com/keenlabs/keen-gem) uses and is based on [dotenv](https://github.com/bkeepers/dotenv).

An example .env file looks like this:

```
KEEN_PROJECT_ID=aaaaaaaaaaaaaaa
KEEN_MASTER_KEY=xxxxxxxxxxxxxxx
KEEN_WRITE_KEY=yyyyyyyyyyyyyyy
KEEN_READ_KEY=zzzzzzzzzzzzzzz
```

If you run `keen` from a directory with this .env file, it will assume the project in context is the one specified by `KEEN_PROJECT_ID`.

To override the project context use the `--project` option:

``` shell
$ keen project:describe --project XXXXXXXXXXXXXXX
```

Similar overrides are available for specifiying API keys: `--master-key`, `--read-key` and `--write-key`.

For example:

``` shell
$ keen project:describe --project XXXXXXXXXXXXXXX --master-key AAAAAAAAAAAAAA
```

Shorter aliases exist as well: `-p` for project, `-k` for master key, `-r` for read key, and `-w` for write key.

``` shell
$ keen project:describe -p XXXXXXXXXXXXXXX -k AAAAAAAAAAAAAA
```

### Usage

keen-cli has a variety of commands, and most are namespaced for clarity.

* `version` - Print version information

##### Projects

* `project:open` - Open the Project Overview page in a browser
* `project:workbench` - Open the Project Workbench page in a browser
* `project:describe` - Get data about the project. Uses the [project row resource](https://keen.io/docs/api/reference/#project-row-resource).
* `project:collections` - Get schema information about the project's collections. Uses the [event resource](https://keen.io/docs/api/reference/#event-resource).

##### Events

`events:add` - Add an event.

Parameters:

+ `--collection`, `-c`: The collection to add the event to. Alternately you can set `KEEN_COLLECTION_NAME` on the environment if you're working with the same collection frequently.
+ `--data`, `-d`: The properties of the event. The value can be JSON or `key=value` pairs delimited by `&` (just like a query string). Data can also be piped in via STDIN.
+ `--file`, `-f`: The name of a file in newline-delimited JSON. If the file is in CSV format, add the `--csv` flag.
+ `--csv`: Specify that the file is in CSV format. The first line of the CSV file must contain column names.

Various examples:

``` shell
# create an empty event
$ keen events:add --collection cli-tests

# use the shorter form of collection
$ keen events:add -c cli-tests

# add a blank event to a collection specified in the .env file:
# KEEN_COLLECTION_NAME=cli-tests
$ keen events:add

# create an event from JSON
$ keen events:add -c cli-tests -d "{ \"username\" : \"dzello\", \"zsh\": 1 }"

# create an event from key value pairs
$ keen events:add -c cli-tests -d "username=dzello&zsh=1"

# pipe in events as JSON
$ echo "{ \"username\" : \"dzello\", \"zsh\": 1 }" | keen events:add -c cli-tests

# pipe in events in querystring format
$ echo "username=dzello&zsh=1" | keen events:add -c cli-test

# specify a file that contains newline delimited json
$ keen events:add --file events.json

# specify a file in CSV format
$ keen events:add --csv --file events.csv

# pipe in events from a file of newline delimited json
# { "username" : "dzello", "zsh" : 1 }
# { "username" : "dkador", "zsh" : 1 }
# { "username" : "gphat", "zsh" : 1 }
$ cat events.json | keen events:add -c cli-test
```

##### Queries

`queries:run` - Runs a query and prints the result in pretty JSON.

Parameters:

+ `--collection`, `-c`: – The collection to query against. Can also be set on the environment via `KEEN_COLLECTION_NAME`.
+ `--analysis-type`, `-a`: The analysis type for the query. Only needed when not using a query command alias.
+ `--group-by`, `-g`: A group by for the query.
+ `--target-property`, `-y`: A target property for the query.
+ `--timeframe`, `-t`: A relative timeframe, e.g. `last_60_minutes`.
+ `--start`, `-s`: The start time of an absolute timeframe.
+ `--end`, `-e`: The end time of an absolute timeframe.
+ `--interval`, `-i`: The interval for a series query.
+ `--filters`, `-f`: A set of filters for the query, passed as JSON.
+ `--percentile`: The percentile value (e.g. 99) for a percentile query.
+ `--property-names`: A comma-separated list of property names. Extractions only.
+ `--latest`: Number of latest events to retrieve. Extractions only.
+ `--email`: Send extraction results via email, asynchronously. Extractions only.
+ `--data`, `-d`: Specify query parameters as JSON instead of query params. Data can also be piped in via STDIN.

Some examples:

``` shell
# run a count
$ keen queries:run --collection cli-tests --analysis-type count
1000

# run a count with collection name from .env
# KEEN_COLLECTION_NAME=cli-tests
$ keen queries:run --analysis-type count
1000

# run a count with a group by
$ keen queries:run --collection cli-tests --analysis-type count --group-by username
[
  {
    "username": "dzello",
    "result": 1000
  }
]

# run a query with a timeframe, target property, group by, and interval
$ keen queries:run --collection cli-tests --analysis-type median --target-property value --group-by cohort --timeframe last_24_hours --interval hourly

{
  "timeframe": {
    "start": "2014-06-27T01:00:00.000Z",
    "end": "2014-06-27T02:00:00.000Z"
  },
  "value": [
  ...
  ...
  ...

# run a query with an absolute timeframe
$ keen queries:run --analysis-type count --start 2014-07-01T00:00:00Z --end 2014-07-31T23:59:59Z
1000

# run an extraction with specific property names
$ keen queries:run --collection minecraft-deaths --analysis-type extraction --property-names player,enemy
[
  {
    "player": "dzello",
    "enemy": "creeper"
  },
  {
    "player": "dkador",
    "enemy": "creeper"
  }
]

# run a query using JSON to specify parameters
$ echo "{ \"event_collection\" : \"minecraft-deaths\", \"target_property\": \"level\" }" | keen queries:run -a average
```

**Query Aliases**

For each type of analysis (e.g. count, average, extraction, etc.) there is an alias that can be used
instead of `queries:run`. The command name is simply the type of analysis, using a dash to delimit words.
Here are a few examples:

``` shell
$ keen count -c logins
1000
$ keen minimum -c cpu-checks -y iowait
0.17
```

Run `keen` with no arguments to see the full list of aliases.

### Changelog

+ 0.1.5 – Support adding events from files with `--file`. Optionally add from CSV with `--csv`.
+ 0.1.4 – Support absolute timeframes via `--start` and `--end` flags
+ 0.1.3 – Add querying via JSON. Add query aliases. Add support for extraction fields.
+ 0.1.2 – Change `project:show` to `project:describe`
+ 0.1.1 – Add `project:collections`
+ 0.1.0 - Initial version

### Contributing

keen-cli is open source, and contributions are very welcome!

Running the tests with:

```
$ bundle exec rake spec
```
