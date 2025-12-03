## Zsh setup

Basic standardized setup for shell can be done following instructions [here](add link to ssh page link). This script copies zshrc, vimrc and tmux.conf (WIP).

* zsh is the recommended shell. You can use your own zshrc if you choose but you have load the modules initialization script (see notes below).

Default zshrc file loads the environment modules and sets up the paths for locating tools and tool bundles. More information on Environment modules and tools can be found.

* `/auto/tools/modulefiles/setup/shell_init.zsh` sourced by default zshrc looks like this:
```
source /auto/tools/installs/modules/4.5.2/init/zsh             <-- setup environment modules
module --silent --force purge                                   <-- purge all existing modules if any
module unuse /auto/tools/installs/modules/4.5.2/modulefiles    <-- remove default path
module use   /auto/tools/modulefiles                           <-- use specific path
module -s load sw/infra.v0.1                                    <-- load infra tools

```


* $HOME/.zshrc needs to be modified to load appropriate tool bundles during initialization as needed.

```
if [[ -o interactive ]]; then
  source /auto/tools/modulefiles/setup/shell_init.zsh
```

## tmux

* tmux is recommended for text based persistent sessions.
