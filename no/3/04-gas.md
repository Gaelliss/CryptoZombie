---
title: Gas
actions: ['checkAnswer', 'hints']
requireLogin: true
material:
  editor:
    language: sol
    startingCode:
      "zombiefactory.sol": |
        pragma solidity ^0.4.19;

        import "./ownable.sol";

        contract ZombieFactory is Ownable {

            event NewZombie(uint zombieId, string name, uint dna);

            uint dnaDigits = 16;
            uint dnaModulus = 10 ** dnaDigits;

            struct Zombie {
                string name;
                uint dna;
                // Add new data here
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

          KittyInterface kittyContract;

          function setKittyContractAddress(address _address) external onlyOwner {
            kittyContract = KittyInterface(_address);
          }

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
      "ownable.sol": |
        /**
         * @title Ownable
         * @dev Ownable kontrakten har eierens adresse, og gir grunnleggende autorisasjonskontroll over
         * funksjoner, dette forenkler implementeringen av "brukerrettigheter".
         */
        contract Ownable {
          address public owner;

          event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

          /**
           * @dev Ownable konstrukt??ren setter den originale `owner`(eieren) av kontrakten som senderen
           * account.
           */
          function Ownable() public {
            owner = msg.sender;
          }


          /**
           * @dev Kaster en feil hvis det kalles av en annen konto enn eieren.
           */
          modifier onlyOwner() {
            require(msg.sender == owner);
            _;
          }


          /**
           * @dev Tillater n??v??rende eier ?? overf??re kontroll over kontrakten til en newOwner.
           * @param newOwner Adressen til ?? overf??re eierskap til.
           */
          function transferOwnership(address newOwner) public onlyOwner {
            require(newOwner != address(0));
            OwnershipTransferred(owner, newOwner);
            owner = newOwner;
          }

        }
    answer: >
      pragma solidity ^0.4.19;

      import "./ownable.sol";

      contract ZombieFactory is Ownable {

          event NewZombie(uint zombieId, string name, uint dna);

          uint dnaDigits = 16;
          uint dnaModulus = 10 ** dnaDigits;

          struct Zombie {
              string name;
              uint dna;
              uint32 level;
              uint32 readyTime;
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
---

ott! N?? vet vi hvordan du oppdaterer viktige deler av DApps mens du forhindrer andre brukere i ?? snakke med v??re kontrakter.

La oss se p?? en annen m??te Solidity er ganske forskjellig fra andre programmeringsspr??k:

## Gas ?????drivstoffet Ethereum DApps kj??rer p??

I Solidity m?? brukerne betale hver gang de utf??rer en funksjon p?? din DApp ved hjelp av en valuta som kalles **_gas_**. Brukere kj??per gass med Ether (valutaen p?? Ethereum), s?? brukerne m?? bruke ETH for ?? kunne utf??re funksjoner p??  din DApp.

Hvor mye gass som kreves for ?? utf??re en funksjon, avhenger av hvor kompleks den funksjonens logikk er. Hver enkelt operasjon har en **_gas cost_** basert p?? hvor mye databehandlingsressurser som kreves for ?? utf??re denne operasjonen (for eksempel ?? skrive til storage er mye dyrere enn ?? legge til to integers). Den totale **_gas cost_**-en av funksjonen din er summen av gasskostnadene for alle sine individuelle operasjoner.

Fordi l??pende funksjoner koster ekte penger for brukerne, er kodeoptimalisering mye viktigere i Ethereum enn i andre programmeringsspr??k. Hvis koden din er slurvete, m?? brukerne betale en h??y pris for ?? utf??re dine funksjoner - og dette kan gi opptil millioner av dollar i un??dvendige avgifter over tusenvis av brukere.

## Hvorfor gas er viktig?

Ethereum er som en stor, sakte, men ekstremt sikker datamaskin. N??r du utf??rer en funksjon, trenger hver enkelt node p?? nettverket ?? kj??re samme funksjon for ?? verifisere utdataene sine - tusenvis av noder som verifiserer hver funksjon, er det som gj??r Ethereum desentralisert, og dataene er uforanderlige og sensurbestandige.

Skaperne av Ethereum ville s??rge for at noen ikke kunne tette opp nettverket med en uendelig loop, eller hogge alle nettverksressursene med veldig intensive beregninger. S?? de gjorde det slik at transaksjoner ikke er gratis, og brukerne m?? betale for beregningstid og lagring.

> Noter: Dette er ikke n??dvendigvis sant for sidekjeder, som de som CryptoZombies-forfatterne bygger p?? Loom Network. Det vil nok ikke v??re fornuftig ?? kj??re et spill som World of Warcraft direkte p?? Ethereum mainnet - gasskostnadene ville v??re uoverkommelig dyrt. Men det kan kj??re p?? en sidekjede med en annen konsensusalgoritme. Vi snakker mer om hvilke typer DApps du ??nsker ?? distribuere p?? sidekjeder vs Ethereum mainnet i en fremtidig leksjon.

## Struct pakking for ?? spare gas

I leksjon 1 nevnte vi at det finnes andre typer `uint`s: `uint8`, `uint16`, `uint32`, etc.

Normalt er det ingen fordel ?? bruke disse undertypene fordi Solidity beholder 256 biter lagringsplass uansett st??rrelsen p?? "uint". For eksempel, ved ?? bruke `uint8` i stedet for `uint` (`uint256`) vil du ikke spare noe gas.


Men det er et unntak til dette: inne i `struct`s.

Hvis du har flere `uint`s inne i en struct, hvis du bruker en mindre st??rrelse `uint` n??r det er mulig kan dette tillate Solidity ?? pakke disse variablene sammen for ?? ta opp mindre lagringsplass. For eksempel:

```
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// `mini` vil koste mindre gass enn `normalt` p?? grunn av struct pakking
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30); 
```

Av denne grunn, inne i en struct, vil du helst bruke de minste integer sub-typene du kan komme unna med.

Du vil ogs?? klynge identiske datatyper sammen (dvs. sett dem ved siden av hverandre i strukturen) slik at Solidity kan minimere n??dvendig lagringsplass. For eksempel, en struktur med
felter `uint c; uint32 a; uint32 b; ` vil koste mindre gass enn en struktur med felt `uint32 a; uint c; uint32 b; `
fordi `uint32`-feltene er gruppert sammen.

## Test det

I denne leksjonen kommer vi til ?? legge til 2 nye funksjoner til v??re zombier: `level` og `readyTime` - sistnevnte vil bli brukt til ?? implementere en nedkj??lings-timer for ?? begrense hvor ofte en zombie kan spise.

S?? la oss hoppe tilbake til `zombiefactory.sol`.

1. Legg til to flere egenskaper til v??r `Zombie` struct: `level` (en `uint32`) og `readyTime` (ogs?? en `uint32`). Vi ??nsker ?? pakke disse datatypene sammen, s?? la oss sette dem p?? slutten av strukturen.

32-bit er mer enn nok til ?? holde zombiens niv?? og tidstempel, s?? dette vil spare oss for noen gasskostnader ved ?? pakke dataene tettere enn ?? bruke en vanlig "uint" (256 bit).
