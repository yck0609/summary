# 常见问题

报错信息 ：fatal: unable to access 'https://github.com/yck0609/summary.git/': schannel: failed to receive handshake, SSL/TLS connection failed



在项目文件夹的命令行窗口执行下面代码，取消git本身的https代理，使用自己本机的代理，


git config --global --unset http.proxy //取消http代理
git config --global --unset https.proxy //取消https代理
