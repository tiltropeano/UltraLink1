# /**
 * @custom:dev-run-script <script_file_path>
 */

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract UltraLink is ERC20, ERC20Burnable, Ownable {
    uint256 private constant MAX_SUPPLY = 100000000 * 10**18;
uint256 private constant INITIAL_SUPPLAX_SUPPLY = 10 * 10**6 * 10**18; // 10 million tokens with 18 decimals
    uint256 private constant OWNER_SUPPLY = INITIAL_SUPPLY * 50 / 100; // 50% of initial supply to owner
uint256 private constant TEAM_SUPPLY = MAX_SUPPLY * 50 / 100;
uint256 private constant MAX_FREE_SUPPLY = OWNER_SUPPLY + TEAM_SUPPLY;
uint256 private constant PRICE_FREE_SUPPLY = 0 ether;
uint256 private constant PRICE_REGULAR_SUPPLY = 0.005 ether;
uint256 private constant MINTABLE_SUPPLY = 90 * 106 * 10**18; // 90 million tokens with 18 decimals
bool public mintingAllowed;
uint256 private ulinkPrice = PRICE_REGULAR_SUPPLY;


    struct Proposal {
        string title;
        string description;
        address proposer;
        uint256 startBlock;
        uint256 endBlock;
        uint256 votesFor;
        uint256 votesAgainst;
        bool executed;
        bool canceled;
    }

    Proposal[] private proposals;

    event ProposalCreated(uint256 indexed proposalIndex, string title, address indexed proposer);
    event Voted(uint256 indexed proposalIndex, address indexed voter, bool inSupport);
    event ProposalExecuted(uint256 indexed proposalIndex, uint256 votesFor, uint256 votesAgainst, bool executed);
    event ProposalCanceled(uint256 indexed proposalIndex);
    event TokenPriceUpdated(uint256 newPrice);
    

    // Aggiunta funzionalità di upgrade del contratto
    address private newContractAddress;

    // Aggiunti controlli di sicurezza
    modifier validAmount(uint256 _amount) {
        require(_amount > 0, "Amount must be greater than zero");
        require(totalSupply() + _amount <= MAX_SUPPLY, "Max supply reached");
        _;
    }

    // Aggiunta funzionalità di lock-up
    mapping(address => uint256) private lockedBalances;
    mapping(address => uint256) private unlockTimes;
 // Aggiunta funzionalità di staking avanzate
    struct StakeInfo {
        uint256 amount;
        uint256 releaseTime;
        bool isUnlocked;
        uint256 reward; // NEW: reward earned by the user
    }

    mapping(address => StakeInfo[]) private stakes;

    uint256 private constant MIN_STAKE_AMOUNT = 0.01 ether;
    uint256 private constant STAKE_DURATION = 365 days;
    uint256 private constant UNLOCK_DURATION = 30 days;
    uint256 private constant APY = 6; // NEW: Annual percentage yield

    event Staked(address indexed user, uint256 amount, uint256 releaseTime);
    event Unstaked(address indexed user, uint256 amount);
    event Unlocked(address indexed user, uint256 amount);
    event RewardPaid(address indexed user, uint256 amount); // NEW: Event for reward paid

// Permette all'utente di fare lo staking di una quantità di token specificata per un determinato periodo di tempo
function stake(uint256 amount) external {
    require(amount >= MIN_STAKE_AMOUNT, "Minimum stake amount not reached");
    require(transferFrom(msg.sender, address(this), amount), "Token transfer failed");

    uint256 releaseTime = block.timestamp + STAKE_DURATION;
    uint256 reward = calculateReward(amount, APY, STAKE_DURATION);
    stakes[msg.sender].push(StakeInfo(amount, releaseTime, false, reward));

    emit Staked(msg.sender, amount, releaseTime);
}

function unlockStake(uint256 stakeIndex) external {
    require(stakeIndex < stakes[msg.sender].length, "Invalid stake index");
    StakeInfo storage currentStake = stakes[msg.sender][stakeIndex];

    require(!currentStake.isUnlocked, "Stake already unlocked");
    require(block.timestamp >= currentStake.releaseTime + UNLOCK_DURATION, "Stake not yet unlocked");
// Aggiunto controllo sulla durata minima dello stake
    require(block.timestamp >= currentStake.releaseTime + STAKE_DURATION, "Stake duration not yet reached");

    uint256 amount = currentStake.amount;
    uint256 reward = currentStake.reward;
      // Aggiunta della nuova condizione per la durata minima dello stake
    require(block.timestamp >= currentStake.releaseTime + STAKE_DURATION, "Stake duration not yet reached");
    currentStake.isUnlocked = true;
    // Controlla che l'utente abbia già approvato il contratto a spendere la quantità di token specificata
    require(ERC20(address(this)).allowance(address(this), msg.sender) >= amount, "Token allowance not set");
    // Controlla che la quantità di token staked sia uguale alla quantità di token effettivamente trasferita al contratto
    require(ERC20(address(this)).balanceOf(address(this)) >= amount, "Insufficient token balance");


    // Trasferisce i token sbloccati e la ricompensa
    require(transfer(msg.sender, amount + reward), "Token transfer failed");

    emit Unlocked(msg.sender, amount);
    emit RewardPaid(msg.sender, reward);
}

// Restituisce il numero di stake attualmente attivi per l'utente
function getNumStakes(address user) external view returns (uint256) {
    return stakes[user].length;
}

// Restituisce le informazioni relative allo stake specificato dall'utente
function getStake(address user, uint256 stakeIndex) external view returns (uint256, uint256, bool, uint256) {
    require(stakeIndex < stakes[user].length, "Invalid stake index");
    StakeInfo memory stakeInfo = stakes[user][stakeIndex];
    return (stakeInfo.amount, stakeInfo.releaseTime, stakeInfo.isUnlocked, stakeInfo.reward);
}

// Calcola la ricompensa per lo stake
function calculateReward(uint256 amount, uint256 apy, uint256 duration) internal pure returns (uint256) {
    uint256 annualReward = (amount * apy) / 100;
    uint256 dailyReward = annualReward / 365;
    return dailyReward * (duration / 1 days);
}


// Funzione di visualizzazione delle informazioni di stake
function getStakes(address user) external view returns (StakeInfo[] memory) {
return stakes[user];
}

// Funzione di visualizzazione dell'ammontare totale staked
function getTotalStaked() external view returns (uint256) {
return ERC20(address(this)).balanceOf(address(this));
}
   constructor() ERC20("UltraLink", "ULINK") {
_mint(msg.sender, OWNER_SUPPLY);}
    
uint256 public constant INITIAL_SUPPLY = 10000000 * 10**18; // Totale supply iniziale
uint256 public constant FREE_TOKENS = INITIAL_SUPPLY * 100 / 100; // 100% della supply iniziale distribuito gratuitamente
uint256 public tokensDistributed; // Contatore per tenere traccia dei token distribuiti

modifier canMint() {
    require(tokensDistributed >= FREE_TOKENS, "Token not yet available for minting.");
    _;
}

modifier underMaxSupply(uint256 amount) {
    require(totalSupply() + amount <= MAX_SUPPLY, "Exceeds maximum supply limit.");
    _;
}

function mint(address to, uint256 amount) public onlyOwner canMint underMaxSupply(amount) validAmount(amount) {
    _mint(to, amount);
}




    function transfer(address to, uint256 value) public override returns (bool) {
        require(to != address(0), "Transfer to the zero address");
        require(value > 0, "Transfer value must be greater than zero");
        require(balanceOf(msg.sender) - lockedBalances[msg.sender] >= value, "Insufficient unlocked balance");

        // Aggiunta funzionalità di lock-up
        if (unlockTimes[msg.sender] > 0 && block.timestamp < unlockTimes[msg.sender]) {
            // controllo il blocco temporale di lock-up
            uint256 lockedAmount = balanceOf(msg.sender) - (lockedBalances[msg.sender]);
            require(value <= lockedAmount, "Transfer value must be less than or equal to the locked balance");
        }

        _transfer(msg.sender, to, value);
        return true;
    }
function checkBalance(address user) public view returns (uint256) {
    return balanceOf(user);
}

function approveTokens(address spender, uint256 amount) public returns (bool) {
    require(amount > 0, "Amount must be greater than zero");

    _approve(msg.sender, spender, amount);
    return true;
}
    function sellTokens(uint256 amount) external {
        require(amount > 0, "Amount must be greater than zero");
        require(balanceOf(msg.sender) - lockedBalances[msg.sender] >= amount, "Insufficient unlocked balance");

        uint256 sellPrice = amount * ulinkPrice;

        _burn(msg.sender, amount);
        payable(msg.sender).transfer(sellPrice);
    }

    function purchaseTokens() external payable validAmount(msg.value / ulinkPrice) {
require(msg.value > 0, "Amount must be greater than zero");
uint256 amount = msg.value / ulinkPrice;
require(balanceOf(address(this)) >= amount, "Insufficient balance");
    _transfer(address(this), msg.sender, amount);
    return;
}

function createProposal(string memory title, string memory description, uint256 durationInBlocks) external {
    require(bytes(title).length > 0, "Title cannot be empty");
    require(bytes(description).length > 0, "Description cannot be empty");
    require(durationInBlocks > 0, "Duration must be greater than zero");

    Proposal memory newProposal = Proposal({
        title: title,
        description: description,
        proposer: msg.sender,
        startBlock: block.number,
        endBlock: block.number + durationInBlocks,
        votesFor: 0,
        votesAgainst: 0,
        executed: false,
        canceled: false
    });

    proposals.push(newProposal);

    emit ProposalCreated(proposals.length - 1, title, msg.sender);
}

function vote(uint256 proposalIndex, bool inSupport) external {
    require(proposalIndex < proposals.length, "Proposal does not exist");

    Proposal storage proposal = proposals[proposalIndex];

    require(!proposal.canceled, "Proposal has been canceled");
    require(!proposal.executed, "Proposal has already been executed");
    require(block.number <= proposal.endBlock, "Voting period has ended");

    if (inSupport) {
        proposal.votesFor += balanceOf(msg.sender);
    } else {
        proposal.votesAgainst += balanceOf(msg.sender);
    }

    emit Voted(proposalIndex, msg.sender, inSupport);
}

function executeProposal(uint256 proposalIndex) external {
    require(proposalIndex < proposals.length, "Proposal does not exist");

    Proposal storage proposal = proposals[proposalIndex];

    require(!proposal.canceled, "Proposal has been canceled");
    require(!proposal.executed, "Proposal has already been executed");
    require(block.number > proposal.endBlock, "Voting period has not ended");
    require(proposal.votesFor > proposal.votesAgainst, "Proposal did not pass");

    proposal.executed = true;

    emit ProposalExecuted(proposalIndex, proposal.votesFor, proposal.votesAgainst, true);
}

function cancelProposal(uint256 proposalIndex) external {
    require(proposalIndex < proposals.length, "Proposal does not exist");

    Proposal storage proposal = proposals[proposalIndex];

    require(!proposal.executed, "Proposal has already been executed");
    require(!proposal.canceled, "Proposal has already been canceled");
    require(msg.sender == proposal.proposer, "Only the proposer can cancel the proposal");

    proposal.canceled = true;

    emit ProposalCanceled(proposalIndex);
}

function updateTokenPrice(uint256 newPrice) external onlyOwner {
    require(newPrice > 0, "Price must be greater than zero");
    ulinkPrice = newPrice;
    emit TokenPriceUpdated(newPrice);
}

function lockTokens(uint256 amount, uint256 unlockTime) external {
    require(amount > 0, "Amount must be greater than zero");
    require(balanceOf(msg.sender) - lockedBalances[msg.sender] >= amount, "Insufficient unlocked balance");
    require(unlockTime > block.timestamp, "Unlock time must be in the future");

    lockedBalances[msg.sender] += amount;
    unlockTimes[msg.sender] = unlockTime;
}

event TokensUnlocked(address indexed owner, uint256 amount);
event Withdrawn(address indexed owner, uint256 amount);

function unlockTokens(uint256 amount, uint256 unlockTime) external {
    require(amount > 0, "Amount must be greater than zero");
    require(balanceOf(msg.sender) - lockedBalances[msg.sender] >= amount, "Insufficient unlocked balance");
    require(unlockTime > block.timestamp, "Unlock time must be in the future");

    lockedBalances[msg.sender] += amount;
    unlockTimes[msg.sender] = unlockTime;
}

function unlockTokens() external {
    require(lockedBalances[msg.sender] > 0, "No tokens are locked");
    require(block.timestamp >= unlockTimes[msg.sender], "Tokens are still locked");
    uint256 amount = lockedBalances[msg.sender];
    lockedBalances[msg.sender] = 0;
    _transfer(address(this), msg.sender, amount);
    emit TokensUnlocked(msg.sender, amount);
}

function withdraw(uint256 amount) external {
    require(balanceOf(msg.sender) >= amount, "Insufficient balance");

    // Update locked balances
    lockedBalances[msg.sender] -= amount;

    // Update unlock times and check for zero balance
    if (lockedBalances[msg.sender] == 0) {
        unlockTimes[msg.sender] = 0;
    }

    // Transfer tokens to the sender
    require(transfer(msg.sender, amount), "Token transfer failed");

    emit Withdrawn(msg.sender, amount);
}

}

