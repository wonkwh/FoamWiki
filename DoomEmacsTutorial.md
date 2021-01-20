# Doom Emacs Tutorial
- https://github.com/hlissner/doom-emacs
- SpaceEmacs와 함께 emacs 의 시작을 도와주는 양대산맥
- key binding은 evil mode 로 vim keymap 을 사용한다. 

## 설치 

### cli 
- Terminal 
	- https://github.com/alacritty/alacritty
		- iterms2 도 좋지만 더 심플하고 빠르다. 
		- 설정파일은 http://jacobzelko.com/workflow/ 보면 중간에 [.alacritty.yml](https://gist.github.com/TheCedarPrince/7743091bd8743a7568b718f30bf707c2) 파일을 공유하고 있다.
		- 위 설정파일은 [fantasque-sans mono](https://github.com/belluzj/fantasque-sans#installation) font를 사용한다 설치해 주자. 
		- 
### emacs, doom-emacs 설치
 - [Getting Started Guide](https://github.com/hlissner/doom-emacs/blob/develop/docs/getting_started.org#with-homebrew) 참조 
	 - 위 설정대로 emacs-plus 를 설치후 실행을 시키지 말고 doom-emacs 설정파일을 실행키시자. 
	 - `./emacs` folder 가 있으면 `doom doctor` 에서 다음과 같은 워닝이 나온다. 해당 폴더를 지워줘야 doom emacs 설정이 적용된다.

			> If Emacs finds an ~/.emacs file, it will ignore ~/.emacs.d, where Doom is
			  typically installed. If you're seeing a vanilla Emacs splash screen, this may explain why.	  
	  - `doom doctor` 에서 markdown-preview, Shellcheck가 없다는 워닝이 나온면 설치해 준다
		```shell
			npm install -g marked
			brew install shellcheck
		```
		- emacs 를 실행시키면 doom emacs 화면을 볼수 있다.
			

 ### evil mode 관련 
 - vim 관련 editting이랑 겹친다. 
 - [[PracticalVimBook]] 참고 
 - [emacs - Evil Mode best practice? - Stack Overflow](https://stackoverflow.com/questions/8483182/evil-mode-best-practice)
	 - 요기 보면 evil 모드에서 emacs key를 이용하는 법이 나온다. 참조하자. 

### projectile 

### swift-mode

 
## reference
- https://seorenn.github.io/note/doom-emacs.html
- http://jacobzelko.com/workflow/
- [이맥스와-함께하는-개발환경](https://blog.shiren.dev/2017-11-13-%EC%9D%B4%EB%A7%A5%EC%8A%A4%EC%99%80-%ED%95%A8%EA%BB%98%ED%95%98%EB%8A%94-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/)

----
- tags: #emacs #vim 
