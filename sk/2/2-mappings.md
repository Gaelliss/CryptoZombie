---
title: Mapovania a adresy
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

          // tu deklaruj mapovania

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

Po??me spravit z na??ej hru multi-player t??m, ??e ka??d??mu zombie v na??ej datab??ze prirad??me vlastn??ka. 

Aby sme to dosiahli, budeme potrebova?? 2 nov?? d??tov?? ??trukt??ry `mapping` a `address`.

## Adresy
## Addresses

Ethereum blockchain je tvoren?? **_????tami_**. M????e?? o nich prem??????a?? podobn??m sp??sobom ako o bankov??ch ????toch. ????et m?? ur??it?? balans **_Etheru_** (kryptomena na Ethereum blockchaine). M????e?? odosiela?? a pr??jma?? platby do a z ostatn??ch u??tov. Rovnako ako ke?? s?? prev??dzan?? peniaze medzi bankov??mi ????tami. 

Ka??d?? ????et m?? svoju `adresu`. Tu je mo??n?? prirovna?? k ????slu bankov??ho ????tu. Je to unik??tny identifik??tor referuj??ci na jeden konkr??tny ????et. M????e vyzera?? napr??klad takto:

`0x0cE446255506E92DF41614C46F1d6df9Cc969183`

(T??to adresa patr?? CryptoZombies t??mu. Ak sa ti CryptoZombies p????i, m????e?? n??m posla?? nejak?? Ether! ???? )

K bli??????m detailom o adres??ch sa dostaneme v nasleduj??cej lekci. Zatia?? sta???? ke?? porozumie?? ??e **adresa je vlastnen?? ??pecifick??m u????vate??om** (alebo aj smart kontraktom).

To znamen?? ??e m????eme adresu pou??i?? ako unik??tny identifik??tor vlastn??ctva zombies. Ke?? u????vate?? vytvor?? nov??ho zombie prostriedn??ctvom na??ej aplik??cie, nastav??me vlastn??ka vytvoren??ho zombie na adresu z ktorej bola zavolan?? funkcia na vytvorenie nov??ho zombie.

## Mapovania

V Lekcii 1 sme sa pozreli na **_??trukt??ry_** (structs) a **_polia_** (arrays). **_Mapovania_** (**_mapping_**) s?? ??al????m sp??sobom ako v Solidity m????eme pracova?? so ??trukturovan??mi d??tami.

Deklar??cia mapovania vyzer?? takto: 

```
// Pre finan??n?? aplik??ciu by sme mohli ma?? mapovanie, v ktorom by ku ka??dej adrese patril `uint` reprezentuj??cu balanc ktor?? je na danom ????te k dispozici??:
mapping (address => uint) public accountBalance;
// Alebo by sme mohli uklada?? a preh??ad??vat ??????vate??sk?? men?? pod??a userId
mapping (uint => string) userIdToName;
```

Mapovanie je v podstate d??tov?? ??lo??isko vo forme k??u??-hodnota. V prvom pr??klade je k??????om je hodnota typu `address` ktorej kore??ponduje hodnota typu `uint`. V druhom pr??klade je k??????om hodnota typu `uint`. Ka??d??mu k??????u potom odpoved?? ur??it?? hodnota typu `string`.

# Vysk????aj si to s??m

Aby sme nejak??m sp??sobom spravovali vlastn??ctvo vytvoren??ch zombie, pou??ijeme dve mapovania. Jedno si bude pam??ta?? adresu vlastn??ka zombie s ur????t??m id. Druh?? mapovanie pou??ijeme na udr??ovanie si po??tu zombies vlastnen??ho nejakou adresou.  

1. Vytvor mapovanie pomenovan?? `zombieToOwner`. K??????om bude `uint` (pou??ijeme id zombie na to aby sme zistili kto je jeho vlastn??kom) a hodnotou mapovania bude adresa (adresa vlastn??ka zombie). Mapovanie deklarujme ako `public`.

2. Vytvor mapovanie s n??zvom `ownerZombieCount`. K?????? mapovania bude typu `address` a hodnotou bude `uint`.
