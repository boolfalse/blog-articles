
## Quick way to solve Git problem: Permission denied

_This was tested with non-root user and works for GitHub & GitLab (for 2FA enabled as well)._

You may encounter the following problem while your development:

```text
git@gitlab.com: Permission denied (publickey,keyboard-interactive).
```

In this article I will provide a way to solve this, and not only.

<img src="https://i.imgur.com/DfZUaa7.jpg" style="width: 100%;">

If you already have setup SSH key on your remote platform (such as GitHub), just skip this part.

**_Adding SSH to your Git account._**

Run this command to generate public & private key files:

```bash
# during creation of SSH files you can leave passphrase empty
# here you can set the private-key & public-key file names/paths
ssh-keygen -t ed25519 -C "<GIT_ACCOUNT_EMAIL>"
```

Print the output of public SSH file and copy the output content.

```bash
# let's assume we have setup default "id_ed25519.pub"
cat ~/.ssh/id_ed25519.pub
```

We will do this for GitHub as an instance. But practically you can do the same for the other Git platforms as well.

Login to your account and go to the **_“Add SSH key”_** section. Use the copied public-key content for adding as a new SSH key into your account.

Test the SSH authentication for GitHub:

```shell
# for GitHub
ssh -T git@github.com

# for GitLab
ssh -T git@gitlab.com
```

**_Fixing the issue:_**

- **_Local cause:_**

Sometimes SSH authentication test fails, and you might get something like this:

```text
git@gitlab.com: Permission denied (publickey,keyboard-interactive).
```

For avoiding this, you can follow to these steps:

```shell
# cd into ".ssh" directory
cd ~/.ssh

# make sure you have private-key file octal permission as 600
stat -c "%a %n" *

# if not, then you can change the permission like this
# in our case we might have this file: "id_ed25519-gitlab"
sudo chmod 600 id_ed25519-gitlab

# [FREQUENTLY USED] if the permissions are correct, start your SSH agent
eval `ssh-agent -s`
# this will output something like this:
"Agent pid 21590"

# [FREQUENTLY USED] now add your SSH private-key as the authorized user
ssh-add ~/.ssh/id_ed25519-gitlab
# you should get something like this
"Identity added: /home/<LOCAL_USER>/.ssh/id_ed25519-gitlab (<ACCOUNT_EMAIL>)"

# now test the SSH authentication again
# you should get some success message, something like this:
"Welcome to GitLab, @<USERNAME>!"
```

- **_Connection cause:_**

Sometimes connection is not working between your machine and the Git server, but you may want to push your code somewhere remotely.
In this kind of cases you can add the alternative/another remote and push the project there:

```shell
git remote rename origin upstream
git remote add origin git@gitlab.com:<USERNAME>/<REPO_NAME>.git
git push origin <BRANCH>
```

For existing repos (with HTTPS usage) change remote origin as SSH if needed:

```shell
git remote set-url origin git@github.com:<USERNAME>/<REPO_NAME>.git
```

**That’s it.**

In the end, just in case here’s some useful resources which you may use:

- [Git push existing repo to a new and different remote repo server?](https://stackoverflow.com/questions/5181845/git-push-existing-repo-to-a-new-and-different-remote-repo-server)
- [How can I pull/push from multiple remote locations?](https://stackoverflow.com/questions/849308/how-can-i-pull-push-from-multiple-remote-locations)
- [Git SSH not working in SSH session](https://askubuntu.com/questions/802812/git-ssh-not-working-in-ssh-session)
- [ssh: connect to host github.com port 22: Connection timed out](https://stackoverflow.com/questions/15589682/ssh-connect-to-host-github-com-port-22-connection-timed-out)
- [GIT SSH is not Working](https://stackoverflow.com/questions/17220325/git-ssh-is-not-working)

***

If you liked this article, feel free to follow me here. 😇

To explore projects working with various modern technologies, you can follow me on [**GitHub**](https://github.com/boolfalse), where I actively publicize much of my work.

For more info you can visit my website [**boolfalse.com**](https://boolfalse.com/)

Thank you !!!
