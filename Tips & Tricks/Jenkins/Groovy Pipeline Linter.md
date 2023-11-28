
For vim users, this is how you can asynchronously lint your groovy and Jenkinsfile code, straight from the Jenkins API.
## API Token

- Go to https://jenkins.qa.local 
- Click your profile
- Click Configure
- Create an API Token

Set this at the bottom of your ~/.bashrc or ~/.zshrc
```bash
export JENKINS_USERNAME="euthuppan"
export JENKINS_URL="https://jenkins.qa.local"
export JENKINS_SECRET="<API_TOKEN_HERE>"
```
## Linter Script

Create the linter script which will make the API call to Jenkins to lint your file

```
mkdir ~/.scripts
vim ~/.scripts/checkci.sh
```

Paste this code into it
```bash
function _checkci() {
	local jenkinsfile="${1}"
	curl --user "$JENKINS_USERNAME:$JENKINS_SECRET" -X POST -F "jenkinsfile=<$jenkinsfile" "$JENKINS_URL/pipeline-model-converter/validate"
}

_checkci "$*"
exit 0
```

Create a symbolic link so you can call `checkci` from anywhere

```bash
ln -snf ~/.scripts/checkci.sh /usr/local/bin/checkci
```

## Vimrc

```bash
augroup
    autocmd BufWritePost *.groovy execute '!checkci %:p'
    autocmd BufWritePost Jenkinsfile execute '!checkci %:p'
augroup
```


From this [gist](https://gist.github.com/MorganGeek/2958ba47630a176733e0136b42557284)

