# Unit 20 - "Looks like we've made our First Contract!"

![contract](https://image.shutterstock.com/z/stock-photo-two-hands-handshake-polygonal-low-poly-hud-illustration-smart-contract-agreement-blockchain-and-1161295627.jpg)

## Background

My new startup has created its own Ethereum-compatible blockchain to help connect financial institutions, and the team wants to build smart contracts to automate some company finances to make everyone's lives easier, increase transparency, and to make accounting and auditing practically automatic!

Fortunately, I've been learning how to program smart contracts with Solidity and I'll create a few `ProfitSplitter` contracts. These contracts will do several things:

* Pay Associate-level employees quickly and easily.

* Distribute profits to different tiers of employees.

* Distribute company shares for employees in a "deferred equity incentive plan" automatically.

## Contract Levels

* **Level One** is an `AssociateProfitSplitter` contract. This will accept Ether into the contract and divide the Ether evenly among the associate level employees. This will allow the Human Resources department to pay employees quickly and efficiently.

* **Level Two** is a `TieredProfitSplitter` that will distribute different percentages of incoming Ether to employees at different tiers/levels. For example, the CEO gets paid 60%, CTO 25%, and Bob gets 15%.

* **Level Three** is a `DeferredEquityPlan` that models traditional company stock plans. This contract will automatically manage 1000 shares with an annual distribution of 250 over 4 years for a single employee.

### Getting Started

Navigated to the [Remix IDE](https://remix.ethereum.org) and created a new contract called `AssociateProfitSplitter.sol` 

![Level 1 Contract](New-Contract1.png)

While developing and testing the contract, used the [Ganache](https://www.trufflesuite.com/ganache) development chain, and point MetaMask to `localhost:8545`

![Set Local Host](Local_Host.png)

### Level One: The `AssociateProfitSplitter` Contract

At the top of my contract, I defined the following `public` variables:

* `employee_one` -- The `address` of the first employee. Make sure to set this to `payable`.

* `employee_two` -- Another `address payable` that represents the second employee.

* `employee_three` -- The third `address payable` that represents the third employee.

Created a constructor function that accepts:

* `address payable _one`

* `address payable _two`

* `address payable _three`

Within the constructor, set the employee addresses to equal the parameter values. This will allow me to avoid hardcoding the employee addresses.

Next, created the following functions:

* `balance` -- This function was set to `public view returns(uint)`, and returns the contract's current balance. This function should always return `0`, since we are sending Ether. If it does not, the `deposit` function is not handling the remainders properly and should be fixed. This will serve as a test function of sorts.

* `deposit` -- This function is set to `public payable` check, ensuring that only the owner can call the function.

  * In this function, performed the following steps:

    * Set a `uint amount` to equal `msg.value / 3;` in order to calculate the split value of the Ether.

    * Transfer the `amount` to `employee_one`.

    * Repeat the steps for `employee_two` and `employee_three`.

    * Since `uint` only contains positive whole numbers, and Solidity does not fully support float/decimals, we must deal with a potential remainder at the end of this function since `amount` will discard the remainder during division.

    * We may either have `1` or `2` wei leftover, so transfer the `msg.value - amount * 3` back to `msg.sender`. This will re-multiply the `amount` by 3, then subtract it from the `msg.value` to account for any leftover wei, and send it back to Human Resources.

* Created a fallback function using `function() external payable`, and call the `deposit` function from within it. This will ensure that the logic in `deposit` executes if Ether is sent directly to the contract. This is important to prevent Ether from being locked in the contract since we don't have a `withdraw` function in this use-case.

#### Test the contract

In the `Deploy` tab in Remix, deployed the contract to the local Ganache chain by connecting to `Injected Web3` and ensuring MetaMask is pointed to `localhost:8545`.

![Set Injected Web3](Set_Injected_Web3.png)

Next, filled in the constructor parameters with the designated `employee` addresses.

![Added Employee Addresses](Employee_Accounts.png)

Tested the `deposit` function by sending various values. 

![Tested the deposit function](Test_1.png)
![Confirmation of Deployed Contract](Deployed_Contract1.png)

### Level Two: The `TieredProfitSplitter` Contract

In this contract, rather than splitting the profits between Associate-level employees, I calculated the rudimentary percentages for different tiers of employees (CEO, CTO, and Bob).

Performed the following:

* Calculate the number of points/units by dividing `msg.value` by `100`.

  * This will allow us to multiply the points with a number representing a percentage. For example, `points * 60` will output a number that is ~60% of the `msg.value`.

* The `uint amount` variable will be used to store the amount to send each employee temporarily. For each employee, the `amount` was set to equal the number of `points` multiplied by the percentage (say, 60 for 60%).

* After calculating the `amount` for the first employee, added the `amount` to the `total` to keep a running total of how much of the `msg.value` we are distributing so far.

* Then, transfered the `amount` to `employee_one`. Repeat the steps for each employee, setting the `amount` to equal the `points` multiplied by their given percentage.

* For example, each transfer should look something like the following for each employee, until after transferring to the third employee:

  * Step 1: `amount = points * 60;`

    * For `employee_one`, distribute `points * 60`.

    * For `employee_two`, distribute `points * 25`.

    * For `employee_three`, distribute `points * 15`.

  * Step 2: `total += amount;`

  * Step 3: `employee_one.transfer(amount);`

* Sent the remainder to the employee with the highest percentage by subtracting `total` from `msg.value`, and sending that to an employee.

* Deployed and tested the contract functionality by depositing various Ether values (greater than 100 wei).

![Level 2 Contract](Contract_Level2.png)
![Level 2 Confirmation](Confirmation_Level2.png)

  * Note: The 100 wei threshold is due to the way we calculate the points. If we send less than 100 wei, for example, 80 wei, `points` would equal `0` because `80 / 100` equals `0` because the remainder is discarded. We will learn more advanced arbitrary precision division later in the course. In this case, we can disregard the threshold as 100 wei is a significantly smaller value than the Ether or Gwei units that are far more commonly used in the real world (most people aren't sending less than a penny's worth of Ether).

### Level Three: The `DeferredEquityPlan` Contract

In this contract, I will be managing an employee's "deferred equity incentive plan" in which 1000 shares will be distributed over 4 years to the employee. I didn't need to work with Ether in this contract, but will be storing and setting amounts that represent the number of distributed shares the employee owns and enforcing the vetting periods automatically.

* Human Resources will be set in the constructor as the `msg.sender`, since HR will be deploying the contract.

* Below the `employee` initialization variables at the top (after `bool active = true;`), set the total shares and annual distribution:

  * Created a `uint` called `total_shares` and set this to `1000`.

  * Created another `uint` called `annual_distribution` and set this to `250`. This equates to a 4 year vesting period for the `total_shares`, as `250` will be distributed per year. Since it is expensive to calculate this in Solidity, I simply set these values manually. 

* The `uint start_time = now;` line permanently stores the contract's start date. I used this to calculate the vested shares later. Below this variable, set the `unlock_time` to equal `now` plus `365 days`. Each distribution period will be incremented.

* The `uint public distributed_shares` will track how many vested shares the employee has claimed and was distributed. By default, this is `0`.

* In the `distribute` function:

  * Added the following `require` statements:

    * Required that `unlock_time` is less than or equal to `now`.

    * Required that `distributed_shares` is less than the `total_shares` the employee was set for.

    * Provided error messages .

  * After the `require` statements, added `365 days` to the `unlock_time`. This calculated next year's unlock time before distributing this year's shares. All calculations are performed like this before distributing the shares.

  * Next, set the new value for `distributed_shares` by calculating how many years have passed since `start_time` multiplied by `annual_distributions`. For example:

    * The `distributed_shares` is equal to `(now - start_time)` divided by `365 days`, multiplied by the annual distribution. If `now - start_time` is less than `365 days`, the output will be `0` since the remainder will be discarded. If it is something like `400` days, the output will equal `1`, meaning `distributed_shares` would equal `250`.

  * The final `if` statement provided checks that in case the employee does not cash out until 5+ years after the contract start, the contract does not reward more than the `total_shares` agreed upon in the contract.

* Deployed and tested contract locally.

  * For this contract, tested the timelock functionality by adding a new variable called `uint fakenow = now;` as the first line of the contract, then replaced every other instance of `now` with `fakenow`. Utilized the following `fastforward` function to manipulate `fakenow` during testing.

  * Added this function to "fast forward" time by 100 days when the contract is deployed (requires setting up `fakenow`):

    ```solidity
    function fastforward() public {
        fakenow += 100 days;
    }
    ```

![Level 3 Contract](Contract_Level3.png)
![Level 3 Confirmation](Confirmation_Level3.png)
