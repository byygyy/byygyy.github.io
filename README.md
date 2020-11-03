#### 项目维护说明
该项目2020年2月29日后停止在gitlab维护，开始在github维护，博客维护流程如下：

1.在source/_posts/新建一个md文件
2.登陆服务器并cd到项目根目录opstime.github.io
3.执行update.sh
4.不要修改根目录下_config.yml文件的git配置，否则会影响项目的运行。

#### 项目迁移流程
该项目最初是运行在gitlab上的，后来迁移到了github，只是没有将hexo deploy 跟github pages关联。利用hexo deploy生成的public目录，将该目录发布到nginx后，用户就可以通过ngxin访问该博客， 速度超快。博客迁移流程如下：

1.在一个新服务器上完整安装hexo
2.git clone 该项目到服务器本地
3.在项目根目录只执行一次
  - npm install hexo-cli -g
  - npm install
  - hexo g
4.安装nginx并将web目录指向当前项目的public目录
5.迁移后常态维护流程请看“博客维护说明”
