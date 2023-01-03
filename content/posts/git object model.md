---
title: "Git Object Model"
date: 2023-01-03T16:27:35+08:00
tag: "git"
---

### 模型介绍

下面将介绍一下`git`命令内部所使用的对象模型



首先进行一次简单的`git`操作: 创建一个文件, 将其`add`到暂存区之后, 再`commit`到本地仓库

```sh
$ cd git_temp
$ git init
$ touch index.html
$ vim index.html # 任意写入一些内容
$ git add index.html
$ git commit -m "initial commit"
```



经过上面的操作之后, 在`.git/objects`目录之中将会出现一些新的文件

```sh 
$ tree .git/objects
.git/objects
├── 04
│   └── ec0f5b9e7b9ab1cd2f187ccb3d213ebe4650fc
├── 5d
│   └── 3a7429769482ee84223e9e925d9e6efdcac231
├── 79
│   └── 77334c269a2c80ae49d40dac95fd8b63a29b99
├── info
└── pack
```

`info`与`pack`目录暂且不管, 重点在于前面3个文件

这些文件均位于一个以2个字符开头的目录下, 长度为38个字符, 实际上, 这些目录名与文件名是`git`使用SHA-1加密之后得到

总字符长度为40, 取前面2个字符作为目录名, 后面38个字符位文件名

这三个文件其实对应着三种对象模型: **blob object**, **tree object**, **commit object**



**blob object**

一个`blob object`对应着工作目录的一个文件, 其看起来像这样

![image-20230103151718620](/image-20230103151718620.png)

其由哈希值与文件内容构成



**tree object**

一个`tree object`包含着当前项目的文件列表, 其看起来像这样

![image-20230103151938429](/image-20230103151938429.png)

每个`tree object`除了包含工作目录的文件名之外, 还拥有指针字段指向列表中文件对应的`blob object`



**commit object**

当使用`git commit`命令之后, 就会创建一个`commit object`, 其指向`tree object`

![image-20230103152308487](/image-20230103152308487.png)



#### 实战

`.git/objects`目录下的文件都是二进制文件, 不能够使用`cat`查看, 需要使用特定的`git`命令来查看文件内容

- 使用`git log`查看`commit object`的哈希值

  ```sh
  $ git log
  commit 04ec0f5b9e7b9ab1cd2f187ccb3d213ebe4650fc (HEAD -> master)
  Author: liver0377 <liverspecial@gmail.com>
  Date:   Tue Jan 3 14:47:19 2023 +0800
  
      initial commit
  ```

  `commit`后面跟着的就是这次`commit object`的哈希值

- 使用`git cat-file commit`查看`commit object`的内容

  ```sh
  $  git cat-file commit 04ec0f5b9e7b9ab1cd2f187ccb3d213ebe4650fc
  tree 7977334c269a2c80ae49d40dac95fd8b63a29b99
  author liver0377 <liverspecial@gmail.com> 1672728439 +0800
  committer liver0377 <liverspecial@gmail.com> 1672728439 +0800
  
  initial commit
  ```

  可以看出, 其中包含了`tree object`的哈希值, 初次之外, 还包含作者, 提交者等信息

- 使用`git ls-tree`查看`tree object`的内容

  ```sh
  $ git ls-tree 7977334c269a2c80ae49d40dac95fd8b63a29b99
  100644 blob 5d3a7429769482ee84223e9e925d9e6efdcac231    index.html
  ```

  该命令输出了该`tree`对象所指向的文件名以及其对象的`blob object`的哈希值

- 使用`git cat-file blob`查看`blob object`的内容

  ```sh
  git cat-file blob 5d3a7429769482ee84223e9e925d9e6efdcac231
  <head>
          <body>
                  hello world
          </body>
  </head>
  ```



### 对象复用

当做出新的`commit`时, 经常会有旧文件没有发生改动, 此时`git`对于这些文件并不会创建新的`blob`对象, 而是会利用指针进行对象复用



**当前的对象模型**

![image-20230103154626249](/image-20230103154626249.png)

此时我们不去修改`index.html`,创建一个新文件`README.md`, 那么在`git commit`之后, `git`总共只会创建三个新的对象

1. `README.md`对应的`blob object`
2. 新的`tree object`
3. 新的`commit object`

旧的`index.html`对应的`blob`因为没有修改, 所有没有为其创造新的`blob object`



**commit 之后的模型**

![image-20230103155149764](/image-20230103155149764.png)

可以注意到旧的未修改的`blob object`对象被复用, 并且新的`commit object`有一个指向上次`commit object`对象的指针



> 实际上, 为了尽可能做到对象复用, tree object也可以指向另一个tree object



#### 实战

- 创建`README.md`, 并执行`add`和`commit`指令

  ```sh
  $ touch README.md
  $ vim README.md
  $ git add README.md
  $ git commit -m "second commit"
  $ tree .git/objects
  .git/objects/
  ├── 04
  │   └── ec0f5b9e7b9ab1cd2f187ccb3d213ebe4650fc
  ├── 5d
  │   └── 3a7429769482ee84223e9e925d9e6efdcac231
  ├── 69
  │   └── 02319ed0faea5dd3f4bc41e7f43d8fd9283977
  ├── 79
  │   └── 77334c269a2c80ae49d40dac95fd8b63a29b99
  ├── c3
  │   └── a8213d075e316ab1a8d5a1537d50f35c3a63e0
  ├── c6
  │   └── 60914581071e0dd19f2f27766e8da8f431d113
  ├── info
  └── pack
  ```

  可以看出多出来3个新对象

- 使用`git log`查看`commit object`的哈希值

  ```sh
  $ git log
  commit c3a8213d075e316ab1a8d5a1537d50f35c3a63e0 (HEAD -> master)
  Author: liver0377 <liverspecial@gmail.com>
  Date:   Tue Jan 3 15:39:50 2023 +0800
  
      second commit
  
  commit 04ec0f5b9e7b9ab1cd2f187ccb3d213ebe4650fc
  Author: liver0377 <liverspecial@gmail.com>
  Date:   Tue Jan 3 14:47:19 2023 +0800
      
      initial commit
  ```

- 使用`git cat-file commit`查看`commit object`对象的内容

  ```sh
  $ git cat-file commit c3a8213d075e316ab1a8d5a1537d50f35c3a63e0
  tree 6902319ed0faea5dd3f4bc41e7f43d8fd9283977
  parent 04ec0f5b9e7b9ab1cd2f187ccb3d213ebe4650fc
  author liver0377 <liverspecial@gmail.com> 1672731590 +0800
  committer liver0377 <liverspecial@gmail.com> 1672731590 +0800
  
  second commit
  ```

  可以看出其指向了一个新的`tree object`, 并且指向了上一次的`commit object`(parent)

- 使用`git ls-tree`查看`tree object`对象内容

  ```sh
  $ git ls-tree 6902319ed0faea5dd3f4bc41e7f43d8fd9283977
  100644 blob c660914581071e0dd19f2f27766e8da8f431d113    README.md
  100644 blob 5d3a7429769482ee84223e9e925d9e6efdcac231    index.html
  ```

  通过比对哈希值可以发现, 这里的`index.html`的哈希值就是上一次的`blob object`的哈希值





### 文件删除

当删除文件时, `git`对象模型所做的操作非常简单, 只要不让新的`commit object`指向删除文件对应的`blob object`即可



假设在新的`commit`中, 我们删除了文件`index.html`, 那么对象模型组织如下

![image-20230103160819037](/image-20230103160819037.png)





