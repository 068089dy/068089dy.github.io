## 安装
npm install
npm install -g hexo

注意：这里在theme/cactus/下留了一个_config_bak.yml备份文件，每次重新down项目下来时，需要复制一下这个文件，名称改为_config.yml，才可以启动预览。theme/cactus/_config.yml文件中有gitTalk的账户信息，不同步到git仓库，每次迁移需要自己新建填写。
```
gitalk:
    enabled: true # 开关开启
    owner: 068089dy # 你的 github 用户名
    repo: 068089dy.github.io # 保存评论的 repo 库
    admin: ['068089dy'] # 管理员，你的 github 用户名
    clientID: 到这里查看github注册的app：https://github.com/settings/developers
    clientSecret: 
```

## 预览
hexo server

## 生成
hexo g

## 发布
hexo clean && hexo g && hexo d

`hexo d`失败的话，可以尝试`npm update`一下。