---
title: Kontrakt Arv
actions: ['checkAnswer', 'hints']
material:
  editor:
    language: sol
    startingCode: |
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

          function _createZombie(string _name, uint _dna) private {
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

      // Start her

    answer: >
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

          function _createZombie(string _name, uint _dna) private {
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

      contract ZombieFeeding is ZombieFactory {

      }

---
V??r spillkode begynner ?? bli ganske lang. I stedet for ?? lage en ekstremt lang kontrakt, er det noen ganger greit ?? dele kodelogikken din p?? tvers av flere kontrakter for ?? organisere koden.

En egenskap av Solidity som gj??r dette mer overkommelig er kontrakt **_arv_**:

```
contract Doge {
  function catchphrase() public returns (string) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string) {
    return "Such Moon BabyDoge";
  }
}
```

`BabyDoge` **_arver_** fra `Doge`. Det betyr at hvis du kompilerer og distribuerer `BabyDoge`,vil den ha adgang til b??de `catchphrase()` og `anotherCatchphrase()` og eventuelle andre offentlige funksjoner vi kan definere i `Doge`).

Dette kan brukes til logisk arv (som med en underklasse, en `katt` er et`dyr`). Men det kan ogs?? brukes til ?? organisere koden din ved ?? gruppere lignende logikk sammen i forskjellige klasser.

# Test det

I de neste kapitlene skal vi implementere funksjonaliteten for v??re zombier ?? mate og formere. La oss sette denne logikken i sin egen klasse som arver alle metodene fra `ZombieFactory`.

1. Lag en kontrakt kalt `ZombieFeeding` under `ZombieFactory`. Denne kontrakten skal arve fra v??r `ZombieFactory`-kontrakt.
