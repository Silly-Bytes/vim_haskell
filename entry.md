# Vim + Haskell

So you're writing in the right language using the right tool already, but lets
put some extra magic under our sleeves.


## Expectations

* Omnicompletion
* Convenient mappings for
    * Building
    * Linting
    * Testing
    * Jump to importations
* Ghc-mod integration
    * Type inserting
    * Case splitting
    * Type asserting
* Hlint integration
* Code formatting
    * Trailing white space
    * Trailing blank lines
    * Spaces over tabs
* Easy arrows generation
* Types abbreviations

Most of this functionality is achieved by using already available tools and
already available Vim plugins for those tools. So I'll assume you have your way
to install the plugins (I'm using
[vim-plug](https://github.com/junegunn/vim-plug)).

Here is my complete
[.vimrc](https://github.com/alx741/dotfiles/blob/master/nvim/.config/nvim/init.vim).


**Important**: Every line of vimrc used should be enclosed in an `:h :augroup`:

    augroup ft_haskell
        au!

        ...

    augroup END


### Omnicompletion

The [neco-ghc](https://github.com/eagletmt/neco-ghc) plugin declares a complete
omnifunction. Use it by defining the local `omnifunc`:

    au FileType haskell setlocal omnifunc=necoghc#omnifunc

![](./omni.gif)


### Convenient mappings

Using [vim-dispatch](https://github.com/tpope/vim-dispatch),
[neomake](https://github.com/neomake/neomake) and `g` prefixed mappings we can
now build with `gj` ([Stack](https://docs.haskellstack.org/en/stable/README/),
lint with `gl`, and test with `gk`. Also use `gI` to jump to the first *import*.

You will need the Stack tool of course and *hlint* that you can install with
`stack install hlint`.

    au FileType haskell nnoremap <buffer> gl :Neomake<CR>
    au FileType haskell nnoremap <buffer> gj :Make build<CR>
    au FileType haskell nnoremap <buffer> gk :Make test<CR>
    au FileType haskell nnoremap <buffer> gI gg /\cimport<CR><ESC>:noh<CR>


### Ghc-mod integration

[ghc-mod](https://hackage.haskell.org/package/ghc-mod) is the *Happy Haskell
Programming package*! With a whole bunch of functionality, here we will be using
just a few:

* Type inserting
* Case splitting
* Type asserting

You need the *ghc-mod* package: `stack install ghc-mod` and the [ghcmod-vim
plugin](https://github.com/eagletmt/ghcmod-vim).

    au FileType haskell nnoremap <silent><buffer> git :GhcModTypeInsert<CR>
    au FileType haskell nnoremap <silent><buffer> gfs :GhcModSplitFunCase<CR>
    au FileType haskell nnoremap <silent><buffer> gtt :GhcModType<CR>


`git` (*g insert type*) will insert the missing type declaration of an
expression, take for instance this Haskell code:

```haskell
module Hello where

f (Just a) = Left a
f Nothing = Right ()
```

With the cursor in the first `f` (the function name) using the `tt` mapping will
produce:

```haskell
module Hello where

f :: Maybe a -> Either a ()
f (Just a) = Left a
f Nothing = Right ()
```

![](./ghcmod.gif)

Neat!, go ahead and play around with the other mappings, you'll be not
disappointed.


### Hlint integration

By default [Neomake](https://github.com/neomake/neomake) will use *hlint* on the
current file when the `:Neomake` command is invoked on a Haskell source file, so
by adding a mapping:

    au FileType haskell nnoremap <buffer> gll :Neomake<CR>

`gll` will open the location list with the lints, which takes us to some
convenience mappings:

    au FileType haskell nnoremap <buffer><silent> gl<space> :call ToggleLocationList()<CR>
    au FileType haskell nnoremap <buffer><silent> glc :sign unplace *<CR>

For toggling the location list `gl<space` and cleaning the lints `glc`.


### Code formatting, Code cleaning

*Hindent* allows to beautify Haskell code, you could used it by setting the
`formatprg` option and then trigger it with the `=` command, but there is a
problem: if your code happens to have any syntax errors, it will be replaced
with a nasty error message. To handle this we're going to use the
[vim-hindent](https://github.com/alx741/vim-hindent) plugin instead, so each
time we save a Haskell source file it will be automatically beatified.

Don't forget to configure it:

```vim
let g:hindent_on_save = 1
let g:hindent_line_length = 80
let g:hindent_indent_size = 4
```

One extra thing left is to align stuff in the code so it looks nicer

```vim
au FileType haskell nmap <silent><buffer> g<space> vii<ESC>:silent!'<,'> EasyAlign /->/<CR>
```

Take for instance this very dumb example for the sake of the argument:

```Haskell
module Test where

f :: Int -> String
f x = case x of
    1   -> "1"
    2 ->   "2"
    3 -> "3"
```

Using `g<space>` we got:

```Haskell
module Test where

f :: Int -> String
f x =
  case x of
    1 -> "1"
    2 -> "2"
    3 -> "3"
```

![](./format.gif)

So much better!


### Easy arrows generation

In Haskell, operators like `->` and `=>` are very common and I find it
cumbersome to type theme manually. Lets define a function:

```vim
function! Make_arrow(type)
    if a:type
        if (matchstr(getline('.'), '\%' . col('.') . 'c.') ==? ' ')
            exe "norm! a->  "
        else
            exe "norm! a ->  "
        endif
        exe "startreplace"
    else
        if (matchstr(getline('.'), '\%' . col('.') . 'c.') ==? ' ')
            exe "norm! a=>  "
        else
            exe "norm! a =>  "
        endif
        exe "startreplace"
    endif
endfunction
```

And some insert mode mappings:

```vim
au FileType haskell inoremap <buffer> ;; <ESC>:call Make_arrow(1)<CR>
au FileType haskell inoremap <buffer> ;: <ESC>:call Make_arrow(0)<CR>
```

So while in insert mode typing `;;` or `;:` will insert `->` or `=>`
respectively. Additionally it will avoid duplicated spaces between the types and
the arrows.


### Types abbreviations

Maybe i'm a terrible typist, but writing the first upper case letter of the most
common types hurst my pinkie. So by using some insert mode abbreviations:

    au FileType haskell inoreab <buffer> int Int
    au FileType haskell inoreab <buffer> integer Integer
    au FileType haskell inoreab <buffer> string String
    au FileType haskell inoreab <buffer> double Double
    au FileType haskell inoreab <buffer> float Float
    au FileType haskell inoreab <buffer> true True
    au FileType haskell inoreab <buffer> false False

Now I can type all lower case without having to bother with the *shift* key and
the capitalized version will be inserted instead.
