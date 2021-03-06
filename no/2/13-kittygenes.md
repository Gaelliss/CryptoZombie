---
title: "Bonus: Kitty Gener"
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

          // Modify function definition here:
          function feedAndMultiply(uint _zombieId, uint _targetDna) public {
            require(msg.sender == zombieToOwner[_zombieId]);
            Zombie storage myZombie = zombies[_zombieId];
            _targetDna = _targetDna % dnaModulus;
            uint newDna = (myZombie.dna + _targetDna) / 2;
            // Add an if statement here
            _createZombie("NoName", newDna);
          }

          function feedOnKitty(uint _zombieId, uint _kittyId) public {
            uint kittyDna;
            (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
            // And modify function call here:
            feedAndMultiply(_zombieId, kittyDna);
          }

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
                randDna = randDna - randDna % 100;
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

        function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) public {
          require(msg.sender == zombieToOwner[_zombieId]);
          Zombie storage myZombie = zombies[_zombieId];
          _targetDna = _targetDna % dnaModulus;
          uint newDna = (myZombie.dna + _targetDna) / 2;
          if (keccak256(_species) == keccak256("kitty")) {
            newDna = newDna - newDna % 100 + 99;
          }
          _createZombie("NoName", newDna);
        }

        function feedOnKitty(uint _zombieId, uint _kittyId) public {
          uint kittyDna;
          (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
          feedAndMultiply(_zombieId, kittyDna, "kitty");
        }

      }
---

V??r funksjonslogikk er n?? fullf??rt ... men la oss legge til i en bonusfunksjon.

La oss gj??re det s?? zombier laget av kattunge har noen unike egenskaper som viser at de er kattzombier.

For ?? gj??re dette kan vi legge til noen spesielle kitty-koder i zombiens DNA.

Hvis du husker fra leksjon 1, bruker vi for ??yeblikket bare de f??rste 12 sifrene i v??rt 16-sifret DNA for ?? bestemme zombiens utseende. S?? la oss bruke de siste 2 ubrukte sifrene til ?? h??ndtere "spesielle" egenskaper. 

Vi sier at katetzombier har `99` som de to siste sifrene i DNA (siden katter har 9 liv). S?? i v??r kode, sier vi `if` en zombie kommer fra en katt, og sett deretter de to siste sifrene i DNA til `99`.

## If statements

If statements i Solidity ser akkurat ut som i javascript:

```
function eatBLT(string sandwich) public {
  // Husk med stringer, m?? vi sammenligne deres keccak256 hashes
  // for ?? sjekke likhet
  if (keccak256(sandwich) == keccak256("BLT")) {
    eat();
  }
}
```

# Test det

La oss implementere kattgener i v??r zombiekode.

1. F??rst la vi endre funksjonsdefinisjonen for `feedAndMultiply` s?? det tar et tredje argument: en `string` som heter `_species`

2. S??, etter at vi har regnet ut den nye zombieens DNA, la oss legge til en `if`-statement som sammenligner `keccak256`-hashene av `_species` og stringen `"kitty"`

3. Inside the `if` statement, we want to replace the last 2 digits of DNA with `99`. One way to do this is using the logic: `newDna = newDna - newDna % 100 + 99;`.

  > Forklaring: Anta at `newDna` er `334455`. S?? `newDna % 100` er `55`, s?? `newDna - newDna % 100` er `334400`. Til slutt legger du `99` for ?? f?? `334499`.

4. Til slutt, Vi m?? endre funksjonsanropet i `feedOnKitty`. N??r det kalles `feedAndMultiply`, legger du til parameteret `"kitty"` til slutten.
