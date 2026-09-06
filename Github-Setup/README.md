The first step to initalising a Github repo on your local machine is to set up your SSH key:
ssh-keygen -t ed25519 -C "your.email@example.com" -f ~/.ssh/keystore
The -f stores the key in ~/.ssh/keystore
There will be 2 files stored in the directory.
Cat the .pub file and copy it.
Go to github - Settings - SSH and GPG key - add new SSH key
paste the ssh key into the text box and give it a name.

The second step is to start the ssh agent.
eval "$(ssh-agent -s)"
you should see an output of Agent pid "numbers"

Add the key to the agent: ssh-add ~/.ssh/keystore

test the connection:
ssh -T git@github.com

Clone the repo
e.g
git clone git@github.com:Username/example-repo.git

then initalise the repo
git init

create your first commit, stage and push it:
git add README.md 
git commit -m "first commit"
git remote add origin git@github.com:Husseinmalki/docker-learning.git
git push

