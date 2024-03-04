
## vim-plug for plugins

```
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```
## Config


```bash
vi ~/.vimrc
```

```
" pre plugin config
set encoding=utf-8
scriptencoding utf-8

set fileencoding=utf-8
"filetype off                  " required

"====================Plugged================"
" plug-vim plugins
call plug#begin('~/.vim/plugged')
Plug 'tpope/vim-sensible' " Default everyone can agree on
Plug 'neoclide/coc.nvim', {'branch': 'release'}
Plug 'Yggdroot/indentLine'
Plug 'morhetz/gruvbox'
Plug 'YarNhoj/chef-syntastic'
Plug 'tpope/vim-sleuth' " Heuristically set buffer options
Plug 'ntpeters/vim-better-whitespace' "Better whitespace highlighting
Plug 'tpope/vim-surround' " quoting made simple cmd : cs"'
Plug 'tpope/vim-repeat' " enable repeating suppproted plugin maps with '.'
Plug 'tpope/vim-endwise' " wisely add 'end' in ruby
Plug 'AndrewRadev/switch.vim' " :Switch
Plug 'tpope/vim-fugitive' " Nice wrapper
Plug 'mhinz/vim-signify' " Shows diff in a sign column

Plug 'maximbaz/lightline-ale'           " lightline ale support
Plug 'macthecadillac/lightline-gitdiff' " lightline git support
Plug 'skywind3000/asyncrun.vim'         " background runner
Plug 'albertomontesg/lightline-asyncrun' " lightline asyncrun support
Plug 'itchyny/lightline.vim'            " light and configurable statusline/tabline
Plug 'editorconfig/editorconfig-vim'

"Plug 'ajh17/VimCompletesMe'             " simple lightweight tap completion
Plug 'dense-analysis/ale' " Async lint engine
Plug 'sheerun/vim-polyglot' " collection of language packs

Plug 'junegunn/fzf'
Plug 'junegunn/fzf.vim'
"Plug 'burnettk/vim-jenkins'
" Check syntax in Vim asynchronously and fix files, with Language Server Protocol (LSP) support

"let g:jenkins_url = 'https://jenkins.qa.local'
"let g:jenkins_username = 'euthuppan'
"let g:jenkins_password = '11a706b95cc5780919ed36be6ce1473ab4'

" dense-analysis/ale options
let g:ale_history_log_output = 1
let g:ale_use_global_executables = 1
" let g:ale_fix_on_save = 1
let g:ale_completion_enabled = 1
"let g:ale_open_list = 1
let g:ale_linters = {
\   '*': ['remove_trailing_lines', 'trim_whitespace'],
\   'Jenkinsfile': ['checkci'],
\   'groovy': ['checkci'],
\}

"augroup
"    autocmd BufWritePost *.groovy execute '!checkci %:p'
"    autocmd BufWritePost Jenkinsfile execute '!checkci %:p'
"augroup

augroup
    au BufNewFile,BufRead groovy setf Jenkinsfile
augroup

"au BufNewFile,BufRead groovy setf groovy
"au BufNewFile,BufRead Jenkinsfile setf groovy

"Plug 'nvim-lua/plenary.nvim'
"Plug 'ckipp01/nvim-jenkinsfile-linter'
"Plug 'prabirshrestha/asyncomplete.vim'  " background completion
"Plug 'prabirshrestha/asyncomplete-lsp.vim' " completion using lsp
"Plug 'prabirshrestha/vim-lsp'
"Plug 'prabirshrestha/asyncomplete-lsp.vim' " completion using lsp
"Plug 'mattn/vim-lsp-settings'


" Use tab for trigger completion with characters ahead and navigate.
" NOTE: Use command ':verbose imap <tab>' to make sure tab is not mapped by
" other plugin before putting this into your config.
inoremap <silent><expr> <TAB>
      \ pumvisible() ? "\<C-n>" :
      \ <SID>check_back_space() ? "\<TAB>" :
      \ coc#refresh()
inoremap <expr><S-TAB> pumvisible() ? "\<C-p>" : "\<C-h>"

function! s:check_back_space() abort
  let col = col('.') - 1
  return !col || getline('.')[col - 1]  =~# '\s'
endfunction

call plug#end()

"=============LightLine Config==========="
" branch name function for lightline
function! LightlineFugitive()
  if exists('*FugitiveHead')
    let branch = FugitiveHead()
    return branch !=# '' ? ''.branch : ''
  endif
  return ''
endfunction

" readonly indicator function for lightline
function! LightlineReadonly()
  return &readonly ? '' : ''
endfunction

" lightline config
let g:lightline = {
\ 'active': {
\   'left': [['mode', 'paste'],
\            ['fugitive', 'readonly', 'filename']],
\   'right': [['percent', 'lineinfo'],
\             ['fileformat', 'fileencoding', 'filetype'],
\             ['linter_checking', 'linter_errors', 'linter_warnings', 'linter_infos', 'linter_ok'],
\             ['asyncrun_status']]
\ },
\ 'separator': { 'left': '', 'right': '' },
\ 'subseparator': { 'left': '', 'right': '' },
\ 'component_expand': {
\   'asyncrun_status': 'lightline#asyncrun#status',
\   'linter_checking': 'lightline#ale#checking',
\   'linter_infos': 'lightline#ale#infos',
\   'linter_warnings': 'lightline#ale#warnings',
\   'linter_errors': 'lightline#ale#errors',
\   'linter_ok': 'lightline#ale#ok',
\ },
\ 'component_function': {
\   'readonly': 'LightlineReadonly',
\   'fugitive': 'LightlineFugitive',
\ },
\ 'component_type': {
\   'linter_checking': 'right',
\   'linter_infos': 'right',
\   'linter_warnings': 'warning',
\   'linter_errors': 'error',
\   'linter_ok': 'right',
\ },
\}


filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
"================Syntastic================="
   set statusline+=%#warningmsg#
   set statusline+=%*

" Language Specific
" HTML
" JS
" Golang
   let g:go_highlight_functions = 1
   let g:go_highlight_methods = 1
   let g:go_highlight_fields = 1
   let g:go_highlight_types = 1
   let g:go_highlight_operators = 1
" Ruby
" Python
  let g:ale_python_flake8_options = '--ignore=E501'

"==================   Theme    ============"
    set t_Co=256
    colorscheme gruvbox
    set background=dark
    syntax on
"================Spaces and Tabs==========="
    set softtabstop=2
    set expandtab
    set shiftwidth=2
"==================UI Config==============="
    set number
    set showcmd
    set cursorline
    filetype indent on
    set showmatch
    set laststatus=2
    set colorcolumn=80
"==================Searching================"
    set incsearch
    set hlsearch
    :hi Search ctermbg=3 ctermfg=8
"==================Movement================="
    set ttyfast
    nnoremap B ^
    nnoremap E $
    set backspace=indent,eol,start
    nnoremap <C-L> <C-W><C-L>
    nnoremap <C-K> <C-W><C-K>
    nnoremap <C-J> <C-W><C-J>
    nnoremap <C-H> <C-W><C-H>
"==============Leader Shortcuts============="
    let mapleader=' '
    inoremap jk <esc>
    nnoremap <leader>ev :vsp $MYVIMRC<CR>
    nnoremap <leader>sv :source $MYVIMRC<CR>
    nnoremap <leader>s :update<CR>
"====================NERDTree=============="
    nnoremap <leader>d :NERDTree<CR>
"    let g:NERDTreeDirArrowExpandable="+"
"    let g:NERDTreeDirArrowCollapsible="~"
"====================Powerline============="
set runtimepath+=$HOME/.local/lib/python2.7/site-packages/powerline/bindings/vim/

" Always show statusline
set laststatus=2

" Copy to clipboard with yy
set clipboard+=unnamed

let g:ale_disable_lsp = 1
let g:ale_sign_column_always = 1
```


## Install Plugins

```
:PlugInstall
```
