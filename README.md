# ssh_key_generation
Describes how to make a new ssh key for github

# CS356 Programming Server

The following commands should work on macOS and Linux terminals
natively, and on Windows through [Git Bash](https://gitforwindows.org).
We also recommend installing VS Code from [here](https://code.visualstudio.com/download).


## Goal 1: Add an SSH key for GitHub (on your laptop)

1. Run `ssh -T git@github.com`: If it works, you already have a working
   SSH key to access your GitHub. Bravo! You can skip all the
   following steps and move to Goal 2.

2. Run `ls ~/.ssh` to check whether you already have an SSH key pair.
   If it shows `id_rsa` and `id_rsa.pub`, you can skip to step 4.

3. Run `ssh-keygen` and press `ENTER` three times to create a key pair
   without a password (if you add a password, memorize it).

4. Run `cat ~/.ssh/id_rsa.pub` to show the content of the public key;
   copy that into the clipboard by selecting with the mouse, then
   right click, copy.

5. Go to [https://github.com/settings/keys](https://github.com/settings/keys),
   select "New SSH Key" (green button), add a title like "Laptop",
   then paste the public key. After adding the key, the command of
   step 1 (`ssh -T git@github.com`) should now work.


## Goal 2: Add an SSH key for the CS356 server (on your laptop)

1. Run `git clone git@github.com:usc-cs356-fall22/akovesdy_keys.git`
   to clone this repository on your laptop; run `cd akovesdy_keys`
   to enter its directory.

2. Run `cp akovesdy akovesdy.pub ~/.ssh` to add the key pair
   to your laptop.

3. Run `chmod 600 ~/.ssh/akovesdy*` to ensure that the keys
   have the correct permissions.

4. Run `code ~/.ssh/config` and add the following blocks, then save
   and exit:

   ```
   Host cs356-work
     User akovesdy
     HostName cs356-work.easygrade.org
     IdentityFile ~/.ssh/akovesdy
     ServerAliveInterval 10
     Compression yes
     ForwardAgent yes

   Host *
     AddKeysToAgent yes
     IdentityFile ~/.ssh/id_rsa
     ## Use on macOS to avoid typing key passwords
     # UseKeychain yes
     ## Use if getting "Bad configuration option: usekeychain"
     # IgnoreUnknown UseKeychain
   ```

5. Run `ssh cs356-work` to connect; you should see a different prompt,
   `akovesdy@cs356-work`. Press `CTRL` + `d` to disconnect.


## Goal 3: Connect and clone a repository on the server

This can be tricky, but it can be very useful whenever you are working
on cloud servers! The idea is that the SSH agent running on your laptop
will handle the authentication for the SSH agent running on the CS356 server,
using your GitHub key.

More details on the [GitHub docs](https://docs.github.com/en/developers/overview/using-ssh-agent-forwarding).

1. On Windows only, run `code ~/.bashrc` and add the following, then
   restart Git Bash:
   ```
   env=~/.ssh/agent.env

   agent_load_env () { test -f "$env" && . "$env" >| /dev/null ; }

   agent_start () {
      (umask 077; ssh-agent >| "$env")
      . "$env" >| /dev/null ; }

   agent_load_env

   # agent_run_state: 0=agent running w/ key; 1=agent w/o key; 2=agent not running
   agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)

   if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
      agent_start
      ssh-add
   elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
      ssh-add
   fi

   unset env
   ```

2. Run `ssh-add -L`: it should list the public key registered with GitHub.
   - If it says *"Could not open a connection to your authentication agent"*,
     and you are using macOS or Linux, run `code ~/.bashrc` and add a line with
     `eval "$(ssh-agent -s)"`. Then, restart the terminal.
   - If it says *"The agent has no identities"* or you don't see the public key
     registered with GitHub [here](https://github.com/settings/keys), then run:
     - `ssh-add ~/.ssh/id_rsa` on Linux or Windows
     - `ssh-add --apple-use-keychain ~/.ssh/id_rsa` on macOS Monterey or Ventura
     - `ssh-add -K ~/.ssh/id_rsa` on older macOS

3. Run `ssh cs356-work` to connect to the server.

4. Run `git clone git@github.com:usc-cs356-fall22/akovesdy_hellolab.git` to
   clone `akovesdy_hellolab` on the server. This is where the magic happens!
   You are using your GitHub private key on the server without ever copying it
   there, through "SSH agent forwarding".

5. Run `git config --global user.name "Agnes Kovesdy"` and
   `git config --global user.email akovesdy@usc.edu` to configure git on
   the server.


## Goal 4: Make a commit from VS Code Remote

Working on the terminal is great for BombLab and AttackLab; for the C
programming assignments (DataLab, CacheLab, MallocLab) you can install VS Code
on your laptop and use its [SSH Remote extension](https://code.visualstudio.com/docs/remote/ssh)
to edit/compile/run files on the server. Very cool!

(If you're an emacs or vim user, you don't need this of course.)

1. Start VS Code (from the terminal with `code` or from the menu).

2. Install the [Remote SSH extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)
   of VS Code (while running on your host OS, not inside the server).

3. Inside VS Code: press `F1` and select `Remote-SSH: Connect to host...`,
   then enter `cs356-work` as the host name. VS Code should connect to the server after
   installing the remote extension (if it asks you to choose the OS, choose Linux);
   now you can edit and compile files remotely.

4. If VS Code doesn't prompt you when you open a `.c` file, go to the "Extension" tab
   (`SHIFT+CTRL+X` or `SHIFT+CMD+X` on macOS), search for the
   [C/C++ Extension Pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools-extension-pack)
   and click `Install on SSH: cs356-work`.

5. Make a change to any file, create a commit VS Code, push it.
   If it works, you're all set!
