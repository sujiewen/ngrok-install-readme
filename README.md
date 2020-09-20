# ngrok-install-readme

Ngrok搭建内网穿透--Centos 6.x, 7.x 

a,服务器环境
  1,安装gcc,yum install gcc -y 
  2,安装git, yum install git -y
  3,安装go语言环境，请安装go(安装1.12.5版本切记，go1.15我试了存在问题，有知道人可以指点一下)
     11》下载：wget https://storage.googleapis.com/golang/go1.12.5.linux-amd64.tar.gz (需要翻墙)
     22》解压：tar -xzf go1.12.5.linux-amd64.tar.gz
     33》配置：
            
          vim ~/.bashrc

            export GOPATH=/(你的目录)/Go
            export GOROOT=/(你的目录)/go
            export PATH=$PATH:$GOROOT/bin
            source ~/.bashrc
          
 b,安装Ngrok
  1，下载Ngrok源码
      cd /usr/local/
      git clone https://github.com/inconshreveable/ngrok.git
      
  2,  生成证书
      cd ngrok    

        export NGROK_DOMAIN="ngrok.xx.com"    //记得域名换成自己的

        openssl genrsa -out root.key 2048

        openssl req -x509 -new -nodes -key root.key -subj "/CN=$NGROK_DOMAIN" -days 365 -out root.pem

        openssl genrsa -out server.key 2048

        openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr

        openssl x509 -req -in server.csr -CA root.pem -CAkey root.key -CAcreateserial -out server.crt -days 365
 3, 替换证书
 
      //一行一行执行，然后会提示是否覆盖，输入 “y” 回车就可以了
      cp root.pem assets/client/tls/ngrokroot.crt

      cp server.crt assets/server/tls/snakeoil.crt

      cp server.key assets/server/tls/snakeoil.key
      
4, 生成服务端

   GOOS=linux GOARCH=amd64 make release-server
5, 生成客户端
    GOOS=windows GOARCH=amd64 make release-client    //windows 64位
    GOOS=windows GOARCH=amd64 make release-client    //windows 32位

    GOOS=darwin GOARCH=386 make release-client        //Mac OS 32位
    GOOS=darwin GOARCH=amd64 make release-client      //Mac OS 64位

    GOOS=linux GOARCH=amd64 make release-client       //Linux  64位

    GOOS=linux GOARCH=arm  make release-client        //ARM 平台
    
c, 启动服务端
  cd /usr/local/ngrok
  ./bin/ngrokd -domain="ngrok.xx.com"  -httpAddr=":80" -httpsAddr=":443" -tunnelAddr=":4443"
d, 启动客户端
  1， ngrok.cfg内容：
     server_addr: "ngrok.xx.com:4443"
     trust_host_root_certs: false
  2， start.bat
     ngrok -config=ngrok.cfg -subdomain test 80
    // test就是你想要访问域名的前缀
    // 80表示本地需要穿透的端口
e,   6.x,7.x添加服务，有点差别，

  1, 6.x
      #!/bin/sh  
      #chkconfig:2345 70 30  
      #description:ngrok  

      ngrok_path=/usr/local/ngrok
      case "$1" in
          start)
              cd /usr/local/ngrok
              echo "start ngrok service.."  
              sh ${ngrok_path}/bin/ngrokd -domain="ngrok.xx.com"  -httpAddr=":80" -httpsAddr=":443" -tunnelAddr=":4443"
              ;;
          *)
          exit 1
          ;;
          
   2, 7.x
      [Unit]
      Description=Share local port(s) with ngrok
      After=syslog.target network.target

      [Service]
      PrivateTmp=true
      Type=simple
      Restart=always
      RestartSec=1min
      StandardOutput=null
      StandardError=null
      ExecStart=/usr/local/ngrok/bin/ngrokd -domain=ngrok.intolearn.com -httpAddr=:80 -httpsAddr=:443 -tunnelAddr=:4443 %i
      ExecStop=/usr/bin/killall ngrok

      [Install]
      WantedBy=multi-user.target


   
    

