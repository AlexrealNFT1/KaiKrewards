// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface IKaiKToken {
    function transfer(address recipient, uint256 amount) external returns (bool);
}

contract KaiKRewards {
    address public owner;
    address public kaiKToken;
    mapping(address => uint256) public userInteractions;
    mapping(address => bool) public authorizedApps; // Tastermonial app servers

    event Rewarded(address indexed user, uint256 amount);
    event AuthorizedApp(address indexed app, bool status);
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not the contract owner");
        _;
    }

    modifier onlyAuthorizedApp() {
        require(authorizedApps[msg.sender], "Not authorized");
        _;
    }

    constructor(address _kaiKToken) {
        owner = msg.sender;
        kaiKToken = _kaiKToken;
    }

    function authorizeApp(address app, bool status) external onlyOwner {
        authorizedApps[app] = status;
        emit AuthorizedApp(app, status);
    }

    function rewardUser(address user) external onlyAuthorizedApp {
        require(user != address(0), "Invalid user address");
        
        userInteractions[user] += 1; // Track interactions

        // Reward 1 $KaiK token per interaction
        require(IKaiKToken(kaiKToken).transfer(user, 1 * (10 ** 18)), "Token transfer failed");

        emit Rewarded(user, 1 * (10 ** 18));
    }
}
