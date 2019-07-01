1. 从远程仓库clone至本地新文件夹
   1. 创建本地新文件夹，并在文件夹中初始化  

        `$ git init`
   2. 启动sparse-checkout

        `$ git config core.sparseCheckout true`  
   3. 设置sparse-checkout文件夹，注意文件或文件夹添加"/"前缀

        ```shell
        $ echo '/folder/*' >> .git/info/sparse-checkout
        $ echo '/file' >> .git/info/sparse-checkout
        ...
        ```
   4. 设置远程仓库  

        `$ git remote add origin git://github.com/cdnjs/cdnjs.git`  
   5. 开始clone  

        `$ git pull origin master`  
    
2. 忽略文件  
   使用 `.gitignore` 文件。注意: 只能忽略从未添加到git中的文件，如果已经提交过，则不能忽略。

3. 删除文件关联  
   
    `$ git rm --cached filename`  
    使本地文件与远程仓库断开联系，可以使已提交的文件使用 `.gitignore`  

    