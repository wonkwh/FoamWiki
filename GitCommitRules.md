
# Git Rules
 - [ ] : [[GitMoji]] 를 이용해서 재정의
  
## Styleguides

### Git Commit Messages
* Use the present tense ("Add feature" not "Added feature")
* Use the imperative mood ("Move cursor to..." not "Moves cursor to...")
* Limit the first line to 72 characters or less
* Reference issues and pull requests liberally after the first line
* When only changing documentation, include `[ci skip]` in the commit title
* Consider starting the commit message with an applicable emoji:
	*    🎨 :art: when improving the format/structure of the code
	*    🐎 :racehorse: when improving performance
	*    🚱 :non-potable_water: when plugging memory leaks
	*    📝 :memo: when writing docs
	*    🐧 :penguin: when fixing something on Linux
	*    🍎 :apple: when fixing something on macOS
	*    🏁 :checkered_flag: when fixing something on Windows
	*    🐛 :bug: when fixing a bug
	*    🔥 :fire: when removing code or files
	*    💚 :green_heart: when fixing the CI build
	*    ✅ :white_check_mark: when adding tests
	*    🔒 :lock: when dealing with security
	*    ⬆️ :arrow_up: when upgrading dependencies
	*    ⬇️ :arrow_down: when downgrading dependencies
	*    👕 :shirt: when removing linter warnings


### MessageFormat
>  [Reference](http://karma-runner.github.io/1.0/dev/git-commit-msg.html)

| Type | Contents |
|--|--|
|feat| new feature for the user, not a new feature for build script
|fix| bug fix for the user, not a fix to a build script
|docs| changes to the documentation
|refactor| refactoring production code, eg. renaming a variable
|style| formatting, missing semi colons, etc; no production code change
|test| adding missing tests, refactoring tests; no production code change)
|chore| updating grunt tasks etc; no production code change

- Example

    ```
    refactor: Refactor subsystem X for readability 

    {body...}

    #1 or resolves #1 // reference issues
    ```

### Branch - Git Flow
- default branch : `dev`
- `main`: production-ready state
- `dev`: latest delivered development changes for the next release
- `feat`: develop new features for the upcoming or a distant future release
- `deploy`: support preparation of a new production release
- `hotfix`: act immediately upon an undesired state of a live production version
- `{feat}/{feature}`
- Example

    ```
    feat/create-note
    ```
	
	----
	- tags: #git, #styleguide