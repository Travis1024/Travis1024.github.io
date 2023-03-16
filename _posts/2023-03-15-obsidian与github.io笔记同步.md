---
title: obsidian与github.io笔记同步
author: Travis <Hongxu Wei>
date: 2023-03-15 16:42:19 +0800
categories: [DailyNotes Space]
tags: [obsidian, github]
math: false
---
obsidian笔记仓库的结构如下，其中TravisNotes为obsidian仓库：

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303151632908.png" alt="image-20230315163216875" style="zoom:80%;" />

github.io笔记存放在`_posts`文件夹中

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303151633496.png" alt="image-20230315163347468" style="zoom:40%;" />

通过shell脚本进行obsidian仓库笔记与github.io中笔记的同步，并且推送到github中

```shell
#!/bin/bash

# Gets the file suffix
function FileSuffix() {
    local filename="$1"
    if [ -n "$filename" ]; then
        echo "${filename##*.}"
    fi
}
suffix=".md"
# tatget
TargetPath="/Users/travis/GithubProjects/Travis1024.github.io/_posts"
# source
FilePath="/Users/travis/Documents/TravisNotesManagement/TravisNotes"

function getAllFiles() {
	for file in `ls $1`
	do
		file_full_path=$1"/"$file
		echo ""
		echo "-----------------------file_full_path: $file_full_path-----------------------"

		# 如果为文件
		if [ -f $file_full_path ]; then
			echo "===== $file_full_path is a FILE ====="
			# 文件名称
			filename=$file
			if [[ "$filename" =~ 20[0-9][0-9]\-.*\.md ]]; then
				file_true_path=$file_full_path
				target_true_path=$TargetPath"/"$filename
				echo "| --->target_true_path: $target_true_path <--- |"
				cp -f $file_true_path $target_true_path
				if [ $? -eq 0 ]; then
					echo "| ---> Copy Successfully! <--- |"
				fi
			fi
		# 如果为文件夹
		elif [ -d $file_full_path ]; then
			# 文件夹名称
			filename=$file
			if [[ "$filename" == ".git" ]] || [[ "$filename" == ".obsidian" ]]; then
				echo "continue"
				continue
			fi

			cd $file_full_path
			echo "===== $file_full_path is a DIRECTORY ====="
			getAllFiles $file_full_path
			cd ..
		# 非文件 and 非文件夹
		else
			echo "!!! $file_full_path is a invalid path !!!"
		fi
	done
}

getAllFiles $FilePath

sleep 0.5s

cd "/Users/travis/GithubProjects/Travis1024.github.io"

echo ""
echo ""
echo "-------------------------Start Git Push Files-------------------------"

echo "-----> (1) start adding files"
git add .
sleep 0.5s

echo "-----> (2) get git status"
git status
sleep 0.5s

echo "-----> (3) start commit"
current_time=$(date "+%Y.%m.%d %H:%M:%S")
git commit -m "update on $current_time"
sleep 0.5s

echo "-----> (4) start push files"
git push

if [ $? -eq 0 ]; then
	echo "-------------------------Git Successfully!-------------------------"
else
	echo "-----------------------------Git Error!-------------------------"
fi
```

