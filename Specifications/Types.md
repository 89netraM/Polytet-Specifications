# Types

This document specifies the special types used in Polytet's communication
protocol.

## Player integer

A player integer is an unsigned integer of a size specified by the server upon
connecting. This varying size allows for an theoretically infinite number of
players.

The length of player integer is detonated by `pi`.

## Piece enum

A one byte enum detonating the different piece in the game.

|  Name | Bits      |
|------:|-----------|
| Empty | `0b_0000` |
|     I | `0b_0001` |
|     J | `0b_0010` |
|     L | `0b_0011` |
|     O | `0b_0100` |
|     S | `0b_0101` |
|     T | `0b_0110` |
|     Z | `0b_0111` |

## Move enum

A one byte enum detonating the different moves a player can perform.

|                     Name | Bits           |
|-------------------------:|----------------|
|              Not allowed | `0b_0000_0000` |
|                Move left | `0b_0000_0001` |
|               Move right | `0b_0000_0010` |
|                Move down | `0b_0000_0100` |
| Rotate counter-clockwise | `0b_0000_1000` |
|         Rotate clockwise | `0b_0001_0000` |