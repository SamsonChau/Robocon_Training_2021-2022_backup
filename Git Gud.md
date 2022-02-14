# Git Gud
This document discusses more Git/Github features for you to git gud <sub><sub><sub>haha!</sub></sub></sub> at git. We recommend you to read through [Git Basics][basics] before reading this.

[basics]: https://github.com/HKUST-Robotics-Team/HKUST-Robotics-Team-SW-Tutorial-2021/blob/main/Advanced%20Notes%201%20-%20Git/Git%20Basics.md

### Branches
- Run `git branch` to see a list of branches on the repository. Your current branch will be indicated with an asterisk on the left.
- To make a new branch, use `git checkout -b <new branch name>`.
- To switch between branches, use `git checkout <branch name>` and commit any changes to the branch.
- To merge changes from another branch into the current branch, run `git merge <other branch name>`.

### Other GitHub Useful Features
- Forking: As a GitHub user, you have the ability to fork any repository you can access, 
copying the repository under your user. 
We do this by clicking the “Fork” button in the top-right.
- Pull Requests: Once you’ve forked a repository and made some changes to your version, 
you may want to request those changes be added to the main version of the repository.

  For example, if you wanted to add a new feature to Bootstrap (an open source framework), you could fork the repository, 
make some changes, and then submit a pull request. This pull request could then be evaluated and 
possibly accepted by the people who run the Bootsrap repository. This process of people making a 
few edits and then requesting that they be merged into a main repository is vital for what is known as 
open source software, or software created by contributions from a number of developers.

### gitignore
When you make commits in a git repository, you choose which files to stage and commit by using git add FILENAME and then git commit. But what if there are some files that you never want to commit? It's too easy to accidentally commit them (especially if you use `git add .` to stage all files in the current directory). That's where a .gitignore file comes in handy. It lets Git know that it should ignore certain files and not track them.

- A .gitignore file is a plain text file where each line contains a pattern for files/directories to ignore. 
- You can put it in any folder in the repository and you can also have multiple .gitignore files. 
```
.DS_STORE
```
- This will ignore any files named .DS_Store, which is a common, hidden file on macOS.

#### Directories
- You can ignore entire directories, just by including their paths and putting a `/` on the end
```
node_modules/
logs/
```

#### Wildcards
The `*` matches 0 or more characters (except `/`). So, for example, `*.log` matches any file ending with the `.log` extension.
Another example is `*~`, which matches any file ending with `~`, such as index.html~.
You can also use `?`, which matches any one character (except `/`).

#### Negation
You can use the `!` prefix to negate a file that would be ignored.
```
*.log
!example.log
```
In this example, `example.log` is not ignored, even though all other files ending with `.log` are ignored.
But be aware, you can't negate a file inside an ignored directory:
```
logs/
!logs/example.log
```
Due to performance reasons, git will still ignore `logs/example.log` here because the entire `logs/` directory is ignored.

#### Double Asterisk
`**` can be used to match any number of directories.

- `**/logs` matches all files or directories named logs (same as the pattern `logs`).
- `**/logs/*.log` matches all files ending with .log in a logs directory.
- `logs/**/*.log` matches all files ending with .log in the logs directory and any of its subdirectories.
- `**` can also be used to match all files inside a directory. For example, `logs/**` matches all files inside of `logs/`.

#### Comments
Lines starting with `#` are comments:
```
# macOS Files
.DS_Store
```
