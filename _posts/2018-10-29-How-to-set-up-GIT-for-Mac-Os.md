---
layout: post
title: How to set-up GIT for Mac Os?
author: FX
---

At the heart of GitHub is an open source version control system (VCS) called [Git](https://git-scm.com/). Git is responsible for everything GitHub-related that happens locally on your computer.

To use Git on the command line, you'll need to download, install, and configure Git on your computer. [Homebrew](https://brew.sh/) is the missing packages manager for MacOS. I’ll assume that you have it installed and running on your computer. If not just head over to their website and follow the installation instructions.

The first step is to install GIT locally. 

Enter the following command:

```brew install git```

In order to maximise security it is highly recommended to connect either through the HTTPS method or via SSH.

If you have enabled double factor authentication it might be difficult to configure. In that case use the SSH config.

After you've checked for existing SSH keys, you can generate a new SSH key to use for authentication, then add it to the ssh-agent and to your Git account.