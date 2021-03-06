---
title: Handling Multiple Return Values
actions: ['checkAnswer', 'hints']
material:
  editor:
    language: sol
    startingCode:
      "zombiefeeding.sol": |
        pragma solidity ^0.4.19;

        import "./zombiefactory.sol";

        contract KittyInterface {
          function getKitty(uint256 _id) external view returns (
            bool isGestating,
            bool isReady,
            uint256 cooldownIndex,
            uint256 nextActionAt,
            uint256 siringWithId,
            uint256 birthTime,
            uint256 matronId,
            uint256 sireId,
            uint256 generation,
            uint256 genes
          );
        }

        contract ZombieFeeding is ZombieFactory {

          address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
          KittyInterface kittyContract = KittyInterface(ckAddress);

          function feedAndMultiply(uint _zombieId, uint _targetDna) public {
            require(msg.sender == zombieToOwner[_zombieId]);
            Zombie storage myZombie = zombies[_zombieId];
            _targetDna = _targetDna % dnaModulus;
            uint newDna = (myZombie.dna + _targetDna) / 2;
            _createZombie("NoName", newDna);
          }

          // Tu definuj funkciu

        }
      "zombiefactory.sol": |
        pragma solidity ^0.4.19;

        contract ZombieFactory {

            event NewZombie(uint zombieId, string name, uint dna);

            uint dnaDigits = 16;
            uint dnaModulus = 10 ** dnaDigits;

            struct Zombie {
                string name;
                uint dna;
            }

            Zombie[] public zombies;

            mapping (uint => address) public zombieToOwner;
            mapping (address => uint) ownerZombieCount;

            function _createZombie(string _name, uint _dna) internal {
                uint id = zombies.push(Zombie(_name, _dna)) - 1;
                zombieToOwner[id] = msg.sender;
                ownerZombieCount[msg.sender]++;
                NewZombie(id, _name, _dna);
            }

            function _generateRandomDna(string _str) private view returns (uint) {
                uint rand = uint(keccak256(_str));
                return rand % dnaModulus;
            }

            function createRandomZombie(string _name) public {
                require(ownerZombieCount[msg.sender] == 0);
                uint randDna = _generateRandomDna(_name);
                _createZombie(_name, randDna);
            }

        }
    answer: >
      pragma solidity ^0.4.19;

      import "./zombiefactory.sol";

      contract KittyInterface {
        function getKitty(uint256 _id) external view returns (
          bool isGestating,
          bool isReady,
          uint256 cooldownIndex,
          uint256 nextActionAt,
          uint256 siringWithId,
          uint256 birthTime,
          uint256 matronId,
          uint256 sireId,
          uint256 generation,
          uint256 genes
        );
      }

      contract ZombieFeeding is ZombieFactory {

        address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
        KittyInterface kittyContract = KittyInterface(ckAddress);

        function feedAndMultiply(uint _zombieId, uint _targetDna) public {
          require(msg.sender == zombieToOwner[_zombieId]);
          Zombie storage myZombie = zombies[_zombieId];
          _targetDna = _targetDna % dnaModulus;
          uint newDna = (myZombie.dna + _targetDna) / 2;
          _createZombie("NoName", newDna);
        }

        function feedOnKitty(uint _zombieId, uint _kittyId) public {
          uint kittyDna;
          (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
          feedAndMultiply(_zombieId, kittyDna);
        }

      }
---

T??to `getKitty` funkcia je prv??m pr??kladom vracania nieko??k??m hodn??t z funkcie. Po??me sa pozrie?? ako tieto n??vratov?? hodnoty spracova??. 

```
function multipleReturns() internal returns(uint a, uint b, uint c) {
  return (1, 2, 3);
}

function processMultipleReturns() external {
  uint a;
  uint b;
  uint c;
  // Takto prirad???? n??vratov?? hodnoty do nieko??k??ch premenn??ch
  (a, b, c) = multipleReturns();
}

// Alebo v pr??pade ??e ??a zauj??ma len jedna z hodn??t:
function getLastReturnValue() external {
  uint c;
  // M????me ostatn?? polo??ky ponecha?? pr??zdne takto:
  (,,c) = multipleReturns();
}
```

# Vysk????aj si to s??m

Je ??as na interakciu s CryptoKitties kontraktom!

Po??me nap??sa?? funkciu ktor?? z??ska g??ny ma??ky z CryptoKitties kontraktu:

1. Vytvor funkciu s n??zvom `feedOnKitty`. Bude pr??jma?? 2 `uint` parametre - `_zombieId` a `_kittyId`. Mala by to by?? `public` funkcia.

2. Funkcia by mala najprv deklarova?? `uint` s n??zvom `kittyDna`.

  > Pozn??mka: V na??om `KittyInterface`, `genes` je `uint256` - ale ak si spomenie?? na Lekciu 1, `uint` je skratkou pre `uint256` - je to to ist??.

3. Funkcia by potom mala vola?? funkciu `kittyContract.getKitty` s parametrom `_kittyId` a ulo??i?? `genes` do `kittyDna`. Ber na vedomie, ??e??`getKitty` vracia kopu n??vratov??ch hodn??t. (10, aby sme boli presn??. Sme dobr??, zr??tali sme ich za teba!). Jedin?? n??vratov?? hodnota ktor?? n??s ale naozaj zauj??ma je t?? posledn??, `genes`. Zr??taj si po??et ??iarok pozorne!

4. Funkcia by mala na z??ver zavola?? `feedAndMultiply` a preda?? jej paremetre `_zombieId` a `kittyDna`.
