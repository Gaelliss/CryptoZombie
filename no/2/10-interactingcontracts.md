---
title: Hva spiser Zombier?
actions: ['checkAnswer', 'hints']
material:
  editor:
    language: sol
    startingCode:
      "zombiefeeding.sol": |
        pragma solidity ^0.4.19;

        import "./zombiefactory.sol";

        // Lag en KittyInterface her

        contract ZombieFeeding is ZombieFactory {

          function feedAndMultiply(uint _zombieId, uint _targetDna) public {
            require(msg.sender == zombieToOwner[_zombieId]);
            Zombie storage myZombie = zombies[_zombieId];
            _targetDna = _targetDna % dnaModulus;
            uint newDna = (myZombie.dna + _targetDna) / 2;
            _createZombie("NoName", newDna);
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

        function feedAndMultiply(uint _zombieId, uint _targetDna) public {
          require(msg.sender == zombieToOwner[_zombieId]);
          Zombie storage myZombie = zombies[_zombieId];
          _targetDna = _targetDna % dnaModulus;
          uint newDna = (myZombie.dna + _targetDna) / 2;
          _createZombie("NoName", newDna);
        }

      }
---

Det er p?? tide ?? mate v??re zombier! Og hva liker zombier ?? spise mest?

Vel, det har seg slik at CryptoZombies elsker ?? spise ...

**CryptoKitties!** ????????????

(Ja, jeg er seri??s ???? )

For ?? gj??re dette m?? vi lese kittyDna fra CryptoKitties smart-kontrakt. Vi kan gj??re det fordi CryptoKitties-dataene er lagret ??pent p?? blockchain. Er ikke blockchain kult ?!

Ikke bekymre deg - v??rt spill kommer ikke til ?? skade noen CryptoKitty. Vi leser bare * CryptoKitties-dataene, vi kan ikke faktisk slette den????

## Samhandl med andre kontrakter

For v??r kontrakt ?? snakke med en annen kontrakt p?? blockchain som vi ikke eier, m?? vi f??rst definere et **_ interface _**.

La oss se p?? et enkelt eksempel. Si at det var en kontrakt p?? blockchainen som s?? ut som dette:

```
contract LuckyNumber {
  mapping(address => uint) numbers;

  function setNum(uint _num) public {
    numbers[msg.sender] = _num;
  }

  function getNum(address _myAddress) public view returns (uint) {
    return numbers[_myAddress];
  }
}
```

Dette ville v??re en enkel kontrakt hvor alle kunne lagre sitt lucky nummer, og det vil bli knyttet til deres Ethereum-adresse. S?? kunne noen andre sl?? opp personens lykkenummer ved ?? bruke adressen deres.

La oss si at vi hadde en ekstern kontrakt som ??nsket ?? lese dataene i denne kontrakten ved hjelp av `getNum`-funksjonen.

F??rst m?? vi definere en **_interface_** av `LuckyNumber` kontrakten:

```
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

Legg merke til at dette ser ut som ?? definere en kontrakt, med noen forskjeller. For en erkl??rer vi bare de funksjonene vi vil samhandle med - i dette tilfellet `getNum` - og vi nevner ikke noen av de andre funksjonene eller state-variablene.

For det andre definerer vi ikke noe inni funksjonen. I stedet for curly-braces (`{` og `}`), avslutter vi bare funksjonen med et semi-kolon (`;`).

S?? det ser ut som et kontraktskjelett. Slik vet kompilatoren at det er et interface.

Ved ?? inkludere dette grensesnittet i v??r dapps kode, vet v??r kontrakt hvordan de andre kontraktens funksjoner ser ut, hvordan du kj??rer dem og hva slags svar du kan forvente.

Vi kommer faktisk til ?? kj??re den andre kontraktens funksjoner i neste leksjon, men for n??, la oss erkl??re v??rt interface for CryptoKitties kontrakten.
# Test det

Vi har sett opp kildekoden for CryptoKitties for deg, og funnet en funksjon kalt `getKitty` som returnerer all kitty-ens data, inkludert dens "gener" (som er hva v??rt zombie spill trenger for ?? danne en ny zombie!).

Funksjonen ser slik ut:

```
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
) {
    Kitty storage kit = kitties[_id];

    // hvis denne variablen er 0 s?? er den ikke gravid
    isGestating = (kit.siringWithId != 0);
    isReady = (kit.cooldownEndBlock <= block.number);
    cooldownIndex = uint256(kit.cooldownIndex);
    nextActionAt = uint256(kit.cooldownEndBlock);
    siringWithId = uint256(kit.siringWithId);
    birthTime = uint256(kit.birthTime);
    matronId = uint256(kit.matronId);
    sireId = uint256(kit.sireId);
    generation = uint256(kit.generation);
    genes = kit.genes;
}
```

Funksjonen ser litt annerledes ut enn vi er vant til. Du kan se det returnerer ... en rekke forskjellige verdier. Hvis du kommer fra et programmeringsspr??k som Javascript, er dette annerledes - i Solidity kan du returnere mer enn en verdi fra en funksjon.

N?? som vi vet hva denne funksjonen ser ut, kan vi bruke den til ?? lage et interface:

1. Definer et interface kalt `KittyInterface`. Husk, Dette ser ut som om n??r vi skaper en ny kontrakt ?????vi bruker `contract` n??kkelordet.

2. Inni interface, definer funksjonen `getKitty` (som skal v??re en copy/paste av funksjonen ovenfor, men med en semi-kolon etter "return"-statementen, i stedet for alt inne i curly-braces.
