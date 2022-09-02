# AAVE

<img src="aave poster.png" alt="drawing" width="600" height="250"/>
## Working of the Interest Lottery.

# 1. A Weekly Drawing

In the lottery contract you'll notice there is a drawing unsigned integer. 
This value is the timestamp of the lottery drawing.
In the constructor, we'll want to set a date for the lottery drawing to occur, which will determine the winner.

## Set a Drawing
Set the lottery drawing to be a week after the contract is deployed.

# 2.Purchasing Tickets

Now that we've set a drawing, it's time to allow participants to enter our lottery system.
In this stage you will need to update the 'purchase' method to allow purchasing of tickets.

## Ticket Purchasing
A. In the purchase function, accept DAI in exchange for tickets. The ticketPrice is set a 100 DAI. <br/>
B. You can call transferFrom on the dai contract to transfer 100 dai from the caller's address to the contract. <br/>
C. The caller will set their allowance beforehand to ensure this works successfully. <br/>

# 3. Earn Interest

As soon as a ticket is purchased, we can start earning interest on it for the lottery. 
Let's take the DAI transferred from each individual ticket purchaser and deposit it immmediately into the AAVE lending pool.

## Ticket Interest
A. In the purchase method, deposit the DAI transferred from the user's account into the AAVE pool. <br/>
B. In the previous stage you moved the DAI from the user to the lottery contract. Now you will need to approve the lottey contract's dai to be spent by the pool before you can call deposit.<br/>
C. After you approve it, call deposit on the pool with onBehalfOf set as the lottery contract address and 0 set as the referral code.<br/>

# 4. Picking a Winner

Since this is an open and transparent lottery system, you will want to randomly select a winner in a way that is fair.
Finding a random value, or more accurately an unpredictable value, on the blockchain can be quite difficult! For the purposes of this code tutorial you can use a globally available unit like the block.timestamp or blockhash of a previous block, 
although you should be aware of the drawbacks of this approach!
Once you have your acceptably random value, you take this number modulo the total number of tickets purchased to decide your winner.

## Emit Winner
A. In the pickWinner method, pick a winner from all of the ticket purchasers. Once you have determined this winner, emit the Winner event with the address of the winner. <br/>
B.Security
Ensure that the pickWinner method can only be called a time occuring after the drawing timestamp.

# 5. Winner Payout

Let's wrap up the pickWinner function by paying out all the participants.
Each participant should get their money back and the winner should additionally recieve all interest earned.

## Withdrawls.
A. For each ticket purchaser, withdraw their initial purchase in DAI and transfer it to them. <br/>
B. You will need to approve the pool to spend the lottery's aDai balance before you can call the withdraw function successfully. Then you'll want to withdraw dai to each participant.<br/>
C. Finally, with the remaining aDai interest, withdraw it to the chosen winner.<br/>
