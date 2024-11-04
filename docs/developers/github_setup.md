#Setting up Git 

The Remotelabz project is hosted on a secure repository on GitHub at https://github.com/remotelabz/.

If you need to commit code, you have to sign it first. 

For Mac and Windows there is a complete guide at https://github.com/microsoft/vscode/wiki/Commit-Signing

Here we will focus on ubuntu 

First install git and gpg (if not already installed)

```bash
apt-get install git
apt-get install gpg
```

#Create GPG keys
To push code on RemoteLabz git, you need to sign your commits with a key linked up to your account.
First create a key with the command
```bash
gpg --full-generate-key
```
Answer all the questions
At the end of the process a key with a key ID is generated
![Screenshot](/images/Developers/git_images/gpg_key_generated.png)

#add GPG key to your github account
Retrieve your key ID by using the command
```bash
gpg --list-secret-keys --keyid-format=long
```
![Screenshot](/images/Developers/git_images/gpg_key_list.png)
Then export the key using the retrieved ID.

```bash
gpg --armor --export your_key_ID
```
then go to your github account and add the key in your account -> settings -> ssh and gpg keys -> add key 


For more informations, you can follow the guide <a href="https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account"> here</a>

Once you've added the key, you will need to order git to sign the commits.But first, Git needs to know that your key exists.
```bash
git config --global user.signingkey <Your Key ID>
```
Then it must be told to use it when signing the commits

```bash
git config --global commit.gpgsign true
```
#VSCode
If you wish to use Git with github, you need to do the following.

##installing git on VSCode

On VSCode, go to plugins and install the Git Extension Pack.
![Screenshot](/images/Developers/git_images/vscode_git_add.png)
##Adding code signing on VSCode

Then, into settings, go to Extension->Git and tick the box `Git enable commit signing`
![Screenshot](/images/Developers/git_images/vscode_git_code_signing.png)



