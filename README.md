# GTasks

gtasks is a command line client for the [google tasks api](https://developers.google.com/google-apps/tasks/) written in python, as a means of learning the language.

## Requirements

gtasks has some dependencies, it is recommend you install them using [pip](http://www.pip-installer.org/en/latest/index.html) [[install guide](http://www.pip-installer.org/en/latest/installing.html)]. The steps I followed were:

    mkdir $HOME/.virtualenvs && cd $HOME/.virtualenvs
    curl -O https://raw.github.com/pypa/virtualenv/master/virtualenv.py
    python virtualenv.py my_new_env
    source my_new_env/bin/activate

The last line activates the virtual python environment for that shell instance.
You may want to add it to your `.bashrc` file to make it more permanent:

    BASE_VIRTUAL_ENV="$HOME/.virtualenvs/base/bin/activate"
    if [ -f $BASE_VIRTUAL_ENV ];then
        source $BASE_VIRTUAL_ENV
    fi

To switch to a different virtual environment you have to source its bin/activate.
You'll find more details in the [pip documentation](http://www.pip-installer.org/en/latest/index.html)

After that, is the google tasks api python client library:

    pip install google-api-python-client

Lastly, you need to add your google api project details. If you don't have one already,
create a project at https://code.google.com/apis/console/

After the project is created, add the `Tasks API` service to it. Then in the
`API Access` tab, click "Create an OAuth 2.0 Client ID...". Give your project a name,
click next and select "Installed application" and click "Create client ID".

Back on the `API Access` tab you will see populated values for `Client ID`,
`Client secret` and `API key`. Take a note of these values as you will use them with `keychain`:

    keychain -s gtasks_id <Client ID>
    keychain -s gtasks_secret <Client secret>
    keychain -s gtasks_key <API key>

You will then need to populate these keys in your system keyring service. In OSX, for example, this is the
keychain Access. I've created [keychain](https://github.com/diffsky/keychain) as a means of allowing OSX users
a convenient way to add their gtasks api settings to the OSX keychain. Internally, gtasks uses the
[keyring package](https://pypi.python.org/pypi/keyring) for an independent way of interacting with the
system keyring.


## Installation

    git clone git://github.com/diffsky/gtasks.git

## Usage

Assuming you have the `gtasks` file somewhere that's in your `$PATH`, then `gtasks -h` will show usage instructions.

gtasks is designed to be very flexibe and supports a number of options. Here are some examples:

    gtasks -ll

lists all your task lists

    gtasks -al list1

creates a new list called "list1"

    gtasks -l list1 list2

lists all tasks in list1 and list2

    gtasks -a -t "hello world"

adds a task with the title "hello world" to your default list

    gtasks -l list1 -e 2 -t "goodbye mars"

edits task number 2, in list1, changing its title to "goodbye mars"

    gtasks -l list1 -d 2

deletes task 2 from list1

    gtasks -c 1

marks task 1 in the default list as complete

    gtasks -e 1 -w today

edits task 1 in the default list, settings its due date to today. Note that wherever
there is a date format, gtasks supports these values:

 - yesterday, yes, today, tod, tomorrow, tom
 - year-month-day example: 2012-06-22
 - {last|next}{mon|tue|wed|thu|fri|sat|sun} examples: nextfri, mon, lasttue
 - *n* examples: 5 (for 5 days ahead), -5 (for 5 days back)

to remove the due date for a task use th `-cw` option when editing the task: `gtasks -e 1 -cw`

    gtasks -st

list tasks in the default lists and show totals for complete and due

    gtasks -L

list *all( tasks in *all lists*. Note that you cannot take an action on tasks returned
from the `-L` option as it is a collection of lists. If you wanted to update a task
in a list returned by `-L` you should use `-l <list title>` to specify the actual list
it belongs to.

    gtasks -L -b

list all tasks in all lists, busting any local cache to ensure the latest data

    gtasks -L -sb tomorrow -sa today -dse

list all tasks in all lists, that are due before tomorrow and after today, don't
show empty lists. Essentially this shows all tasks due today.


And list gtasks that are due:
```
gtasks -L -sdo -dse
```

As you can see, lots of options/flexability.


## TODO

 - <del>add a dashboard output</del> Achievable via bash:

```
#!/bin/bash
#
# gtasks_dashboard
#
TXTDEF='\033[0m'          # everything back to defaults
TXTRED='\033[0;31;1m'     # red text
TXTYEL='\033[0;33;1m'     # yellow text

TOTALS=$(gtasks -b -L -sto | grep -A 2 "OVERALL")
OVERDUE=$(echo $TOTALS | grep -o "Overdue: [0-9]\+;" | grep -o "[0-9]\+")
TODAY=$(echo $TOTALS | grep -o "Due Today: [0-9]\+;" | grep -o "[0-9]\+")

let TOTAL=OVERDUE+TODAY
if [ $TOTAL != "0" ];then
    if [ $TOTAL != "1" ];then
        TP='s'
    fi

    if [ $OVERDUE != "0" ];then
        OVERDUE_MSG="$TXTRED$OVERDUE overdue$TXTDEF"
    fi
    if [ $TODAY != "0" ];then
        TODAY_MSG="$TXTYEL$TODAY due today$TXTDEF"
    fi

    if [ $OVERDUE != "0" ];then
        MSG="$OVERDUE_MSG"
        if [ $TODAY != "0" ];then
            MSG="$MSG and $TODAY_MSG"
        fi
    elif [ $TODAY != "0" ];then
        MSG="$TXTYEL$TODAY due today$TXTDEF"
    fi

    echo -e "You have $MSG task$TP"
fi
```

optional crontab for gtasks_dashboard:

```
# Check for tasks
# note may need to `source /full/path/to/home/dir/.bashrc;` before gtasks_dashboard
0,30 * * * * gtasks_dashboard > /full/path/to/home/dir/.gtasks/dashboard
function parse_tasks {
  if [ -f $HOME/.gtasks/dashboard ]; then
    cat $HOME/.gtasks/dashboard
    echo -e "\r\n"
    rm $HOME/.gtasks/dashboard 2> /dev/null
    # introduce a random <1 second delay in an attempt to prevent multiple terms
    # from trying to delete the file at the same time
    # http://www.unix.com/shell-programming-scripting/7197-sleep-under-one-second.html
    #perl -e "select(undef,undef,undef,.${RANDOM})"
    #if [ -f $HOME/.gtasks/dashboard ]; then
    #  rm $HOME/.gtasks/dashboard
    #fi
  fi
}
```

 - move tasks (note api does not support moving between tasks)
 - support indented tasks


