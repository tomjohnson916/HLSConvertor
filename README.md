Install Vagrant and load Ubuntu virutal OS
============

See:
[vagrant](https://www.vagrantup.com/downloads.html)
[boxes trusty64](https://atlas.hashicorp.com/ubuntu/boxes/trusty64) 
```
# virtual port to Ubuntu, using virtual box.
vagrant init ubuntu/trusty64; vagrant up --provider virtualbox

# make sure the virtual machine's memory is above 1GB
# make sure the virtual machine using bridge mode by commenting out the line in ~/Vagrantfile:
#   config.vm.network "public_network"
vagrant ssh
```

Install kurento-media-server, node.js
============
```
# Install kurento-media-server on the Ubuntu virtual OS: (user:vagrant paw:vagrant)
echo "deb http://ubuntu.kurento.org trusty-dev kms6" | sudo tee /etc/apt/sources.list.d/kurento-dev.list
wget -O - http://ubuntu.kurento.org/kurento.gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install -y kurento-media-server-6.0-dev
sudo apt-get install -y build-essential libtool autotools-dev  automake indent astyle git
sudo apt-get install -y libboost-all-dev libjson-glib-dev bison flex uuid-dev libsoup2.4-dev 

sudo service kurento-media-server-6.0 start

curl -sL https://deb.nodesource.com/setup_4.x | sudo bash -
sudo apt-get install -y nodejs
sudo npm install npm -g

npm install
npm start
```


Building and installing HLS Convertor
=====================================
```
cd HLSConvertor/h-l-s-convertor
mkdir .build
cd .build
cmake .. -DGENERATE_JS_CLIENT_PROJECT=TRUE
make

# copy the generated so file to system's lib folder.
sudo cp ./src/server/*.so /usr/lib/x86_64-linux-gnu/gstreamer-1.5/
sudo cp ./src/gst-plugins/*.so /usr/lib/x86_64-linux-gnu/kurento/modules/

# copy the generated js code to current project.
cp -r ./js ../../kurento-one2many-call/node_modules/kurento-client/node_modules/kurento-module-hlsconvertor/
vi ../../kurento-one2many-call/node_modules/kurento-client/lib/index.js
  # (Add one line)
  register('kurento-module-hlsconvertor')
```


cd node_modules/kurento-client/
vi lib/index.js
  (Add one line)
  register('kurento-module-hlsconvertor')

**********************************
* building kurento media server
**********************************
See: https://www.kurento.org/docs/6.0.0/mastering/develop_kurento_modules.html
On Ubuntu virtual box:
sudo apt-get update
sudo apt-get install git
git config --global user.name "rentao"
git config --global user.email "tao@swarmnyc.com"

sudo apt-get install kurento-media-server-6.0-dev
kurento-module-scaffold.sh HLSConvertor ./HLSConvertor/


cd HLSConvertor/h-l-s-convertor/src
cmake ..
make

cmake .. -DGENERATE_JS_CLIENT_PROJECT=TRUE
(to generate the js code used by npm[node.js])

vi /etc/default/kurento-media-server-6.0
  (Add below 2 lines)
  export KURENTO_MODULES_PATH=/home/vagrant/swarmnyc/HLSConvertor/h-l-s-convertor/src/src
  export GST_PLUGIN_PATH=/home/vagrant/swarmnyc/HLSConvertor/h-l-s-convertor/src/src
sudo service kurento-media-server-6.0 restart



*****************************************
* test kurento media server by node.js
*****************************************
Download the tutorial project of kurento from:
  https://github.com/Kurento/kurento-tutorial-node

Suppose we are using the kurento-hello-world to test:
cd kurento-tutorial-node/kurento-hello-world/
cd node_modules/kurento-client/
vi lib/index.js
  (Add one line)
  register('kurento-module-hlsconvertor')
# copy the generated js code to current project.
cp -r /home/vagrant/swarmnyc/HLSConvertor/h-l-s-convertor/src/js/ ./node-modules/kurento-module-hlsconvertor/
vi package.json
  (Add one line in the "dependencies" block)
  "kurento-module-hlsconvertor": "0.0.1-dev",
# run the backend server.
cd ../..
vi server.js
  (Change the line, the actual ip address of the running kurento media server)
        ws_uri: 'ws://192.168.17.197:8888/kurento'

  (Add below code where webRtcEndpoint is created)
                pipeline.create('HLSConvertor', function(error, hlsconvertor) {
                    console.log("-------------------------creating HLSConvertor-------------------------\n"+webRtcEndpoint);
                    webRtcEndpoint.connect(hlsconvertor);
                    console.log("-------------------------creating HLSConvertor finished!-------------------------");
                });
node server.js



******************************************
* Install nvm and nodejs if necessary
******************************************
Install nvm to upgrade nodejs from: https://github.com/creationix/nvm
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.26.1/install.sh | bash
. ~/.nvm/nvm.sh
nvm install 0.12.7

