# karamay
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.34;

contract BaseDoubleOrNothing {
    address public immutable owner;
    uint256 public treasuryBalance;

    event GamePlayed(
        address indexed player,
        uint256 bet,
        bool won,
        uint256 payout,
        uint256 timestamp
    );

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can call this");
        _;
    }

    function play() external payable {
        require(msg.value == 0.001 ether, "Must pay exactly 0.001 ETH");

        // 伪随机 50/50
        bool won = uint256(keccak256(abi.encodePacked(
            block.prevrandao,
            msg.sender,
            block.timestamp,
            block.number
        ))) % 2 == 0;

        uint256 payout = 0;
        if (won) {
            payout = 0.002 ether;           // 翻倍
            payable(msg.sender).transfer(payout);
        } else {
            treasuryBalance += msg.value;   // 清零进入金库
        }

        emit GamePlayed(msg.sender, msg.value, won, payout, block.timestamp);
    }

    function getTreasury() external view returns (uint256) {
        return treasuryBalance;
    }

    function withdrawTreasury() external onlyOwner {
        uint256 amount = treasuryBalance;
        treasuryBalance = 0;
        payable(owner).transfer(amount);
    }

    function getContractInfo() external pure returns (string memory) {
        return "BaseDoubleOrNothing - Pay 0.001 ETH to double or nothing!";
    }
}
