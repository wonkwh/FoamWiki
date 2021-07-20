# Xcode Tip

## Shortcut
- ⌥ (Option / Alt) , ⌘(Cmd), ⇧(Shift),  ⌃(Ctrl)
### Editor 
- `ctrl - i` : formatting code
- `⌘ ⌃ e` , `cmd - control + e` : edit all in scope
- `option - control + 3` : Find Next Occurrence
- `option + command + [`. `option + command + ]` : move line up or down
- `option + command + L` : 마우스위치를 센처로 
###  navigator
- `⌘ ⇧ J` ,  `(Cmd + Shift + J)` - 현재 파일위치의 folder 위치로 navigator focus 된다. 유용함.

## Debug

## Profile

## Some Behavior Recommendations
- https://www.raywenderlich.com/2325-supercharging-your-xcode-efficiency#toc-anchor-004

## plugin 
### xvim2
- bigsur 이상 12.4 버전부터 추가 플러그인을 붙이면 apple signin이 안되는 문제가 나온다. 
   - 2 copy 로 하나는 dev용으로 다른 하나는 apple signin 용으로 쓰는 방법 으로 해야함 
      - https://github.com/XVimProject/XVim2/issues/340#issuecomment-768653857
      ```
      - Download from developer.apple.com/download/more
      - Put the downloaded .xip file in a working directory somewhere
      - 2b) If you want two copies, then use two working directories
      - cd to the directory and type "xip -x filename" (22 GB)
      - After 20-30 minutes, you will get a directory called Xcode.app will appear in the current directory (31GB)
      - Rename the Xcode.app directory to something_else.app and move it to /Applications
      - You're good to go. Run your new app like you would any other.
      - As was already pointed out, you probably shouldn't run both copies at the same time!
      ```
   
   - `xcode-select` 로 xcode-beta 를 선택하자 
        > xcode-select -p //현재 설정되어 있는 xcode 확인
        > sudo xcode-select -s /Applications/[Your xcode].app/Contents/Developer
    - 위와 같이 설정하면 `xed` 로 console 에서 선택시 해당 xcode를 open할수 있다. 
        
## Reference
- [Supercharging Your Xcode Efficiency | raywenderlich.com](https://www.raywenderlich.com/2325-supercharging-your-xcode-efficiency)
   - Some Behavior Recommendations 참조해서 setting 변경
- [10 Tips and Shortcuts You Should Be Using Right Now in Xcode | by Mike Pesate | Better Programming | Medium](https://medium.com/better-programming/10-tips-shortcuts-you-should-be-using-right-now-on-xcode-2e9e1b01511e)
- 
----
- tags : #xcode  #shortcut #debug 