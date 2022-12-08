# Wormholes güncellemesi 08.12.2022 (0.10.1)


## Kurulum için servis dosyası oluşturuyoruz aşağıdaki komudu yazın 
```
nano ruesandora.sh 
```
## Açılan ekranda CTRL+K ile tektek satırları silin aşağıdaki kodları tek seferde copy past yapın CTRL+X ve Y ile kaydedip çıkın


```
#!/bin/bash

sleep 2

echo "Update package ......."
sleep 2
sudo apt update && apt upgrade -y

echo "Install docker ......."
sleep 2
sudo apt update && sudo apt upgrade -y

sleep 2
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
apt-cache policy docker-ce
sudo apt install docker-ce -y

echo "Install program ....."
sleep 2
cd $HOME
#check docker cmd
which docker >/dev/null 2>&1
if  [ $? -ne 0 ] ; then
        echo "docker not found, please install first!"
        echo "ubuntu:sudo apt install docker.io -y"
        echo "centos:yum install  -y docker-ce "
        echo "fedora:sudo dnf  install -y docker-ce"
        exit
fi
#check docker service
docker ps > /dev/null 2>&1
if [ $? -ne 0 ] ; then

        echo "docker service is not running! you can use command start it:"
        echo "sudo service docker start"
        exit
fi

docker stop wormholes > /dev/null 2>&1
docker rm wormholes > /dev/null 2>&1
docker rmi wormholestech/wormholes:v1 > /dev/null 2>&1
if [ -d /wm/.wormholes/wormholes ]; then
    read -p "If you want to clear the historical data of the wormholes blockchain, please press y to clear it, if not, please press Enter directly:" xyz
    if [ "$xyz" = 'y' ];then
        cp /wm/.wormholes/wormholes/nodekey /wm/nodekey
        rm -rf /wm/.wormholes
        mkdir -p /wm/.wormholes/wormholes
        mv /wm/nodekey /wm/.wormholes/wormholes/nodekey
    else
        echo "not empty"
    fi
else
    read -p "Please enter your private key:" ky
fi

if [ -n "$ky" ]; then
    if [ ${#ky} -eq 64 ];then
        mkdir -p /wm/.wormholes/wormholes
        echo $ky > /wm/.wormholes/wormholes/nodekey
    elif [ ${#ky} -eq 66 ] && ([ ${ky:0:2} == "0x" ] || [ ${ky:0:2} == "0X" ]);then
        mkdir -p /wm/.wormholes/wormholes
        echo ${ky:2:64} > /wm/.wormholes/wormholes/nodekey
    else
        echo "the nodekey format is not correct"
        exit -1
    fi
fi
docker run -id -e KEY=$ky  -p 30303:30303 -p 8545:8545 -v /wm/.wormholes:/wm/.wormholes --name wormholes wormholestech/wormholes:v1


echo "Your private key is:"
sleep 6
docker exec -it wormholes /usr/bin/cat /wm/.wormholes/wormholes/nodekey
#cat /wm/.wormholes/geth/nodekey
```


## Şimdi Kurulum için servis dosyamızı çalıştırıyoruz /// V0.10.1
```
bash ./ruesandora.sh 
```

## Versiyon kontrol ediyoruz güncel versiyon V0.10.1
```
curl -X POST -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_version","id":64}' http://127.0.0.1:8545
```
## Çıktı
```
V0.10.1
```



## Node bloklarını izlemek servis dosyası oluşturuyoruz aşağıdaki komudu yazın ( Daha önceki bunu kurduysanız bu adımıgeçin)
```
nano monitor.sh 
```
## açılan ekranda CTRL+K ile tektek satırları silin aşağıdaki kodları tek seferde copy past yapın CTRL+X ve Y ile kaydedip çıkın
```
#!/bin/bash
function info(){
     cn=0
     while true
     do
             echo "$cn second."
             echo "node $1"
             rs=`curl -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","id":64}' https://api.wormholestest.com 2>/dev/null`
             blockNumbers=$(parse_json $rs "result")
             echo "Block height of the whole network: $((16#${blockNumbers:2}))"
             rs1=`curl -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"net_peerCount","id":64}' 127.0.0.1:$1 2>/dev/null`
             count=$(parse_json $rs1 "result")
             echo "Number of node connections: $((16#${count:2}))"
             rs2=`curl -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","id":64}' 127.0.0.1:$1 2>/dev/null`
             blckNumber=$(parse_json $rs2 "result")
             echo "Block height of the current peer: $((16#${blckNumber:2}))"
             sleep 5
             clear
             let cn+=5
     done
}

function parse_json(){
      if [[ $# -gt 1 ]] && [[ $1 =~ $2 ]];then
         echo "${1//\"/}"|sed "s/.*$2:\([^,}]*\).*/\1/"
      else
         echo "0x0"
     fi
}

function main(){
     if [[ $# -eq 0 ]];then
             info 8545
     else
             info $1
     fi
}

main "$@"
```

## Güncel blokları görmek için servis dosyası çalıştırıyoruz aşağıdaki komudu yazın 
```
bash ./monitor.sh
```
