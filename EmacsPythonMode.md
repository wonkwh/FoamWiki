# Emacs Python Mode

## pyenv 설치

```zsh
brew update
brew install pyenv
```

### update config
```
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
```

### path 추가 
```zsh
echo 'export PYENV\_ROOT="$HOME/.pyenv"' \>> ~/.zshrc
echo 'export PATH="$PYENV\_ROOT/bin:$PATH"' \>> ~/.zshrc
```

### 수동으로 zsh에 추가
```zsh
export PYENV\_ROOT\="$HOME/.pyenv"
export PATH\="$PYENV\_ROOT/bin:$PATH"
```

pyenv install --list
pyenv install 3.9.1
pyenv global 3.9.1
python --version

## reference
- [Doom Emacs의 Python 프로그래밍 환경 - Seorenn Note](https://seorenn.github.io/note/doom-emacs-python-env.html)
- [pyenv로 파이썬 버전 관리하기 - Seorenn Note](https://seorenn.github.io/note/pyenv-manage-python-version.html)
- [Python Programming in Emacs | Chao's Blog (upenn.edu)](https://www.seas.upenn.edu/~chaoliu/2017/09/01/python-programming-in-emacs/)

----
- tags: #python #emacs 