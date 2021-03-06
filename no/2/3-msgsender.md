---
title: Msg.sender
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
              // start her
              NewZombie(id, _name, _dna);
          }

          function _generateRandomDna(string _str) private view returns (uint) {
              uint rand = uint(keccak256(_str));
              return rand % dnaModulus;
          }

          function createRandomZombie(string _name) public {
              uint randDna = _generateRandomDna(_name);
              _createZombie(_name, randDna);
          }

      }
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
              uint randDna = _generateRandomDna(_name);
              _createZombie(_name, randDna);
          }

      }
---

N?? som vi har v??re mappings for ?? holde oversikt over hvem som eier en zombie, vil vi oppdatere `_createZombie`-metoden for ?? bruke dem.

For ?? gj??re dette m?? vi bruke noe som heter `msg.sender`.

## msg.sender

I Solidity er det visse globale variabler som er tilgjengelige for alle funksjoner. En av disse er `msg.sender`, som refererer til `address` til personen (eller smart kontrakt) som kalte gjeldende funksjon.

> Noter: I Solidity m?? utf??rsel av funksjon alltid begynne med en ekstern utf??rsel. En kontrakt vil bare sitte p?? blockchain og gj??re ingenting f??r noen utf??rer en av dens funksjoner. S?? det vil alltid v??re en `msg.sender`.

Her er et eksempel p?? bruk av `msg.sender` og oppdatering av en `mapping`:

```
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _myNumber) public {
  // Oppdater din `favoriteNumber`-mapping for ?? lagre `_myNumber` under `msg.sender`
  favoriteNumber[msg.sender] = _myNumber;
  // ^ Syntaxen for lagring av data i en mapping er akkurat som med arrays
}

function whatIsMyNumber() public view returns (uint) {
  // Hent verdien som er lagret i avsenderens adresse
????// Vil v??re `0` hvis avsenderen ikke har kalt` setMyNumber` enn??
  return favoriteNumber[msg.sender];
}
```

I dette trivielle eksempelet kan noen kalle `setMyNumber` og lagre en `uint` i kontrakten v??r, som vil v??re knyttet til adressen deres. Da n??r de kaller ??whatIsMyNumber??, ville de bli returnert den `uint`-en som de lagret.

Ved ?? bruke `msg.sender` f??r du sikkerheten til Ethereum blockchain - den eneste m??ten noen kan modifisere andres data ville v??re ?? stjele den private n??kkelen som er knyttet til deres Ethereum-adresse.

# Test det

La oss oppdatere v??r `_createZombie`-metode fra leksjon 1 for ?? tilordne eierskap til zombie til den som kj??rte funksjonen.

1. F??rst, etter at vi f??tt tilbake den nye zombiens `id`, la oss oppdatere v??r `zombieToOwner`-mapping for ?? lagre `msg.sender` under den `id`-en.

2. S??, la oss ??ke `ownerZombieCount` for denne `msg.sender`-en.

I Solidity kan du ??ke en `uint` med `++`, akkurat som i javascript:

```
uint number = 0;
number++;
// `number` er n?? `1`
```

Ditt endelige svar for dette kapitlet skal v??re 2 linjer med kode.
