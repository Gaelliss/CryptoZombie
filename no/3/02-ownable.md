---
title: Eierskap i kontrakter
actions: ['checkAnswer', 'hints']
requireLogin: true
material:
  editor:
    language: sol
    startingCode:
      "zombiefactory.sol": |
        pragma solidity ^0.4.19;

        // 1. Importer her

        // 2. Arv kontrakt her:
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

          function setKittyContractAddress(address _address) external {
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

La du merke til sikkerhetshullet i forrige kapittel?

`setKittyContractAddress` er `external`, s?? hvem som helst kan kalle den! Det betyr at alle som kj??rer funksjonen, kunne endre adressen til CryptoKitties kontrakten, og bryte v??r app for alle sine brukere.

Vi vil ha muligheten til ?? oppdatere denne adressen i kontrakten v??r, men vi vil ikke at alle skal kunne oppdatere den.

For ?? h??ndtere saker som denne, en vanlig praksises som har oppst??tt er ?? lage kontrakter `Ownable` - noe som betyr at de har en eier (deg) som har spesielle rettigheter.

## OpenZeppelin's `Ownable` kontrakter

Nedenfor er den `Ownable` kontrakten hentet fra **_OpenZeppelin_** Solidity library. OpenZeppelin er et library med sikre og samfunnsmessige smart-kontrakter som du kan bruke i dine egne DApps. Etter denne leksjonen, mens du venter p?? utgivelsen av leksjon 4, anbefaler vi at du sjekker ut deres nettsted for ?? l??re mer!

Gi kontrakten under en gjennomgang. Du kommer til ?? se ting vi ikke har l??rt enda, men ikke v??r bekymret, vi kommert til ?? snakke om dem senere.

```
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
```

Et par nye ting her har vi ikke sett f??r:

- Constructors: `function Ownable ()` er en **_constructor_**, som er en valgfri spesialfunksjon som har samme navn som kontrakten. Det vil bli utf??rt bare ??n gang, n??r kontrakten er f??rst opprettet.
- Funksjonsmodifikatorer: `modifier onlyOwner ()`. Modifikatorer er en slags halvfunksjon som brukes til ?? endre andre funksjoner, vanligvis for ?? kontrollere krav f??r utf??relsen. I dette tilfellet kan `onlyOwner` brukes til ?? begrense tilgangen slik **kun eieren** av kontrakten kan kj??re denne funksjonen. Vi snakker mer om funksjonsmodifikatorer i neste kapittel, og hva `_;` gj??r.
- `indexed` n??kkelordet: ikke bekymre deg om dette, vi trenger ikke det enda.

S?? `Ownable` kontrakten gj??r i utgangspunktet f??lgende:

1. N??r en kontrakt er opprettet, setter konstrukt??ren `owner` til `msg.sender` (personen som distribuerte den)

2. Det legger til en `onlyOwner` modifikator, som begrenser tilgangen til visse funksjoner til bare `owner`

3. Det tillater deg ?? overf??re kontrakten til en ny `owner`

`onlyOwner` er et s?? vanlig krav til kontrakter at de fleste Solidity DApps starter med ?? copy / paste-e denne `Ownable` kontrakten, aog s?? arver deres f??rste kontrakt fra den.

Siden vi vil begrense `setKittyContractAddress` til `onlyOwner`, gj??r vi det samme med v??r kontrakt.

## Test det

Vi har tatt og kopiert `Ownable` kontrakten inn i en ny fil, `ownable.sol` for deg. La oss ta og la `ZombieFactory` arve fra den.

1. Modifiser koden v??r slik at vi `import`erer innholdet av `ownable.sol`. Hvis du ikke husker hvordan du gj??r dette, ta en titt p?? `zombiefeeding.sol`.

2. Modifiser `ZombieFactory` kontrakten til ?? arve fra `Ownable`. Igjen kan du ta en titt p?? `zombiefeeding.sol` hvis du ikke husker hvordan dette er gjort.
