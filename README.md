# Bitcoin Dev Quickstart 

This step-by-step tutorial provides a hands-on walkthrough for quickly getting up and running with Bitcoin basics, from a developer's perspective. From building and installation of the headless Bitcoin daemon (bitcoind), interacting with it using the command line, and building a demo C++ application that can send RPC commands and process responses.

The tutorial is written for Ubuntu 18.04. Adjust accordingly for your own environment.

## Preliminary Steps

1. Install Prerequisites

Install the following dependencies.


```
sudo apt install openssl autoconf automake libboost-dev libboost-all-dev libevent-dev

sudo add-apt-repository ppa:bitcoin/bitcoin
sudo apt-get update
sudo apt-get install libdb4.8-dev libdb4.8++-dev
```

2. Clone the bitcoin repository and change into its directory

```
$ git clone https://github.com/bitcoin/bitcoin.git
$ cd bitcoin
```

3. Configure the build


```
$ ./autogen.sh && ./configure
```

4. Building and installing the project


```
$ make -j4 && sudo make install
```

5. Check the installation


```
$ which bitcoind
```

Example response:


```
/usr/local/bin/bitcoind
```


```
$ which bitcoin-cli
```

Example response:


```
/usr/local/bin/bitcoin-cli
```

## Configure the Daemon

1. Download a sample configuration file


```
$ wget https://raw.githubusercontent.com/bitcoin/bitcoin/master/share/examples/bitcoin.conf -O ~/.bitcoin/bitcoin.conf
```

2. Generate a user and password


```
$ ./share/rpcauth/rpcauth.py user
```

Example response:


```
String to be appended to bitcoin.conf:
rpcauth=user:bf2a249c8580766a1e4fcd9f777533b0$700645074a2f4a4ffd35cf1ec7ada6d8f148053ca4be729c3f54e63ce4664a2e
Your password:
l-sHsqLHrstIh1ZuOVIDjaLcRuYn-Xoy8prVgjboRnc=
```

NOTE: Record your password for later use.

3. Open ~/.bitcoin/bitcoin.conf, and search for the following (or similar) line:


```
# You can even add multiple entries of these to the server conf file, and client can use any of them:
# rpcauth=bob:b2dd077cb54591a2f3139e69a897ac$4e71f08d48b4347cf8eff3815c0e25ae2e9a4340474079f55705f40574f4ec99
```


4. Below that line, add the rpcauth information generated by rpcauth.py.

Example:
```
rpcauth=user:bf2a249c8580766a1e4fcd9f777533b0$700645074a2f4a4ffd35cf1ec7ada6d8f148053ca4be729c3f54e63ce4664a2e

```

## Running and Testing bitcoind

1. Start the daemon


```
$ bitcoind -daemon
```

Example response:


```
Bitcoin Core starting
```

2. Show help

```
$ bitcoind -?
```

3. Test CLI commands


```
$ bitcoin-cli -getinfo
```

Example response:


```
{
  "version": 209900,
  "blocks": 156419,
  "headers": 641060,
  "verificationprogress": 0.003607647832804664,
  "timeoffset": -1,
  "connections": 10,
  "proxy": "",
  "difficulty": 1090715.680051267,
  "chain": "main",
  "keypoolsize": 1000,
  "paytxfee": 0.00000000,
  "balance": 0.00000000,
  "relayfee": 0.00001000,
  "warnings": "This is a pre-release test build - use at your own risk - do not use for mining or merchant applications"
}
```

4. Stopping the daemon


```
bitcoin-cli stop
```

NOTE: If you may see the following error message if running commands too quickly after starting bitcoind:


```
error code: -28
error message:
Loading block index..
```

If you see this kind of error, wait another second or two and try again.

## The RPC Interface

1. Start the daemon (if stopped).


```
$ bitcoind -daemon
```

2. Use the following curl command to test communication with the the RPC interface.


```
$ curl --user user --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getblockchaininfo", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:8332
```

Example response:


```
Enter host password for user 'user':
```

3. Enter or paste the password generated earlier (e.g. l-sHsqLHrstIh1ZuOVIDjaLcRuYn-Xoy8prVgjboRnc=)

Example response:


```json
{"result":{"chain":"main","blocks":229810,"headers":641062,"bestblockhash":"000000000000019606fe717e1ec8832aadd0b531a9fdb3fc9b66d4396e4c0e6c","difficulty":6695826.282596251,"mediantime":1365171545,"verificationprogress":0.02829406551238997,"initialblockdownload":true,"chainwork":"0000000000000000000000000000000000000000000000340a2573e03179ff18","size_on_disk":7956125601,"pruned":false,"softforks":{"bip34":{"type":"buried","active":true,"height":227931},"bip66":{"type":"buried","active":false,"height":363725},"bip65":{"type":"buried","active":false,"height":388381},"csv":{"type":"buried","active":false,"height":419328},"segwit":{"type":"buried","active":false,"height":481824}},"warnings":"This is a pre-release test build - use at your own risk - do not use for mining or merchant applications"},"error":null,"id":"curltest"}
```

## Demo Example C++

1. Install the following:


```
$ sudo apt install libcurlpp-dev libcurl4-openssl-dev
```

2. Create a file called main.cpp:


```cpp
#include <curlpp/cURLpp.hpp>
#include <curlpp/Easy.hpp>
#include <curlpp/Options.hpp>

#include <string>
#include <sstream>
#include <iostream>

using namespace curlpp::options;

int main(int argc, char *argv[])
{

    if(argc < 3) {
    std::cerr << argv[0] << ": Wrong number of arguments" << std::endl 
	      << "Usage: " << argv[0] << " username password"
	      << std::endl;
    return EXIT_FAILURE;
    }

    std::string username(argv[1]);
    std::string password(argv[2]);

    std::string userpass = username + ":" + password;

    try
	{

		curlpp::Cleanup btcCleanup;

		curlpp::Easy btcRequest;

        std::list<std::string> header; 
        header.push_back("Content-Type: application/json"); 
    
        btcRequest.setOpt(new curlpp::options::HttpHeader(header)); 

        // Set the daemon's URL.
        curlpp::options::Url btcUrl(std::string("http://127.0.0.1:8332"));

        // Set the username and password.
        curlpp::options::UserPwd usrPsswd(userpass);

        // Create the data payload
        std::string json = R"({"jsonrpc": "1.0", "id":"curltest", "method": "getblockchaininfo", "params": [] })";

        // Add options to the request
        btcRequest.setOpt(btcUrl);
        btcRequest.setOpt(usrPsswd);
        btcRequest.setOpt(curlpp::options::PostFields(json));
	    btcRequest.setOpt(curlpp::options::PostFieldSize(json.size()));

		// Send the request 
		btcRequest.perform();
	}

	catch(curlpp::RuntimeError & e)
	{
		std::cout << e.what() << std::endl;
	}

	catch(curlpp::LogicError & e)
	{
        // Result is sent to standard output.
		std::cout << e.what() << std::endl;
	}
    
  return 0;

}
```

3. Build the demo application:
 
 
```  
$ g++ -Wall main.cpp -o btctest -Llib/x86_64-linux-gnu -lcurlpp -Wl,-Bsymbolic-functions -Wl,-z,relro -lcurl -Iinclude -I/usr/include/x86_64-linux-gnu
```

4. Run the demo with your username and password as command line arguements. Example:


```
./btctest user l-sHsqLHrstIh1ZuOVIDjaLcRuYn-Xoy8prVgjboRnc=
```

Example response:


```json
{"result":{"chain":"main","blocks":303520,"headers":641075,"bestblockhash":"000000000000000024450e52f5d08622114b503713243aab15f3f7f989e76267","difficulty":10455720138.48484,"mediantime":1401561878,"verificationprogress":0.07238580672791109,"initialblockdownload":true,"chainwork":"000000000000000000000000000000000000000000007829790f458ed3674540","size_on_disk":22091527638,"pruned":false,"softforks":{"bip34":{"type":"buried","active":true,"height":227931},"bip66":{"type":"buried","active":false,"height":363725},"bip65":{"type":"buried","active":false,"height":388381},"csv":{"type":"buried","active":false,"height":419328},"segwit":{"type":"buried","active":false,"height":481824}},"warnings":"This is a pre-release test build - use at your own risk - do not use for mining or merchant applications"},"error":null,"id":"curltest"}
```

## Congratulations!

You have successfully installed and run the Bitcoin bitcoind daemon, and created a command line application to interact with it to fetch information.

Next steps:

Use `bitcoin-cli -rpcwait help` to find other RPC calls to add to the application.

