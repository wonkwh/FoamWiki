# VimEditor 
- [[VImTips]]

## nvim config
- [siduck76/neovim-dots: my neovim configs](https://github.com/siduck76/neovim-dots)
### key mapping
> `space` 를 Leader Key로 

 - select any text + <kbd> y </kbd> copies the selected text
   - <kbd> Ctrl </kbd> <kbd> n </kbd> toggle neovim tree
   - <kbd> Shift </kbd> <kbd> n </kbd> toggle line number
   - <kbd> Ctrl </kbd> <kbd> a </kbd> copies everything in the current file
   - 
- terminal
   - <kbd> Ctrl </kbd> <kbd> l </kbd> Open terminal vertically over right
   - <kbd> Ctrl </kbd> <kbd> x </kbd> Open terminal horizontally below the current window
  
  

- file + telescope(project)
   - leader + <kbd> f </kbd> <kbd> b </kbd> open all buffers , with telescope
   - leader + <kbd> f </kbd> <kbd> p </kbd> search and preview images with telescope
   - leader + <kbd> f </kbd> <kbd> f </kbd> find files in the current DIR , with telescope
   - leader + <kbd> f </kbd> <kbd> o </kbd> open recently edited files , with telescope
   - leader + <kbd> f </kbd> <kbd> h </kbd> opens up a manpage like thing but for all vim related things , with telescope
   - leader + <kbd> f </kbd> <kbd> m </kbd> formats or beautifies the code in current window via neoformat
     (currently only html ,css , js can be formatted . To be able to use this keybind you need to install the formatter locally for your language , in my case prettier was required only so I installed it. check this <a> https://github.com/sbdchd/neoformat</a>).
- `<C-u>`, `<C-d>`, `<C-b>`, `<C-f>`, `<C-y>` and `<C-e>` : Smooth scrolling for window movement commands.

### lsp, dap setting


## Vim, NVim 관련 링크들
- https://www.chrisatmachine.com/neovim/
   - [ChristianChiarulli/nvim: Truly the Ultimate Neovim Config NVCode (github.com)](https://github.com/ChristianChiarulli/nvim)
- [My Writing & Coding Workflow | the cedar ledge (jacobzelko.com)](http://jacobzelko.com/workflow/)
   - tmux, neovim, alacritty, tmux, zsh
- https://www.reddit.com/r/programming/comments/ihe99i/learn_vim_the_smart_way_a_book_to_learn_the_good/ #vim 
	- https://stackoverflow.com/questions/1218390/what-is-your-most-productive-shortcut-with-vim/1220118#1220118
	- https://github.com/iggredible/Learn-Vim 
- [siduck76/neovim-dots: my neovim configs (github.com)](https://github.com/siduck76/neovim-dots)
   - 요걸 기준으로 정리하자..
----
## Link
- [[VimConfig]], [[PracticalVimBook]]
----
- tags: #vim #neovim 