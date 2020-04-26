
# 安装区块浏览器

```bash
cd /opt
wget http://cdn.npm.taobao.org/dist/node/v10.0.0/node-v10.0.0-linux-x64.tar.gz
tar xzvf node-v10.0.0-linux-x64.tar.gz
echo 'export PATH="/opt/node-v10.0.0-linux-x64/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
npm -g install yarn cnpm
cnpm install compression-webpack-plugin --save-dev

echo 'registry=https://registry.npm.taobao.org' >> ~/.npmrc
echo 'sass_binary_site=https://npm.taobao.org/mirrors/node-sass/' >> ~/.npmrc
echo 'phantomjs_cdnurl=http://npm.taobao.org/mirrors/phantomjs' >> ~/.npmrc
echo 'ELECTRON_MIRROR=http://npm.taobao.org/mirrors/electron/' >> ~/.npmrc


apt -y install git
cd ~/ && git clone https://github.com/my-graphene/ui
cd ui

# sed -i  "s/RUI/RUI/g" `grep -rl RUI . | grep -v node_modules | grep -v git`

yarn
npm install



cd node_modules/bitsharesjs-ws/
sed -i  "s/BTS/RUI/g" `grep -rl BTS . | grep -v git`
sed -i  "s/GPH/RUI/g" `grep -rl GPH . | grep -v git`
cd ../../


vim app/api/apiConfig.js

npm start

cd ~/ui
nohup npm start &

tail -f nohup.out


出现错误时，需要重新安装，删除已安装的模块和lock.json，清除npm内部缓存

rm -rf node_modules/
npm cache clean
npm install

```
