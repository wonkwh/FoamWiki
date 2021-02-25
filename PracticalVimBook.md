# Practical Vim

## part 0
### the vim way

#### tip1

#### tip2

#### tip3

----
## part 1: Modes

----
## part 2: Files

## part 3:  보다 빠르게 
(Getting Around Faster)

### 파일내에서 Motions 으로 이동하기 
(Navigate Inside Files with Motions)
- [Learn-Vim/ch05\_moving\_in\_file.md at master · iggredible/Learn-Vim (github.com)](https://github.com/iggredible/Learn-Vim/blob/master/ch05_moving_in_file.md)

#### tip 47 손을 중앙에 유지하세요 
(Keep Your Fingers on the Home Row)

- home row란 
	-  that means the left-hand fingers rest on a, s, d, and f, while the right-hand fingers rest on j, k, l, and ; keys.
-  h, j, k, l key로 화살키를 대체하자.
-  `;` key는 새끼 손가락으로 
-  만약 익숙해지지 안는다면 강제하는 법은 vimrc 에 다음을 추가하는 법이 있다

		noremap <Up> <Nop>
		noremap <Down> <Nop>
		noremap <Left> <Nop>
		noremap <Right> <Nop>

#### tip 48:  실제 라인과 보여지는 라인수를 구별하자
- 실제 라인을 보려면 
	- `set number` 
- `gj`, `gk` 등의 키는 display line 이동 키

| Command | Move Cursor       |
| ------- | ----------------- |
| j       | Down real line    |
| gj      | Down display line |
| k       | up real           |
| gk      | up display        |

- vimrc에 다음을 추가하면 j, k 키를 display line 이동으로 변경가능

			nnoremap k gk 
			nnoremap gk k
			nnoremap j gj
			nnoremap gj j
			
#### tip 49: 단어단위로 음직이기 
![[Pasted image 20210226051559.png]]
	
- practice 
	- `ea`, `gea` key: 단어뒤에 추가 
		goes faster
		
#### tip 50: 글자로 찾기 
- `f{char}`, `;`, `,` key 
- `zz` key 
----
## part 3: Register



## vim 단축키
![](vim.png)
![](vim2.png)

----
- tags: #book #vim 