---
title: Mapping og Adresser
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

          // deklarer mapping her

          function _createZombie(string _name, uint _dna) private {
              uint id = zombies.push(Zombie(_name, _dna)) - 1;
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

La oss lage v??r multiplayer av spill ved ?? gi zombiene i v??r database en eier.

For ?? gj??re dette trenger vi 2 nye datatyper:`mapping` og `address`.

## Adresser

Ethereum blockchain best??r av **_accounts_**, som du kan tenke p?? som bankkontoer. En konto har en balanse mellom **_Ether_** (den valutaen som brukes i Ethereum-blokkkjeden), og du kan sende og motta Ether-betalinger til andre kontoer, akkurat som at bankkontoen din kan overf??re penger til andre bankkontoer.

Hver konto har en `address`, som du kan tenke p?? som et bankkontonummer. Det er en unik identifikator som peker p?? den kontoen, og det ser slik ut:

`0x0cE446255506E92DF41614C46F1d6df9Cc969183`

(Denne adressen tilh??rer CryptoZombies-teamet. Hvis du setter pris p?? CryptoZombies, kan du sende oss noen Ether! ????)

Vi vil komme inn i det nitty gritty av adresser i en senere leksjon, men for n?? trenger du bare ?? forst?? at **en adresse eies av en bestemt bruker** (eller en smart-kontrakt).

S?? vi kan bruke den som en unik ID for eierskap av v??re zombier. N??r en bruker oppretter nye zombier ved ?? samhandle med v??r app, setter vi eierskap av de zombiene til Ethereum-adressen som kalte funksjonen.

## Mapping

I Leksjon 1 s?? vi p??  **_structs_** og **_arrays_**. **_Mappings_**er en annen m??te ?? lagre organisert data p?? i Solidity.

Definere en `mapping` ser slik ut:

```
// For en finansiell app, lagrer du en uint som holder brukerens kontosaldo:
mapping (address => uint) public accountBalance;
// Eller kan brukes til ?? lagre / opps??k brukernavn basert p?? userId
mapping (uint => string) userIdToName;
```

En mapping er i hovedsak et n??kkelverdi lager for lagring og opps??king av data. I det f??rste eksemplet er n??kkelen en `address` og verdien er en`uint`, og i det andre eksempelet er n??kkelen en `uint` og verdien en`string`.


# Test det

For ?? lagre zombie eierskap, skal vi bruke to mappings: en som holder styr p?? adressen som eier en zombie, og en annen som holder styr p?? hvor mange zombier en eier har.

1. Lag en mapping kalt `zombieToOwner`. N??kkelen vil v??re en `uint` (vi lagrer og ser opp zombie basert p?? id) og verdien en`address`.La oss gj??re denne mappingen `public`.

2. Lag en mapping kalt `ownerZombieCount`, hvor n??kkelen er en `address` og verdien er en `uint`.
