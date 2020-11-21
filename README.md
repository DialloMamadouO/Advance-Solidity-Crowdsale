# Advanced-Solidity-Crowdsale

In this project, we will create a crowdsale PupperCoin token to fund a network development. The goal of this crowdsale is to raise 300 Ether, so we will enable refund if the crowdsale is successfull and the goal is met within 24 weeks.
We will leverage the OpenZeppelin Solidity library to create an ERC20 token that will be minted through the Crowdsale contract which will manage the entire process, allowing users to send Ether and get back PupperCoin (PUP).

## Creating the project
We will use Remix to create two files, one for a standard ERC20Mintable token and a second for a standard crowdsale, PupperCoinCrowdsale.

## Designing the contracts

### ERC20 PupperCoin
For this contract, we will use a standard ERC20Mintable and ERC20Detailed contract and hardcode 18 as the decimals parameter but we will leave the initial_supply parameter alone for now.



contract PupperCoin is ERC20, ERC20Detailed, ERC20Mintable {

    constructor(
    
        string memory name,
        string memory symbol,
        uint initial_supply
        
    )
    
        ERC20Detailed(name, symbol, 18)
        public
        
    {
        // constructor can stay empty
    }
}



### PupperCoinCrowdsale
We will leverage a standard crowdasle and bootstrap that contract to inherit the following OpenZeppelin contracts: 

Crowdsale
MintedCrowdsale
CappedCrowdsale
TimedCrowdsale
RefundablePostDeliveryCrowdsale


We provided parameters for all of the features of our crowdsale, such as the name, symbol, wallet for fundraising, goal, cap, openingtime and closingtime.
We hardcoded a rate of 1, to maintain parity with Ether units (1 TKN per Ether, or 1 TKNbit per wei). Since PupperCoinCrowdsale inherited all the above contracts, we called them into the constructor. We passed the open and close times, to now and now + 24 weeks to set the times properly from our PupperCoinCrowdsaleDeployer contract.


PupperCoinCrowdsaleContract:

 contract PupperCoinSale is Crowdsale, MintedCrowdsale, CappedCrowdsale, TimedCrowdsale, RefundablePostDeliveryCrowdsale {
 
    constructor(
    
        uint rate, //  rate in 1 TKN per Ether, or 1 TKNbit per wei)
        address payable wallet, // sale beneficiary
        PupperCoin token, // the PupperCoin itself that the PupperCoinSale will work with
        uint cap, // Total cap in Ether
        uint goal, //minimum goal in Ether
        uint open, //Opening time
        uint close //Closing time
         
    )
        MintedCrowdsale()
        CappedCrowdsale(cap)
        TimedCrowdsale(open, close)
        RefundableCrowdsale(goal)
        Crowdsale(rate, wallet, token)
        public
   


### PupperCoinCrowdSaleDeployer

Now, below PupperCoinSale, at the top of the PupperCoinSaleDeployer contract, we added the following variables to store the addresses of the PupperCoin and PupperCoinSale contracts that this contract will be deploying:

An address public called token_sale_address, which will store PupperCoinSale's address once deployed.

An address public called token_address, which will store PupperCoin's address once deployed.

Inside of the constructor,


We created the PupperCoin token by defining a variable like PupperCoin token and setting it to equal new PupperCoin (). Inside of the parameters of new PupperCoin, we passed in the name and symbol variables. For the initial_supply variable that PupperCoin expects, we passed in 0.

For example:

    PupperCoin token = new PupperCoin(name, symbol, 0);


Then, we stored the address of the token by using token_address = address(token).


Next, we created the PupperCoinSale contract instance using the same logic we used when creating PupperCoin token. We stored the variable in PupperCoinSale puppercoinSale and set the parameters:


rate, hard-coded to 1 in order to maintain the same units as Ether.

wallet, passed in from the main constructor, this is the wallet that will get paid all Ether raised by the PupperCoinSale.

token, the actual token variable where PupperCoin is stored.

cap, the total cap in Ether.

goal, the 300 Ether target goal.

open, opening time of the crowdsale.

close, closing time of the crowdsale.

Once again, we stored the address of the PupperCoinSale in the token_sale_address variable for easy access later.


Finally, we set the PupperCoinSale contract as a minter, then renounce "mintership" from the PupperCoinSaleDeployer contract, like so:

token.addMinter(token_sale_address);

token.renounceMinter();


We need to do this because when we set our token as ERC20Mintable, the msg.sender is automatically set as the default minter. Since PupperCoinSaleDeployer is actually msg.sender in this case, this step will ensure that the PupperCoinSale is the actual minter, as expected.


contract PupperCoinSaleDeployer {

    address public token_sale_address;
    address public token_address;
    
    constructor(
        string memory name,
        string memory symbol,
        address payable wallet, // this address will receive all Ether raised by the sale
        uint cap, // Total cap in Ether
        uint goal // The minimum goal in Ether

    )
         public
         
    {
        // create the PupperCoin and keep its address handy
        PupperCoin token = new PupperCoin(name, symbol, 0);
        token_address = address(token);

        // create the PupperCoinSale and tell it about the token, set the goal, and set the open and close times to now and now + 24 weeks,
        PupperCoinSale puppercoinSale = new PupperCoinSale(1, wallet, token, cap, goal, now, now + 24 weeks);
        token_sale_address = address(puppercoinSale);

        // make the PupperCoinSale contract a minter, then have the PupperCoinSaleDeployer renounce its minter role
        token.addMinter(token_sale_address);
        token.renounceMinter();
    }
}


### Testing the Crowdsale
To send Ether and get PUP back you will need to use the deployed contract's approve function, you enter your address and the amount you are spending and confirm the transaction on MetaMask to complete the transaction (see screnshot).




