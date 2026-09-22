# Install GitBash
1. Download from https://git-scm.com/downloads
2. Install git and git-bash

## Setup user name and email
1. git config --global user.name [<username>]
2. git config --global user.email [<email-address>]

## To check the above command
3. git config --global user.name
4. git config --global user.email

## Configure cross-platform line endings
5. git config --global core.autocrlf input 
6. git config --global core.safecrlf false

## To check the above command
7. git config --global --list

## Use this if you face issues with LF/CRLF
- echo "* text=auto eol=lf" > .gitattributes

## Create a new repo
1. git init
2. git add .
3. git commit -m "Initial commit"
4. git remote add origin <repository-url>
5. git push -u origin main

## Delete local branch
git branch -D [branch-name]

## Delete remote branch
git push origin --delete [branch-name]
