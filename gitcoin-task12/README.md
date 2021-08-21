                     *               Document Porting An Existing Ethereum DApp To Polyjuice*

##### 1. Setup the Godwoken Testnet Network in MetaMask


From the network selection dropdown, select "Custom RPC".

![image](https://user-images.githubusercontent.com/48971958/130314970-b14d101d-3f51-4b39-a192-66c9d8288826.png)


MetaMask Network Menu
From there you will be presented with a form to specify the network settings.

MetaMask Add Network
Enter the following details.

![image](https://user-images.githubusercontent.com/48971958/130314997-d369ad9e-22a3-4d25-ba25-0538f673fa96.png)


```
Network Name: Godwoken Testnet
RPC URL: https://godwoken-testnet-web3-rpc.ckbapp.dev
Chain ID: 71393
Currency Symbol: <Leave Empty>
Block Explorer URL: <Leave Empty>
```

  
##### 2. Clone Myself Ethereum dapp 
  


```
cd ~/projects
git clone git@github.com:bianyafeng/nervos-simple-dapp.git
cd nervos-simple-dapp
yarn
yarn build
yarn start:ganache
```
Ganache should now be running and creating blocks.

Switch back to your web browser. Open MetaMask and switch your network to Localhost 8545.

In a second terminal, start the UI server.


```
cd ~/projects/nervos-simple-dapp
yarn ui
```

The server should now be started, and you can open a browser tab to http://localhost:3000 to view the dApp UI!


  
  

##### 3. Install Polyjuice Dependencies

 We will begin porting this Ethereum application to use Nervos' Layer 2. The first step is to install the required dependencies for working with Godwoken and Polyjuice. Use the following command in the main project directory to install these dependencies in your application.

```
cd ~/projects/nervos-simple-dapp
yarn add @polyjuice-provider/web3@0.0.1-rc7 nervos-godwoken-integration@0.0.6
```
  
  
##### 4. Configure the Web3 Provider for the Polyjuice Web3 Provider
   
We will configure the Polyjuice Web3 Provider for the application. We replaces the normal web3 provider that may be currently in use for Ethereum with one for the Godwoken Testnet.

```
cd ~/projects/nervos-simple-dapp/src
vi config.ts
```
We will add  a new config file called config.ts to our source code and Add the following content 
  
```
 export const CONFIG = {
    WEB3_PROVIDER_URL: 'https://godwoken-testnet-web3-rpc.ckbapp.dev',
    ROLLUP_TYPE_HASH: '0x4cc2e6526204ae6a2e8fcf12f7ad472f41a1606d5b9624beebd215d780809f6a',
    ETH_ACCOUNT_LOCK_CODE_HASH: '0xdeec13a7b8e100579541384ccaf4b5223733e4a5483c3aec95ddc4c1d5ea5b22'
  };
```
We will update the main UI in the file 
```
~/projects/nervos-simple-dapp/src/ui/app.tsx.
```

Next, we add the following lines in the main dependency importation section of the file.
  
```
import { PolyjuiceHttpProvider } from '@polyjuice-provider/web3';
import { CONFIG } from '../config';
```

This imports the Polyjuice Web3 Provider, which we will use in a moment, and the config file that we just created.

Next we prepare a few constants, create the Polyjuice Provider, and use the Polyjuice Provider with a Web3 instance.
  
    
```
    const godwokenRpcUrl = CONFIG.WEB3_PROVIDER_URL;
    const providerConfig = {
    rollupTypeHash: CONFIG.ROLLUP_TYPE_HASH,
    ethAccountLockCodeHash: CONFIG.ETH_ACCOUNT_LOCK_CODE_HASH,
    web3Url: godwokenRpcUrl};
    const provider = new PolyjuiceHttpProvider(godwokenRpcUrl, providerConfig);
    const web3 = new Web3(provider);
```
**notes** :above const web3 = new Web3(provider) wil replace const web3 = new Web3((window as any).ethereum);

 Now our application is setup to communicate with Polyjuice using Web3!

##### 5. Set High Gas Limit
  
Open the following files in your editor.
```
~/projects/nervos-simple-dapp/src/lib/contracts/DonationWrapper.ts
~/projects/nervos-simple-dapp/src/lib/contracts/ERC20Wrapper.tsx
```

We define a simple object that contains the gas property used by MetaMask.

```
const DEFAULT_SEND_OPTIONS = {
    gas: 6000000
};
```

This can be added in the top region of the file, and we will be using this constant in several other places.

We will be adding it into the DonationWrapper.ts object passed to send() function :


```
async makeDonation(value:number,fromAddress:string)  {
	const tx = await this.contract.methods.makeDonation(value).send({
            ...DEFAULT_SEND_OPTIONS,
            from: fromAddress,
            value
        });
        return tx;
	}

   async withdrawBalance(fromAddress:string)  {
	const tx = await this.contract.methods.withdrawBalance().send({
            ...DEFAULT_SEND_OPTIONS,
            from: fromAddress
            
        });
        return tx;
	}

    async deploy(fromAddress: string) {
        const deployTx = await (this.contract
            .deploy({
                data: DonationJSON.bytecode,
                arguments: []
            })
            .send({
                ...DEFAULT_SEND_OPTIONS,
                from: fromAddress,
                to: '0x0000000000000000000000000000000000000000'
            } as any) as any);

        this.useDeployed(deployTx.contractAddress);
        return deployTx.transactionHash;
    }
```

  
##### 6. Display Polyjuice Address in Your Application
 
Every Ethereum address can be translated into a Polyjuice address on Nervos' Layer 2. This can be done using the AddressTranslator class. 

Open the following file in you editer

```
~/projects/nervos-simple-dapp/src/ui/app.tsx.
```

We add the following line in the main dependency importation section of the file.


```
import { AddressTranslator } from 'nervos-godwoken-integration';
```

  
We can then use the following code to find the Polyjuice address.


```
useEffect( () => {
        if (accounts?.[0]) {
            const addressTranslator = new AddressTranslator();
            setPolyjuiceAddress(addressTranslator.ethAddressToGodwokenShortAddress(accounts?.[0]));
            
       
        } else {
            setPolyjuiceAddress(undefined);
         
        }
    }, [accounts?.[0]]);
```


  
##### 7. View the Completed Godwoken dapp 


```
cd nervos-simple-dapp
yarn
yarn build
yarn ui
```
Switch your MetaMask network to Godwoken Testnet,
open your browser  http://localhost:3000

<img width="1230" alt="task7_1" src="https://user-images.githubusercontent.com/48971958/130315061-6b1e693c-d202-42b3-ac0c-e29a6fe258db.png">



