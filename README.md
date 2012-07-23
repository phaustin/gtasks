# GTasks

gtasks is a command line client for the [google tasks api](https://developers.google.com/google-apps/tasks/) written in python, aimed at OSX (for keychain integration).
It was written as a way to learn python, so the approach may be *unpythonic* in places,
which I will correct as my experience in the language improves.

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

Next you should install [keychain](https://github.com/diffsky/keychain), which
will later allow you to add your gtasks api settings to the OSX keychain.

After that, is the google tasks api python client library:

    pip install google-api-python-client

Lastly, you need to add your google api project details. If you don't have one already,
create a project at https://code.google.com/apis/console/

After the project is created, add the `Tasks API` service to it. Then in the
`API Access` tab, click "Create an OAuth 2.0 Client ID...". Give your project a name,
click next and select "Installed application" and click "Create client ID".

Back on the `API Access` tab you will see populated values for `Client ID`,
`Client secret` and `API key`. Take a note of these values as you will use them with `keychain`:

    keychain -s <Client ID> gtasks_id
    keychain -s <Client secret> gtasks_secret
    keychain -s <API key> gtasks_key

## Installation

    git clone git://github.com/diffsky/gtasks.git

## Usage

Assuming you have the `gtasks.py` file somewhere that's in your `$PATH`, then `gtasks.py -h` will show usage instructions.
In the below examples I have a symlink `gtasks` that points to my location of `gtasks.py`

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


As you can see, lots of options/flexability.


## TODO

 - add a dashboard output, the aim is to have something that can be added to my command prompt
 - move tasks (note api does not support moving between tasks)
 - support indented tasks



