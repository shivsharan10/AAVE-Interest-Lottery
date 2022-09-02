																																																																																																																						//SPDX-License-Identifier: MIT
pragma solidity ^0.7.5;

import "./IERC20.sol";
import "./ILendingPool.sol";

contract Lottery {
	// the timestamp of the drawing event
	uint public drawing;
	// the price of the ticket in DAI (100 DAI)
	uint ticketPrice = 100e18;
	mapping(address => bool) public hasTicket;
	address[] usersAddresses;

	ILendingPool pool = ILendingPool(0x7d2768dE32b0b80b7a3454c06BdAc94A69DDc7A9);
	IERC20 aDai = IERC20(0x028171bCA77440897B824Ca71D1c56caC55b68A3); 
	IERC20 dai = IERC20(0x6B175474E89094C44Da98b954EedeAC495271d0F);

	constructor() {
        drawing = block.timestamp + 1 weeks;
	}

	function purchase() external {
        require(hasTicket[msg.sender] == false, "You have already purchased a ticket.");
		dai.transferFrom(msg.sender, address(this), ticketPrice);
		hasTicket[msg.sender] = true;
		usersAddresses.push(msg.sender);
		
		
		/*As soon as a ticket is purchased, we can start earning interest on it for the lottery 
		depositing the DAI transferred from the user's account into the AAVE pool.*/
		dai.approve(address(pool), ticketPrice);
		pool.deposit(address(dai), ticketPrice, address(this), 0);
	}

	event Winner(address);

	function pickWinner() external {
        require(block.timestamp >= drawing, "You need to wait.");

		/*Using the blockhash of the previus block and the balance of the AAVE dai pool 
		as sources of randomness to pick the winner from the array of users addresses*/
		uint winner = (uint(blockhash(block.number - 1)) * dai.balanceOf(address(pool))) % usersAddresses.length;
		emit Winner(usersAddresses[winner]);

		aDai.approve(address(pool), aDai.balanceOf(address(this)));
		
		for(uint i = 0; i < usersAddresses.length; i++) {
			pool.withdraw(address(dai),ticketPrice, usersAddresses[i]);
		}

		pool.withdraw(address(dai), type(uint).max, usersAddresses[winner]);
	}
}
