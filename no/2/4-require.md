---
title: Require
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
              // start her
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
              require(ownerZombieCount[msg.sender] == 0);
              uint randDna = _generateRandomDna(_name);
              _createZombie(_name, randDna);
          }

      }
---

I leksjon 1 gjorde vi det slik at brukere kan lage nye zombier ved ?? kj??re `createRandomZombie` og skrive inn et navn. Men hvis brukerne kunne fortsette ?? kj??re denne funksjonen for ?? lage ubegrensede zombier i sin h??r, ville spillet ikke v??re veldig morsomt.

La oss gj??re det slik at hver spiller kun kan kj??re denne funksjonen en gang. P?? den m??ten vil nye spillere kalle det n??r de f??rst starter spillet for ?? lage den f??rste zombieen i sin h??r.

Hvordan kan vi gj??re det slik at denne funksjonen kun kan kalles en gang per spiller?

For det bruker vi `require`. `require` gj??r det slik at funksjonen vil kaste en feil og slutte ?? utf??re hvis noen tilstand ikke er sant:

```
function sayHiToVitalik(string _name) public returns (string) {
  // Sammenligner if _name er lik "Vitalik". Kaster en feil og avslutter hvis ikke sant.
???? // (Side notat: Solidity har ingen native string sammenligning, s?? vi
???? // sammenligner stringene med keccak256 hashes for ?? se om strengene er like)
???? require(keccak256 (_name) == keccak256 ("Vitalik"));
???? // Hvis det er sant, fortsett med funksjonen:
???? return "Hi!";
}
```

Hvis du kj??rer denne funksjonen med `sayHiToVitalik (" Vitalik ")`, vil den returnere "Hi!". Hvis du kj??rer det med noe annet input, vil det kaste en feil og ikke utf??re.

Dermed er `require` ganske nyttig for ?? verifisere visse forhold som m?? v??re sanne f??r du kj??rer en funksjon.

# Test det

I v??rt zombispill ??nsker vi ikke at brukeren skal kunne lage ubegrensede zombier i sin h??r ved ?? gjenta "createRandomZombie" - det ville gj??re spillet ikke s?? morsomt.

La oss bruke `require` for ?? sikre at denne funksjonen bare blir utf??rt en gang per bruker, n??r de lager sin f??rste zombie.

1. Sett en `require`-setning i begynnelsen av `createRandomZombie`. Funksjonen b??r sjekke for at `ownerZombieCount[msg.sender]` er lik `0`, og kaster en feil ellers.

> Noter: I Solidity, spiller det ingen rolle hvilket begrep du legger f??rst - begge ordrene er likeverdige. Men siden v??r svarkontroll er veldig grunnleggende, vil den bare akseptere ett svar som riktig - det forventer at `ownerZombieCount[msg.sender]` kommer f??rst.
