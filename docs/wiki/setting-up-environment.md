# Steps to setup the environment

* We now always require that b2c is built using **JDK 17**
* Install the latest stable verison of Android Studio.
* Checkout the application main [b2c repository](https://github.com/deliveryhero/pd-mob-b2c-android) for the Application.
* Install [Homebrew](https://brew.sh/)
* [Jazz Up Your “ZSH” Terminal](https://www.freecodecamp.org/news/jazz-up-your-zsh-terminal-in-seven-steps-a-visual-guide-e81a8fd59a38/)
* Optional: set Login shell default to `/bin/zsh`
* Optional: Install [iTerm2](https://www.iterm2.com/index.html) and [Oh My Zsh](https://ohmyz.sh/#install)
* Add ADB to PATH on OSX:
```
touch .zshrc
# replace {username} with your local user
export PATH=$PATH:/Users/{username}/Library/Android/sdk/platform-tools/
source .zshrc
```
* Follow these [steps](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/code-readability-style/) to setup the code style in Android Studio
* Prefered git setup and alias for none force push
```
git config --global pull.rebase true
git config --global alias.pushf "push --force-with-lease"
git config --global push.default current
```

## M1 Mac Troubleshooting
This section contains guidance for troubleshooting common issues on M1 Mac.

### Homebrew

When installing Homebrew, you might be experiencing an issue regarding missing path as shown below:

```
Warning: /opt/homebrew/bin is not in your PATH.
```

Please just follow the instructions showing in terminal itself:

```
==> Next steps:
 Run these two commands in your terminal to add Homebrew to your PATH:

    echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/xxx/.zprofile
    eval "$(/opt/homebrew/bin/brew shellenv)"
```

Then `brew` should work now!
