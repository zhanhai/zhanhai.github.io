
## inside contract

* Calling other contracts in one contract is in one transaction?

* When contracts are executed, after block being mined?

* How to estimate gas for executing contracts?
1. tx size
2. contract computation
3. contract storage 

* What will happen if gas is depleted during the contract computation?


* How can contracts communicate with oracle?


## development

* Why we have to identify variables as in memory?

* usage of external

* How to test with other tokens?

* How to trade ETH for ERC20 tokens? How to approve/escrow ETH to a contract?

* How to modularize contracts?

* Best practice of contracts?

* memory vs calldata

* ENS

* Revert


## client

* How to query contract status/balance?

* How to watch and check events?

* async await



1. Every node is either red or black.
2. The root is black.
3. Every leaf (NIL) is black.
4. If a node is red, then both its children are black.
5. For each node, all simple paths from the node to descendant leaves contain the same number of black nodes.


invariant

a. Node z is red.
b. If  z.p is the root, then  z.p is black.
c. If the tree violates any of the red-black properties, then it violates at most one of them, and the violation is of either property 2 or property 4. If the tree violates property 2, it is because  z is the root and is red. If the tree violates property 4, it is because both  z and  z.p are red.

RB-INSERT-FIXUP(T,z)

	while  z.p.color == RED 
2.		if  z.p ==  z.p.p.left
3.			y = z.p.p.right 

			if y.color == RED 				// case 1
5.				z.p.color = BLACK 
				y.color = BLACK 
				z.p.p.color = RED
				z = z.p.p
			else if  z ==  z.p.right 		//case 2
10.					z = z.p
					LEFT-ROTATE(T, z)

12.				z.p.color = BLACK 			// case 3
				z.p.p.color = RED 
				RIGHT-ROTATE(T,z.p.p)
		else (same as then clause with “right” and “left” exchanged)

16.	T:root:color = BLACK


