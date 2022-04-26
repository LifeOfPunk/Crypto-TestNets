
cd $HOME

wget -O go1.17.1.linux-amd64.tar.gz https://golang.org/dl/go1.17.linux-amd64.tar.gz

rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.17.1.linux-amd64.tar.gz && rm go1.17.1.linux-amd64.tar.gz

echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile

echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile

echo 'export GO111MODULE=on' >> $HOME/.bash_profile

echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
