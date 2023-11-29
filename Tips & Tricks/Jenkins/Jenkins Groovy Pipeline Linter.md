
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

## .vimrc 

If you use Plugged for plugin management

```
Plug 'dense-analysis/ale'

" dense-analysis/ale options
let g:ale_history_log_output = 1
let g:ale_use_global_executables = 1
let g:ale_fix_on_save = 1
let g:ale_completion_enabled = 1
let g:ale_open_list = 1
let g:ale_linters = {
\   '*': ['remove_trailing_lines', 'trim_whitespace'],
\   'Jenkinsfile': ['checkci'],
\   'groovy': ['checkci'],
\}
```

Do this in vim if you haven't installed ale yet

```
:PlugInstall
```

## Extending ale plugin

This essentially extends the ale plugin to call the checkci.sh script when a groovy file is being edited.

```
mkdir ~/.vim/plugged/ale/ale_linters/groovy
vim ~/.vim/plugged/ale/ale_linters/groovy/checkci.vim
```

Place this in `~/.vim/plugged/ale/ale_linters/groovy/checkci.vim`

```
call ale#linter#Define('groovy', {
\   'name': 'checkci',
\   'executable': 'checkci',
\   'command': 'checkci %s',
\   'callback': 'ale_linters#groovy#checkci#HandleJenkinsValidator',
\})

function! ale_linters#groovy#checkci#HandleJenkinsValidator(buffer, lines) abort
    " Regular expression to match messages:
    " They look like:
    " WorkflowScript: 7: Expected a step @ line 7, column 17.
    let l:pattern = '\v^WorkflowScript: (\d+): (.*) \@ line (\d+), column (\d+)\.(\_.*)$'

    " For each match, update the l:output list:
    let l:output = []

    for l:match in ale#util#GetMatches(a:lines, l:pattern)
        let l:code = l:match[5]

        call add(l:output, {
        \   'lnum': l:match[3] + 0,
        \   'col': l:match[4] + 0,
        \   'text': l:code . ': ' . l:match[2],
        \})
    endfor

    return l:output
endfunction
```

Inspired by this [gist](https://gist.github.com/MorganGeek/2958ba47630a176733e0136b42557284)

