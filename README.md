# Set Up Plutus Playground on macOS: A Step-by-Step Guide for Intel and M1 Macs
Originally composed by Christophe Garant (last update 2022 Jan 23)

Forked and modified by Aaron Prunty (2022 Aug 08)

## These instructions have been tested on the following Mac specs
- Macbook Pro 2015, macOS Monterey (v12.1), Intel Quad Core i7
- Macbook Pro 2013, macOS Catalina (v10.15.7), Intel Quad Core i7

## Set up the required plutus and plutus-apps repositories
If you're reading these instructions to set up Plutus Playground, there's a good chance you already have a Cardano node up and running. So, you may have a directory called **$HOME/cardano-src/** that contains your [cardano-node](https://github.com/input-output-hk/cardano-node) installation. We will install Plutus Playground under **$HOME/cardano-src** as well so that all Cardano-related goodies are kept together:
```
cd $HOME/cardano-src/
```
We need *both* [plutus](https://github.com/input-output-hk/plutus) and [plutus-apps](https://github.com/input-output-hk/plutus-apps) repositories for the Plutus Playground setup to go smoothly. So, we clone them to our local machine here using git:
```
git clone https://github.com/input-output-hk/plutus.git
```
```
git clone https://github.com/input-output-hk/plutus-apps.git
```
We will checkout a specific branch of plutus-apps used in the Plutus Pioneers Week 1 Cohort 3, and then update and build with *cabal*. It's important that we build with *cabal* now; otherwise our playground code won't compile later--instead, we will get a "module not found" error.

In the terminal, change to the plutus-apps directory
```
cd plutus-apps/
```
and then checkout the required branch:
```
git checkout 41149926c108c71831cfe8d244c83b0ee4bf5c8a
```
Finally, update and build code with cabal:
```
cabal update
```
```
cabal build
```

## Installing Nix (caution: here there be dragons...)
If you already have Nix installed on your Mac because you're a Nix poweruser and you've used it to manage and build a bunch of packages, then you probably know what you're doing and can likely just skip ahead to editing the `/etc/nix/nix.conf` file on step 3 below.

However, if you already have Nix installed on your Mac because this is your 100th attempt at installing Plutus Playground and you were told to install Nix along the way but the instructions were more hopeful than certain the outcome will work, then it's absolutely a good idea to wipe your Mac completely clean of Nix and do a fresh install. Follow [these instructions from Nix.org](https://nixos.org/manual/nix/stable/installation/installing-binary.html#macos) to remove Nix from your Mac.

**Once you have completed the uninstall instructions from Nix.org, restart your Mac!!!**

Now, here is the correct way to set up Nix on macOS to run Plutus Playground:
1. Install Nix per the [Nix documentation for macOS](https://nixos.org/download.html#nix-install-macos)
```
sh <(curl -L https://nixos.org/nix/install)
```
_Note: Most outdated tutorials say to add `--daemon` or `--darwin-use-unencrypted-nix-store-volume`; however official Nix documentation and others on Cardano Stack Exchange (CSE) claim this is no longer needed._

2. **Restart your computer!!! (yes, again)**

3. Next, we need to edit the `/etc/nix/nix.conf` file using administrator priveleges. You can do this using your favorite text editor (eg, `sudo nano /etc/nix/nix.conf` or `sudo vim /etc/nix/nix.conf`. However, *be careful* to consider which processor your Mac is using. Choose one of the nix.conf file contents below and copy-and-paste them into your `/etc/nix/nix.conf` file:

 **If you are running an Intel Mac, use this version:**
```
build-users-group           = nixbld

substituters                = https://hydra.iohk.io https://cache.nixos.org/
trusted-public-keys         = hydra.iohk.io:f/Ea+s+dFdN+3Y/G+FDgSq+a5NEWhJGzdjvKNGv0/EQ= cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY=volume

sandbox                     = false
extra-sandbox-paths         = /System/Library/Frameworks /System/Library/PrivateFrameworks /usr/lib /private/tmp /private/var/tmp /usr/bin/env

experimental-features       = nix-command
extra-experimental-features = flakes
```
**If you are running an M1 Mac, use this version:**
```
build-users-group           = nixbld

substituters                = https://hydra.iohk.io https://cache.nixos.org/
trusted-public-keys         = hydra.iohk.io:f/Ea+s+dFdN+3Y/G+FDgSq+a5NEWhJGzdjvKNGv0/EQ= cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY=volume

system                      = x86_64-darwin
extra-platforms             = x86_64-darwin aarch64-darwin

sandbox                     = false
extra-sandbox-paths         = /System/Library/Frameworks /System/Library/PrivateFrameworks /usr/lib /private/tmp /private/var/tmp /usr/bin/env

experimental-features       = nix-command
extra-experimental-features = flakes
```
Save and quit the editor to close the `/etc/nix/nix.conf` file.
    
_Note: many older posts say that it's necessary to include in your `/etc/nix/nix.conf` file the `iohk.cachix.org` substitor and its corresponding `iohk.cachix.org-1:DpRUyj7h7V830dp/i6Nti+NEO2/nhblbov/8MW7Rqoo=` trusted-public-key. However, according to this [post on Cardano Stack Exchange](https://cardano.stackexchange.com/questions/6700/macos-monterey-i7-what-is-the-best-nix-conf-file-to-use), it may no longer be necessary to include these. Apparently, they are defunct or no longer updated, so they should be safe to omit. I have tested setting up nix and the Plutus Playground environment without the `iohk.chachix.org` binaries and can confirm that the set up was successful without them._
    
Ref: (this took a lot of research and investigation to get this right)
- [CSE: Recommendation to have sandbox=false](https://cardano.stackexchange.com/a/6745/4012)
- [macOS Monterey i7, what is the best `nix.conf` file to use?](https://cardano.stackexchange.com/questions/6700/macos-monterey-i7-what-is-the-best-nix-conf-file-to-use)
- [cardano-plutus-apps-install-m1 by ranzwo](https://github.com/renzwo/cardano-plutus-apps-install-m1/blob/main/README.md)
        
4. **Restart your computer!!! (don't even ask)**

Still with us? As a sanity check, execute the following commands in your terminal and verify that Nix is installed correctly:
```
nix doctor --verbose
```
This should produce the output:
```
$ nix doctor --verbose
Running checks against store uri: daemon
[PASS] PATH contains only one nix version.
[PASS] All profiles are gcroots.
[PASS] Client protocol matches store protocol.
```
And verify the version:
```
nix --version
```
which should give:
```
$ nix --version
nix (Nix) 2.10.3
```

## Building the Plutus Playground Server and Client
You're going to want to grab a cup of coffee for this one, folks. The initial build and setup of for the `nix-shell` needed for the `plutus-playground-server` and `plutus-playground-client` takes at least an hour to complete. Ready? Here we go...

Change to the `plutus-apps/` directory:
```
cd $HOME/cardano-src/plutus-apps/
```

Next, verify that git HEAD is pointing to the correct branch that we checked out earlier (the 'tag' of the head should be 41149926c):
```
git status
```
which gives the output:
```
$ git status
HEAD detached at 41149926c
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   plutus-playground-client/package-lock.json
	modified:   plutus-playground-client/webpack.config.js

no changes added to commit (use "git add" and/or "git commit -a")
```

Next, we build the plutus playground server and client. This will take 10's of minutes to complete:
```
nix-build -A plutus-playground.server -A plutus-playground.client
```

Once the initial nix build is finished, we will fire up the `nix-shell`. This is the step that will likely take more than an hour to finish, but will be much quicker the next time we launch the nix-shell. The nix-shell is the "Haskell requirements environment". Enter the command below and enjoy that cup of coffee:
```
nix-shell
```

When the nix-shell has finished and launched, your terminal prompt should change color to indicate the nix-shell is active. Within the nix-shell, we change into the `plutus-playground-server` directory and start the plutus server:
```
cd plutus-playground-server
```
```
GC_DONT_GC=1 plutus-playground-server
```
_Note: without GC_DONT_GC=1, it seems the playground won't compile (will post "error cannot find module")_

_Note2: what is GC_DONT_GC=1? [CSE by @angerman](https://cardano.stackexchange.com/a/6676/4012)_
> "GC_DONT_GC=1 disables the boehm garbage collector in nix. 
> There are edge cases that our nix expressions can hit, which cause nix to segfault."
			
Once the Plutus Playground Server is ready, you should see something like this:
```
plutus-playground-server: for development use only
[Info] Running: (Nothing,Webserver {_port = 8080, _maxInterpretationTime = 80s})
Initializing Context
Initializing Context
Warning: GITHUB_CLIENT_ID not set
Warning: GITHUB_CLIENT_SECRET not set
Warning: JWT_SIGNATURE not set
Interpreter ready
```

Next, we will start the Plutus Playground Client in a new terminal tab (or window). Hit (command+t) or (command+n) to open up a new terminal and change into the `plutus-apps/` directory:
```
cd $HOME/cardano-src/plutus-apps/
```
and then fire up another nix-shell (this should be faster!):
```
nix-shell
```

Once the new nix-shell has launched, we need to update the `npm` dependancies:
```
sudo GC_DONT_GC=1 npm install -g npm
```
_Note: Must include GC_DONT_GC=1 
[CSE: Unable to start /plutus-apps client application](https://cardano.stackexchange.com/questions/6320/unable-to-start-plutus-apps-client-application)_

Almost there!

We next need to configure the `localhost` settings in the `webpack.config.js` file in the `plutus-playground-client` directory. Change into this directory and let's edit that file:
```
cd plutus-playground-client
```

With your favorite text editor, open the `webpack.config.js` file, search for `devServer`, and update the following parameters:
- https to 'false'
- host to '0.0.0.0'
- in API target, make "http" not "https"
- add extra "timeout" line

In sum, the `devServer` parameters should look like this:
```
module.exports = {
    devtool,
    devServer: {
        contentBase: path.join(__dirname, "dist"),
        compress: true,
        port: 8009,
        host: '0.0.0.0',
        https: false,
        stats: "errors-warnings",
        proxy: {
            "/api": {
                target: "http://localhost:8080",
                timeout: 1000 * 60 * 10,
            },
        },
    },
```

Now save and close the file.
_Note: This is needed to connect client to server with enough timeout buffer, and configure localhost config file such that the playground can run without firewall blocking it, from my understanding._

Ref:
- Http and localhost edits [CSE: @prodineeritecht comment to Playground client can't connect to playground server (all localhost)](https://cardano.stackexchange.com/a/6329/4012)
- Add timeout [CSE: @Frank DelPidio comment to Plutus Pioneer Program - Problem with plutus playground client](https://cardano.stackexchange.com/a/1992/4012)

Finally, let's start the Plutus Playground Client! In the nix-shell inside the `plutus-playground-client` directory, enter:
```
GC_DONT_GC=1 npm run start
```
_Note: before, we tried to use just `npm run start`, but the local host did not work. When in doubt, GC_DONT_GC_=1

When the client is ready, you should see something like this:
```
webpack compiled with 1 warning
ℹ ｢wdm｣: Compiled with warnings.
```

Now, open up a new tab or window in your browser and enter the following link:

[http://localhoust:8009/](http://localhost:8009/)

_Note: we're using http here--not http**s**! https was blocked by proxy firewall_

Plutus playground should appear in your browser (local host)

Cheers!

## Diary of Troubleshooting
Herein is a diary play-by-play of my failed attempts, troubleshooting, 
and blood trail of Cardano Stack Exchange questions opened, as well as help from Plutus Discord.

- Installed Nix using `sh <(curl -L https://nixos.org/nix/install) --daemon`
- Used the following `nix.conf` file

        build-users-group = nixbld
        substituters        = https://hydra.iohk.io https://iohk.cachix.org https://cache.nixos.org/
        trusted-public-keys = hydra.iohk.io:f/Ea+s+dFdN+3Y/G+FDgSq+a5NEWhJGzdjvKNGv0/EQ= iohk.cachix.org-1:DpRUyj7h7V830dp/i6Nti+NEO2/nhblbov/8MW7Rqoo= cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY=volume

- Built the Nix by `GC_DONT_GC=1 nix-build -A plutus-playground.client -A plutus-playground.server`
- Ran the configuration/requirements shell by `CG_DONT_GC=1 nix-shell`

**Issues encountered**
- After running nix-shell in ~/plutus-apps, ran check on nix by `nix doctor --verbose`. Had the following error.

        Running checks against store uri: daemon
        [FAIL] Multiple versions of nix found in PATH:
            /nix/store/iq3ra93h9kxmnrw3zlxmqn8ng5w62dra-nix-2.5.1/bin
            /nix/store/j6fqvmsfhl4frzqn2f3bzhn8hr16j5q5-nix-2.5.1/bin
  			
 - Proceed to run the `nix-build` again, ran all night, timed out or froze.  
    - Read comment here that https://cardano.stackexchange.com/questions/3878/sandbox-error-when-running-nix-build
    - Remove the sanbox and extra-sandbox-paths from nix/conf file
 
- Nix-build worked, but then nix-shell had the following error:
 
        error: getting status of /nix/var/nix/daemon-socket/socket: Operation not permitted

- Solution was to totally uninstall Nix and start fresh, and fix the nix.conf file
    - Ref [CSE: nix doctor --verbose [FAIL] Multiple versions of nix found in PATH, on MacOS i7](https://cardano.stackexchange.com/questions/6648/nix-doctor-verbose-fail-multiple-versions-of-nix-found-in-path-on-macos-i7)
    - Ref [CSE: Sandbox error when running nix build on macOS](https://cardano.stackexchange.com/a/6745/4012)

**How to Uninstall Nix**
- See [CSE solution to nix doctor --verbose [FAIL] Multiple versions of nix found in PATH, on MacOS i7](https://cardano.stackexchange.com/a/6706/4012)
    - Run script from [Github: kintsugi/uninstall-nix-osx.sh](https://gist.github.com/kintsugi/47e5e9f9d7c3a3a2f6004b6271365f8c)
    - AND/OR follow [Github: @zimbatm comment on issue](https://github.com/NixOS/nix/issues/1402#issuecomment-312496360)
    - RESTART COMPUTER
    - In terminal, ~/plutus-apps directory, `nix doctor --verbose`. Should be 3 passes.
    

## Nix helpful commands
| command | notes |
| :---- | :---- |
| nix doctor --verbose			 	         | precheck |
| nix --version					             | to check it is installed correctly |
| nix-channel --update --verbose	       	 |seems to not work|
| nix-env -iA nixpkgs.nix-update	       	 |works, command for non NixOS|
| nix-build --show-trace			         |to get details of fails|
| nix-collect-garbage			             |cleans up garbage old files https://www.mankier.com/1/nix-collect-garbage|
| --extra-experimental-features nix-command  |some commands are 'experimental' and need this flag set|


## References, Resources, and Credit

1. [CSE, macOS Monterey i7, what is the best `nix.conf` file to use?, Garant](https://cardano.stackexchange.com/questions/6700/macos-monterey-i7-what-is-the-best-nix-conf-file-to-use)
    - For macOS i7 
2. [cardano-plutus-apps-install-m1 by @renzwo](https://github.com/renzwo/cardano-plutus-apps-install-m1/blob/main/README.md)
    - For masOS M1, but good notes besides the nix.conf file extra stuff for M1.  Yarn install didn't work for me.
    
3. [MacOS Setup for Plutus Pioneer Program by @Til-D](https://github.com/Til-D/cardano-plutus)
	- macOS i7, probably the most useful tutorial reference.
	
4. [Plutus developer environment setup on MacOS Monterey by @Punkbit](https://www.punkbit.com/hacking/plutus-developer-environment-setup-on-macos-monterey/)
	- the `yarn install` did not work for me, the npm did. 
	
5. [Cardano Stack Exchange - Plutus Pioneers Program](https://cardano.stackexchange.com/questions/tagged/plutus-pioneer-program)

6. [Nix Documentation](https://nixos.org/manual/nix/stable/introduction.html)


## Contact Info
### Christophe Garant
- github: [ccgarant](https://github.com/ccgarant)
- cardano stack exchange: [ccgarant](https://cardano.stackexchange.com/users/4012/ccgarant)
- discord: ccgarant#8489
- Bitcoin & Cardano, since 2017; Ergo since 2020
- Plutus Pioneers Program, Cohort 3 (Jan 2022)

### Aaron Prunty
- github: [aaronprunty] (https://github.com/aaronprunty)
- cardano stack exchange: [mrscatterbrain](https://cardano.stackexchange.com/users/6780/mrscatterbrain)

