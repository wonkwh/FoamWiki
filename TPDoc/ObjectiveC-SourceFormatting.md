# Objective-C File source file formatting

- 설치
```
brew install clang-format
```

- Sublime Text를 이용하려면 #sublimeText
	- Packge Control : Install Package -->  Clang Format
	- Custom 설정
		- Preferences --> Package Settings --> Clang Format 의  Settings-User와 Custom Style- User
- VSCode 에서는 
   - clang-format extension 설치
   - `~/.clang-format` 에 설정파일 저장 #vscode 
## reference
- https://github.com/TextureGroup/Texture/blob/master/.clang-format
	- Texture library에서 쓰는 clang-format config file 
	- json type으로 바꿔스려면 https://onlineyamltools.com/convert-yaml-to-json 이용
----
- tags: #Objective-C 