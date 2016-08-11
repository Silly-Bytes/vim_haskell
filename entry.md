# Vim + Haskell

So you're writing in the right language using the right tool already, but lets
put some extra magic under our leaves.


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
* Code formatting
* Code cleaning
    * Trailing whitespace
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

![](./neco-ghc.png)
