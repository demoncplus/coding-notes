# -

右键powershell管理员打开
dir .查看当前目录
-m 文件名 删除当前文件夹
cd 文件名 进入某个文件夹
cd ..进入上一文件夹
pwd 查看当前路径
Remove-Item -LiteralPath "路径" -Recurse -Force 彻底删除该文件夹
Test-Path "路径" 删除后测试，返回false表示删除成功

创建初始化项目：
cd ~\Documents
mkdir test
cd test
git init
"Git练习项目" > README.md
git add .
git commit -m "初始化项目"

练习1 观察工作区状态
code：
"第一篇笔记" > note.txt
git status

New-Item note.txt 创建空文件
"第一篇笔记" > note.txt等价于 "第一篇笔记" | Set-Content note.txt
> 会覆盖原文件内容，想保留旧内容并继续写，用 >>；eg: "第二行内容" >> note.txt
查看状态git status，这时候应该出现在Untracked files中



练习2 观察差异
code：
"第二行" >> note.txt
git diff
git add note.txt
git diff
git diff --staged
git diff HEAD

git diff              工作区 vs 缓存区
git diff --staged     缓存区 vs 最近提交
git diff HEAD         当前全部修改 vs 最近提交



练习3 创建多个小提交
code:
git commit -m "添加学习笔记"
"用户功能" | Set-Content user.txt
git add user.txt
git commit -m "添加用户示例"
"订单功能" | Set-Content order.txt
git add order.txt
git commit -m "添加订单示例"
git log --oneline

整套操作可以理解为：
note.txt  → 提交 1：添加学习笔记
user.txt  → 提交 2：添加用户示例
order.txt → 提交 3：添加订单示例
你看：是先做了事情，再做个笔记

git log –oneline: 会保存在当前项目隐藏的 .git 数据库中，不会写入 note.txt
会显示类似：
c81a3f2 添加订单示例
9db75a1 添加用户示例
4a0ce82 添加学习笔记
左边是提交 ID，右边是 -m 写的提交说明。最新提交排在最上面。
查看最新一次提交：git log -1 –oneline
查看最新提交的详细信息：git show HEAD
查看所有详细提交记录：git log


练习4 部分暂存
目标：使用交互式暂存选择要提交的修改。
"修改一" >> note.txt
"修改二" >> note.txt
git diff
git add -p -- 文件名  表示“只处理这个文件”
git diff
git diff --staged
交互界面中，y 表示暂存当前块，n 表示跳过，s 表示拆分，q 表示退出，p决定哪些修改需要下一次提交。
如果两处修改距离太近，Git 可能无法拆分，这时可以编辑文件后分次暂存。


练习 5 查看历史和文件来源
目标：检查提交、单个文件历史和每一行的最后修改者。
git log --all --graph --decorate --oneline
git show HEAD
git log --oneline -- note.txt
git blame note.txt
git ls-files
检查：能够指出 HEAD 当前指向哪个提交，并找到 note.txt 的修改历史。
阶段二 分支合并和临时工作
这一阶段训练真实开发中最常见的功能分支流程，并主动制造一次冲突。
练习 6 创建功能分支
目标：在独立分支完成新功能。
git switch -c feature/login
"登录功能" | Set-Content login.txt
git add login.txt
git commit -m "实现登录功能"
git branch -vv
git log --all --graph --decorate --oneline
检查：星号应出现在 feature/login 前，main 不应包含 login.txt 的提交。
练习 7 合并功能分支
目标：把功能分支合并回 main。
git switch main
git merge feature/login
git log --all --graph --decorate --oneline
git branch -d feature/login
检查：main 应包含登录提交。删除已合并分支时不应报未合并警告。
练习 8 制造和解决冲突
目标：理解冲突是 Git 无法替你决定最终内容。
git switch -c feature/message
"功能分支内容" | Set-Content README.md
git add README.md
git commit -m "功能分支修改说明"
git switch main
"主分支内容" | Set-Content README.md
git add README.md
git commit -m "主分支修改说明"
git merge feature/message
git status
检查：README.md 应显示为冲突文件。打开文件，删除冲突标记并保留最终内容，然后执行：
git add README.md
git commit -m "解决说明文件冲突"
如果想放弃本次合并，可以在提交前执行 git merge --abort。
练习 9 使用 Stash 切换任务
目标：在不提交半成品的情况下临时清空工作区。
"尚未完成" | Set-Content unfinished.txt
git stash push -u -m "未完成的任务"
git status
git stash list
git stash show -p stash@{0}
git stash pop
git status
检查：stash 后工作区应干净，pop 后 unfinished.txt 应重新出现。
阶段三 远程协作和版本发布
不用 GitHub 也可以在电脑上模拟远程仓库和第二位开发者，安全练习 fetch、pull 和 push。
练习 10 建立本地模拟远程仓库
目标：理解远程仓库不等于当前工作目录。
cd "$env:USERPROFILE\Documents"
git init --bare test-remote.git
cd test
git remote add origin ..\test-remote.git
git push -u origin main
git remote -v
git branch -vv
检查：origin 应指向 test-remote.git，main 应显示跟踪 origin/main。
练习 11 模拟第二位开发者
目标：在两个工作目录之间练习协作。
cd "$env:USERPROFILE\Documents"
git clone test-remote.git test-developer-b
cd test-developer-b
git config user.name "Developer B"
git config user.email "developer-b@example.com"
"开发者 B 的内容" | Set-Content developer-b.txt
git add developer-b.txt
git commit -m "开发者 B 添加文件"
git push
检查：远程 main 应获得 Developer B 的提交。
练习 12 比较 Fetch 和 Pull
目标：先只获取远程信息，再决定是否合并。
cd "$env:USERPROFILE\Documents\test"
git fetch origin
git log --all --graph --decorate --oneline
git diff main..origin/main
git merge --ff-only origin/main
检查：fetch 后 origin/main 前进而 main 未立即前进。merge 后 main 才包含新提交。git pull 大致等于 fetch 后 merge，练习时拆开执行更容易理解。
练习 13 忽略文件和创建标签
目标：避免提交临时文件，并给稳定版本打标签。
"*.log" | Set-Content .gitignore
"临时日志" | Set-Content debug.log
git status
git check-ignore -v debug.log
git add .gitignore
git commit -m "添加忽略规则"
git tag -a v1.0.0 -m "第一个练习版本"
git tag
git show v1.0.0
检查：debug.log 不应进入提交，标签 v1.0.0 应指向最新提交。
阶段四 历史整理和错误恢复
这一阶段包含会改写历史或移动分支指针的命令。只在 test 练习项目中执行，并在每一步检查提交图。
练习 14 使用 Rebase 更新功能分支
目标：把功能提交重新放到最新 main 后面。
git switch -c feature/rebase
"功能步骤一" | Set-Content rebase.txt
git add rebase.txt
git commit -m "功能步骤一"
git switch main
"主分支更新" | Set-Content main-update.txt
git add main-update.txt
git commit -m "更新主分支"
git switch feature/rebase
git rebase main
git log --all --graph --decorate --oneline
检查：feature/rebase 应形成线性历史，功能提交的 commit ID 会变化。不要随意 rebase 已被其他人使用的公共分支。
练习 15 交互式整理提交
目标：合并零散提交并修改提交说明。
"一步" | Set-Content cleanup.txt
git add cleanup.txt
git commit -m "步骤一"
"二步" | Add-Content cleanup.txt
git add cleanup.txt
git commit -m "步骤二"
"修正" | Add-Content cleanup.txt
git add cleanup.txt
git commit -m "修一下"
git rebase -i HEAD~3
检查：保留第一行 pick，把后两行改为 squash 或 fixup。完成后应只剩一个整理后的提交。
练习 16 复制单个提交
目标：使用 cherry-pick 只复制需要的提交，而不是合并整个分支。
git switch main
git switch -c hotfix
"紧急修复" | Set-Content hotfix.txt
git add hotfix.txt
git commit -m "修复紧急问题"
git log -1 --oneline
git switch feature/rebase
git cherry-pick 提交ID
检查：把命令中的 提交ID 替换成 hotfix 的真实 ID。当前分支应出现内容相同但 ID 不同的新提交。
练习 17 比较撤销工具
目标：根据修改所处位置选择 restore、revert 或 reset。
"临时错误" | Add-Content note.txt
git restore note.txt
"需要提交" | Add-Content note.txt
git add note.txt
git restore --staged note.txt
git add note.txt
git commit -m "用于撤销练习"
git revert HEAD
检查：restore 撤销文件或取消暂存，revert 新建反向提交并保留历史。
练习 18 使用 Reflog 找回提交
目标：模拟误移动分支后恢复提交。
"重要内容" | Set-Content important.txt
git add important.txt
git commit -m "添加重要内容"
git log -1 --oneline
git reset --hard HEAD~1
git reflog
git branch rescue 提交ID
git switch rescue
检查：把 提交ID 替换为 reflog 中添加重要内容对应的 ID。rescue 分支应恢复 important.txt。reset --hard 会丢弃未提交修改，本练习只允许在 test 中执行。
练习 19 使用 Bisect 和 Worktree
目标：定位引入问题的提交，并在第二目录同时处理分支。
git bisect start
git bisect bad
git bisect good 正常提交ID
# 每次检查当前版本后执行 git bisect good 或 git bisect bad
git bisect reset

git worktree add -b hotfix/worktree ..\test-hotfix main
git worktree list
git worktree remove ..\test-hotfix
检查：bisect 应最终指出第一个错误提交。worktree list 应短暂显示两个工作目录。
常用命令速查
•	git status：不知道下一步时先检查状态。
•	git diff：查看未暂存修改。
•	git diff --staged：检查下一次提交的内容。
•	git log --all --graph --decorate --oneline：显示完整提交图。
•	git switch -c 分支名：创建并切换分支。
•	git merge 分支名：把指定分支合入当前分支。
•	git stash -u：连同未跟踪文件一起临时保存。
•	git fetch：获取远程更新但不自动合并。
•	git revert 提交ID：创建安全的反向提交。
•	git reflog：查看 HEAD 移动记录并寻找丢失提交。
•	git cherry-pick 提交ID：复制指定提交到当前分支。
•	git bisect：二分定位引入问题的提交。
撤销选择
•	未暂存修改使用 git restore 文件名，会丢弃该文件的修改。
•	已经暂存使用 git restore --staged 文件名，会保留文件修改。
•	最近提交尚未共享时可以使用 git commit --amend。
•	已共享提交优先使用 git revert 提交ID。
•	分支指针移动错误时先使用 git reflog 找回原提交。
危险命令
git reset --hard
git clean -fd
git push --force
git branch -D 分支名
•	reset --hard 会丢弃未提交修改。
•	clean -fd 会删除未跟踪文件和目录，先用 git clean -nd 预览。
•	push --force 会改写远程历史，团队仓库优先使用 --force-with-lease。
•	branch -D 会强制删除未合并分支，先用 git log 分支名检查提交。
十四天练习计划
•	第 1 天完成练习 1 和 2，能解释工作区和暂存区。
•	第 2 天完成练习 3 和 4，能拆分不同目的的提交。
•	第 3 天完成练习 5，能阅读提交图和文件历史。
•	第 4 天完成练习 6 和 7，能独立完成分支开发和合并。
•	第 5 天完成练习 8，能解决和放弃一次冲突。
•	第 6 天完成练习 9，能临时保存和恢复工作。
•	第 7 天复习阶段一和二，不看手册完成完整流程。
•	第 8 天完成练习 10 和 11，能解释本地和远程仓库。
•	第 9 天完成练习 12 和 13，能使用 fetch、忽略规则和标签。
•	第 10 天完成练习 14，能解释 merge 与 rebase 的区别。
•	第 11 天完成练习 15 和 16，能整理和复制提交。
•	第 12 天完成练习 17 和 18，能安全撤销并找回提交。
•	第 13 天完成练习 19，能使用 bisect 和 worktree。
•	第 14 天完成综合考试。
综合考试
•	[ ] 创建 test-exam 仓库并完成第一次提交。
•	[ ] 创建 feature 分支，完成两个目的不同的小提交。
•	[ ] 在 main 和 feature 修改同一行，解决一次合并冲突。
•	[ ] 建立本地 bare 远程仓库，完成第一次 push。
•	[ ] 模拟第二位开发者提交，并使用 fetch 检查后合并。
•	[ ] 故意移动一次分支指针，再使用 reflog 建立 rescue 分支恢复。
•	[ ] 使用标签标记最终版本，并输出完整提交图。
忘记命令时怎么查
git help -a
git 命令 -h
git help 命令
例如忘记 switch 的选项，可以运行 git switch -h。提问时提供已经执行的命令、完整报错、git status 输出，以及你希望保留还是丢弃修改。
完成标准
当你能够不看答案完成综合考试，并能解释每个命令改变了工作区、暂存区、HEAD、分支还是远程引用，就已经具备日常开发所需的 Git 能力。命令不需要全部背诵，理解状态和恢复路径比记住参数更重要。


![4个区域理解](images/image.png)
![常用命令](images/2.jpg)


