# Sidechain Launch Kit

## Introduction

This directory contains python scripts to tests and explore side chains. 

This document walks through the steps to setup a side chain running on your local
machine and make your first cross chain transfers.

## Get Ready

This section describes how to install the python dependencies, create the
environment variables, and create the configuration files that scripts need to
run correctly.

### Build rippled

Checkout the `sidechain` branch from the rippled repository, and follow the
usual process to build rippled.

### Set up Python environment

To make it easy to manage your Python environment with `xrpl-py`, including switching between versions, install `pyenv` and follow these steps:

* Install [`pyenv`](https://github.com/pyenv/pyenv):

        brew install pyenv

    For other installation options, see the [`pyenv` README](https://github.com/pyenv/pyenv#installation).

* Use `pyenv` to install the optimized version for `xrpl-py` (currently 3.9.1):

        pyenv install 3.9.1

* Set the [global](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md#pyenv-global) version of Python with `pyenv`:

        pyenv global 3.9.1

### Set up shell environment

To enable autocompletion and other functionality from your shell, add `pyenv` to your environment.

These steps assume that you're using a [Zsh](http://zsh.sourceforge.net/) shell. For other shells, see the [`pyenv` README](https://github.com/pyenv/pyenv#basic-github-checkout).


* Add `pyenv init` to your Zsh shell:

        echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.zshrc

* Source or restart your terminal:

        . ~/.zshrc

### Manage dependencies and virtual environments

To simplify managing library dependencies and the virtual environment, `xrpl-py` uses [`poetry`](https://python-poetry.org/docs).

* [Install `poetry`](https://python-poetry.org/docs/#osx-linux-bashonwindows-install-instructions):

        curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python -
        poetry install

### Environment variables

The python scripts need to know the locations of two files and one directory.
These can be specified either through command line arguments or by setting
environment variables. 

1. The location of the rippled executable used for main chain servers. Either
   set the environment variable `RIPPLED_MAINCHAIN_EXE` or use the command line
   switch `--exe_mainchain`. Until a new RPC is integrated into the main branch
   (this will happen very soon), use the code built from the sidechain branch as
   the main chain exe.
2. The location of the rippled executable used for side chain servers. Either
   set the environment variable `RIPPLED_SIDECHAIN_EXE` or use the command line
   switch `--exe_sidechain`. This should be the rippled executable built from
   the sidechain branch.
3. The location of the directory that has the rippled configuration files.
   Either set the environment variable `RIPPLED_SIDECHAIN_CFG_DIR` or use the
   command line switch `--cfgs_dir`. The configuration files do not exist yet.
   There is a script to create these for you. For now, just choose a location
   where the files should live and make sure that directory exists.
   
Setting environment variables can be very convient. For example, a small script
can be sourced to set these environment variables when working with side chains.


### Creating configuration files

Assuming rippled is built, the three environment variables are set, and the
python environment is activated, run the following script:
```
bin/sidechain/python/create_config_files.py --usd
```

There should now be many configuration files in the directory specified by the
`RIPPLED_SIDECHAIN_CFG_DIR` environment variable. The `--usd` creates a sample
cross chain assert for USD -> USD transfers.

## Running the interactive shell

There is an interactive shell called `RiplRepl` that can be used to explore
side chains. It will use the configuration files built above to spin up test
rippled main chain running in standalone mode as well as 5 side chain federators
running in regular consensus mode. 

To start the shell, run the following script:
```
bin/sidechain/python/riplrepl.py
```

The shell will not start until the servers have synced. It may take a minute or
two until they do sync. The script should give feedback while it is syncing.

Once the shell has started, the following message should appear:
```
Welcome to the sidechain test shell.   Type help or ? to list commands.

RiplRepl> 
```

Type the command `server_info` to make sure the servers are running. An example output would be:
```
RiplRepl> server_info
           pid                                  config  running server_state  ledger_seq complete_ledgers
main 0  136206  main.no_shards.mainchain_0/rippled.cfg     True    proposing          75             2-75
side 0  136230                 sidechain_0/rippled.cfg     True    proposing          92             1-92
     1  136231                 sidechain_1/rippled.cfg     True    proposing          92             1-92
     2  136232                 sidechain_2/rippled.cfg     True    proposing          92             1-92
     3  136233                 sidechain_3/rippled.cfg     True    proposing          92             1-92
     4  136234                 sidechain_4/rippled.cfg     True    proposing          92             1-92
```

Of course, you'll see slightly different output on your machine. The important
thing to notice is there are two categories, one called `main` for the main chain
and one called `side` for the side chain. There should be a single server for the
main chain and five servers for the side chain.

Next, type the `balance` command, to see the balances of the accounts in the address book:
```
RiplRepl> balance
                           balance currency peer limit
     account                                          
main root    99,999,989,999.999985      XRP           
     door             9,999.999940      XRP           
side door    99,999,999,999.999954      XRP           
```

There are two accounts on the main chain: `root` and `door`; and one account on the side chain: `door`. These are not user accounts. Let's add two user accounts, one on the main chain called `alice` and one on the side chain called `bob`. The `new_account` command does this for us.

```
RiplRepl> new_account mainchain alice
RiplRepl> new_account sidechain bob
```

This just added the accounts to the address book, but they don't exist on the
ledger yet. To do that, we need to fund the accounts with a payment. For now,
let's just fund the `alice` account and check the balances. The `pay` command
makes a payment on one of the chains:

```
RiplRepl> pay mainchain root alice 5000
RiplRepl> balance
                           balance currency peer limit
     account                                          
main root    99,999,984,999.999969      XRP           
     door             9,999.999940      XRP           
     alice            5,000.000000      XRP           
side door    99,999,999,999.999954      XRP           
     bob                  0.000000      XRP      
```

Finally, let's do something specific to side chains: make a cross chain payment.
The `xchain` command makes a payment between chains:

```
RiplRepl> xchain mainchain alice bob 4000
RiplRepl> balance
                           balance currency peer limit
     account                                          
main root    99,999,984,999.999969      XRP           
     door            13,999.999940      XRP           
     alice              999.999990      XRP           
side door    99,999,995,999.999863      XRP           
     bob              4,000.000000      XRP           
```

Note: the account reserve on the side chain is 100 XRP. The cross chain amount
must be greater than 100 XRP or the payment will fail.

Making a cross chain transaction from the side chain to the main chain is similar:
```
RiplRepl> xchain sidechain bob alice 2000
RiplRepl> balance
                           balance currency peer limit
     account                                          
main root    99,999,984,999.999969      XRP           
     door            11,999.999840      XRP           
     alice            2,999.999990      XRP           
side door    99,999,997,999.999863      XRP           
     bob              1,999.999990      XRP    
```

If you typed `balance` very quickly, you may catch a cross chain payment in
progress and the XRP may be deducted from bob's account before it is added to
alice's. If this happens just wait a couple seconds and retry the command. Also
note that accounts pay a ten drop fee when submitting transactions.

Finally, exit the program with the `quit` command:
```
RiplRepl> quit
Thank you for using RiplRepl. Goodbye.


WARNING: Server 0 is being stopped. RPC commands cannot be sent until this is restarted.
```

Ignore the warning about the server being stopped.

## Conclusion

Those two cross chain payments are a "hello world" for side chains. It makes sure
you're environment is set up correctly.