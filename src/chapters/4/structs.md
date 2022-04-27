# Structs

Structs group together many variables of different types in a single named value type. They may be used inside mappings and arrays and may contain mappings and arrays as fields. We can declare them with the keyword `struct`:

```solidity
struct Point {
    int256 x;
    int256 y;
}

struct Rectangle {
    Point: topLeft;
    Point: bottomRight;
}

struct Player {
    string username;
    address account;
    uint256 gamesWon;
    uint256 gamesLost;
    uint256[] gameIds;
}
```

We can define them in a few ways:

```solidity
Point memory origin = Point({x: 0, y: 0});

Rectangle memory rect = Rectangle(
    Point(10, 0),
    Point(0, 20)
);

Player player; // Defaults to storage
player.username = "horsefacts";
player.account = address(0x1234);
player.gamesWon = 5;
// gamesLost will remain default value of 0
// gameIds will remain default value of []
```

And access their fields using the dot operator:

```solidity
origin.x
> 0

rect.topLeft.x
> 10

player.gameIds.push(1);
player.gameIds[0]
> 1
```

Structs that contain mappings or dynamic arrays must have data location `storage` and must be created using the second syntax, since mappings and dynamic arrays cannot be created in memory.
